## Better way 48. __init_subclass__를 사용해 하위 클래스를 검증하라

메타클래스는 type을 상속해 정의된다. 기본적인 경우 메타클래스는 \_\_new\_\_ 메서드를 통해 자신과 연관된 클래스의 내용을 받는다. 

```python
class Meta(type):
    def __new__(meta, name, bases, class_dict):
        print(f'* 실행: {name}의 메타 {meta}.__new__')
        print('기반클래스들:', bases)
        print(class_dict)
        return type.__new__(meta, name, bases, class_dict)

class MyClass(metaclass=Meta):
    stuff = 123

    def foo(self):
        pass

class MySubclass(MyClass):
    other = 567

    def bar(self):
        pass

>>>
* 실행: MyClass의 메타 <class '__main__.Meta'>.__new__
기반클래스들: ()
{'__module__': '__main__', '__qualname__': 'MyClass', 'stuff': 123, 'foo': <function MyClass.foo at 0x7fcb8c9919e0>}
* 실행: MySubclass의 메타 <class '__main__.Meta'>.__new__
기반클래스들: (<class '__main__.MyClass'>,)
{'__module__': '__main__', '__qualname__': 'MySubclass', 'other': 567, 'bar': <function MySubclass.bar at 0x7fcb8c9913b0>}
```

파이썬 3.6부터는 메타클래스를 정의하지 않고 같은 동작을 구현할 수 있는 더 단순한 구문(\_\_init_subclass\_\_ 메서드를 정의하는 방식)이 추가됐다. 

```python
# 파이썬 3.6 이전
class ValidatePolygon(type):
    def __new__(meta, name, bases, class_dict):
        # Polygon 클래스의 하위 클래스만 검증한다
        if bases:
            if class_dict['sides'] < 3:
                raise ValueError('다각형 변은 3개 이상이어야 함')
        return type.__new__(meta, name, bases, class_dict)

class Polygon(metaclass=ValidatePolygon):
    sides = None # 하위 클래스는 이 애트리뷰트에 값을 지정해야 한다
    @classmethod
    def interior_angles(cls):
        return (cls.sides - 2) * 180

class Triangle(Polygon):
    sides = 3

assert Triangle.interior_angles() == 180

# 파이썬 3.6 이후
class BetterPolygon:
    sides = None  # 하위클래스에서 이 애트리뷰트의 값을 지정해야 함

    def __init_subclass__(cls):
        super().__init_subclass__()
        if cls.sides < 3:
            raise ValueError('다각형 변은 3개 이상이어야 함')

    @classmethod
    def interior_angles(cls):
        return (cls.sides - 2) * 180

class Hexagon(BetterPolygon):
    sides = 6

assert Hexagon.interior_angles() == 720
```

기존 표준 파이썬 메타클래스 방식의 또 다른 문제점은 클래스 정의마다 메타클래스를 단 하나만 지정할 수 있다는 것이다. \_\_init_subclass\_\_ 메서드를 사용하면 이 문제도 해결할 수 있다. 

```python
class Filled:
    color = None  # 하위 클래스에서 이 애트리뷰트 값을 지정해야 한다

    def __init_subclass__(cls):
        super().__init_subclass__()
        if cls.color not in ('red', 'green', 'blue'):
            raise ValueError('지원하지 않는 color 값')

class BetterPolygon:
    sides = None  # 하위클래스에서 이 애트리뷰트의 값을 지정해야 함

    def __init_subclass__(cls):
        super().__init_subclass__()
        if cls.sides < 3:
            raise ValueError('다각형 변은 3개 이상이어야 함')

    @classmethod
    def interior_angles(cls):
        return (cls.sides - 2) * 180

class RedTriangle(Filled, BetterPolygon):
    color = 'red'
    sides = 3

ruddy = RedTriangle()
assert isinstance(ruddy, Filled)
assert isinstance(ruddy, BetterPolygon)

print('class 이전')

# 오류가 나는 부분. 오류를 보고 싶으면 커멘트를 해제할것
class BlueLine(Filled, Polygon): 
    color = 'blue'
    sides = 2

print('class 이후')

>>>
class 이전
ValueError: 다각형 변은 3개 이상이어야 함
```

정리: \_\_init_subclass\_\_ 메서드가 정의된 클래스를 상속 받아 특정 클래스 정의하게 되면, **정의할 때** \_\_init_subclass\_\_ 메서드가 작동한다.

심지어 __init_subclass__를 다이아몬드 상속 같은 복잡한 경우에도 사용할 수 있다.

```python
class Top:
    def __init_subclass__(cls):
        super().__init_subclass__()
        print(f'{cls}의 Top')

class Left(Top):
    def __init_subclass__(cls):
        super().__init_subclass__()
        print(f'{cls}의 Left')

class Right(Top):
    def __init_subclass__(cls):
        super().__init_subclass__()
        print(f'{cls}의 Right')

class Bottom(Left, Right):
    def __init_subclass__(cls):
        super().__init_subclass__()
        print(f'{cls}의 Bottom')

>>>
<class '__main__.Left'>의 Top
<class '__main__.Right'>의 Top
<class '__main__.Bottom'>의 Top
<class '__main__.Bottom'>의 Right
<class '__main__.Bottom'>의 Left
```

Bottom 클래스에서 Top 클래스에 이르는 상속 경로가 두 가지(Left를 통하는 경로와 Right를 통하는 경로)지만, 각 클래스마다 Top.__init_subclass__는 단 한 번만 호출된다.

**기억해야 할 내용**
- 메타클래스의 \_\_new\_\_ 메서드는 class 문의 모든 본문이 처리된 직후에 호출된다.
- 메타클래스를 사용해 클래스가 정의된 직후이면서 클래스가 생성되기 직전인 시점에 클래스 정의를 변경할 수 있다. 하지만 메타클래스는 원하는 목적을 달성하기에 너무 복잡해지는 경우가 많다.
- __init_subclass__를 사용해 하위 클래스가 정의된 직후, 하위 클래스 타입이 만들어지기 직전에 해당 클래스가 원하는 요건을 잘 갖췄는지 확인하라.
- \_\_init_subclass\_\_ 정의 안에서 super().__init_subclass__를 호출해 여러 계층에 걸쳐 클래스를 검증하고 다중 상속을 제대로 처리하도록 하라.