## Better way 50. __set_name__으로 클래스 애트리뷰트를 표시하라

디스크립터에 \_\_set_name\_\_ 특별 메서드(파이썬 3.6부터 도입됨)

클래스가 정의될 때마다 파이썬은 해당 클래스 안에 들어 있는 디스크립터 인스턴스의 __set_name__을 호출한다. __set_name__은 디스크립터 인스턴스를 소유 중인 클래스와 디스크립터 인스턴스가 대입될 애트리뷰트 이름을 인자로 받는다.

```python
class Field:
    def __init__(self):
        self.name = None
        self.internal_name = None

    def __set_name__(self, owner, name):
        # 클래스가 생성될 때 모든 디스크립터에 대해 이 메서드가 호출된다
        self.name = name
        self.internal_name = '_' + name

    def __get__(self, instance, instance_type):
        if instance is None:
            return self
        return getattr(instance, self.internal_name, '')

    def __set__(self, instance, value):
        setattr(instance, self.internal_name, value)

class FixedCustomer:
    first_name = Field()
    last_name = Field()
    prefix = Field()
    suffix = Field()

cust = FixedCustomer()
print(f'이전: {cust.first_name!r} {cust.__dict__}')
cust.first_name = '메르센'
print(f'이후: {cust.first_name!r} {cust.__dict__}')

>>>
이전: '' {}
이후: '메르센' {'_first_name': '메르센'}
```

**기억해야 할 내용**
- 메타클래스를 사용하면 어떤 클래스가 완전히 정의되기 전에 클래스의 애트리뷰트를 변경할 수 있다.
- 디스크립터와 메타클래스를 조합하면 강력한 실행 시점 코드 검사와 선언적인 동작을 만들 수 있다.
- \_\_set_name\_\_ 특별 메서드를 디스크립터 클래스에 정의하면 디스크립터가 포함된 클래스의 프로퍼티 이름을 처리할 수 있다.
- 디스크립터가 변경한 클래스의 인스턴스 딕셔너리에 데이터를 저장하게 만들면 메모리 누수를 피할 수 있고, weakref 내장 메서드를 사용하지 않아도 된다.