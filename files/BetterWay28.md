## Better way 28. 컴프리헨션 내부에 제어 하위 식을 세 개 이상 사용하지 말라

리스트 안에 리스트가 들어 있는 형태로 정의한 행렬을 모든 원소가 들어 있는 평평한 단일 리스트로 단순화하고 싶다고 하자. 컴프리헨션에 하위 식을 두 개 포함시키면 이런 처리가 가능하다. 각각의 하위 식은 컴프리헨션에 들어간 순서대로 왼쪽에서 오른쪽으로 실행된다.

```python
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
flat = [x for row in matrix for x in row]
print(flat)

>>>
[1, 2, 3, 4, 5, 6, 7, 8, 9]
```

**기억해야 할 내용**
- 컴프리헨션은 여러 수준의 루프를 지원하며 각 수준마다 여러 조건을 지원한다.
- 제어 하위 식이 세 개 이상인 컴프리헨션은 이해하기 매우 어려우므로 가능하면 피해야 한다.