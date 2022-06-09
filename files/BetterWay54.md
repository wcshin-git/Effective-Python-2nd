## Better way 54. 스레드에서 데이터 경합을 피하기 위해 Lock을 사용하라

GIL이 다중 CPU에서 파이썬 스레드들이 병렬적으로 실행될 수 없게 막는다면, 파이썬 스레드들이 프로그램의 데이터 구조에 동시에 접근할 수 없게 막는 락 역할도 해줘야 하지 않을까? 

하지만 그런 역할을 해주지 못한다. 여러 스레드가 같은 데이터 구조에 동시에 접근하면 위험하다. 이런 인터럽트로 인해 실질적으로는 언제든지 데이터 구조에 대한 불변 조건이 위반될 수 있고, 그에 따라 프로그램의 상태가 오염될 수 있다. 

예를 들어 병렬적으로 여러 가지의 개수를 세는 프로그램을 작성한다고 하자.  

```python
class Counter:
    def __init__(self):
        self.count = 0

    def increment(self, offset):
        self.count += offset

def worker(sensor_index, how_many, counter):
    for _ in range(how_many):
        # 센서를 읽는다
        counter.increment(1)

from threading import Thread

how_many = 10**5
counter = Counter()

threads = []
for i in range(5):
    thread = Thread(target=worker,
                    args=(i, how_many, counter))
    threads.append(thread)
    thread.start()

for thread in threads:
    thread.join()

expected = how_many * 5
found = counter.count
print(f'카운터 값은 {expected}여야 하는데, 실제로는 {found} 입니다')

>>>
카운터 값은 500000여야 하는데, 실제로는 404231 입니다
```

이 코드는 병렬로 센서마다 하나씩 worker 스레드를 실행하고, 모든 스레드가 값을 다 읽을 때까지 기다린다.

그런데 결과가 예상과 전혀 다르다. 왜 이런 일이 벌어졌을까? 파이썬 인터프리터는 실행되는 모든 스레드를 강제로 공평하게 취급해서 각 스레드의 실행 시간을 거의 비슷하게 만든다. 이를 위해 파이썬은 실행 중인 스레드를 일시 중단시키고 다른 스레드를 실행시키는 일이 반복된다. 문제는 파이썬이 스레드를 언제 중단시킬지 알 수 없다는 점이다. 심지어 원자적(atomic)인 것처럼 보이는 연산을 수행하는 도중에도 파이썬이 스레드를 일시 중단시킬 수 있다.  예를 들어 counter.count += 1은 간단해 보이지만 실제로는 세 가지 연산으로 이뤄진다.

```python
value = getattr(counter, 'count')
result = value + 1
setattr(counter, 'count', result)
```

카운터를 증가시키는 파이썬 스레드는 세 연산 사이에서 일시 중단될 수 있다. 이런 일시 중단으로 인해 스레드 간 연산 순서가 뒤섞이면서 value의 이전 값을 카운터에 대입하는 일이 생길 수 있다. 

이와 같은 데이터 경합이나 다른 유형의 데이터 구조 오염을 해결하기 위해 파이썬은 threading 내장 모듈 안에 여러 가지 튼튼한 도구를 제공한다. 가장 간단하지만 유용한 도구로 Lock 클래스가 있다.

락을 사용하면 Counter 클래스가 여러 스레드의 동시 접근으로부터 자신의 현재 값을 보호할 수 있다. 다음 코드에서는 with 문을 사용해 락을 획득하고 해제한다. 

```python
from threading import Thread

from threading import Lock

class LockingCounter:
    def __init__(self):
        self.lock = Lock()
        self.count = 0

    def increment(self, offset):
        with self.lock:
            self.count += offset

def worker(sensor_index, how_many, counter):
    for _ in range(how_many):
        # 센서를 읽는다
        counter.increment(1)

from threading import Thread

how_many = 10**5
counter = LockingCounter()

threads = []
for i in range(5):
    thread = Thread(target=worker,
                    args=(i, how_many, counter))
    threads.append(thread)
    thread.start()

for thread in threads:
    thread.join()

expected = how_many * 5
found = counter.count
print(f'카운터 값은 {expected}여야 하는데, 실제로는 {found} 입니다')

>>>
카운터 값은 500000여야 하는데, 실제로는 500000 입니다
```

**기억해야 할 내용**
- 파이썬에는 GIL이 있지만, 파이썬 프로그램 코드는 여전히 여러 스레드 사이에 일어나는 데이터 경합으로부터 자신을 보호해야 한다.
- 코드에서 여러 스레드가 상호 배제 락(뮤텍스) 없이 같은 객체를 변경하도록 허용하면 코드가 데이터 구조를 오염시킬 것이다.
- 여러 스레드 사이에서 프로그램의 불변 조건을 유지하려면 threading 내장 모듈의 Lock 클래스를 활용하라.