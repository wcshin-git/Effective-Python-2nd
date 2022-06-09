## Better way 30. 리스트를 반환하기보다는 제너레이터를 사용하라

문자열에서 띄어쓰기 기준으로 단어의 시작 인덱스를 담은 리스트를 반환하고 싶다고 하자.

```python
def index_words(text):
    result = []
    if text:
        result.append(0)
    for index, letter in enumerate(text):
        if letter == ' ':
            result.append(index + 1)
    return result

address = '컴퓨터(영어: Computer, 문화어: 콤퓨터, 순화어:전산기)는 진공관'
result = index_words(address)
print(result)

>>>
[0, 8, 18, 23, 28, 38]
```

이 코드의 문제는, 코드에 잡음이 많고 핵심을 알아보기 어렵다는 것. 이 함수를 개선하는 방법은 제너레이터를 사용하는 것이다. 제너레이터는 yield 식을 사용하는 함수에 의해 만들어진다. 

```python
def index_words_iter(text):
    if text:
        yield 0
    for index, letter in enumerate(text):
        if letter == ' ':
            yield index + 1
```

이 함수가 호출되면 제너레이터 함수가 실제로 실행되지 않고 즉시 **이터레이터를 반환**한다. 이터레이터가 next 내장 함수를 호출할 때마다 이터레이터는 제너레이터 함수를 다음 yield식까지 진행시킨다. 제너레이터가 yield에 전달하는 값은 이터레이터에 의해 호출되는 쪽에 반환된다.

```python
it = index_words_iter(address)
print(next(it))
print(next(it))

>>>
0
8
```

제너레이터가 반환하는 이터레이터를 리스트 내장 함수에 넘기면 필요할 때 쉽게 리스트로 변환할 수 있다.

```python
result = list(index_words_iter(address))
print(result)

>>>
[0, 8, 18, 23, 28, 38]
```

**기억해야 할 내용**
- 제너레이터를 사용하면 결과를 리스트에 합쳐서 반환하는 것보다 더 깔끔하다.
- 제너레이터가 반환하는 이터레이터는 제너레이터 함수의 본문에서 yield가 반환하는 값들로 이뤄진 집합을 만들어낸다.
- 제너레이터를 사용하면 작업 메모리에 모든 입력과 출력을 저장할 필요가 없으므로 입력이 아주 커도 출력 시퀀스를 만들 수 있다.