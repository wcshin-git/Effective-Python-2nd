## Better way 72. 정렬된 시퀀스를 검색할 때는 bisect를 사용하라

메모리에 정렬된 리스트로 존재하는 커다란 데이터를 검색하고 싶은 경우가 자주 있다. 파이썬 내장 bisect 모듈은 순서가 정해져 있는 리스트에 대해 이런 유형의 검사를 더 효과적으로 수행한다. bisect_left 함수를 사용하면 정렬된 원소로 이뤄진 시퀀스에 대해 이진 검색(binary search)을 효율적으로 수행할 수 있다. bisect_left가 반환하는 인덱스는 리스트에 찾는 값의 원소가 존재하는 경우 이 원소의 인덱스이며, 리스트에 찾는 값의 원소가 존재하지 않는 경우 정렬 순서상 해당 값을 삽입해야 할 자리의 인덱스다.

(bisect 모듈이 사용하는 이진 검색 알고리즘의 복잡도는 로그 복잡도)

```python
data = list(range(10**5))

from bisect import bisect_left

index = bisect_left(data, 91234) # 정확히 일치
assert index == 91234

index = bisect_left(data, 91234.56) # 근접한 값과 일치
assert index == 91235

index = bisect_left(data, 91234.23) # 근접한 값과 일치(찾는 값 이상의 값 중 근접한 값을 찾음)
assert index == 91235
```

**기억해야 할 내용**
- 리스트에 들어 있는 정렬된 데이터를 검색할 때 index 메서드를 사용하거나 for 루프와 맹목적인 비교를 사용하면 선형 시간이 걸린다.
- bisect 내장 모듈의 bisect_left 함수는 정렬된 리스트에서 원하는 값을 찾는 데 로그 시간이 걸린다. 따라서 다른 접근 방법보다 훨씬 빠르다.