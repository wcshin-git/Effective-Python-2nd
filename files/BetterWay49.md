## Better way 49. __init_subclass__를 사용해 클래스 확장을 등록하라

```python
# 개선 전
import json

class BetterSerializable:
    def __init__(self, *args):
        self.args = args

    def serialize(self):
        return json.dumps({
            'class': self.__class__.__name__,
            'args': self.args,
        })

    def __repr__(self):
        name = self.__class__.__name__
        args_str = ', '.join(str(x) for x in self.args)
        return f'{name}({args_str})'

registry = {}

def register_class(target_class):
    registry[target_class.__name__] = target_class  # eg. registry = {'EvenBetterPoint2D': __main__.EvenBetterPoint2D}

def deserialize(data):
    params = json.loads(data)
    name = params['class']
    target_class = registry[name]
    return target_class(*params['args'])

class EvenBetterPoint2D(BetterSerializable):
    def __init__(self, x, y):
        super().__init__(x, y)
        self.x = x
        self.y = y

register_class(EvenBetterPoint2D)

before = EvenBetterPoint2D(5, 3)
print('이전: ', before)
data = before.serialize()
print('직렬화한 값:', data)
after = deserialize(data)
print('이후: ', after)

>>>
이전:  EvenBetterPoint2D(5, 3)
직렬화한 값: {"class": "EvenBetterPoint2D", "args": [5, 3]}
이후:  EvenBetterPoint2D(5, 3)
```

이 방식의 문제점은 register_class 호출을 잊어버릴 수 있다는 것이다. BetterSerializable의 하위 클래스를 정의했음에도 불구하고, 클래스 정의문 다음에 register_class를 호출하는 것을 잊어버리면 deserialize할 때 에러가 발생한다. 이를 개선할 수 없을까?

```python
# 개선 후
class BetterRegisteredSerializable(BetterSerializable):
    def __init_subclass__(cls):
        super().__init_subclass__()
        register_class(cls)

class Vector1D(BetterRegisteredSerializable): # __init_subclass__ 메서드가 정의된 클래스를 상속 받아 특정 클래스 정의하게 되면, **정의할 때** __init_subclass__ 메서드가 작동한다.
    def __init__(self, magnitude):
        super().__init__(magnitude)
        self.magnitude = magnitude

before = Vector1D(6)
print('이전: ', before)
data = before.serialize()
print('직렬화한 값:', data)
print('이후: ', deserialize(data))

>>>
이전:  Vector1D(6)
직렬화한 값: {"class": "Vector1D", "args": [6]}
이후:  Vector1D(6)

print(registry)
>>>
{'EvenBetterPoint2D': __main__.EvenBetterPoint2D,
 'Vector1D': __main__.Vector1D}
```

**기억해야 할 내용**
- 클래스 등록은 파이썬 프로그램을 모듈화할 때 유용한 패턴이다.
- 메타클래스를 사용하면, 프로그램 안에서 기반 클래스를 상속한 하위 클래스가 정의될 때마다 등록 코드를 자동으로 실행할 수 있다.
- 메타클래스를 클래스 등록에 사용하면 클래스 등록 함수를 호출하지 않아서 생기는 오류를 피할 수 있다.
- 표준적인 메타클래스 방식보다는 __init_subclass__가 더 낫다. \_\_init_subclass\_\_ 쪽이 더 깔끔하고 초보자가 이해하기에도 더 쉽다.