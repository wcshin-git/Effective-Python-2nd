## Better way 43. 커스텀 컨테이너 타입은 collections.abc를 상속하라

모든 파이썬 클래스는 함수와 애트리뷰트를 함께 캡슐화하는 일종의 컨테이너라고 할 수 있다. 

시퀀스처럼 사용법이 간단한 클래스를 정의할 때는 파이썬 내장 리스트 타입의 하위 클래스를 만들고 싶을 것이다. 예를 들어 멤버들의 빈도를 계산하는 메서드가 포함된 커스텀 리스트 타입이 필요하다고 가정하자.

FrequencyList는 리스트의 하위 클래스로 만듦으로써 리스트가 제공하는 모든 표준 함수를 FrequencyList에서도 사용할 수 있으며, 필요한 기능을 제공하는 메서드를 얼마든지 추가할 수 있다. 

```python
class FrequencyList(list):
    def __init__(self, members):
        super().__init__(members)

    def frequency(self):
        counts = {}
        for item in self:
            counts[item] = counts.get(item, 0) + 1
        return counts

foo = FrequencyList(['a', 'b', 'a', 'c', 'b', 'a', 'd'])
print('길이: ', len(foo))

foo.pop() # pop()은 리스트의 맨 마지막 요소를 돌려주고 그 요소는 삭제한다.
print('pop한 다음:', repr(foo))
print('빈도:', foo.frequency())

>>>
길이:  7
pop한 다음: ['a', 'b', 'a', 'c', 'b', 'a']
빈도: {'a': 3, 'b': 2, 'c': 1}
```

이제 리스트처럼 느껴지면서 인덱싱이 가능한 객체를 제공하고 싶은데, 리스트의 하위 클래스로 만들고 싶지는 않다고 가정해보자.

파이썬에서는 특별한 이름의 인스턴스 메서드를 사용해 컨테이너의 동작을 구현한다. 인덱스를 사용해 다음과 같이 시퀀스에 접근하는 코드는 

```python
bar = [1, 2, 3]
assert bar[0] == 1
```

다음 특별 메서드로 해석된다.

```python
assert bar.__getitem__(0) == 1
```

이제 이진 트리 클래스를 시퀀스(리스트나 튜플)의 의미구조를 사용해 다룰 수 있는 클래스를 만들어보자. 

```python
class BinaryNode:
    def __init__(self, value, left=None, right=None):
        self.value = value
        self.left = left
        self.right = right

class IndexableNode(BinaryNode):
    def _traverse(self):
        if self.left is not None:
            yield from self.left._traverse()
        yield self
        if self.right is not None:
            yield from self.right._traverse()

    def __getitem__(self, index):
        for i, item in enumerate(self._traverse()):
            if i == index:
                return item.value
        raise IndexError(f'인덱스 범위 초과: {index}')

tree = IndexableNode(
    10,
    left=IndexableNode(
        5,
        left=IndexableNode(2),
        right=IndexableNode(
            6,
            right=IndexableNode(7))),
    right=IndexableNode(
        15,
        left=IndexableNode(11)))

print('LRR:', tree.left.right.right.value)
print('인덱스 0:', tree[0])
print('인덱스 1:', tree[1])
print('11이 트리 안에 있나?', 11 in tree)
print('17이 트리 안에 있나?', 17 in tree)
print('트리:', list(tree))

>>>
LRR: 7
인덱스 0: 2
인덱스 1: 5
11이 트리 안에 있나? True
17이 트리 안에 있나? False
트리: [2, 5, 6, 7, 10, 11, 15]
```

BinaryNode 클래스가 시퀀스처럼 작동하게 하려면 트리 노드를 깊이 우선 순회(depth first traverse)하는 커스텀 \_\_getitem\_\_ 메서드 구현을 제공하면 된다.

문제는 __getitem__을 구현하는 것만으로는 리스트 인스턴스에서 기대할 수 있는 모든 시퀀스 의미 구조를 제공할 수 없다는 데 있다.

```python
len(tree)

>>>
TypeError: object of type 'IndexableNode' has no len()
```

len 내장 함수는 __len__ 이라는 이름의 특별 메서드를 구현해야 제대로 작동한다. 커스텀 시퀀스 타입은 이 메서드를 꼭 구현해야 한다.

```python
class SequenceNode(IndexableNode):
    def __len__(self):
        for count, _ in enumerate(self._traverse(), 1):
            pass
        return count
```

안타깝지만 어떤 클래스가 올바른 시퀀스가 되려면 두 메서드(\_\_getitem\_\_, \_\_len\_\_)를 구현하는 것만으로는 충분하지 않다. count나 index 메서드도 구현해야 한다. 자신만의 컨테이너 타입을 직접 정의하는 것은 생각보다 훨씬 어려운 일임을 알 수 있다.

파이썬을 사용할 때 흔히 발생하는 이런 어려움을 덜어주기 위해 내장 collections.abc 모듈 안에는 컨테이너 타입에 정의해야 하는 전형적인 메서드를 모두 제공하는 추상 기반 클래스 정의가 여러 가지 들어 있다. 이런 추상 기반 클래스의 하위 클래스를 만들고 필요한 메서드 구현을 잊어버리면, collections.abc 모듈이 실수한 부분을 알려준다. 

```python
from collections.abc import Sequence

class BadType(Sequence):
    pass

foo = BadType()

>>>
TypeError: Can't instantiate abstract class BadType with abstract methods __getitem__, __len__
```

collections.abc에서 가져온 추상 기반 클래스가 요구하는 모든 메서드를 구현하면 index나 count와 같은 추가 메서드 구현을 거저 얻을 수 있다.

```python
class BetterNode(SequenceNode, Sequence):
    pass

tree = BetterNode(
    10,
    left=BetterNode(
        5,
        left=BetterNode(2),
        right=BetterNode(
            6,
            right=BetterNode(7))),
    right=BetterNode(
        15,
        left=BetterNode(11))
)

print('7의 인덱스:', tree.index(7))
print('10의 개수:', tree.count(10))

>>>
7의 인덱스: 3
10의 개수: 1
```

**기억해야 할 내용**
- 간편하게 사용할 경우에는 파이썬 컨테이너 타입(리스트나 딕셔너리 등)을 직접 상속하라.
- 커스텀 컨테이너를 제대로 구현하려면 수많은 메서드를 구현해야 한다는 점에 주의하라.
- 커스텀 컨테이너 타입이 collections.abc에 정의된 인터페이스를 상속하면 커스텀 컨테이너 타입이 정상적으로 작동하기 위해 필요한 인터페이스와 기능을 제대로 구현하도록 보장할 수 있다.