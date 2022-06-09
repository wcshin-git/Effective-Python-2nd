## Better way 36. 이터레이터나 제너레이터를 다룰 때는 itertools를 사용하라

**여러 이터레이터 연결하기** 

```python
import itertools

# chain
## 여러 이터레이터를 하나의 순차적인 이터레이터로 합치고 싶을 때
it = itertools.chain([1, 2, 3], [4, 5, 6])
print(list(it))

>>>
[1, 2, 3, 4, 5, 6]

# repeat
## 한 값을 계속 반복해 내놓고 싶을 때
it = itertools.repeat('안녕', 3)
print(list(it))

>>>
['안녕', '안녕', '안녕']

# cycle
## 어떤 이터레이터가 내놓는 원소들을 계속 반복하고 싶을 때
it = itertools.cycle([1, 2])
result = [next(it) for _ in range (10)]
print(result)

>>>
[1, 2, 1, 2, 1, 2, 1, 2, 1, 2]

# tee
## 한 이터레이터를 병렬적으로 두 번째 인자로 지정된 개수의 이터레이터로 만들고 싶을 때 
it1, it2, it3 = itertools.tee(['하나', '둘'], 3)
print(list(it1))
print(list(it2))
print(list(it3))

>>>
['하나', '둘']
['하나', '둘']
['하나', '둘']

# zip_longest
## zip 내장 함수의 변종으로, 여러 이터레이터 중 짧은 쪽 이터레이터의 원소를 다 사용한 경우 fillvalue로 지정한 값을 채워 넣어준다.
keys = ['하나', '둘', '셋']
values = [1, 2]

normal = list(zip(keys, values))
print('zip:', normal)

it = itertools.zip_longest(keys, values, fillvalue='없음')
longest = list(it)
print('zip_longest:', longest)

>>>
zip: [('하나', 1), ('둘', 2)]
zip_longest: [('하나', 1), ('둘', 2), ('셋', '없음')]

```

**이터레이터에서 원소 거르기**

```python
# islice
## 이터레이터를 복사하지 않으면서 원소 인덱스를 이용해 슬라이싱하고 싶을 때 
values = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

first_five = itertools.islice(values, 5)
print('앞에서 다섯 개:', list(first_five))

middle_odds = itertools.islice(values, 2, 8, 2)
print('중간의 홀수들:', list(middle_odds))

>>>
앞에서 다섯 개: [1, 2, 3, 4, 5]
중간의 홀수들: [3, 5, 7]

# takewhile
## 이터레이터에서 주어진 술어(predicate)가 False를 반환하는 첫 원소가 나타날 때까지 원소를 돌려준다.
values = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
less_than_seven = lambda x: x < 7
it = itertools.takewhile(less_than_seven, values)
print(list(it))

>>>
[1, 2, 3, 4, 5, 6]

# dropwhile
## takewhile의 반대
values = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
less_than_seven = lambda x: x < 7
it = itertools.dropwhile(less_than_seven, values)
print(list(it))

>>>
[7, 8, 9, 10]

# filterfalse
## filter 내장 함수의 반대. 주어진 이터레이터에서 술어가 False를 반환하는 모든 원소를 돌려준다.
values = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
evens = lambda x: x % 2 == 0

filter_result = filter(evens, values)
print('Filter:', list(filter_result))

filter_false_result = itertools.filterfalse(evens, values)
print('Filter false:', list(filter_false_result))

>>>
Filter: [2, 4, 6, 8, 10]
Filter false: [1, 3, 5, 7, 9]

```

**이터레이터에서 원소의 조합 만들어내기**

```python
# accumulate
## 파라미터를 두 개 받는 함수를 반복 적용하면서 이터레이터 원소를 값 하나로 줄여준다.
values = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
sum_reduce = itertools.accumulate(values)
print('합계:', list(sum_reduce))

def sum_modulo_20(first, second):
    output = first + second
    return output % 20

modulo_reduce = itertools.accumulate(values, sum_modulo_20)
print('20으로 나눈 나머지의 합계:', list(modulo_reduce))

>>>
합계: [1, 3, 6, 10, 15, 21, 28, 36, 45, 55]
20으로 나눈 나머지의 합계: [1, 3, 6, 10, 15, 1, 8, 16, 5, 15]

# product
## 하나 이상의 이터레이터에 들어 있는 아이템들의 데카르트 곱(Cartesian product)을 반환한다.
single = itertools.product([1, 2], repeat=2)
print('리스트 한 개:', list(single))

multiple = itertools.product([1, 2], ['a', 'b'])
print('리스트 두 개:', list(multiple))

>>>
리스트 한 개: [(1, 1), (1, 2), (2, 1), (2, 2)]
리스트 두 개: [(1, 'a'), (1, 'b'), (2, 'a'), (2, 'b')]

# permutations
## 이터레이터가 내놓는 원소들로부터 만들어낸 길이 N인 순열을 돌려준다. 원소들이 모두 다 다르다고 가정하고 순열을 만들어낸다.
it = itertools.permutations([1, 2, 3, 4], 2)
print(list(it))

>>>
[(1, 2), (1, 3), (1, 4), (2, 1), (2, 3), (2, 4), (3, 1), (3, 2), (3, 4), (4, 1), (4, 2), (4, 3)]

# combinations
## 이터레이터가 내놓는 원소들로부터 만들어낸 길이 N인 조합을 돌려준다. 원소들이 모두 다 다르다고 가정하고 조합을 만들어낸다.
it = itertools.combinations([1, 2, 3, 4], 2)
print(list(it))

>>>
[(1, 2), (1, 3), (1, 4), (2, 3), (2, 4), (3, 4)]

# combinations_with_replacement
## combination과 같지만 원소의 반복을 허용한다. 원소들이 모두 다 다르다고 가정하에 중복 조합을 만든다.
it = itertools.combinations_with_replacement([1, 2, 3, 4], 2)
print(list(it))

>>>
[(1, 1), (1, 2), (1, 3), (1, 4), (2, 2), (2, 3), (2, 4), (3, 3), (3, 4), (4, 4)]

```

**기억해야 할 내용**
- 이터레이터나 제너레이터를 다루는 itertools 함수는 세 가지 범주로 나눌 수 있다.
    - 여러 이터레이터를 연결함
    - 이터레이터의 원소를 걸러냄
    - 원소의 조합을 만들어냄
- 파이썬 인터프리터에서 help(itertools)를 입력한 후 표시되는 문서를 살펴보면 더 많은 고급 함수와 추가 파라미터를 알 수 있으며, 이를 사용하는 유용한 방법도 확인할 수 있다.