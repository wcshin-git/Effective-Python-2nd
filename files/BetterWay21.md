## Better way 21. 변수 영역과 클로저의 상호작용 방식을 이해하라

숫자로 이뤄진 list를 정렬하되, 정렬한 리스트의 앞쪽에는 우선순위를 부여한 몇몇 숫자를 위치시켜야 한다고 가정하자.

```python
def sort_priority(values, group):
    def helper(x):
        if x in group:
            return (0, x)
        return (1, x)
    values.sort(key=helper)

numbers = [8, 3, 1, 2, 5, 4, 7, 6]
group = {2, 3, 5, 7}
sort_priority(numbers, group)
print(numbers)

>>>
[2, 3, 5, 7, 1, 4, 6, 8]
```

이 함수가 잘 작동하는 이유

→ 파이썬이 클로저(closure)를 지원: 클로저란 자신이 정의된 영역 밖의 변수를 참조하는 함수다. 클로저로 인해 도우미 함수가 sort_priority 함수의 group 인자에 접근할 수 있다.

여기서 이 함수가 우선순위가 높은 원소가 있는지 여부도 반환하게 만들어보자.

```python
def sort_priority2(numbers, group):
    found = False
    def helper(x):
        if x in group:
            found = True # 문제를 쉽게 해결할 수 있을 것 같다
            return (0, x)
        return (1, x)
    numbers.sort(key=helper)
    return found

numbers = [8, 3, 1, 2, 5, 4, 7, 6]
group = {2, 3, 5, 7}
found = sort_priority2(numbers, group)
print('발견:', found)
print(numbers)

>>>
발견: False    # 이상한 결과가 나옴!
[2, 3, 5, 7, 1, 4, 6, 8]
```

왜 이럴까? 여러분이 식 안에서 변수를 참조할 때 파이썬 인터프리터는 이 참조를 해결하기 위해 다음 순서로 영역을 뒤진다. 

1. 현재 함수의 영역
2. 현재 함수를 둘러싼 영역(현재 함수를 둘러싸고 있는 함수 등)
3. 현재 코드가 들어 있는 모듈의 영역(전역 영역이라고도 부름)
4. 내장 영역(len, str 등의 함수가 들어 있는 영역)

식이 참조하는 이름에 해당하는 변수가 이 네 가지 영역에 없으면 NameError 예외가 발생한다.

변수에 값을 대입하는 것은 다른 방식으로 작동한다. 변수가 현재 영역에 이미 정의돼 있다면 그 변수의 값만 새로운 값으로 바뀐다. 하지만 변수가 현재 영역에 정의돼 있지 않다면 파이썬은 변수 대입을 변수 정의로 취급한다. 따라서 helper 함수의 클로저 안에서 대입문 found = True는 helper 영역 안에 새로운 변수를 정의하는 것으로 취급되지, sort_priority2 안에서 기존 변수에 값을 대입하는 것으로 취급되지는 않는다. 이를 **영역 지정 버그**라고 부르기도 함.

파이썬에는 클로저 밖으로 데이터를 끌어내는 특별한 구문이 있다: nonlocal문. nonlocal의 유일한 한계점은 (전역 영역을 더럽히지 못하도록) 모듈 수준 영역까지 변수 이름을 찾아 올라가지 않는다는 것뿐이다. 만약 모듈 수준 영역까지 변수 이름을 찾아야 한다면 global 문을 사용하기. 

```python
def sort_priority2(numbers, group):
    found = False
    def helper(x):
        nonlocal found       # 추가함
        if x in group:
            found = True
            return (0, x)
        return (1, x)
    numbers.sort(key=helper)
    return found
```

하지만 어떤 경우라도 nonlocal문을 사용하지 말라고 경고하고 싶다. 특히 함수가 길고 nonlocal문이 지정한 변수와 대입이 이뤄지는 위치의 거리가 멀면 함수 동작을 이해하기 더 힘들어진다.

**기억해야 할 내용**
- 클로저 함수는 자신이 정의된 영역 외부에서 정의된 변수도 참조할 수 있다.
- 기본적으로 클로저 내부에 사용한 대입문은 클로저를 감싸는 영역에 영향을 끼칠 수 없다.
- 클로저가 자신을 감싸는 영역의 변수를 변경한다는 사실을 표시할 때는 nonlocal문을 사용하라.
- 간단한 함수가 아닌 경우에는 nonlocal문을 사용하지 말라.