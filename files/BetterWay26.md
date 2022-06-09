## Better way 26. functools.wrap을 사용해 함수 데코레이터를 정의하라.

데코레이터는 자신이 감싸고 있는 함수가 호출되기 전과 후에 코드를 추가로 실행해준다.

```python
def trace(func):
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        print(f'{func.__name__}({args!r}, {kwargs!r}) '
              f'-> {result!r}')
        return result
    return wrapper

@trace
def fibonacci(n):
    """Return n 번째 피보나치 수"""
    if n in (0, 1):
        return n
    return (fibonacci(n - 2) + fibonacci(n - 1))

fibonacci(4)

>>>
fibonacci((0,), {}) -> 0
fibonacci((1,), {}) -> 1
fibonacci((2,), {}) -> 1
fibonacci((1,), {}) -> 1
fibonacci((0,), {}) -> 0
fibonacci((1,), {}) -> 1
fibonacci((2,), {}) -> 1
fibonacci((3,), {}) -> 2
fibonacci((4,), {}) -> 3
3
```

이 코드는 잘 작동하지만 의도하지 않은 부작용이 있음. 데코레이터가 반환하는 함수(앞에서 감싸진 fibonacci 함수)의 이름이 fibonacci가 아니게 된다. 

```python
print(fibonacci)

>>>
<function trace.<locals>.wrapper at 0x7fd506a94680>
```

이렇게 되면 help(fibonacci)해도 fibonacci 함수의 맨 앞부분에 있는 독스트링이 출력되지 않고 엉뚱한 게 출력된다. 또한 데코레이터가 감싸고 있는 원래 함수의 위치를 찾을 수 없기 때문에 객체 직렬화도 깨진다.

문제를 해결하는 방법은 fuctools 내장 모듈에 정의된 wraps 도우미 함수를 사용하는 것이다. 이 함수는 데코레이터 작성을 돕는 데코레이터이다.

wraps를 wapper 함수에 적용하면 wraps가 데코레이터 내부에 들어가는 함수에서 중요한 메타데이터를 복사해 wrapper 함수에 적용해준다.

```python
from functools import wraps

def trace(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        print(f'{func.__name__}({args!r}, {kwargs!r}) '
              f'-> {result!r}')
        return result
    return wrapper

@trace
def fibonacci(n):
    """Return n 번째 피보나치 수"""
    if n in (0, 1):
        return n
    return (fibonacci(n - 2) + fibonacci(n - 1))
```

이제 help(fibonacci)했을 때 대코레이터로 감싸진 함수에 대해서도 원하는 결과를 볼 수 있고, pickle 객체 직렬화도 제대로 작동한다. 

**기억해야 할 내용**
- 파이썬 데코레이터는 실행 시점에 함수가 다른 함수를 변경할 수 있게 해주는 구문이다.
- 데코레이터를 사용하면 디버거 등 인트로스펙션을 사용하는 도구가 잘못 작동할 수 있다.
- 직접 데코레이터를 구현할 때 인트로스펙션에서 문제가 생기지 않길 바란다면 functools 내장 모듈의 wraps 데코레이터를 사용하라.