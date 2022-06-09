## Better way 35. 제너레이터 안에서 throw로 상태를 변화시키지 말라

제너레이터 안에서 Exception을 다시 던질 수 있는 throw 메서드가 있다. 어떤 제너레이터에 대해 throw가 호출되면 이 제너레이터는 값을 내놓은 yield로부터 평소처럼 제너레이터 실행을 계속하는 대신, throw가 제공한 Exception을 다시 던진다.

```python
class MyError(Exception):
    pass

def my_generator():
    yield 1
    yield 2
    yield 3

it = my_generator()
print(next(it))  # 1을 내놓음
print(next(it))  # 2를 내놓음
print(it.throw(MyError('test error'))) # MyError: test error
```

**기억해야 할 내용**
- throw 메서드를 사용하면 제너레이터가 마지막으로 실행한 yield 식의 위치에서 예외를 다시 발생시킬 수 있다.
- throw를 사용하면 가독성이 나빠진다. 예외를 잡아내고 다시 발생시키는 데 준비 코드가 필요하며 내포 단계가 깊어지기 때문이다.
- 제너레이터에서 예외적인 동작을 제공하는 더 나은 방법은 __iter__메서드를 구현하는 클래스를 사용하면서 예외적인 경우에 상태를 전이시키는 것이다.