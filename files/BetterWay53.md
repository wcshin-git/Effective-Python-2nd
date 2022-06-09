## Better way 53. 블로킹 I/O의 경우 스레드를 사용하고 병렬성을 피하라

파이썬의 표준 구현을 Cpython이라고 한다(대부분의 파이썬의 경우 내부는 C언어로 구현되어 있다. 이를 CPython이라고 한다). Cpython은 두 단계를 거쳐서 파이썬 프로그램을 실행한다. 첫 번째 단계는 소스 코드를 구문 분석해서 바이트코드로 변환한다. 그 후 Cpython은 바이트코드를 스택 기반 인터프리터를 통해 실행한다. 바이트코드 인터프리터에는 파이썬 프로그램이 실행되는 동안 일관성 있게 유지해야 하는 상태가 존재한다. Cpython은 **전역 인터프리터 락**(Global Interpreter Lock, GIL)이라는 방법을 사용해 일관성을 강제로 유지한다. (스레드가 동시에 앞으로 나아가지 않는다. 한 스레드씩)

근본적으로 GIL은 Cpython이 선점형 멀티스레드로 인해 영향을 받는 것을 방지한다. 선점형 멀티스레드에서는 한 스레드가 다른 스레드의 실행을 중간에 인터럽트시키고 제어를 가져올 수 있다. 이런 인터럽트가 예기치 못한 때 발생하면 인터프리터의 상태(eg. 쓰레기 수집기의 참조 카운드)가 오염될 수 있다.GIL은 Cpython 자체와 Cpython이 사용하는 C확장 모듈이 실행되면서 인터럽트가 함부로 발생하는 것을 방지해, 인터프리터 상태가 제대로 유지되고 바이트코드 명령들이 제대로 실행되도록 만든다. 

파이썬도 다중 실행 스레드를 지원하지만, **GIL로 인해 여러 스레드 중 어느 하나만 앞으로 진행할 수 있다.** 따라서 병렬 처리를 수행하고자 스레드를 사용한다면 여러분은 크게 실망할 것이다.(다중 CPU 에서 병렬 실행을 위해서는 다중 프로세스를 이용하는 multiprocessing 모듈을 사용해야 한다.)

파이썬으로 계산량이 매우 많은 작업을 수행하고 싶다고 하자. eg) 인수찾기 알고리즘.

```python
def factorize(number):
    for i in range(1, number + 1):
        if number % i == 0:
            yield i

import time

numbers = [2139079, 1214759, 1516637, 1852285]
start = time.time()

for number in numbers:
    list(factorize(number))

end = time.time()
delta = end - start
print(f'총 {delta:.3f} 초 걸림')

>>>
총 0.333 초 걸림
```

이번에는 파이썬에서 다중 스레드를 사용해 계산 수행을 시도해보자.

```python
def factorize(number):
    for i in range(1, number + 1):
        if number % i == 0:
            yield i

from threading import Thread
# threading.Thread 로부터 파생클래스를 만드는 방식은
# Thread 클래스를 파생하여 쓰레드가 실행할 run() 메서드를 재정의해서 사용하는 방식
class FactorizeThread(Thread):
    def __init__(self, number):
        super().__init__()
        self.number = number

    def run(self):
        self.factors = list(factorize(self.number))

import time

numbers = [2139079, 1214759, 1516637, 1852285]
start = time.time()

threads = []
for number in numbers:
    thread = FactorizeThread(number) # 각 수마다 스레드를 시작해 병렬로 인수를 찾을 수 있다. 
    thread.start() # start() 메서드는 내부적으로 run() 메서드를 호출
    threads.append(thread)

for thread in threads:
    thread.join() # 모든 스레드가 끝날 때까지 기다린다.

end = time.time()
delta = end - start
print(f'총 {delta:.3f} 초 걸림')

>>>
총 0.368 초 걸림
```

놀랍게도 스레드를 하나만 써서 순차적으로 factorize를 실행할 때보다 시간이 더 오래 걸린다. 이 결과는 표준 Cpython 인터프리터에서 프로그램을 사용할 때 GIL이 미치는 영향을 잘 보여준다.

Cpython에서도 다중 코어를 활용할 수 있는 방법이 있다. 하지만 이 방법은 표준 Thread 클래스를 사용하지 않으며, 코딩하는 데 상당한 노력이 필요하다. 이런 한계에도 불구하고 파이썬이 스레드를 지원하는 이유는?

블로킹 I/O를 다루기 위해서다. 블로킹 I/O는 파이썬이 특정 시스템 콜을 사용할 때 일어난다. 파이썬 프로그램은 시스템 콜을 사용해 컴퓨터 운영체제가 자기 대신 외부 환경과 상호작용하도록 의뢰한다. 파일 쓰기나 읽기, 네트워크와 상호작용하기, 디스플레이 장치와 통신하기 등의 작업이 블로킹 I/O에 속한다. **스레드를 사용하면 운영체제가 시스템 콜 요청에 응답하는 데 걸리는 시간 동안 파이썬 프로그램이 다른 일을 할 수 있다.** 

**기억해야 할 내용**
- 파이썬 스레드는 GIL(전역 인터프리터 락)로 인해 다중 CPU 코어에서 병렬로 실행될 수 없다.
- GIL이 있음에도 불구하고 파이썬 스레드는 여전히 유용하다. 스레드를 사용하면 여러 일을 동시에 진행하는 프로그램을 쉽게 기술할 수 있기 때문이다.
- 파이썬 스레드를 사용해 여러 시스템 콜을 병렬로 할 수 있다. 이를 활용하면 블로킹 I/O와 계산을 동시에 수행할 수 있다.