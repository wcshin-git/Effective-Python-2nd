## Better way 42. 비공개 애트리뷰트보다는 공개 애트리뷰트를 사용하라

파이썬에서 클래스의 애트리뷰트에 대한 가시성은 공개와 비공개, 두 가지밖에 없다.

```python
class MyObject:
    def __init__(self):
        self.public_field = 5
        self.__private_field = 10 # 애트리뷰트 이름 앞에 밑줄 두 개 붙이면 비공개 필드가 된다.

    def get_private_field(self):
        return self.__private_field  # 메서드에서는 비공개 애트리뷰트에 접근 가능

foo = MyObject()
assert foo.public_field == 5

assert foo.get_private_field() == 10

foo.__private_field # 오류남. 클래스 외부에서 비공개 필드에 접근하면 에러가 발생
```

  클래스 메서드는 자신을 둘러싸고 있는 class 블록 내부에 들어 있기 때문에 해당 클래스의 비공개 필드에 접근할 수 있다.

```python
class MyOtherObject:
    def __init__(self):
        self.__private_field = 71

    @classmethod
    def get_private_field_of_instance(cls, instance):
        return instance.__private_field

bar = MyOtherObject()
assert MyOtherObject.get_private_field_of_instance(bar) == 71
```

하위 클래스는 부모 클래스의 비공개 필드에 접근할 수 없다.

```python
class MyParentObject:
    def __init__(self):
        self.__private_field = 71

class MyChildObject(MyParentObject):
    def get_private_field(self):
        return self.__private_field

baz = MyChildObject()

baz.get_private_field() # 에러남
```

하지만 아예 접근이 불가능한 것은 아님. 클래스 위부에서든 하위 클래스에서든 원하는 클래스의 비공개 애트리뷰트에 접근할 수 있다.

```python
assert foo._MyObject__private_field == 10
assert baz._MyParentObject.__private_field == 71
print(baz.__dict__)

>>>
{'_MyParentObject__private_field': 71}
```

그러나 비공개 애트리뷰트를 사용하면 확장이나 하위 클래스의 오버라이드를 귀찮게 하고 깨지기 쉽게 만들 뿐이다.

```python
# 확장 전
class MyStringClass:
    def __init__(self, value):
        self.__value = value

    def get_value(self):
        return str(self.__value)

foo = MyStringClass(5)
assert foo.get_value() == '5'

class MyIntegerSubclass(MyStringClass):
    def get_value(self):
        return int(self._MyStringClass__value)

foo = MyIntegerSubclass('5')
assert foo.get_value() == 5

# 확장 후
class MyBaseClass:
    def __init__(self, value):
        self.__value = value

    def get_value(self):
        return self.__value

class MyStringClass(MyBaseClass):
    def get_value(self):
        return str(super().get_value())  # 변경됨

class MyIntegerSubclass(MyStringClass):
    def get_value(self):
        return int(self._MyStringClass__value)  # 변경되지 않음

foo = MyIntegerSubclass(5)

foo.get_value() # 변경되지 않은 부분 때문에 에러남
```

비공개 애트리뷰트를 사용할지 진지하게 고민해야 하는 유일한 경우는 하위 클래스의 필드와 이름이 충돌할 수 있는 경우뿐이다. 자식 클래스가 실수로 부모 클래스가 이미 정의한 애트리뷰트를 정의하면 충돌이 생길 수 있다. 

```python
class ApiClass:
    def __init__(self):
        self._value = 5

    def get(self):
        return self._value

class Child(ApiClass):
    def __init__(self):
        super().__init__()
        self._value = 'hello'  # 충돌

a = Child()
print(f'{a.get()} 와 {a._value} 는 달라야 합니다.')

>>>
hello 와 hello 는 달라야 합니다.
```

이런 문제가 발생할 위험성을 줄이려면, 부모 클래스 쪽에서 자식 클래스의 애트리뷰트 이름이 자신의 애트리뷰트 이름과 겹치는 일을 방지하기 위해 비공개 애트리뷰트를 사용할 수 있다.

```python
class ApiClass:
    def __init__(self):
        self.__value = 5    # 밑줄 2개!

    def get(self):
        return self.__value # 밑줄 2개!

class Child(ApiClass):
    def __init__(self):
        super().__init__()
        self._value = 'hello' # OK!

a = Child()
print(f'{a.get()} 와 {a._value} 는 다릅니다.')

>>>
5 와 hello 는 다릅니다.
```

**기억해야 할 내용**
- 파이썬 컴파일러는 비공개 애트리뷰트를 자식 클래스나 클래스 외부에서 사용하지 못하도록 엄격히 금지하지 않는다.
- 여러분의 내부 API에 있는 클래스의 하위 클래스를 정의하는 사람들이 여러분이 제공하는 클래스의 애트리뷰트를 사용하지 못하도록 막기보다는 애트리뷰트를 사용해 더 많은 일을 할 수 있게 허용하라.
- 비공개 애트리뷰트로 (외부나 하위 클래스의) 접근을 막으려고 시도하기보다는 보호된 필드를 사용하면서 문서에 적절한 가이드를 남겨라.
- 여러분이 코드 작성을 제어할 수 없는 하위 클래스에서 이름 충돌이 일어나는 경우를 막고 싶을 때만 비공개 애트리뷰트를 사용할 것을 권한다.