## Better way 34. send로 제너레이터에 데이터를 주입하지 말라

```python
def my_generator():
    received = yield 1
    print(f'받은 값 = {received}')

it = iter(my_generator())
output = next(it)  # 첫 번째 제너레이터 출력을 얻는다
print(f'출력값 = {output}')

>>>
출력값 = 1
```

여기서 next(it)를 한 번 더 하면?

```python
next(it)

>>>
받은 값 = None
StopIteration error 발생
```

for 루프나 next 내장 함수로 제너레이터를 이터레이션하지 않고 send 메서드를 호출하면, 제너레이터가 재개(resume)될 때 yield가 send에 전달된 파라미터 값을 반환한다. 하지만 방금 시작한 제너레이터는 아직 yield식에 도달하지 못했기 때문에 최초로 send를 호출할 때 인자로 전달할 수 있는 유일한 값은 None뿐이다.(다른 값을 전달하려고 하면 에러남)

```python
it = iter(my_generator())
output = it.send(None)     # 첫 번째 제너레이터 출력을 얻는다
print(f'출력값 = {output}')

>>>
출력값 = 1
```

여기서 it.send('안녕!')를 한 번 더 하면?

```python
it.send('안녕!')

>>>
받은 값 = 안녕!
StopIteration error 발생
```

하지만 이 코드의 문제점은 코드를 처음 봤을 때 이해하기 어렵다는 것이다. 대입문의 오른쪽에 yield를 사용하는 것은 직관적이지 않다. 그리고 제너레이터의 고급 기능을 잘 모를 경우에는 send와 yield 사이의 연결을 알아보기 어렵다.

**기억해야 할 내용**
- send 메서드를 사용해 데이터를 제너레이터에 주입할 수 있다. 제너레이터는 send로 주입된 값을 yield식이 반환하는 값을 통해 받으며, 이 값을 변수에 저장해 활용할 수 있다.
- send와 yield from식을 함께 사용하면 제너레이터의 출력에 None이 불쑥불쑥 나타나는 의외의 결과를 얻을 수도 있다.
- 합성할 제너레이터들의 입력으로 이터레이터를 전달하는 방식이 send를 사용하는 방식보다 더 낫다. send는 가급적 사용하지 말라.