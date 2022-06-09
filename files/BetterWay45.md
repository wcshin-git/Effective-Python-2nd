## Better way 45. 애트리뷰트를 리팩터링하는 대신 @property를 사용하라

리키 버킷(leaky bucket) 알고리즘을 구현한다고 하자. 리키 버킷 알고리즘은 가용 용량을 수비할 때마다 시간을 검사해서 주기가 달라질 경우에는 이전 주기에 미사용한 가용 용량이 새로운 주기로 넘어오지 못하게 막는다.

 

```python
from datetime import datetime, timedelta

class Bucket:
    def __init__(self, period):
        self.period_delta = timedelta(seconds=period)
        self.reset_time = datetime.now()
        self.quota = 0

    def __repr__(self):
        return f'Bucket(quota={self.quota})'

def fill(bucket, amount):
    now = datetime.now()
    if (now - bucket.reset_time) > bucket.period_delta:
        bucket.quota = 0
        bucket.reset_time = now
    bucket.quota += amount

def deduct(bucket, amount):
    now = datetime.now()
    if (now - bucket.reset_time) > bucket.period_delta:
        return False # 새 주기가 시작됐는데 아직 버킷 할당량이 재설정되지 않았다
    if bucket.quota - amount < 0:
        return False # 버킷의 가용 용량이 충분하지 못하다
    else:
        bucket.quota -= amount
        return True  # 버킷의 가용 용량이 충분하므로 필요한 분량을 사용한다

bucket = Bucket(60)
fill(bucket, 100)
print(bucket)

if deduct(bucket, 99):
    print('99 용량 사용')
else:
    print('가용 용량이 작아서 99 용량을 처리할 수 없음')
print(bucket)

if deduct(bucket, 3):
    print('3 용량 사용')
else:
    print('가용 용량이 작아서 3 용량을 처리할 수 없음')
print(bucket)

>>>
Bucket(quota=100)
99 용량 사용
Bucket(quota=1)
가용 용량이 작아서 3 용량을 처리할 수 없음
Bucket(quota=1)
```

여기서, 이번 주기에 재설정된 가용 용량인 max_quota와 이번 주기에 버킷에서 소비한 용량의 합계인 quota_consumed를 추적하도록 클래스를 변경하고 싶다면?

```python
class NewBucket:
    def __init__(self, period):
        self.period_delta = timedelta(seconds=period)
        self.reset_time = datetime.now()
        self.max_quota = 0
        self.quota_consumed = 0

    def __repr__(self):
        return (f'NewBucket(max_quota={self.max_quota}, '
                f'quota_consumed={self.quota_consumed})')

    @property
    def quota(self):
        return self.max_quota - self.quota_consumed

    @quota.setter
    def quota(self, amount):
        delta = self.max_quota - amount
        if amount == 0:
            # 새로운 주기가 되고 가용 용량을 재설정하는 경우
            self.quota_consumed = 0
            self.max_quota = 0
        elif delta < 0:
            # 새로운 주기가 되고 가용 용량을 추가하는 경우
            assert self.quota_consumed == 0
            self.max_quota = amount
        else:
            # 어떤 주기 안에서 가용 용량을 소비하는 경우
            assert self.max_quota >= self.quota_consumed
            self.quota_consumed += delta

bucket = NewBucket(60)
print('최초', bucket)
fill(bucket, 100)
print('보충 후', bucket)

if deduct(bucket, 99):
    print('99 용량 사용')
else:
    print('가용 용량이 작아서 99 용량을 처리할 수 없음')
print('사용 후', bucket)

if deduct(bucket, 3):
    print('3 용량 사용')
else:
    print('가용 용량이 작아서 3 용량을 처리할 수 없음')

print('여전히', bucket)

>>>
최초 NewBucket(max_quota=0, quota_consumed=0)
보충 후 NewBucket(max_quota=100, quota_consumed=0)
99 용량 사용
사용 후 NewBucket(max_quota=100, quota_consumed=99)
가용 용량이 작아서 3 용량을 처리할 수 없음
여전히 NewBucket(max_quota=100, quota_consumed=99)
```

**기억해야 할 내용**
- @property를 사용해 기존 인스턴스 애트리뷰트에 새로운 기능을 제공할 수 있다.
- @property를 사용해 데이터 모델을 점진적으로 개선하라.
- @property 메서드를 너무 과하게 쓰고 있다면, 클래스와 클래스를 사용하는 모든 코드를 리팩터링하는 것을 고려하라.