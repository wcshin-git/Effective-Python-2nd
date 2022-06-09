메타클래스를 사용하면 파이썬의 class문을 가로채서 클래스가 정의될 때마다 특별한 동작을 제공할 수 있다.

파이썬에서는 @property를 사용하면 getter, setter를 간단하게 구현할 수 있음.

- **@property**
- **@메서드이름.setter**

```python
# from 파이썬 코딩도장
# 변경 전
class Person:
    def __init__(self):
        self.__age = 0
 
    def get_age(self):           # getter
        return self.__age
    
    def set_age(self, value):    # setter
        self.__age = value
 
james = Person()
james.set_age(20)
print(james.get_age())

# 변경 후
class Person:
    def __init__(self):
        self.__age = 0
 
    @property
    def age(self):           # getter
        return self.__age
 
    @age.setter
    def age(self, value):    # setter
        self.__age = value
 
james = Person()
james.age = 20      # 인스턴스.속성 형식으로 접근하여 값 저장
print(james.age)    # 인스턴스.속성 형식으로 값을 가져옴
```

getter, setter 메서드의 이름을 잘 보면 둘다 age입니다. 그리고 getter에는 @property가 붙어있고, setter에는 @age.setter가 붙어있습니다. 즉, 값을 가져오는 메서드에는 @property 데코레이터를 붙이고, 값을 저장하는 메서드에는 @메서드이름.setter 데코레이터를 붙이는 방식입니다.

특히 @property와 @age.setter를 붙이면 james.age처럼 메서드를 속성처럼 사용할 수 있습니다. 값을 저장할 때는 james.age = 20처럼 메서드에 바로 값을 할당하면 되고, 값을 가져올 때도 james.age처럼 메서드에 바로 접근하면 됩니다.

## Better way 44. 세터와 게터 메서드 대신 평범한 에트리뷰트를 사용하라

파이썬에서는 명시적인 세터나 게터 메서드를 구현할 필요가 전혀 없다. 대신 다음과 같이 항상 단순한 공개 애트리뷰트로부터 구현을 시작하라.

```python
class Resistor:
    def __init__(self, ohms):
        self.ohms = ohms
        self.voltage = 0
        self.current = 0

r1 = Resistor(50e3)
r1.ohms = 10e3

r1.ohms += 5e3
```

나중에 애트리뷰트가 설정될 때 특별한 기능을 수행해야 한다면, 애트리뷰트를 @property와 @메서드이름.setter로 옮겨갈 수 있다. 다음 코드는 voltage 프로퍼티에 값을 대입하면 current 값이 바뀐다.

 

```python
class Resistor:
    def __init__(self, ohms):
        self.ohms = ohms
        self.voltage = 3082  # 시험삼아 3082로 설정
        self.current = 0     # 재밌는 사실: self.voltage = 3082이 가장 마지막에 쓰이면 self.current값이 0이 아닌 프로퍼티에 의해 계산된 값으로 초기화된다.

class VoltageResistance(Resistor):
    def __init__(self, ohms):
        super().__init__(ohms)
        self._voltage = 0  # 이게 없으면 self.voltage의 초기값이 3082인데 이를 0으로 초기화

    @property
    def voltage(self):           # getter
        return self._voltage

    @voltage.setter
    def voltage(self, voltage):  # setter.  getter와 이름이 똑같아야 함
        self._voltage = voltage   # self._voltage = voltage로 안 쓰고 self.voltage = voltage로 쓰면 무한루프에 빠짐
        self.current = self._voltage / self.ohms

r2 = VoltageResistance(1e3)
print(r2.voltage)
print(f'이전: {r2.current:.2f} 암페어')
r2.voltage = 10
print(f'이후: {r2.current:.2f} 암페어')

>>>
0
이전: 0.00 암페어
이후: 0.01 암페어
```

self.voltage를 self._voltage로 바꿔치기 했다고 이해할 수 있음!

장점: self.voltage = value를 할 때마다 self.current도 함께 바뀐다! 

전달된 값에 대한 검증을 수행하는 것도 가능하다.

```python
class BoundedResistance(Resistor):
    def __init__(self, ohms):
        super().__init__(ohms)

    @property
    def ohms(self):
        return self._ohms

    @ohms.setter
    def ohms(self, ohms):
        if ohms <= 0:
            raise ValueError(f'저항 > 0이어야 합니다. 실제 값: {ohms}')
        self._ohms = ohms

r3 = BoundedResistance(1e3)

# 오류가 나는 부분. 오류를 보고 싶으면 커멘트를 해제할것
#r3.ohms = 0

# 생성자에 잘못된 값을 넘기는 경우에도 오류가 발생.
#BoundedResistance(-5)
```

생성자에 잘못된 값을 넘기는 경우에도 오류가 발생하는 이유는 BoundedResistance.__init__이 Resistor.__init__을 호출하고 이 초기화 메서드는 다시 self.ohms = -5라는 대입문을 실행하기 때문이다. 이 대입으로 인해 BoundedResistance에 있는 @ohms.setter 메서드가 호출되고, 이 세터 메서드는 객체 생성이 끝나기 전에 즉시 저항을 검증하는 코드를 실행한다. 

@property를 사용해 부모 클래스에 정의된 애트리뷰트를 불변으로 만들 수도 있다. 

```python
class FixedResistance(Resistor):
    def __init__(self, ohms):
        super().__init__(ohms)

    @property
    def ohms(self):
        return self._ohms

    @ohms.setter
    def ohms(self, ohms):
        if hasattr(self, '_ohms'):
            raise AttributeError("Ohms는 불변객체입니다")
        self._ohms = ohms

r4 = FixedResistance(1e3)
# 오류가 나는 부분. 오류를 보고 싶으면 커멘트를 해제할것
#r4.ohms = 2e3
```

**주의사항**

게터 프로퍼티 메서드 안에서 다른 애트리뷰트를 설정하면 안 된다. 게터 프로퍼티 메서드에서 다른 애트리뷰트를 설정하면 코드가 아주 이상하게 작동할 수 있기 때문이다.

```python
class Resistor:
    def __init__(self, ohms):
        self.ohms = ohms
        self.current = 0
        self.voltage = 0

class MysteriousResistor(Resistor):
    @property
    def ohms(self):
        self.voltage = self._ohms * self.current
        return self._ohms

    @ohms.setter
    def ohms(self, ohms):
        self._ohms = ohms

r7 = MysteriousResistor(10)
r7.current = 0.01
print(f'이전: {r7.voltage:.2f}')
r7.ohms
print(f'이후: {r7.voltage:.2f}')

>>>
이전: 0.00
이후: 0.10
```

게터나 세터를 정의할 때 가장 좋은 방법은 관련이 있는 개체 속성을 **@property.setter 메서드 안에서만 변경**하는 것이다. 또한 클래스를 사용하는 쪽에서는 케터나 세터를 통하는 애트리뷰트가 다른 일반적인 파이썬 객체와 마찬가지로 빠르고 사용하기 쉬울 것으로 예상하기 때문에, 더 복잡하거나 느린 연산의 경우에는 일반적인 메서드를 사용하라.

@property의 가장 큰 단점은 애트리뷰트를 처리하는 메서드가 하위 클래스 사이에서만 공유될 수 있다는 것이다. 서로 관련이 없는 클래스 사이에 같은 프로퍼티 게터나 세터 구현을 공유할 수는 없다. 하지만 파이썬은 재사용 가능한 프로퍼티 로직을 구현할 때는 물론 다른 여러 용도에도 사용할 수 있는 디스크립터(descriptor)를 제공한다.

**기억해야 할 내용**
- 새로운 클래스 인터페이스를 정의할 때는 간단한 공개 애트리뷰트에서 시작하고, 세터나 게터 메서드를 가급적 사용하지 말라.
- 객체에 있는 애트리뷰트에 접근할 때 특별한 동작이 필요하면 @property로 이를 구현할 수 있다.
- @property 메서드를 만들 때는 최소 놀람의 법칙을 따르고 이상한 부작용을 만들어내지 말라.
- @property 메서드가 빠르게 실행되도록 유지하라. 느리거나 복잡한 작업의 경우에는 프로퍼티 대신 일반적인 메서드를 사용하라.