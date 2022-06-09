## Better way 55. Queue를 사용해 스레드 사이의 작업을 조율하라

파이썬 프로그램이 동시에 여러 일을 수행한다면 각 작업을 잘 조율해야 한다. 동시성 작업을 처리할 때 가장 유용한 방식은 함수 파이프라인이다.

예를 들어 디지털 카메라에서 이미지 스트림을 계속 가져와 이미지 크기를 변경하고 온라인 포토 갤러리에 저장하고 싶다고 하자. 

```python
def download(item):
    return item

def resize(item):
    return item

def upload(item):
    return item

from collections import deque
from threading import Lock

class MyQueue:
    def __init__(self):
        self.items = deque()
        self.lock = Lock()

    def put(self, item):
        with self.lock:
            self.items.append(item)

    def get(self):
        with self.lock:
            return self.items.popleft()

from threading import Thread
import time

class Worker(Thread):
    def __init__(self, func, in_queue, out_queue):
        super().__init__()
        self.func = func
        self.in_queue = in_queue
        self.out_queue = out_queue
        self.polled_count = 0
        self.work_done = 0

    def run(self):
        while True:
            self.polled_count += 1
            try:
                item = self.in_queue.get()
            except IndexError:
                time.sleep(0.01) # 할 일이 없음
            else: # except 절이 실행되지 않았을 때 else절 진행
                result = self.func(item)
                self.out_queue.put(result)
                self.work_done += 1

download_queue = MyQueue()
resize_queue = MyQueue()
upload_queue = MyQueue()

done_queue = MyQueue()
threads = [
    Worker(download, download_queue, resize_queue),
    Worker(resize, resize_queue, upload_queue),
    Worker(upload, upload_queue, done_queue),
]

for thread in threads:
    thread.start()

for _ in range(1000):
    download_queue.put(object())

while len(done_queue.items) < 1000:
    # 기다리는 동안 유용한 작업을 수행한다
    pass

processed = len(done_queue.items)
polled = sum(t.polled_count for t in threads)
print(f'{processed} 개의 아이템을 처리했습니다, '
      f'이때 폴링을 {polled} 번 했습니다.')

# 작업이 끝나도 무한대기함. 프로그램을 강제종료시킬것
>>>
1000 개의 아이템을 처리했습니다, 이때 폴링을 3009 번 했습니다.
```

문제점

첫째, 모든 작업이 다 끝났는지 검사하기 위해 추가로 done_queue에 대해 바쁜 대기를 수행해야 한다.

둘째, Worker의 run 메서드가 루프를 무한히 반복한다. 현재 구현에서는 작업자 스레드에게 루프를 중단할 시점임을 알려줄 뚜렷한 방법이 없다.

셋째, 파이프라인 진행이 막히면 프로그램이 임의로 중단될 수 있다. 첫 번째 단계가 빠르게 진행되는데 두 번째 단계가 느리게 진행되면. 첫 번째 단계와 두 번째 단계를 연결하는 큐의 크기가 계속 늘어난다. 메모리를 다 소진하여 프로그램이 죽음.

여기서 얻을 수 있는 교훈은 파이프라인이 나쁘다는 것이 아니라, 제대로 작동하는 생산자-소비자 큐를 직접 구현하기가 어렵다는 것이다. 따라서 굳이 직접 할 필요가 없다.

**대안: Queue**

queue 내장 모듈에 있는 Queue 클래스는 앞에서 설명한 모든 문제를 해결할 수 있는 기능을 제공한다.

Queue는 새로운 데이터가 나타날 때까지 get 메서드가 블록되게 만들어서 작업자의 바쁜 대기 문제를 해결한다. 

```python
from queue import Queue
from threading import Thread

my_queue = Queue()

def consumer():
    print('소비자 대기')
    my_queue.get()  # 다음에 보여줄 put()이 실행된 다음에 시행된다
    print('소비자 완료')

thread = Thread(target=consumer)
thread.start()

print('생산자 데이터 추가')
my_queue.put(object())     # 앞에서 본 get()이 실행되기 전에 실행된다.
print('생산자 완료')
thread.join()

>>>
소비자 대기
생산자 데이터 추가
생산자 완료
소비자 완료
```

파이프라인 중간이 막히는 경우를 해결하기 위해 Queue 클래스에서는 두 단계 사이에 허용할 수 있는 미완성 작업의 최대 개수를 지정할 수 있다. 이렇게 버퍼 크기를 정하면 큐가 이미 가득 찬 경우 put이 블록된다.

```python
from queue import Queue
from threading import Thread
import time

my_queue = Queue(1)  # 버퍼 크기 1

def consumer():
    time.sleep(0.1)  # 대기
    my_queue.get()  # 두 번째로 실행됨
    print('소비자 1')
    my_queue.get()  # 네 번째로 실행됨
    print('소비자 2')
    print('소비자 완료')

thread = Thread(target=consumer)
thread.start()

my_queue.put(object()) # 첫 번째로 실행됨
print('생산자 1')
my_queue.put(object()) # 세 번째로 실행됨
print('생산자 2')
print('생산자 완료')
thread.join()
```

Queue 클래스의 task_done 메서드를 통해 작업의 진행을 추적할 수 있다.

```python
from queue import Queue
from threading import Thread
import time

in_queue = Queue()

def consumer():
    print('소비자 대기')
    work = in_queue.get()  # 두 번째로 실행됨
    print('소비자 작업중')
    # Doing work
    print('소비자 완료')
    in_queue.task_done()  # 세 번째로 실행됨

thread = Thread(target=consumer)
thread.start()

print('생산자 데이터 추가')
in_queue.put(object())    # 첫 번째로 실행됨
print('생산자 대기')
in_queue.join()           # 네 번째로 실행됨    # queue에 join을 거는것은, queue.task_done이 호출되기를  기다림
print('생산자 완료')
thread.join()

>>>
소비자 대기
생산자 데이터 추가
생산자 대기
소비자 작업중
소비자 완료
생산자 완료
```

**기억해야 할 내용**
- 순차적인 작업을 동시에 여러 파이썬 스레드에서 실행되도록 조직하고 싶을 때, 특히 I/O위주의 프로그램인 경우라면 파이프라인이 매우 유용하다.
- 동시성 파이프라인을 만들 때 발생할 수 있는 여러 가지 문제(바쁜 대기, 작업자에게 종료를 알리는 방법, 잠재적인 메모리 사용량 폭발 등)를 잘 알아두라.
- Queue 클래스는 튼튼한 파이프라인을 구축할 때 필요한 기능인 블로킹 연산, 버퍼 크기 지정, join을 통한 완료 대기를 모두 제공한다.