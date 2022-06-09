## Better way 38. 간단한 인터페이스의 경우 클래스 대신 함수를 받아라

API가 실행되는 과정에서 여러분이 전달한 함수를 실행하는 경우, 이런 함수를 훅(hook)이라고 부른다. 

예시) 리스트 타입의 sort 메서드에 key 훅으로 len 내장 함수를 전달.

```python
names = ['소크라테스', '아르키메데스', '플라톤', '아리스토텔레스']
names.sort(key=len)
```

defaultdict에는 딕셔너리 안에 없는 키에 접근할 경우 호출되는 인자가 없는 함수를 전달할 수 있다.

다음 코드는 존재하지 않는 키에 접근할 때 로그를 남기고 0을 디폴트 값으로 반환한다.

```python
def log_missing():
    print('키 추가됨')
    return 0

from collections import defaultdict
current = {'초록': 12, '파랑': 3}
increments = [
    ('빨강', 5),
    ('파랑', 17),
    ('주황', 9),
]
result = defaultdict(log_missing, current)
print('이전:', dict(result))
for key, amount in increments:
    result[key] += amount
print('이후:', dict(result))

>>>
이전: {'초록': 12, '파랑': 3}
키 추가됨
키 추가됨
이후: {'초록': 12, '파랑': 20, '빨강': 5, '주황': 9}
```

defaultdict에 전달하는 디폴트 값 훅이 존재하지 않는 키에 접근한 총횟수를 세고 싶은 경우?

파이썬에서는 클래스에 \_\_call\_\_ 특별 메서드를 정의할 수 있다. \_\_call\_\_을 사용하면 객체를 함수처럼 호출할 수 있다. 이런 방식으로 정의돼서 호출될 수 있는 모든 객체를 호출 가능(callable) 객체라고 부른다.

```python
class BetterCountMissing:
    def __init__(self):
        self.added = 0

    def __call__(self):
        self.added += 1
        return 0

counter = BetterCountMissing()
assert counter() == 0
assert callable(counter)

#
counter = BetterCountMissing()
result = defaultdict(counter, current) # __call__에 의존함
for key, amount in increments:
    result[key] += amount
assert counter.added == 2
```

**기억해야 할 내용**
- 파이썬의 여러 컴포넌트 사이에 간단한 인터페이스가 필요할 때는 클래스를 정의하고 인스턴스화하는 대신 간단히 함수를 사용할 수 있다.
- 파이썬 함수나 메서드는 일급 시민이다. 따라서 (다른 타입의 값과 마찬가지로) 함수나 함수 참조를 식에 사용할 수 있다.
- \_\_call\_\_ 특별 메서드를 사용하면 클래스의 인스턴스인 객체를 일반 파이썬 함수처럼 호출할 수 있다.
- 상태를 유지하기 위한 함수가 필요한 경우에는 상태가 있는 클로저를 정의하는 대신 \_\_call\_\_ 메서드가 있는 클래스를 정의할지 고려해보라.