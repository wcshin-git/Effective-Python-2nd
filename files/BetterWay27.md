## Better way 27. map과 filter 대신 컴프리헨션을 사용하라

```python
a = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

alt = map(lambda x: x**2, filter(lambda x: x % 2 == 0, a))

even_squares = [x**2 for x in a if x % 2 == 0] # 리스트 컴프리핸션

```

딕셔너리와 집합에도 리스트 컴프리헨션과 동등한 컴프리헨션이 있다. 각각 딕셔너리 컴프리헨션과 집합 컴프리헨션이라고 한다.

```python
even_squares_dict = {x: x**2 for x in a if x % 2 == 0}
threes_cubed_set = {x**3 for x in a if x % 3 == 0}
print(even_squares_dict)
print(threes_cubed_set)

>>>
{2: 4, 4: 16, 6: 36, 8: 64, 10: 100}
{216, 729, 27}
```

**기억해야 할 내용**
- 리스트 컴프리헨션은 lambda 식을 사용하지 않기 때문에 같은 일을 하는 map과 filter내장 함수를 사용하는 것보다 더 명확하다.
- 리스트 컴프리헨션을 사용하면 쉽게 입력 리스트의 원소를 건너뛸 수 있다. 하지만 map을 사용하는 경우에는 filter의 도움을 받아야만 한다.
- 딕셔너리와 집합도 컴프리헨션으로 생성할 수 있다.