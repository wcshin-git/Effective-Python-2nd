## Better way 25. 위치로만 인자를 지정하게 하거나 키워드로만 인자를 지정하게 해서 함수 호출을 명확하게 만들라

복잡한 함수의 경우 호출자로 하여금 키워드만 사용하는 인자를 통해 의도를 명확히 밝히도록 요구하는 편이 좋다. 키워드만 사용하는 인자는 키워드를 반드시 사용해 지정해야 하며, 절대 위치를 기반으로는 지정할 수 없다. 

다음은 safe_division 함수가 키워드만 사용하는 인자만 받도록 만든 코드다. 인자 목록에 있는 * 기호는 위치 인자의 마지막과 키워드만 사용하는 인자의 시작을 구분해준다. 

```python
def safe_division_c(number, divisor, *,         # 변경
                    ignore_overflow=False,
                    ignore_zero_division=False):
    try:
        return number / divisor
    except OverflowError:
        if ignore_overflow:
            return 0
        else:
            raise
    except ZeroDivisionError:
        if ignore_zero_division:
            return float('inf')
        else:
            raise

safe_division_c(1.0, 10**500, True, False)

>>>
TypeError: safe_division_c() takes 2 positional arguments but 4 were given
```

파이썬 3.8에는 위치로만 지정하는 인자가 있다. 위치로만 지정하는 인자는 반드시 위치만 사용해 인자를 지정해야 하고 키워드 인자로는 쓸 수 없다.(앞에서 설명한 키워드만 사용하는 인자와는 정반대)

다음 safe_division 함수는 처음 두 필수 인자를 위치로만 지정하는 인자로 지정한다. 인자 목록의 / 기호는 위치로만 지정하는 인자의 끝을 표시한다.

```python
def safe_division_d(numerator, denominator, /, *, # 변경
                    ignore_overflow=False,
                    ignore_zero_division=False):
...

safe_division_d(numerator=2, denominator=5)

>>>
TypeError: safe_division_d() got some positional-only arguments passed as keyword arguments: 'numerator, denominator'
```

이렇게 만듦으로써 앞의 파라미터 두 개의 이름을 numerator, denominator가 아니라 다른 걸로 바꿔도 코드가 망가지지 않는다.(위치 기반 인자로 함수 호출을 하게끔 강요하고 있으니까)

인자목록에서 /와 * 기호 사이에 있는 모든 파라미터는 위치를 사용해 전달할 수도 있고 이름을 키워드로 사용해 전달할 수도 있다.

```python
def safe_division_e(numerator, denominator, /,
                    ndigits=10, *,                 # 변경
                    ignore_overflow=False,
                    ignore_zero_division=False):
...
# ndigits은 위치를 사용해 전달할 수도 있고 키워드를 사용해 전달할 수도 있음 
```

**기억해야 할 내용**
- 키워드로만 지정해야 하는 인자를 사용하면 호출하는 쪽에서 특정 인자를 반드시 키워드를 사용해 호출하도록 강제할 수 있다. 이로 인해 함수 호출의 의도를 명확히 할 수 있다. 키워드로만 지정해야 하는 인자는 인자 목록에서 * 다음에 위치한다.
- 위치로만 지정해야 하는 인자를 사용하면 호출하는 쪽에서 키워드를 사용해 인자를 지정하지 못하게 만들 수 있고, 이에 따라 함수 구현과 함수 호출 지점 사이의 결합을 줄일 수 있다. 위치로만 지정해야 하는 인자는 인자 목록에서 /앞에 위치한다.
- 인자 목록에서 / 와 * 사이에 있는 파라미터는 키워드를 사용해 전달해도 되고 위치를 기반으로 전달해도 된다. 이런 동작은 파이썬 함수 파라미터의 기본 동작이다.