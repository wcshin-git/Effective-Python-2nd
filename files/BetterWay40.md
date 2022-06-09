## Better way 40. super로 부모 클래스를 초기화하라.

자식 클래스에서 부모 클래스를 초기화하는 오래된 방법은 바로 자식 인스턴스에서 부모 클래스의 \_\_init\_\_ 메서드를 직접 호출하는 것이다.

```python
class MyBaseClass:
    def __init__(self, value):
        self.value = value

class MyChildClass(MyBaseClass):
    def __init__(self):
        MyBaseClass.__init__(self, 5)
```

이 접근 방법은 기본적인 클래스 계층의 경우에는 잘 작동하지만, 다른 경우에는 잘못될 수도 있다.

어떤 클래스가 다중 상속에 의해 영향을 받는 경우(일반적으로는 다중 상속을 피해야 한다. Better way 41: '기능을 합성할 때는 믹스인 클래스를 사용하라' 참고), 상위 클래스의 \_\_init\_\_ 메서드를 직접 호출하면 프로그램이 예측할 수 없는 방식으로 작동할 수 있다. 

다중 상속을 사용하는 경우 생기는 문제 중 하나는 모든 하위 클래스에서 \_\_init\_\_ 호출의 순서가 정해져 있지 않다는 것이다. 이는 클래스 정의에서 부모 클래스를 나열한 순서와 부모 클래스의 생성자를 호출하는 순서가 일치하지 않을 수 있다는 뜻이다.

```python
class MyBaseClass:
    def __init__(self, value):
        self.value = value

class TimesTwo:
    def __init__(self):
        self.value *= 2

class PlusFive:
    def __init__(self):
        self.value += 5

#
class OneWay(MyBaseClass, TimesTwo, PlusFive):
    def __init__(self, value):
        MyBaseClass.__init__(self, value)
        TimesTwo.__init__(self)
        PlusFive.__init__(self)

foo = OneWay(5)
print('첫 번째 부모 클래스 순서에 따른 값은 (5 * 2) + 5 =', foo.value)

>>>
첫 번째 부모 클래스 순서에 따른 값은 (5 * 2) + 5 = 15

#
class AnotherWay(MyBaseClass, PlusFive, TimesTwo):
    def __init__(self, value):
        MyBaseClass.__init__(self, value)
        TimesTwo.__init__(self)
        PlusFive.__init__(self)

bar = AnotherWay(5)
print('두 번째 부모 클래스 순서에 따른 값은', bar.value)

>>>
두 번째 부모 클래스 순서에 따른 값은 15
```

다이아몬드 상속으로 인해 다른 문제가 생길 수도 있다. 다이아몬드 상속이란 어떤 클래스가 두 가지 서로 다른 클래스를 상속하는데, 두 상위 클래스의 상속 계층을 거슬러 올라가면 같은 조상 클래스가 존재하는 경우를 뜻한다. 다이아몬드 상속이 이뤄지면 공통 조상 클래스의 \_\_init\_\_ 메서드가 여러 번 호출될 수 있기 때문에 코드가 예상치 않은 방식으로 작동할 수 있다.

```python
class TimesSeven(MyBaseClass):
    def __init__(self, value):
        MyBaseClass.__init__(self, value)
        self.value *= 7

class PlusNine(MyBaseClass):
    def __init__(self, value):
        MyBaseClass.__init__(self, value)
        self.value += 9

class ThisWay(TimesSeven, PlusNine):
    def __init__(self, value):
        TimesSeven.__init__(self, value)
        PlusNine.__init__(self, value)

foo = ThisWay(5)
print('(5 * 7) + 9 = 44가 나와야 하지만 실제로는', foo.value)

>>>
(5 * 7) + 9 = 44가 나와야 하지만 실제로는 14
```

이러한 문제를 해결하기 위해 파이썬에는 super라는 내장 함수와 표준 메서드 결정 순서(MRO)가 있다. super를 사용하면 다이아몬드 계층의 공통 상위 클래스를 단 한 번만 호출하도록 보장한다. MRO는 상위 클래스를 초기화하는 순서를 정의한다. 

```python
class TimesSevenCorrect(MyBaseClass):
    def __init__(self, value):
        super().__init__(value)
        self.value *= 7

class PlusNineCorrect(MyBaseClass):
    def __init__(self, value):
        super().__init__(value)
        self.value += 9

class GoodWay(TimesSevenCorrect, PlusNineCorrect):
    def __init__(self, value):
        super().__init__(value)

foo = GoodWay(5)
print('7 * (5 + 9) = 98이 나와야 하고 실제로도', foo.value)

>>>
7 * (5 + 9) = 98이 나와야 하고 실제로도 98
```

처음에는 순서가 거꾸로 된 것처럼 보일 것이다. TimesSevenCorrect.__init__이 먼저 호출돼야 하는 것 아닐까? 결과가 (5 * 7) + 9 = 44여야 하지 않을까? 대답은 '아니오'다. 호출 순서는 이 클래스에 대한 MRO 정의를 따른다. 상속 다이아몬드의 정점에 도달하면 각 초기화 메서드는 각 클래스의 __init__이 호출된 순서의 역순으로 작업을 수행하기 때문. 

super 함수에 두 가지 파라미터를 넘길 수 있다. 첫 번째 파라미터는 여러분이 접근하고 싶은 MRO 뷰를 제공할 부모 타입이고, 두 번째 파라미터는 첫 번째 파라미터로 지정한 타입의 MRO 뷰에 접근할 때 사용할 인스턴스다. 하지만 여러분이 클래스 정의 안에서 아무 인자도 지정하지 않고 super를 호출해도 파이썬 컴파일러가 자동으로 올바른 파라미터(__class__와 self)를 넣어준다. 따라서 아래 세 가지 사용법은 모두 동일하다.

```python
class ExplicitTrisect(MyBaseClass):
    def __init__(self, value):
        super(ExplicitTrisect, self).__init__(value)
        self.value /= 3

class AutomaticTrisect(MyBaseClass):
    def __init__(self, value):
        super(__class__, self).__init__(value)
        self.value /= 3

class ImplicitTrisect(MyBaseClass):
    def __init__(self, value):
        super().__init__(value)
        self.value /= 3

assert ExplicitTrisect(9).value == 3
assert AutomaticTrisect(9).value == 3
assert ImplicitTrisect(9).value == 3
```

super에 파라미터를 제공해야 하는 유일한 경우는 자식 클래스에서 상위 클래스의 특정 기능에 접근해야 하는 경우뿐이다(예: 특정 기능을 감싸거나 재사용해야 하는 경우).

**기억해야 할 내용**
- 파이썬은 표준 메서드 결정 순서(MRO)를 활용해 클래스 초기화 순서와 다이아몬드 상속 문제를 해결한다.
- 부모 클래스를 초기화할 때는 super 내장 함수를 아무 인자 없이 호출하라. super를 아무 인자 없이 호출하면 파이썬 컴파일러가 자동으로 올바른 파라미터를 넣어준다.