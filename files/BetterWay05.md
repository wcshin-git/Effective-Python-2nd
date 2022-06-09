## Better way 5. 복잡한 식을 쓰는 대신 도우미 함수를 작성하라

딕셔너리의 get 메서드는 딕셔너리 안에 키에 해당하는 값이 없을 때 두 번째 인자를 반환한다.

빈 문자열, 빈 list, 0은 모두 암묵적으로 False로 평가된다.

```python
my_values = {'빨강': ['5'], '파랑': ['0'], '초록': ['']}

green_str = my_values.get('초록', ['']) # print(repr(green_str)): ['']
if green_str[0]:
    green = int(green_str[0])
else:
    green = 0

# 도우미 함수 사용시
def get_first_int(values, key, default=0):
    found = values.get(key, [''])
    if found[0]:
        return int(found[0])
    return default

green = get_first_int(my_values, '초록')
```

**기억해야할 내용**
- 파이썬 문법을 사용하면 아주 복잡하고 읽기 어려운 한 줄짜리 식을 쉽게 작성할 수 있다.
- 복잡한 식을 도우미 함수로 옮겨라. 특히 같은 로직을 반복해 사용할 때는 도우미 함수를 꼭 사용하라.
- 불(boolean) 연산자 or나 and를 식에 사용하는 것보다 if/else식을 쓰는 편이 더 가독성이 좋다.