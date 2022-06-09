## Better way 23. 키워드 인자로 선택적인 기능을 제공하라

함수 호출 시 위치 기반 인자를 지정하려면 키워드 인자보다 앞에 지정해야 한다.

```python
def remainder(number, divisor):
    return number % divisor

remainder(number=20, 7)

>>>
SyntaxError: positional argument follows keyword argument
```

딕셔너리의 내용물을 사용해 remainder와 같은 함수를 호출하고 싶다면 **연산자를 사용할 수 있다. **연산자는 파이썬이 딕셔너리에 들어 있는 값을 함수에 전달하되 각 값에 대응하는 키를 키워드로 사용하도록 명령한다. 

```python
my_kwargs = {
    'number': 20,
    'divisor': 7,
}

assert remainder(**my_kwargs) == 6
```

```python
# **연산자를 위치 인자나 키워드 인자와 섞어서 함수를 호출할 수 있다. 다만 중복되는 인자가 없어야 한다.
my_kwargs = {
    'divisor': 7,
}

assert remainder(number=20, **my_kwargs) == 6

# **연산자를 여러 번 사용할 수도 있다. 다만 여러 딕셔너리에 겹치는 키가 없어야 한다.
my_kwargs = {
    'number': 20,
}

other_kwargs = {
    'divisor': 7,
}

assert remainder(**my_kwargs, **other_kwargs) == 6
```

아무 키워드 인자나 받는 함수를 만들고 싶다면, 모든 키워드 인자를 dict에 모아주는 **kwargs라는 파라미터를 사용한다. 

```python
def print_parameters(**kwargs):
    for key, value in kwargs.items():
        print(f'{key} = {value}')

print_parameters(alpha=1.5, beta=9, 감마=4)

>>>
alpha = 1.5
beta = 9
감마 = 4
```

**기억해야 할 내용**
- 함수 인자를 위치에 따라 지정할 수도 있고, 키워드를 사용해 지정할 수도 있다.
- 키워드를 사용하면 위치 인자만 사용할 때는 혼동할 수 있는 여러 인자의 목적을 명확히 할 수 있다.
- 키워드 인자와 디폴트 값을 함께 사용하면 기본 호출 코드를 마이그레이션하지 않고도 함수에 새로운 기능을 쉽게 추가할 수 있다.
- 선택적 인자는 항상 위치가 아니라 키워드를 사용해 전달돼야 한다.