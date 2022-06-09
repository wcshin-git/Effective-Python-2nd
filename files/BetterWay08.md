## Better way 8. 여러 이터레이터에 대해 나란히 루프를 수행하려면 zip을 사용하라

같은 인덱스 위치에 관련 있는 원소들을 가진 두 리스트를 동시에 이터레이션할 경우.

zip은 둘 이상의 이터레이터를 지연 계산 제너레이터(lazy generator)를 사용해 묶어준다. zip 제너레이터는 각 이터레이터의 다음 값이 들어 있는 튜플을 반환한다. 이 튜플을 for문에서 바로 언패킹할 수 있다.

```python
names = ['Cecilia', '남궁민수', '毛泽东']
counts = [len(n) for n in names]
for name, count in zip(names, counts):
		...
```

하지만 입력 이터레이터의 길이가 서로 다를 때는 zip이 어떻게 작동하는지에 주의해야 한다.

```python
names.append('Rosalind')
for name, count in zip(names, counts):
    print(name)

>>>
Cecilia
남궁민수
毛泽东
```

새로 추가한 원소인 'Rosalind'에 대한 출력은 없다. zip의 출력은 가장 짧은 입력의 길이와 같다. 하지만 긴 이터레이터의 뒷부분을 버리는 zip 기능이 바람직하지 못할 때도 있다. zip에 전달한 리스트의 길이가 같지 않을 것으로 예상한다면 itertools 내장 모듈에 들어 있는 zip_longest를 대신 사용하는 것을 고려하라.

```python
import itertools
for name, count in itertools.zip_longest(names, counts):
    print(f'{name}: {count}')

>>>
Cecilia: 7
남궁민수: 4
毛泽东: 3
Rosalind: None
```

zip_longest는 존재하지 않는 값(여기서는 counts 리스트에서 'Rosalind' 문자열의 길이에 해당하는 값)을 자신에게 전달된 fillvalue로 대체한다. 디폴트 fillvalue는 None이다.

**기억해야 할 내용**
- zip 내장 함수를 사용해 여러 이터레이터를 나란히 이터레이션할 수 있다.
- zip은 튜플을 지연 계산하는 제너레이터를 만든다. 따라서 무한히 긴 입력에도 zip을 쓸 수 있다.
- 입력 이터레이터의 길이가 서로 다르면 zip은 아무런 경고도 없이 가장 짧은 이터레이터 길이까지만 튜플을 내놓고 더 긴 이터레이터의 나머지 원소는 무시한다.
- 가장 짧은 이터레이터에 맞춰 길이를 제한하지 않고 길이가 서로 다른 이터레이터에 대해 루프를 수행하려면 itertools 내장 모듈의 zip_longest 함수를 사용하라.