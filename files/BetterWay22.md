## Better way 22. 변수 위치 인자를 사용해 시각적인 잡음을 줄여라

위치 인자(positional argument)를 가변적으로 받을 수 있을까? 이를 가변 인자(varargs)나 스타 인자(star args)라고 부르기도 한다.

수정 전

```python
def log(message, values):
    if not values:
        print(message)
    else:
        values_str = ', '.join(str(x) for x in values)
        print(f'{message}: {values_str}')

log('내 숫자는 ', [1, 2])
log('안녕 ', [])

>>>
내 숫자는 : 1, 2
안녕
```

수정 후

```python
def log(message, *values):  # 달라진 유일한 부분
    if not values:
        print(message)
    else:
        values_str = ', '.join(str(x) for x in values)
        print(f'{message}: {values_str}')

log('내 숫자는 ', [1, 2])
log('안녕 ')  # 훨씬 좋다

>>>
내 숫자는 : 1, 2
안녕
```

가변인자를 써도 함수 본문은 바뀌지 않는다. 단지 호출하는 코드만 바뀐다. 

이미 시퀀스(리스트 등)가 있는데 log와 같은 가변 인자 함수에 시퀀스를 사용하고 싶다면 *연산자를 사용하면 된다. *연산자는 파이썬이 시퀀스의 원소들을 함수의 위치 인자로 넘길 것을 명령한다. 

```python
favorites = [7, 33, 99]
log('좋아하는 숫자는', *favorites)

>>>
좋아하는 숫자는: 7, 33, 99
```

가변적인 위치 인자를 받는 데는 두 가지 문제점이 있다.

1. 이러한 선택적인 위치 인자가 함수에 전달되기 전에 항상 튜플로 변환됨. 따라서 *args를 받는 함수는 인자 목록에서 가변적인 부분에 들어가는 인자의 개수가 처리하기 좋을 정도로 충분히 작다는 사실을 이미 알고 있는 경우에 가장 적합하다. (인자 개수가 너무 많으면 OOM으로 프로그램이 중단될 수 있음)
2. 함수에 새로운 위치 인자를 추가하면 해당 함수를 호출하는 모든 코드를 변경해야만 한다. 

```python
def log(sequence, message, *values):
    if not values:
        print(f'{sequence} - {message}')
    else:
        values_str = ', '.join(str(x) for x in values)
        print(f'{sequence} - {message}: {values_str}')

log('좋아하는 숫자는', 7, 33)      # 예전 방식 코드는 깨짐
>>>
좋아하는 숫자는 - 7: 33
```

이런 가능성을 완전히 없애려면 *args를 받아들이는 함수를 확장할 때는 키워드 기반의 인자만 사용해야 한다.

**기억해야 할 내용**
- def 문에서 *args를 사용하면 함수가 가변 위치 기반 인자를 받을 수 있다.
- *연산자를 사용하면 가변 인자를 받는 함수에게 시퀀스 내의 원소들을 전달할 수 있다.
- 제너레이터에 *연산자를 사용하면 프로그램이 메모리를 모두 소진하고 중단될 수 있다.
- *args를 받는 함수에 새로운 위치 기반 인자를 넣으면 감지하기 힘든 버그가 생길 수 있다.