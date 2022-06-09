## Better way 37. 내장 타입을 여러 단계로 내포시키기보다는 클래스를 합성하라

내포 단계가 두 단계 이상이 되면 더 이상 딕셔너리, 리스트, 튜플 계층을 추가하지 말아야 한다. 딕셔너리 안에 딕셔너리를 포함시키면 다른 프로그래머들이 코드를 읽기 어려워지고, 여러분 스스로도 유지 보수의 '악몽' 속으로 들어가는 셈이다. 코드에서 값을 관리하는 부분이 점점 복잡해지고 있음을 깨달은 즉시 해당 기능을 클래스로 분리해야 한다. 

완전한 클래스가 제공하는 유연성이 필요하지 않고 가변운 불변 데이터 컨테이너가 필요하다면 namedtuple을 사용하라. 

```python
# 튜플은 튜플에 있는 요소를 접근할 때 순서를 기억해서 순서로 접근해야 하는 단점이 있음

from collections import namedtuple
Grade = namedtuple('Grade', ('score', 'weight')) # namedtuple(타입명, (이름1, 이름2))
# namedtuple이 반환하는 값은 클래스이다. 
```

또한, 내부 상태를 표현하는 딕셔너리가 복잡해지면 이 데이터를 관리하는 코드를 여러 클래스로 나눠서 재작성하라.

```python
from collections import defaultdict

class Subject:
    def __init__(self):
        self._grades = []

    def report_grade(self, score, weight):
        self._grades.append(Grade(score, weight))

    def average_grade(self):
        total, total_weight = 0, 0
        for grade in self._grades:
            total += grade.score * grade.weight
            total_weight += grade.weight
        return total / total_weight

class Student:
    def __init__(self):
        self._subjects = defaultdict(Subject)

    def get_subject(self, name):
        return self._subjects[name]

    def average_grade(self):
        total, count = 0, 0
        for subject in self._subjects.values():
            total += subject.average_grade()
            count += 1
        return total / count

class Gradebook:
    def __init__(self):
        self._students = defaultdict(Student)

    def get_student(self, name):
        return self._students[name] # self._students.keys()에 없는 name이 들어오면(eg. '알버트 아인슈타인') 에러를 발생시키는 게 아니라 다음과 같이 pair를 default로 만듦. {'알버트 아인슈타인': <__main__.Student at 0x7f9a5e831350>}

book = Gradebook()
albert = book.get_student('알버트 아인슈타인')
math = albert.get_subject('수학')
math.report_grade(75, 0.05)
math.report_grade(65, 0.15)
math.report_grade(70, 0.80)
gym = albert.get_subject('체육')
gym.report_grade(100, 0.40)
gym.report_grade(85, 0.60)
print(albert.average_grade())

>>>
80.25
```

**기억해야 할 내용**
- 딕셔너리, 긴 튜플, 다른 내장 타입이 복잡하게 내포된 데이터를 값으로 사용하는 딕셔너리를 만들지 말라.
- 완전한 클래스가 제공하는 유연성이 필요하지 않고 가벼운 불변 데이터 컨테이너가 필요하다면 namedtuple을 사용하라.
- 내부 상태를 표현하는 딕셔너리가 복잡해지면 이 데이터를 관리하는 코드를 여러 클래스로 나눠서 재작성하라.