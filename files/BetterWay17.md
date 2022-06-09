## Better way 17. 내부 상태에서 원소가 없는 경우를 처리할 때는 setdefault보다 defaultdict를 사용하라.

setdefault라는 이름은 여전히 헷갈림. defaultdict을 사용하자.

```python
from collections import defaultdict
class Visits:
    def __init__(self):
        self.data = defaultdict(set)

    def add(self, country, city):
        self.data[country].add(city)

visits = Visits()
visits.add('영국', '바스')
visits.add('영국', '런던')
print(visits.data)

>>>
defaultdict(<class 'set'>, {'영국': {'바스', '런던'}})
```