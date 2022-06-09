## Better way 69. 정확도가 매우 중요한 경우에는 decimal을 사용하라

decimal 내장 모듈에 들어 있는 Decimal 클래스는 디폴트로 소수점 이하 28번째 자리까지 고정소수점 수 연산을 제공한다. 심지어 자릿수를 더 늘릴 수도 있다.

```python
rate = 1.45
seconds = 3*60 + 42
cost = rate * seconds / 60
print(cost)

print(round(cost, 2))

from decimal import Decimal

rate = Decimal('1.45')
seconds = Decimal(3*60 + 42)
cost = rate * seconds / Decimal(60)
print(cost)

>>>
5.364999999999999
5.36
5.365
```

Decimal 인스턴스에 값을 지정하는 방법은 두 가지가 있다. 첫째, 숫자가 들어 있는 str을 Decimal 생성자에 전달하는 방법이다. 이렇게 하면 파이썬 부동소수점 수의 근본적인 특성으로 인해 발생하는 정밀도 손실을 막을 수 있다. 둘째, int나 float 인스턴스를 생성자에 전달하는 방법이다. 다음 코드를 보면 두 생성 방법의 결과가 서로 다름을 알 수 있다.

```python
print(Decimal('1.45'))
print(Decimal(1.45))

>>>
1.45
1.4499999999999999555910790149937383830547332763671875
```

Decimal 생성자에 정수를 넘기는 경우에는 이런 문제가 생기지 않는다.

```python
print(Decimal('456'))
print(Decimal(456))

>>>
456
456
```

**기억해야 할 내용**
- 파이썬은 실질적으로 모든 유형의 숫자 값을 표현할 수 있는 내장 타입과 클래스를 제공한다.
- 돈과 관련된 계산 등과 같이 높은 정밀도가 필요하거나 근삿값 계산을 원하는 대로 제어해야 할 때는 Decimal 클래스가 이상적이다.
- 부동소수점 수로 계산한 근사값이 아니라 정확한 답을 계산해야 한다면 Decimal 생성자에 float 인스턴스 대신 str 인스턴스를 넘겨라.