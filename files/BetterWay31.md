## Better way 31. 인자에 대해 이터레이션할 때는 방어적이 돼라

이터레이터는 결과를 단 한 번만 만들어낸다. 이미 StopIteration 예외가 발생한 이터레이터나 제너레이터를 다시 이터레이션하면 아무 결과도 없을 수 없다.

```python
def read_visits(data_path):
    with open(data_path) as f:
        for line in f:
            yield int(line)

it = read_visits('my_numbers.txt')
print(list(it))
print(list(it)) # 이미 모든 원소를 다 소진했다

>>>
[15, 35, 80]
[]
```

혼란스럽게도 이미 소진된 이터레이터에 대해 이터레이션을 수행해도 아무런 오류가 발생하지 않는다. 이런 문제를 회피하기 위한 좋은 방법은 이터레이터 프로토콜을 구현한 새로운 컨테이너 클래스를 제공하는 것이다. 

파이썬에서는 for x in foo와 같은 구문을 사용하면, 실제로는 iter(foo)를 호출한다. iter 내장함수는 foo.__iter__라는 특별 메서드를 호출한다. __iter__ 메서드는 반드시 이터레이터 객체를 반환해야 한다. for루프는 반환받은 이터레이터 객체가 데이터를 소진(이터레이터는 StopIteration 예외를 던진다)할 때까지 반복적으로 이터레이터 객체에 대해 next 내장 함수를 호출한다. 

설명이 복잡해 보이지만, 실제로 코드를 작성할 때 여러분이 정의하는 클래스에서 __iter__ 메서드를 제너레이터로 구현하기만 하면 이 모든 동작을 만족시킬 수 있다. 

```python
class ReadVisits:
    def __init__(self, data_path):
        self.data_path = data_path

    def __iter__(self):
        with open(self.data_path) as f:
            for line in f:
                yield int(line)

def normalize(numbers):
    total = sum(numbers)
    result = []
    for value in numbers:
        percent = 100 * value / total
        result.append(percent)
    return result

path = 'my_numbers.txt'
visits = ReadVisits(path)
percentages = normalize(visits)
print(percentages)
assert sum(percentages) == 100.0

>>>
[11.538461538461538, 26.923076923076923, 61.53846153846154]
```

이 코드가 잘 작동하는 이유는 normalize 함수 안의 sum메서드가 ReadVisits.__iter__를 호출해서 새로운 이터레이터 객체를 할당하기 때문이다. 각 숫자를 정규화하기 위한 for 루프도 __iter__를 호출해서 두 번째 이터레이터 객체를 만든다. 두 이터레이터 객체는 서로 독립적으로 진행되고 소진된다. 

위 normalize함수 구현에서 입력으로 컨테이너여야 한다. 이를 검사하는 함수를 다음과 같이 만들 수 있다.

```python
from collections.abc import Iterator

def normalize_defensive(numbers):
    if isinstance(numbers, Iterator):
        raise TypeError('컨테이너를 제공해야 합니다') # 이터레이터면 sum같은 걸 했을 때 다 소진되고 뒤에 for문 같은 걸 추가적으로 돌릴 수 없음. 반면 컨테이너는 iter를 할 때마다 새로운 iterator를 만듦.
    total = sum(numbers)
    result = []
    for value in numbers:
        percent = 100 * value / total
        result.append(percent)
    return result

visits = [15, 35, 80]
percentages = normalize_defensive(visits)
assert sum(percentages) == 100.0

visits = ReadVisits(path)
percentages = normalize_defensive(visits)
assert sum(percentages) == 100.0

visits = [15, 35, 80]
it = iter(visits)
percentages = normalize_defensive(it) # 에러.
#이 함수는 컨테이너가 아닌 이터레이터를 입력으로 받으면 에러 발생.
```

normalize_defensive함수는 리스트와 ReadVisits에 대해 모두 제대로 작동한다. 리스트나 ReadVisits 모두 이터레이터 프로토콜을 따르는 **이터러블 컨테이너**이기 때문이다. 

**기억해야 할 내용**
- 입력 인자를 여러 번 이터레이션하는 함수나 메서드를 조심하라. 입력받은 인자가 이터레이터면 함수가 이상하게 작동하거나 결과가 없을 수 있다.
- 파이썬의 이터레이터 프로토콜은 컨테이너와 이터레이터가 iter, next 내장 함수나 for 루프 등의 관련 식과 상호작용하는 절차를 정의한다.
- __iter__ 메서드를 제너레이터로 정의하면 쉽게 이터러블 컨테이너 타입을 정의할 수 있다.
- 어떤 값이 (컨테이너가 아닌) 이터레이터인지 감지하려면, 이 값을 iter 내장 함수에 넘겨서 반환되는 값이 원래 값과 같은지 확인하면 된다. 다른 방법으로 collections.abc.Iterator 클래스를 isinstance와 함께 사용할 수도 있다.

질문: $\text{컨테이너} \subset \text{이터레이터}?$ 여기서의 컨테이너는 __contains__를 구현한 Container를 의미하는 게 아님. 그저 iter를 할 때마다 새로운 iterator를 만든 것이라고 생각하기.