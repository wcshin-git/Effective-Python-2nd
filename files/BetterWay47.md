## Better way 47. 지연 계산 애트리뷰트가 필요하다면 \_\_getattr\_\_, \_\_getattribute\_\_, \_\_setattr\_\_을 사용하라

어떤 클래스 안에 **\_\_getattr\_\_** 메서드 정의가 있으면, 이 객체의 인스턴스 딕셔너리에서 찾을 수 없는 애트리뷰트에 접근할 때마다 __getattr__이 호출된다. 

```python
class LazyRecord:
    def __init__(self):
        self.exists = 5

    def __getattr__(self, name):
        value = f'{name}의 값'
        setattr(self, name, value)
        return value

data = LazyRecord()
print('이전:', data.__dict__)
print('foo:', data.foo)
print('이후:', data.__dict__)

>>>
이전: {'exists': 5}
foo: foo의 값
이후: {'exists': 5, 'foo': 'foo의 값'}
```

**\_\_getattribute\_\_** 라는 특별 메서드는 객체의 애트리뷰트에 접근할 때마다 호출된다. 

```python
class ValidatingRecord:
    def __init__(self):
        self.exists = 5
    def __getattribute__(self, name):
        print(f'* 호출: __getattr__({name!r})')
        try:
            value = super().__getattribute__(name)
            print(f'* {name!r} 찾음, {value!r} 반환')
            return value
        except AttributeError:
            value = f'{name}을 위한 값'
            print(f'* {name!r}를 {value!r}로 설정')
            setattr(self, name, value)
            return value

data = ValidatingRecord()
print('exists: ', data.exists)
print('첫 번째 foo: ', data.foo)
print('두 번째 foo: ', data.foo)

>>>
* 호출: __getattr__('exists')
* 'exists' 찾음, 5 반환
exists:  5
* 호출: __getattr__('foo')
* 'foo'를 'foo을 위한 값'로 설정
첫 번째 foo:  foo을 위한 값
* 호출: __getattr__('foo')
* 'foo' 찾음, 'foo을 위한 값' 반환
두 번째 foo:  foo을 위한 값
```

참고로 hasattr(data, 'foo')처럼 hasattr 내장 함수를 써도 __getattr__이나 __getattribute__가 호출됨.

**\_\_setattr\_\_** 은 인스턴스의 애트리뷰트에 (직접 대입하든 setattr 내장 함수를 통해서든) 대입이 이뤄질 때마다 항상 호출된다.

```python
class SavingRecord:
    def __setattr__(self, name, value):
        # 데이터를 데이터베이스 레코드에 저장한다
        super().__setattr__(name, value)

class LoggingSavingRecord(SavingRecord): # 질문: SavingRecord를 굳이 상속 받아야 하나? 상속 받지 않고 해도 잘 됨
    def __setattr__(self, name, value):
        print(f'* 호출: __setattr__({name!r}, {value!r})')
        super().__setattr__(name, value)

data = LoggingSavingRecord()
print('이전: ', data.__dict__)
data.foo = 5
print('이후: ', data.__dict__)
data.foo = 7
print('최후:', data.__dict__)
```

__getattribute__와 __setattr__의 문제점은 여러분이 원하든 원하지 않든 어떤 객체의 모든 애트리뷰트에 접근할 때마다 함수가 호출된다는 것이다. 만약 어떤 객체와 관련된 딕셔너리에 키가 있을 때만 이 객체의 애트리뷰트에 접근하고 싶다면? 

```python
# 잘못된 코드. 무한루프에 빠짐.
class BrokenDictionaryRecord:
    def __init__(self, data):
        self._data = data
    def __getattribute__(self, name):
        print(f'* 호출: __getattribute__({name!r})')
        return self._data[name]

data = Brokedata = BrokenDictionaryRecord({'foo': 3})

data.foo
```

__getattribute__가 self._data에 접근해서 __getattribute__가 다시 호출되기 때문.

해결방법은 super().__getattribute__를 호출해 인스턴스 애트리뷰트  딕셔너리에서 값을 가져오는 것이다. 이렇게 하면 재귀를 피할 수 있다.

```python
class DictionaryRecord:
    def __init__(self, data):
        self._data = data

    def __getattribute__(self, name):
        print(f'* 호출: __getattribute__({name!r})')
        data_dict = super().__getattribute__('_data')
        return data_dict[name]

data = DictionaryRecord({'foo': 3})
print('foo: ', data.foo)

>>>
* 호출: __getattribute__('foo')
foo:  3
```

**기억해야 할 내용**
- __getattr__과 __setattr__을 사용해 객체의 애트리뷰트를 지연해 가져오거나 저장할 수 있다.
- __getattr__은 애트리뷰트가 존재하지 않을 때마 호출되지만, __getattribute__는 애트리뷰트를 읽을 때마다 항상 호출된다는 점을 이해하라.
- __getattribute__와 __setattr__에서 무한 재귀를 피하려면 super()에 있는(즉, object 클래스에 있는) 메서드를 사용해 인스턴스 애트리뷰트에 접근하라.