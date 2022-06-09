## Better way 14. 복잡한 기준을 사용해 정렬할 때는 key 파라미터를 사용하라

리스트의 sort 메서드는 리스트의 내용을 원소 타입에 따른 자연스러운 순서를 사용해 오름차순으로 정렬한다. 

```python
numbers = [93, 86, 11, 68, 70]
numbers.sort()
print(numbers)

>>>
[11, 68, 70, 86, 93]
```

sort에는 key라는 파라미터가 있다. key는 함수여야 한다. key함수에는 정렬 중인 리스트의 원소가 전달된다. key함수가 반환하는 값은 원소 대신 정렬 시준으로 사용할, 비교 가능한(즉, 자연스러운 순서가 정의된) 값이어야만 한다. 

```python
class Tool:
    def __init__(self, name, weight):
        self.name = name
        self.weight = weight

    def __repr__(self):
        return f'Tool({self.name!r}, {self.weight})'  # !r을 붙이면 raw string이라 하여 escape문이 동작되지 않고, 있는 그대로 출력된다.

tools = [
    Tool('수준계', 3.5),
    Tool('해머', 1.25),
    Tool('스크류드라이버', 0.5),
    Tool('끌', 0.25),
]

print('미정렬:', repr(tools))
tools.sort(key=lambda x: x.name)
print('\n정렬: ', tools)

>>>
미정렬: [Tool('수준계', 3.5), Tool('해머', 1.25), Tool('스크류드라이버', 0.5), Tool('끌', 0.25)]

정렬:  [Tool('끌', 0.25), Tool('수준계', 3.5), Tool('스크류드라이버', 0.5), Tool('해머', 1.25)]
```

여러 기준을 사용해 정렬하고 싶을 때는?

```python
power_tools = [
    Tool('드릴', 4),
    Tool('원형 톱', 5),
    Tool('착암기', 40),
    Tool('연마기', 4),
]

power_tools.sort(key=lambda x: (x.weight, x.name)) # 먼저 weight으로 정렬하고 그 후 name으로 정렬.
print(power_tools)

power_tools.sort(key=lambda x: (x.weight, x.name),
                 reverse=True) # 모든 비교 기준을 내림차순으로 만든다.
print(power_tools)

power_tools.sort(key=lambda x: (-x.weight, x.name))  # 먼저 weight으로 내림차순 정렬, 그 후 name으로 오름차순 정렬.
print(power_tools)

>>>
[Tool('드릴', 4), Tool('연마기', 4), Tool('원형 톱', 5), Tool('착암기', 40)]
[Tool('착암기', 40), Tool('원형 톱', 5), Tool('연마기', 4), Tool('드릴', 4)]
[Tool('착암기', 40), Tool('원형 톱', 5), Tool('드릴', 4), Tool('연마기', 4)]
```

아쉽게도 모든 타입에 부호 반전을 사용할 수는 없다. str타입에는 안 됨. 이 때는 sort를 두 번 호출하는 방식으로 정렬해야 한다.

```python
# 먼저 weight으로 내림차순 정렬, 그 후 name으로 오름차순 정렬
# sort 호출 순서는 역순이라는 것에 주의!
power_tools.sort(key=lambda x: x.name)   # name 기준 오름차순
power_tools.sort(key=lambda x: x.weight, # weight 기준 내림차순
                 reverse=True)
print(power_tools)

>>>
[Tool('착암기', 40), Tool('원형 톱', 5), Tool('드릴', 4), Tool('연마기', 4)]
```

이와 같은 접근 방법을 사용하면 여러 다른 타입의 정렬 기준을 여러분이 원하는 방향으로 서로 조합할 수 있다. 그렇지만 key 함수를 사용해 tuple을 반환하고 여기에 단항 부호 반전연산을 활용하는 접근 방식이 코드가 더 적고 읽기도 쉽다. 꼭 필요할 때만 sort를 호출하는 방법을 사용하는 것을 권장한다.

**기억해야 할 내용**
- 리스트 타입에 들어 있는 sort 메서드를 사용하면 원소 타입이 문자열, 정수, 튜플 등과 같은 내장 타입인 경우 자연스러운 순서로 리스트의 원소를 정렬할 수 있다.
- 원소 타입에 특별 메서드를 통해 자연스러운 순서가 정의돼 있지 않으면 sort 메서드를 쓸 수 없다.
- sort 메서드의 key 파라미터를 사용하면 리스트의 각 원소 대신 비교에 사용할 객체를 반환하는 도우미 함수를 제공할 수 있다.
- key 함수에서 튜플을 반환하면 여러 정렬 기준을 하나로 엮을 수 있다. 단항 부호 반전 연산자를 사용하면 정렬 순서를 반대로 바꿀 수 있다.
- 부호를 바꿀 수 없는 타입의 경우 여러 정렬 기준을 조합하려면 각 정렬 기준마다 reverse 값으로 정렬 순서를 지정하면서 sort 메서드를 여러 번 사용해야 한다.이때 정렬 기준의 우선순위가 점점 높아지는 순서로 sort를 호출해야 한다.