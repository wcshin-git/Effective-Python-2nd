## Better way 73. 우선순위 큐로 heapq를 사용하는 방법을 알아두라

때로는 프로그램에서 원소를 받은 순서가 아니라 원소 간의 상대적인 중요도에 따라 원소를 정렬해야 하는 경우가 있다. 이런 목적에는 우선순위 큐(priority queue)가 적합하다. 

예를 들어 도서관에서 대출한 책을 관리는 프로그램을 작성한다고 하자. 만기일(due_date)을 넘긴 경우에는 연체 사실을 통지하는 메시지를 보내는 시스템이 필요하다. 안타깝지만 책의 최대 대출 기간이 얼마나 최근에 발간된 책인지, 얼마나 유명한 책인지 등의 요소에 따라 달라지므로, 이 경우에는 FIFO 큐를 사용할 수 없다. 오늘 대출한 책이 내일 대출한 책보다 만기일이 더 늦을 수도 있다.

```python
class Book:
    def __init__(self, title, due_date):
        self.title = title
        self.due_date = due_date

def add_book(queue, book):
    queue.append(book)
    queue.sort(key=lambda x: x.due_date, reverse=True)

queue = []
add_book(queue, Book('돈키호테', '2020-06-07'))
add_book(queue, Book('프랑켄슈타인', '2020-06-05'))
add_book(queue, Book('레미제라블', '2020-06-08'))
add_book(queue, Book('전쟁과 평화', '2020-06-03'))

class NoOverdueBooks(Exception):
    pass

def next_overdue_book(queue, now):
    if queue:
        book = queue[-1]
        if book.due_date < now:
            queue.pop() # pop에 인자를 주지 않으면 제일 마지막 원소가 뽑힌다.
            return book

    raise NoOverdueBooks

now = '2020-06-10'
found = next_overdue_book(queue, now)
print(found.title)
found = next_overdue_book(queue, now)
print(found.title)

>>>
전쟁과 평화
프랑켄슈타인

def return_book(queue, book):
    queue.remove(book)

queue = []
book = Book('보물섬', '2020-06-04')

add_book(queue, book)
print('반납 전:', [x.title for x in queue])

return_book(queue, book)
print('반납 후:', [x.title for x in queue])

try:
    next_overdue_book(queue, now)
except NoOverdueBooks:
    pass         # 이 문장이 실행되리라 예상함
else:
    assert False # 이 문장은 결코 실행되지 않음

>>>
반납 전: ['보물섬']
반납 후: []
```

하지만 이 해결방법의 계산 복잡도는 이상적이지 않다. 연체된 책을 검사하거나 제거하는 비용은 상수이지만, 책을 추가할 때마다 전체 리스트를 다시 정렬해야 하는 추가 비용이 들어간다. 책이 반납되면, 이 책을 큐에서 선형 검색으로 찾은 후 제거해야 한다. 책을 제거하면 리스트에 있는 모든 원소를 하나씩 뒤로 옮겨야 하는데, 이 경우의 비용도 선형보다 더 커진다. 

이런 문제는 우선순위 큐를 구현함으로써 해결할 수 있다. 다행히 파이썬은 원소가 들어 있는 리스트에 대해 우선순위 큐를 효율적으로 구현하는 내장 heapq 모듈을 제공한다. heapq에서 힙(heap)은 여러 아이템을 유지하되 새로운 원소를 추가하거나 가장 작은 원소(만기가 가장 빠른 책)를 제거할 때 로그 복잡도가 드는 데이터 구조다.

다음 코드는 heapq 모듈을 사용해 add_book 함수를 다시 구현한 것이다. 큐는 여전히 일반 리스트다. heappush 함수는 앞에서 사용한 list.append 호출을 대신한다. 그리고 이 큐에 대해 더 이상 list.sort를 호출할 필요가 없다.

```python
rom heapq import heappush

def add_book(queue, book):
    heappush(queue, book)

queue = []
# 오류가 나는 부분. 오류를 보고 싶으면 커멘트를 해제할것
add_book(queue, Book('작은 아씨들', '2020-06-05'))
add_book(queue, Book('타임 머신', '2020-05-30'))

>>>
TypeError: '<' not supported between instances of 'Book' and 'Book'
```

에러가 나는 이유는, 우선순위 큐에 들어갈 원소들이 서로 비교 가능하고 원소 사이에 자연스러운 정렬 순서가 존재해야 heapq 모듈이 제대로 작동하기 때문이다. functools 내장 모듈이 제공하는 total_ordering 클래스 데코레이터를 사용하고 __it__ 특별 메서드를 구현하면, 빠르게 Book 클래스에 비교 기능과 자연스러운 정렬 순서를 제공할 수 있다. 

```python
import functools

@functools.total_ordering
class Book:
    def __init__(self, title, due_date):
        self.title = title
        self.due_date = due_date

    def __lt__(self, other):
        return self.due_date < other.due_date

# @total_ordering 으로 데코레이팅 된 클래스는 __lt__(), __gt__(), __le__(), __ge__() 중 하나만 정의하고, __eq__() 만 정의하면 나머지 비교 연산자(__ne__() 등)도 모두 사용 가능합니다.
```

이제는 heapq.heappush 함수를 사용해도 아무 문제없이 책을 우선순위 큐에 등록할 수 있다. 

```python
queue = []
add_book(queue, Book('오만과 편견', '2020-06-01'))
add_book(queue, Book('타임 머신', '2020-05-30'))
add_book(queue, Book('죄와 벌', '2020-06-06'))
add_book(queue, Book('폭풍의 언덕', '2020-06-12'))
```

대출 만기를 넘긴 책을 검사하려면 리스트의 마지막 원소가 아니라 척 번째 원소를 살펴본 다음, list.pop 함수 대신에 heapq.heappop함수를 사용한다. 

```python
from heapq import heappop

def next_overdue_book(queue, now):
    if queue:
        book = queue[0]  # 만기가 가장 이른 책이 맨 앞에 있다
        if book.due_date < now:
            heappop(queue)  # 연체된 책을 제거한다
            return book

    raise NoOverdueBooks

now = '2020-06-02'

book = next_overdue_book(queue, now)
print(book.title)

book = next_overdue_book(queue, now)
print(book.title)

>>>
타임 머신
오만과 편견
```

**기억해야 할 내용**
- 우선순위 큐를 사용하면 선입선출이 아니라 원소의 중요도에 따라 원소를 처리할 수 있다.
- 리스트 연산을 사용해 우선순위를 구현하면 크기가 커짐에 따라 프로그램의 선능이 선형보다 더 빠르게 나빠진다.
- heapq 내장 모듈은 효율적으로 규모 확장이 가능한 우선순위 큐를 구현하는 데 필요한 모든 기능을 제공한다.
- heapq를 사용하려면 우선순위를 부여하려는 원소들이 자연스러운 순서를 가져야 한다. 이는 원소를 표현하는 클래스에 대해 __it__와 같은 특별 메서드가 있어야 한다는 뜻이다.