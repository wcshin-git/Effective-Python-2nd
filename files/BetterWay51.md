## Better way 51. 합성 가능한 클래스 확장이 필요하면 메타클래스보다는 클래스 데코레이터를 사용하라

클래스 데코레이터는 함수 데코레이터처럼 사용할 수 있다. 클래스 선언 앞에 @기호와 데코레이터 함수를 적으면 된다. 이때 데코레이터 함수는 인자로 받은 클래스를 적절히 변경해서 재생성해야 한다. 

```python
def my_class_decorator(klass):
    klass.extra_param = '안녕'
    return klass

@my_class_decorator
class MyClass:
    pass

print(MyClass)
print(MyClass.extra_param)

>>>
<class '__main__.MyClass'>
안녕
```

**기억해야 할 내용**
- 클래스 데코레이터는 class 인스턴스를 파라미터로 받아서 이 클래스르 변경한 클래스나 새로운 클래스를 반환해주는 간단한 함수다.
- 준비 코드를 최소화하면서 클래스 내부의 모든 메서드나 애트리뷰트를 변경하고 싶을 때 클래스 데코레이터가 유용하다.
- 메타클래스는 서로 쉽게 합성할 수 없지만, 여러 클래스 데코레이터를 충돌 없이 사용해 똑같은 클래스를 확장할 수 있다.