## Better way 7. range보다는 enumerate를 사용하라

enumerate는 이터레이터를 지연 계산 제너레이터(lazy generator)로 감싼다. enumerate는 루프 인덱스와 이터레이터의 다음 값으로 이뤄진 쌍을 넘겨준다(yield). 다음 코드는 next 내장 함수를 사용해 다은 원소를 가져온다.

참고: [lazy generator란?](https://itholic.github.io/python-lazy-evaluation/)

참고: [iterator와 generator](https://mingrammer.com/translation-iterators-vs-generators/)

generator는 iterator를 생성해주는 function. 

```python
flavor_list = ['바닐라', '초콜릿', '피칸', '딸기']
it = enumerate(flavor_list)
print(next(it))
print(next(it))

>>>
(0, '바닐라')
(1, '초콜릿')

```

**기억해야 할 내용**
- enumerate를 사용하면 이터레이터에 대해 루프를 돌면서 이터레이터에서 가저오는 원소의 인덱스까지 얻는 코드를 간결하게 작성할 수 있다.
- range에 대해 루프를 돌면서 시퀀스의 원소를 인덱스로 가져오기보다는 enumerate을 사용하라.
- enumerate의 두 번째 파라미터로 어디서부터 원소를 가져오기 시작할지 지정할 수 있다(디폴트 값은 0이다)