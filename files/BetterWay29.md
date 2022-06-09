## Better way 29. 대입식을 사용해 컴프리헨션 안에서 반복 작업을 피하라

```python
stock = {
    '못': 125,
    '나사못': 35,
    '나비너트': 8,
    '와셔': 24,
}

order = ['나사못', '나비너트', '클립']

found = {name: get_batches(stock.get(name, 0), 8)
         for name in order
         if get_batches(stock.get(name, 0), 8)}
print(found)

>>>
{'나사못': 4, '나비너트': 1}
```

이 코드는 get_batches(stock.get(name, 0)이 반복된다는 단점이 있다. 이러한 문제에 대한 쉬운 해법은 파이썬 3.8에 도입된 왈러스 연산자(:=)를 사용하는 것이다. 왈러스 연산자를 사용하면 컴프리헨션의 일부분에 대입식을 만들 수 있다. 

```python
found = {name: batches for name in order
         if (batches := get_batches(stock.get(name, 0), 8))}
```

대입식 batches := get_batches(stock.get(name, 0), 8)을 사용하면 stock 딕셔너리에서 각 order 키를 한 번만 조회하고 get_batches를 한 번만 호출해서 그 결과를 batches 변수에 저장할 수 있다. 

**기억해야 할 내용**
- 대입식을 통해 컴프리헨션이나 제너레이터 식의 조건 부분에서 사용한 값을 같은 컴프리헨션이나 제너레이터의 다른 위치에서 재사용할 수 있다. 이를 통해 가독성과 성능을 향상시킬 수 있다.
- 조건이 아닌 부분에도 대입식을 사용할 수 있지만, 그런 형태의 사용은 피해야 한다.