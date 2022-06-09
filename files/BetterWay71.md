## Better way 71. 생산자-소비자 큐로 deque를 사용하라

프로그램을 작성할 때 자주 쓰는 기능으로 선입선출(First-In First-Out; FIFO) 큐(대기열)가 있다. 선입선출 큐(FIFO 큐)를 생산자-소비자 큐라고 부르기도 한다. 프로그래머들은 종종 파이썬 내장 리스트 타입을 FIFO 큐로 쓰곤 한다.

```python
import random

class Email:
    def __init__(self, sender, receiver, message):
        self.sender = sender
        self.receiver = receiver
        self.message = message

    def __str__(self):
        return f"Email({self.sender},{self.receiver},{self.message})"

class NoEmailError(Exception):
    pass

def try_receive_email():
    # Email 인스턴스를 하나 반환한거나, NoEmailError를 발생시킨다
    # 99%확률로 Email을 발생시킴
    if (i := random.randint(0,100)) > 99:
        raise NoEmailError
    else:
        return Email(f'홍길동{i}', '성춘향', '아니 자네가 왜 여기에!')

def produce_emails(queue):
    while True:
        try:
            email = try_receive_email()
        except NoEmailError:
            return
        else:
            queue.append(email)  # 생산자

def consume_one_email(queue):
    if not queue:
        return
    email = queue.pop(0) # 소비자. pop(0)는 리스트의 첫 번째 원소를 제거하고, 제거한 첫 번째 값을 호출자에게 돌려준다.
    print(f"read email: {email}")

def loop(queue, keep_running):
    while keep_running():
        produce_emails(queue)
        consume_one_email(queue)

# 99%확률로 True를 반환
def my_end_func():
    return random.randint(0, 100) > 1

loop([], my_end_func)

>>>
read email: Email(홍길동65,성춘향,아니 자네가 왜 여기에!)
read email: Email(홍길동76,성춘향,아니 자네가 왜 여기에!)
read email: Email(홍길동33,성춘향,아니 자네가 왜 여기에!)
read email: Email(홍길동65,성춘향,아니 자네가 왜 여기에!)
read email: Email(홍길동73,성춘향,아니 자네가 왜 여기에!)
...
```

이와 같은 생산자-소비자 문제에서는 큐로 리스트를 사용해도 어느 정도까지는 코드가 잘 작동한다. 하지만 크기(리스트 안에 들어 있는 원소 개수)가 늘어나면 리스트 타입의 성능은 선형보다 더 나빠진다. 리스트를 FIFO 큐로 사용할 때 성능이 어떤지 분석하기 위해 timeit 내장 모듈을 사용해 마이크로 벤치마크를 수행할 수 있다. 

```python
import timeit

def print_results(count, tests):
    avg_iteration = sum(tests) / len(tests)
    print(f'\n원소 수: {count:>5,} 걸린시간: {avg_iteration:.6f}초')
    return count, avg_iteration

def list_append_benchmark(count):
    def run(queue):
        for i in range(count):
            queue.append(i)

    tests = timeit.repeat(
        setup='queue = []',
        stmt='run(queue)', # 실행할 코드 및 함수
        globals=locals(),
        repeat=1000,
        number=1)
		# tests: list  len(): 1000

    return print_results(count, tests)

def print_delta(before, after):
    before_count, before_time = before
    after_count, after_time = after
    growth = 1 + (after_count - before_count) / before_count
    slowdown = 1 + (after_time - before_time) / before_time
    print(f'데이터 크기 {growth:>4.1f}배, 걸린 시간 {slowdown:>4.1f}배')

baseline = list_append_benchmark(500)
for count in (1_000, 2_000, 3_000, 4_000, 5_000):
    comparison = list_append_benchmark(count)
    print_delta(baseline, comparison)

>>>
원소 수:   500 걸린시간: 0.000022초

원소 수: 1,000 걸린시간: 0.000043초
데이터 크기  2.0배, 걸린 시간  2.0배

원소 수: 2,000 걸린시간: 0.000083초
데이터 크기  4.0배, 걸린 시간  3.8배

원소 수: 3,000 걸린시간: 0.000124초
데이터 크기  6.0배, 걸린 시간  5.6배

원소 수: 4,000 걸린시간: 0.000165초
데이터 크기  8.0배, 걸린 시간  7.5배

원소 수: 5,000 걸린시간: 0.000208초
데이터 크기 10.0배, 걸린 시간  9.4배
```

이 결과는 리스트 타입에 있는 append 메서드가 거의 상수 시간이 걸린다는 것을 보여준다.

이번에는 큐의 맨 앞에서 원소를 제거하는 pop(0) 호출을 벤치마크한다.

```python
def list_pop_benchmark(count):
    def prepare():
        return list(range(count))

    def run(queue):
        while queue:
            queue.pop(0)

    tests = timeit.repeat(
        setup='queue = prepare()',
        stmt='run(queue)',
        globals=locals(),
        repeat=1000,
        number=1)

    return print_results(count, tests)

baseline = list_pop_benchmark(500)

for count in (1_000, 2_000, 3_000, 4_000, 5_000):
    comparison = list_pop_benchmark(count)
    print_delta(baseline, comparison)

>>>
원소 수:   500 걸린시간: 0.000075초

원소 수: 1,000 걸린시간: 0.000178초
데이터 크기  2.0배, 걸린 시간  2.4배

원소 수: 2,000 걸린시간: 0.000506초
데이터 크기  4.0배, 걸린 시간  6.8배

원소 수: 3,000 걸린시간: 0.000983초
데이터 크기  6.0배, 걸린 시간 13.2배

원소 수: 4,000 걸린시간: 0.001618초
데이터 크기  8.0배, 걸린 시간 21.7배

원소 수: 5,000 걸린시간: 0.002425초
데이터 크기 10.0배, 걸린 시간 32.5배
```

놀랍게도 리스트에서 pop(0)을 사용해 원소를 큐에서 빼내는 시간이 큐 길이가 늘어남에 따라 큐 길이의 제곱에 비례해 늘어나는 것을 볼 수 있다. 이유는 pop(0)을 하면 리스트의 모든 남은 원소를 제 위치로 옮겨야 해서, 결과적으로 전체 리스트 내용을 다시 재대입하기 때문이다. 

파이썬 collections 내장 모듈에는 deque 클래스가 들어 있다. deque는 양방향 큐(double-ended queue) 구현이며 '데크'라고 읽는다. 데크의 시작과 끝 지점에 원소를 넣거나 빼는 데는 상수 시간이 걸린다. 따라서 FIFO 큐를 구현할 때 이상적이다. 

```python
import collections

def deque_append_benchmark(count):
    def prepare():
        return collections.deque()

    def run(queue):
        for i in range(count):
            queue.append(i)

    tests = timeit.repeat(
        setup='queue = prepare()',
        stmt='run(queue)',
        globals=locals(),
        repeat=1000,
        number=1)
    return print_results(count, tests)

baseline = deque_append_benchmark(500)
for count in (1_000, 2_000, 3_000, 4_000, 5_000):
    comparison = deque_append_benchmark(count)
    print_delta(baseline, comparison)

>>>
원소 수:   500 걸린시간: 0.000023초

원소 수: 1,000 걸린시간: 0.000040초
데이터 크기  2.0배, 걸린 시간  1.7배

원소 수: 2,000 걸린시간: 0.000081초
데이터 크기  4.0배, 걸린 시간  3.5배

원소 수: 3,000 걸린시간: 0.000121초
데이터 크기  6.0배, 걸린 시간  5.2배

원소 수: 4,000 걸린시간: 0.000167초
데이터 크기  8.0배, 걸린 시간  7.2배

원소 수: 5,000 걸린시간: 0.000203초
데이터 크기 10.0배, 걸린 시간  8.7배
```

```python
def dequeue_popleft_benchmark(count):
    def prepare():
        return collections.deque(range(count))

    def run(queue):
        while queue:
            queue.popleft()

    tests = timeit.repeat(
        setup='queue = prepare()',
        stmt='run(queue)',
        globals=locals(),
        repeat=1000,
        number=1)

    return print_results(count, tests)

baseline = dequeue_popleft_benchmark(500)
for count in (1_000, 2_000, 3_000, 4_000, 5_000):
    comparison = dequeue_popleft_benchmark(count)
    print_delta(baseline, comparison)

>>>
원소 수:   500 걸린시간: 0.000025초

원소 수: 1,000 걸린시간: 0.000045초
데이터 크기  2.0배, 걸린 시간  1.8배

원소 수: 2,000 걸린시간: 0.000090초
데이터 크기  4.0배, 걸린 시간  3.6배

원소 수: 3,000 걸린시간: 0.000134초
데이터 크기  6.0배, 걸린 시간  5.4배

원소 수: 4,000 걸린시간: 0.000166초
데이터 크기  8.0배, 걸린 시간  6.6배

원소 수: 5,000 걸린시간: 0.000204초
데이터 크기 10.0배, 걸린 시간  8.1배
```

deque의 popleft를 사용하면 리스트의 pop(0)과 달리 대기열 길이에 선형적으로 비례해 시간이 늘어난다!

**기억해야 할 내용**
- 생산자는 append를 호출해 원소를 추가하고 소비자는 pop(0)을 사용해 원소를 받게 만들면 리스트 타입을 FIFO 큐로 사용할 수 있다. 하지만 리스트를 FIFO 큐로 사용하면, 큐 길이가 늘어남에 따라 pop(0)의 성능이 선형보다 더 크게 나빠지기 때문에 문제가 될 수 있다.
- collections 내장 모듈에 들어 있는 deque 클래스는 큐 길이와 관계없이 상수 시간 만에 append와 popleft를 수행하기 때문에 FIFO 큐 구현에 이상적이다.