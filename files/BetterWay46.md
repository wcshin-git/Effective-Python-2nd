## Better way 46. 재사용 가능한 @property 메서드를 만들려면 디스크립터를 사용하라

@property의 가장 큰 문제점은 재사용성이다. @property가 데코레이션하는 메서드를 같은 클래스에 속하는 여러 애트리뷰트로 사용할 수는 없다. 그리고 서로 무관한 클래스 사이에서 @property 데코레이터를 적용한 메서드를 재사용할 수도 없다.

```python
class Exam:
    def __init__(self):
        self._writing_grade = 0
        self._math_grade = 0

    @staticmethod     # 정적 메서드는 인스턴스 속성, 인스턴스 메서드가 필요 없을 때 사용
    def _check_grade(value):
        if not (0 <= value <= 100):
            raise ValueError(
                '점수는 0과 100 사이입니다')

    @property
    def writing_grade(self):
        return self._writing_grade

    @writing_grade.setter
    def writing_grade(self, value):
        self._check_grade(value)
        self._writing_grade = value

    @property
    def math_grade(self):
        return self._math_grade

    @math_grade.setter
    def math_grade(self, value):
        self._check_grade(value)
        self._math_grade = value
```

이런 경우 파이썬에서 적용할 수 있는 더 나은 방법은 디스크립터를 사용하는 것이다. 디스크립터 프로토콜은 파이썬 언어에서 애트리뷰트 접근을 해석하는 방법을 접근한다. 디스크립터 클래스는 \_\_get\_\_과 \_\_set\_\_ 메서드를 제공하고, 이 두 메서드를 사용하면 별다른 준비 코드 없이도 원하는 점수 검증 동작을 재사용할 수 있다.

```python
# 버그 수정 전
class Grade:
    def __init__(self):
        self._value = 0
    def __get__(self, instance, instance_type):
        return self._value
    def __set__(self, instance, value):
        if not (0 <= value <= 100):
            raise ValueError(
                '점수는 0과 100 사이입니다')
        self._value = value

class Exam:
    # 클래스 애트리뷰트
    math_grade = Grade()
    writing_grade = Grade()
    science_grade = Grade()

first_exam = Exam()
first_exam.writing_grade = 82
first_exam.science_grade = 99
print('쓰기', first_exam.writing_grade)
print('과학', first_exam.science_grade)

second_exam = Exam()
second_exam.writing_grade = 75
print(f'두 번째 쓰기 점수 {second_exam.writing_grade}; 맞음')
print(f'첫 번째 쓰기 점수 {first_exam.writing_grade}; 틀림, '
      f'82점이어야 함')

>>>
쓰기 82
과학 99
두 번째 쓰기 점수 75; 맞음
첫 번째 쓰기 점수 75; 틀림, 82점이어야 함
```

Exam 인스턴에 있는 이런 디스크립터 애트리뷰트에 대한 접근을 파이썬이 어떻게 처리하는지 이해하는 것이 중요하다. 

```python
exam = Exam()
exam.writing_grade = 40
은 다음과 같이 해석된다.

Exam.__dict__['writing_grade'].__set__(exam, 40)

exam.writing_grade
는 다음과 같이 해석된다.

Exam.__dict__['writing_grade'].__get__(exam, Exam)
```

(이런 동작을 이끌어내는 것은 object의 \_\_getattribute\_\_ 메서드다)

다만 위 구현의 문제는 writing_grade 클래스 애트리뷰트로 한 **Grade 인스턴스를 모든 Exam 인스턴스가 공유한다는 점**이다. (호출될 때마다 인스턴스를 독립적으로 만들지 않음) 이를 해결하려면 Grade 클래스가 각각의 유일한 Exam 인스턴스에 대해 따로 값을 추적하게 해야 한다. 

```python
# 버그 수정 후
class Grade:
    def __init__(self):
        self._values = {}

    def __get__(self, instance, instance_type):
        if instance is None:   # 질문: 이 if문이 왜 있어야하지? 관용적으로 다 쓰던데... 인스턴스가 아니라 클래스에서 디스크립터를 호출하는 경우에는 instance가 None이다. 이 경우를 대비.
            return self
        return self._values.get(instance, 0)

    def __set__(self, instance, value):
        if not (0 <= value <= 100):
            raise ValueError(
                '점수는 0과 100 사이입니다')
        self._values[instance] = value

class Exam:
    # 클래스 애트리뷰트    # 질문: 이 애트리뷰트들을 __init__ 안에서 정의해 인스턴스 애트리뷰트로 만들면 안 되나?
    math_grade = Grade()
    writing_grade = Grade()
    science_grade = Grade()

first_exam = Exam()
first_exam.writing_grade = 82
first_exam.science_grade = 99
print('쓰기', first_exam.writing_grade)
print('과학', first_exam.science_grade)

second_exam = Exam()
second_exam.writing_grade = 75
print(f'두번째 쓰기 점수 {second_exam.writing_grade} 맞음')
print(f'첫번째 쓰기 점수 {first_exam.writing_grade} 맞음')

>>>
쓰기 82
과학 99
두번째 쓰기 점수 75 맞음
첫번째 쓰기 점수 82 맞음

## 참고: 클래스 애트리뷰트를 인스턴스 애트리뷰트로 넣으면 Grade()의 인스턴스를 공유하는 문제 해결 가능...
class Exam:
    # 인스턴스 애트리뷰트
    def __init__(self):
        math_grade = Grade()
        writing_grade = Grade()
        science_grade = Grade()

first_exam = Exam()
first_exam.writing_grade = 82
first_exam.science_grade = 99
print('쓰기', first_exam.writing_grade)
print('과학', first_exam.science_grade)

second_exam = Exam()
second_exam.writing_grade = 75
print(f'두 번째 쓰기 점수 {second_exam.writing_grade}; 맞음')
print(f'첫 번째 쓰기 점수 {first_exam.writing_grade}; 맞음, '
      f'82점이어야 함')

>>>
쓰기 82
과학 99
두 번째 쓰기 점수 75; 맞음
첫 번째 쓰기 점수 82; 맞음, 82점이어야 함
```

이 구현에도 문제가 하나 있는데, 바로 메모리를 누수(leak)시킨다는 점이다. _value 딕셔너리는 프로그램이 실행되는 동안 \_\_set\_\_ 호출에 전달된 모든 Exam 인스턴스에 대한 참조를 저장하고 있다. 이로 인해 Exam 인스턴스에 대한 참조 카운터가 절대로 0이 될 수 없고, 따라서 쓰레기 수집기(garbage collector)가 인스턴스 메모리를 결코 재활용하지 못한다.

이 문제를 해결하기 위해 파이썬 weakref 내장 모듈을 사용할 수 있다. 이 모듈은 WeakKeyDictionary라는 특별한 클래스를 제공하며, _value에 사용한 단순한 딕셔너리 대신 이 클래스를 쓸 수 있다. WeakKeyDictionary의 독특한 부분은 딕셔너리에 객체를 저장할 때 일반적인 강한 참조(strong reference) 대신에 **약한 참조**(weak reference)를 사용한다는 점이다. 파이썬 쓰레기 수집기는 약한 참조로만 참조되는 객체가 사용 중인 메모리를 언제든지 재활용할 수 있다. 따라서 WeakKeyDictionary를 사용해 _value에 저장된 Exam 인스턴스가 더 이상 쓰이지 않는다면(해당 객체를 가리키는 모든 강한 참조가 사라졌다면), 쓰레기 수집기가 해당 메모리를 재활용할 수 있으므로 더 이상 메모리 누수가 없다.

```python
# 메모리 누수 개선 후
from weakref import WeakKeyDictionary
class Grade:
    def __init__(self):
        self._values = WeakKeyDictionary()    # 바뀐 부분

    def __get__(self, instance, instance_type):
        if instance is None:
            return self
        return self._values.get(instance, 0)

    def __set__(self, instance, value):
        if not (0 <= value <= 100):
            raise ValueError(
                '점수는 0과 100 사이입니다')
        self._values[instance] = value
```

**기억해야 할 내용**
- @property 메서드의 동작과 검증 기능을 재사용하고 싶으면 디스크립터 클래스를 만들라.
- 디스크립터 클래스를 만들 때는 메모리 누수를 방지하기 위해 WeakKeyDictionary를 사용하라.
- __getattribute__가 디스크립터 프로토콜을 사용해 애트리뷰트 값을 읽거나 설정하는 방식을 정확히 이해하라.