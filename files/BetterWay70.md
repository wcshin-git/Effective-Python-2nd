## Better way 70. 최적화하기 전에 프로파일링을 하라

파이썬의 동적인 특성으로 인해 실행 시간 성능이 예상과 달라 놀랄 때가 있다. 가장 좋은 접근 방법은 프로그램을 최적화하기 전에 여러분의 직관을 무시하고 직접 프로그램 성능을 측정하는 것이다. 파이썬은 프로그램의 각 부분이 실행 시간을 얼마나 차지하는지 결정할 수 있게 해주는 프로파일러를 제공한다. 프로파일러가 있기 때문에 프로그램에서 가장 문제가 되는 부분을 집중적으로 최적화하고 프로그램에서 속도에 영향을 미치지 않는 부분은 무시할 수 있다. 

예를 들어 삽입 정렬을 사용해 데이터 리스트를 정렬하는 함수를 정의하자. 삽입 정렬의 핵심 메커니즘은 데이터 조각을 삽입할 위치를 찾는 함수다.  다음은 입력 배열을 선형 검색하는 아주 비효율적인 insert_value 함수이다.

```python
def insertion_sort(data):
    result = []
    for value in data:
        insert_value(result, value)
    return result

def insert_value(array, value):  # 선형검색: 비효율적
    for i, existing in enumerate(array):
        if existing > value:
            array.insert(i, value)
            return
    array.append(value)

from random import randint
max_size = 10**4
data = [randint(0, max_size) for _ in range(max_size)]
test = lambda: insertion_sort(data)
```

파이썬에는 두 가지 내장 프로파일러가 있다. 하나는 순수하게 파이썬으로 작성됐고(profile) 다른 하나는 C확장 모듈로 돼 있는데(cProfile), cProfile 내장 모듈이 더 낫다. 프로파일 대상 프로그램의 성능에 최소로 영향을 미치기 때문이다. 순수 파이썬 버전은 부가 비용이 너무 많이 들어서 결과가 왜곡될 수 있다. 

파이썬 프로그램을 프로파일링할 때는 외부 시스템의 성능이 아니라 코드 자체의 성능을 측정하도록 유의해야 한다. 네트워크나 디스크에 있는 자원에 접근하는 함수에 주의하라.

cProfile 모듈에 있는 profile 객체를 인스턴스화하고, 이 인스턴스의 runcall 메서드를 사용해 테스트 함수를 실행한다. 

```python
from cProfile import Profile

profiler = Profile()
profiler.runcall(test)
```

테스트 함수가 실행되고 나면 pstats 내장 모듈에 들어 있는 Stats 클래스를 사용해 성능 통계를 추출할 수 있다. Stats에 들어 있는 여러 메서드를 사용해 관심 대상 프로파일 정보를 선택하고 정렬하는 방법을 조정해 내가 관심 있는 항목만 표시할 수 있다.

```python
from pstats import Stats

stats = Stats(profiler)
stats.strip_dirs()
stats.sort_stats('cumulative')   # 누적 통계
stats.print_stats()

>>>
20003 function calls in 1.256 seconds

   Ordered by: cumulative time

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.000    0.000    1.256    1.256 <ipython-input-3-4e4ec2c94850>:17(<lambda>)
        1    0.002    0.002    1.256    1.256 <ipython-input-3-4e4ec2c94850>:1(insertion_sort)
    10000    1.239    0.000    1.255    0.000 <ipython-input-3-4e4ec2c94850>:7(insert_value)
     9992    0.016    0.000    0.016    0.000 {method 'insert' of 'list' objects}
        8    0.000    0.000    0.000    0.000 {method 'append' of 'list' objects}
        1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
```

ncalls: 프로파일링 기간 동안 함수가 몇 번 호출됐는지

tottime: 프로파일링 기간 동안 함수를 실행하는 데 걸린 시간의 합계. 대상 함수가 다른 함수를 호출한 경우, 이 다른 함수를 실행하는 데 걸린 시간은 제외된다.

tottime percall: 프로파일링 기간 동안 함수가 호출될 때마다 걸린 시간의 평균을 보여준다. 대상 함수가 다른 함수를 호출한 경우, 이 다른 함수를 실행하기 위해 걸린 시간은 제외된다.  이 값은 tottime을 ncalls로 나눈 값과 같다.

cumtime: 함수를 실행할 때 걸린 누적 시간을 보여준다. 이 시간에는 대상 함수가 호출한 다른 함수를 실행하는 데 걸린 시간이 모두 포함된다. 

cumtime percall: 프로파일링 기간 동안 함수가 호출될 때마다 걸린 누적시간의 평균을 보여준다. 이 시간에는 대상 함수가 호출한 다른 함수를 실행하는 데 걸린 시간이 모두 포함된다. 이 값은 cumtime을 ncalls로 나눈 값과 같다.

위의 통계를 보면, 우리 코드에서 누적 시간으로 CPU를 가장 많이 사용한 함수는 insert_value 함수라는 점을 알 수 있다. 이 함수를 bisect 내장 모듈을 사용해 다시 구현했다.

```python
from bisect import bisect_left

def insert_value(array, value):
    i = bisect_left(array, value)
    array.insert(i, value)
```

```python
profiler = Profile()
profiler.runcall(test)

stats = Stats(profiler)
stats.strip_dirs()
stats.sort_stats('cumulative')   # 누적 통계
stats.print_stats()

>>>
30003 function calls in 0.029 seconds

   Ordered by: cumulative time

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.000    0.000    0.029    0.029 <ipython-input-3-4e4ec2c94850>:17(<lambda>)
        1    0.002    0.002    0.029    0.029 <ipython-input-3-4e4ec2c94850>:1(insertion_sort)
    10000    0.004    0.000    0.027    0.000 <ipython-input-7-9e3ac1f6182f>:3(insert_value)
    10000    0.018    0.000    0.018    0.000 {method 'insert' of 'list' objects}
    10000    0.005    0.000    0.005    0.000 {built-in method _bisect.bisect_left}
        1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
```

이전의 insert_value 함수에 비해 누적 실행 시간이 많이 줄었음을 확인할 수 있다. 

전체 프로그램을 프로파일링했는데, 공통 유틸리티 함수가 대부분의 실행 시간을 차지한다는 사실을 발견할 때도 있다. 프로그램의 여러 부분에서 이런 유틸리티 함수를 호출하기 때문에 프로파일러의 디폴트 출력을 사용하면 이런 상황을 제대로 이해하기 어려울 수 있다.

```python
def my_utility(a, b):
    c = 1
    for i in range(100):
        c += a * b

def first_func():
    for _ in range(1000):
        my_utility(4, 5)

def second_func():
    for _ in range(10):
        my_utility(1, 3)

def my_program():
    for _ in range(20):
        first_func()
        second_func()

profiler = Profile()
profiler.runcall(my_program)

stats = Stats(profiler)
stats.strip_dirs()
stats.sort_stats('cumulative')   # 누적 통계
stats.print_stats()

>>>
20242 function calls in 0.109 seconds

   Ordered by: cumulative time

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.000    0.000    0.109    0.109 <ipython-input-1-224d37765872>:14(my_program)
       20    0.003    0.000    0.108    0.005 <ipython-input-1-224d37765872>:6(first_func)
    20200    0.106    0.000    0.106    0.000 <ipython-input-1-224d37765872>:1(my_utility)
       20    0.000    0.000    0.001    0.000 <ipython-input-1-224d37765872>:10(second_func)
        1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
```

my_utility 함수가 실행 시간의 대부분을 차지하는 것은 분명하다. 하지만 이 함수가 왜 이렇게 많이 호출됐는지 즉시 알아보기는 어렵다. 이런 상황을 처리하고자 파이썬 프로파일러는 각 함수를 프로파일링한 정보에 대해 그 함수를 호출한 함수들이 얼마나 기여했는지를 보여주는 print_callers 메서드를 제공한다.

```python
stats.print_callers()

>>>
Ordered by: cumulative time

Function                                          was called by...
                                                      ncalls  tottime  cumtime
<ipython-input-1-224d37765872>:14(my_program)     <- 
<ipython-input-1-224d37765872>:6(first_func)      <-      20    0.003    0.108  <ipython-input-1-224d37765872>:14(my_program)
<ipython-input-1-224d37765872>:1(my_utility)      <-   20000    0.105    0.105  <ipython-input-1-224d37765872>:6(first_func)
                                                         200    0.001    0.001  <ipython-input-1-224d37765872>:10(second_func)
<ipython-input-1-224d37765872>:10(second_func)    <-      20    0.000    0.001  <ipython-input-1-224d37765872>:14(my_program)
{method 'disable' of '_lsprof.Profiler' objects}  <- 

```

왼쪽에 호출된 함수를, 오른쪽에 그 함수를 호출한 함수를 보여준다. 위 표를 보면 first_func가 my_utility 함수를 가장 많이 썼다는 점을 분명히 알 수 있다.

**기억해야 할 내용**
- 파이썬 프로그램을 느리게 하는 원인이 불분명한 경우가 많으므로 프로그램을 최적화하기 전에 프로파일링하는 것이 중요하다.
- profile 모듈 대신 cProfile 모듈을 사용하라. cProfile이 더 정확한 프로파일링 정보를 제공한다.
- 함수 호출 트리를 독립적으로 프로파일링하고 싶다면 Profile 객체의 runcall 메서드를 사용하기만 하면 된다.
- Stats 객체를 사용하면 프로파일링 정보 중에서 프로그램 성능을 이해하기 위해 살펴봐야 할 부분만 선택해 출력할 수 있다.