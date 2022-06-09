## Better way 13. 슬라이싱보다는 나머지를 모두 잡아내는 언패킹을 사용하라

기본 언패킹(Better way 6)의 한 가지 한계점은 언패킹할 시퀀스의 길이를 미리 알고 있어야 한다는 것이다. 

이런 상황을 더 잘 다룰 수 있도록 파이썬은 별표식(starred expression)을 사용해 모든 값을 담는 언패킹을 할 수 있게 지원한다. 

```python
car_ages = [0, 9, 4, 8, 7, 20, 19, 1, 6, 15]
car_ages_descending = sorted(car_ages, reverse=True)
oldest, second_oldest, *others = car_ages_descending
print(oldest, second_oldest, others)

>>>
20 19 [15, 9, 8, 7, 6, 4, 1, 0]
```

별표 식을 다른 위치에 쓸 수도 있다.

```python
oldest, *others, youngest = car_ages_descending
print(oldest, youngest, others)

>>>
20 0 [19, 15, 9, 8, 7, 6, 4, 1]

*others, second_youngest, youngest = car_ages_descending
print(youngest, second_youngest, others)

>>>
0 1 [20, 19, 15, 9, 8, 7, 6, 4]

# 이렇게도 가능. (별표식을 우측에)
a = [1, 2]
first, second, thrid = *a, 3
```

별표식만 사용해 언패킹할 수는 없다. 또한 한 수준의 언패킹 패턴에 별표 식을 두 개 이상 쓸 수도 없다.

```python
# 별표식만 사용해 언패킹할 수 없음
*others = car_ages_descending

>>>
SyntaxError: starred assignment target must be in a list or tuple

# 한 수준의 언패킹 패턴에 별표 식을 두 개 이상 쓸 수 없음
first, *middle, *second_middle, last = [1, 2, 3, 4]
>>>
SyntaxError: two starred expressions in assignment
```

별표식은 항상 list 인스턴스가 된다. 언패킹 시퀀스에 남는 원소가 없으면 별표 식 부분은 빈 리스트가 된다. 

**기억해야할 내용**
- 언패킹 대입에 별표 식을 사용하면 언패킹 패턴에서 대입되지 않는 모든 부분을 리스트에 잡아낼 수 있다.
- 별표 식은 언패킹 패턴의 어떤 위치에든 놓을 수 있다. 별표 식에 대입된 결과는 항상 리스트가 되며, 이 리스트에는 별표식이 받은 값이 0개 또는 그 이상 들어간다.
- 리스트를 서로 겹치지 않게 여러 조각으로 나눌 경우, 슬라이싱과 인덱싱을 사용하기 보다는 나머지를 모두 잡아내는 언패킹을 사용해야 실수할 여지가 훨씬 줄어든다.