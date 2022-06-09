## Better way 20. None을 반환하기보다는 예외를 발생시켜라

함수의 return 값이 None인지 검사하는 대신, 실수로 False인지 검사할 경우 버그가 발생할 수 있다.

```python
def careful_divide(a, b):
    try:
        return a / b
    except ZeroDivisionError:
        return None

x, y = 0, 5
result = careful_divide(x, y)
if not result:
    print('잘못된 입력') # 버그: result가 None일 때 뿐만 아니라 0일 때도 작동
```

더 나은 방법은 None을 반환하지 않고 에러를 발생시키기.

```python
def careful_divide(a, b):
    try:
        return a / b
    except ZeroDivisionError as e:
        raise ValueError('잘못된 입력')

x, y = 5, 0
try:
    result = careful_divide(x, y)
except ValueError:
    print('잘못된 입력')
else:
    print('결과는 %.1f 입니다' % result)

>>>
잘못된 입력
```

**기억해야 할 내용**
- 특별한 의미를 표시하는 None을 반환하는 함수를 사용하면 None과 다른 값(예: 0이나 빈 문자열)이 조건문에서 False로 평가될 수 있기 때문에 실수하기 쉽다.
- 특별한 상황을 표현하기 위해 None을 반환하는 대신 예외를 발생시켜라. 문서에 예외 정보를 기록해 호출자가 예외를 제대로 처리하도록 하라.
- 함수가 특별한 경우를 포함하는 그 어떤 경우에도 절대로 None을 반환하지 않는다는 사실을 타입 애너테이션으로 명시할 수 있다.