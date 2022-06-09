## Better way 41. 기능을 합성할 때는 믹스인 클래스를 사용하라

파이썬은 다중 상속을 처리할 수 있게 지원하는 객체지향 언어이지만 다중 상속은 피하는 편이 좋다.

다중 상속이 제공하는 편의와 캡슐화가 필요하지만 다중 상속으로 인해 발생할 수 있는 골치 아픈 경우는 피하고 싶다면, 믹스인(mix-in)을 사용할지 고려해보라. 믹스인은 자식 클래스가 사용할 메서드 몇 개만 정의하는 클래스다. 믹스인 클래스에는 자체 애트리뷰트 정의가 없으므로 믹스인 클래스의 \_\_init\_\_ 메서드를 호출할 필요도 없다.

```python
class ToDictMixin:   # 참고로 class ToDictMixin(): 해도 똑같음.
    def to_dict(self):
        return self._traverse_dict(self.__dict__)

    def _traverse_dict(self, instance_dict):
        output = {}
        for key, value in instance_dict.items():
            output[key] = self._traverse(key, value)
        return output

    def _traverse(self, key, value):
        if isinstance(value, ToDictMixin):
            return value.to_dict()
        elif isinstance(value, dict):
            return self._traverse_dict(value)
        elif isinstance(value, list):
            return [self._traverse(key, i) for i in value]
        elif hasattr(value, '__dict__'):
            return self._traverse_dict(value.__dict__)
        else:
            return value

class BinaryTree(ToDictMixin):
    def __init__(self, value, left=None, right=None):
        self.value = value
        self.left = left
        self.right = right

tree = BinaryTree(10,
                  left=BinaryTree(7, right=BinaryTree(9)),
                  right=BinaryTree(13, left=BinaryTree(11)))
print(tree.to_dict())

>>>
{'value': 10, 'left': {'value': 7, 'left': None, 'right': {'value': 9, 'left': None, 'right': None}}, 'right': {'value': 13, 'left': {'value': 11, 'left': None, 'right': None}, 'right': None}}
```

믹스인을 서로 합성할 수도 있다.

```python
import json

class JsonMixin:
    @classmethod
    def from_json(cls, data):
        kwargs = json.loads(data)
        return cls(**kwargs)

    def to_json(self):
        return json.dumps(self.to_dict())  # to_dict()는 ToDictMixin에 정의되어 있음

class DatacenterRack(ToDictMixin, JsonMixin):
    def __init__(self, switch=None, machines=None):
        self.switch = Switch(**switch)
        self.machines = [
            Machine(**kwargs) for kwargs in machines]

class Switch(ToDictMixin, JsonMixin):
    def __init__(self, ports=None, speed=None):
        self.ports = ports
        self.speed = speed

class Machine(ToDictMixin, JsonMixin):
    def __init__(self, cores=None, ram=None, disk=None):
        self.cores = cores
        self.ram = ram
        self.disk = disk

serialized = """{
    "switch": {"ports": 5, "speed": 1e9},
    "machines": [
        {"cores": 8, "ram": 32e9, "disk": 5e12},
        {"cores": 4, "ram": 16e9, "disk": 1e12},
        {"cores": 2, "ram": 4e9, "disk": 500e9}
    ]
}"""

deserialized = DatacenterRack.from_json(serialized)
roundtrip = deserialized.to_json()
assert json.loads(serialized) == json.loads(roundtrip)
```

**기억해야 할 내용**
- 믹스인을 사용해 구현할 수 있는 기능을 인스턴스 애트리뷰트와 __init__을 사용하는 다중 상속을 통해 구현하지 말라.
- 믹스인 클래스가 클래스별로 특화된 기능을 필요로 한다면 인스턴스 수준에서 끼워 넣을 수 있는 기능(정해진 메서드를 통해 해당 기능을 인스턴스가 제공하게 만듦)을 활용하라.
- 믹스인에는 필요에 따라 인스턴스 메스드는 물론 클래스 메서드도 포함될 수 있다.
- 믹스인을 합성하면 단순한 동작으로부터 더 복잡한 기능을 만들어낼 수 있다.