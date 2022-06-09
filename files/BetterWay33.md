## Better way 33. yield from을 사용해 여러 제너레이터를 합성하라

yield from은 근본적으로 파이썬 인터프리터가 여러분 대신 for 루프를 내포시키고 yield 식을 처리하도록 만든다. 

```python
def move(period, speed):
    for _ in range(period):
        yield speed

def pause(delay):
    for _ in range(delay):
        yield 0

def animate():  # 너무 반복적.
    for delta in move(4, 5.0):
        yield delta
    for delta in pause(3):
        yield delta
    for delta in move(2, 3.0):
        yield delta

def render(delta):
    print(f'Delta: {delta:.1f}')
    # 화면에서 이미지를 이동시킨다

def run(func):
    for delta in func():
        render(delta)

run(animate)

>>>
Delta: 5.0
Delta: 5.0
Delta: 5.0
Delta: 5.0
Delta: 0.0
Delta: 0.0
Delta: 0.0
Delta: 3.0
Delta: 3.0
```

```python
def animate_composed():
    yield from move(4, 5.0)
    yield from pause(3)
    yield from move(2, 3.0)

run(animate_composed)
```

만약 제너레이터를 합성한다면 가급적 yield from을 사용하라.

**기억해야 할 내용**
- yield from식을 사용하면 여러 내장 제너레이터를 모아서 제너레이터 하나로 합성할 수 있다.
- 직접 내포된 제너레이터를 이터레이션하면서 각 제너레이터의 출력을 내보내는 것보다 yield from을 사용하는 것이 성능 면에서 더 좋다.