## Better way 68. copyreg를 사용해 pickle을 더 신뢰성 있게 만들라

```python
class GameState:
    def __init__(self):
        self.level = 0
        self.lives = 4

state = GameState()
state.level += 1     # 플레이어가 레벨을 깼다
state.lives -= 1     # 플레이어가 재시도해야 한다

print(state.__dict__)

import pickle

state_path = 'game_state.bin'
with open(state_path, 'wb') as f:
    pickle.dump(state, f)

with open(state_path, 'rb') as f:
    state_after = pickle.load(f)

print(state_after.__dict__)

>>>
{'level': 1, 'lives': 3}
{'level': 1, 'lives': 3}
```

이런 접근 방법을 사용하면 시간이 지나면서 게인 기능이 확장될 때 문제가 발생한다. GameState 클래스에 새로운 필드를 추가했다고 가정해보자.

```python
class GameState:
    def __init__(self):
        self.level = 0
        self.lives = 4
        self.points = 0 # 새로운 필드

state = GameState()
serialized = pickle.dumps(state) # dump에 s붙으면: the dumps() does not deal with writing the pickled object hierarchy into the disk file.
state_after = pickle.loads(serialized)
print(state_after.__dict__)

with open('game_state.bin', 'rb') as f:
    state_after = pickle.load(f)

print(state_after.__dict__)

assert isinstance(state_after, GameState)

>>>
{'level': 0, 'lives': 4, 'points': 0}
{'level': 1, 'lives': 3}
```

위에 GameState 정의에는 points라는 필드가 있는데 'game_state.bin'에는 points가 없음. 그럼에도 isinstance(state_after, GameState)가 True임. 여러분이 pickle을 사용하는 방식이 아주 간단한 수준을 벗어나는 순간, pickle 모듈의 동작은 예상할 수 없는 방식으로 망가지기 시작한다. 

copyreg 내장 모듈을 사용하면 이런 문제를 쉽게 해결할 수 있다. 파이썬 객체를 직렬화하고 역직렬화할 때 사용할 함수를 등록할 수 있으므로 pickle의 동작을 제어할 수 있고, 그에 따라 pickle 동작의 신뢰성을 높일 수 있다.

```python
class GameState:
    def __init__(self, level=0, lives=4, points=0):
        self.level = level
        self.lives = lives
        self.points = points

def pickle_game_state(game_state): # 이 함수가 반환하는 튜플 객체에는 언피클 시 사용할 함수와 언피클 시 이 함수에 전달해야 하는 파라미터 정보가 들어있어야 한다.
    kwargs = game_state.__dict__
    return unpickle_game_state, (kwargs,)

def unpickle_game_state(kwargs):
    return GameState(**kwargs)

import copyreg

copyreg.pickle(GameState, pickle_game_state)

state = GameState()
state.points += 1000
serialized = pickle.dumps(state)
state_after = pickle.loads(serialized)
print(state_after.__dict__)

class GameState:
    def __init__(self, level=0, lives=4, points=0, magic=5):
        self.level = level
        self.lives = lives
        self.points = points
        self.magic = magic   # 추가한 필드

print('이전:', state.__dict__)
state_after = pickle.loads(serialized)
print('이후:', state_after.__dict__)

>>>
{'level': 0, 'lives': 4, 'points': 1000}
이전: {'level': 0, 'lives': 4, 'points': 1000}
이후: {'level': 0, 'lives': 4, 'points': 1000, 'magic': 5}
```

**기억해야 할 내용**
- 신뢰할 수 있는 프로그램 사이에 객체를 직렬화하고 역직렬화할 때는 pickle 내장 모듈이 유용하다.
- 시간이 지남에 따라 클래스가 바뀔(애트리뷰트의 추가나 삭제) 수 있으므로 이전에 피클한 객체를 역직렬화하면 문제가 생길 수 있다.
- 직렬화한 객체의 하위 호환성을 보장하고자 copyreg 내장 모듈과 pickle을 함께 사용하라.