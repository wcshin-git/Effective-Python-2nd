## Better way 4. C스타일 형식 문자열을 str.format과 쓰기보다는 f-문자열을 통한 인터폴레이션을 사용하라

형식화(formatting)는 미리 정의된 문자열에 데이터 값을 끼워 넣어서 사람이 보기 좋은 문자열로 저장하는 과정.

**% 형식화 연산자(C스타일 형식화)**

파이썬에서 문자열을 형식화하는 가장 일반적인 방법은 % 형식화 연산자를 사용하는 것이다. 이 연산자 왼쪽에 들어가는 미리 정의된 텍스트 템플릿을 형식 문자열이라고 부른다. 형식 문자열은 연산자 왼쪽에 있는 값을 끼워 넣을 자리를 표현하기 위해 %d와 같은 **형식 지정자**를 사용한다. 

```python
key = 'my_var'
value = 1.234
formatted = '%-10s = %.2f' % (key, value)
print(formatted)

>>>
my_var    = 1.23
```

하지만 이는 네 가지 문제가 있다.

- 형식화 식에서 tuple 내 데이터 값의 순서를 바꾸거나 값의 타입을 바꾸면 오류가 발생. 이런 오류를 피하려면 % 연산자의 좌우가 서로 잘 맞는지 계속 검사해야 함.
- 형식화 하기 전에 값을 살짝 변경해야 한다면 tuple의 길이가 너무 길어져서 여러 줄에 나눠 써야 하는데, 그로 인해 가독성이 나빠진다.
- 형식화 문자열에서 같은 값을 여러 번 사용하고 싶다면 튜플에서 같은 값을 여러 번 반복해야 한다.
- 형식화 식에 딕셔너리를 사용하면 1번과 3번 문제를 해결할 수 있지만, 가독성이 나쁘다는 2번 문제는 더 악화시킨다.

**내장 함수 format과 str.format**

내장 함수 format

```python
a = 1234.5678
formatted = format(a, ',.2f')
print(formatted)

b = 'my 문자열'
formatted = format(b, '^20s')
print('*', formatted, '*')

>>>
1,234.57
*        my 문자열        *
```

str.format

%d와 같은 C 스타일 형식화 지정자를 사용하는 대신 **위치 지정자** {}를 사용할 수 있다. 기본적으로 형식화 문자열의 위치 지정자는 format 메서드에 전달된 인자 중 순서상 같은 위치에 있는 인자를 가리킨다. 각 위치 지정자에는 콜론 뒤에 형식 지정자를 붙여 넣어 문자열에 값을 넣을 때 어떤 형식으로 변환할지 정할 수 있다.

```python
key = 'my_var'
value = 1.234
formatted = '{:<10} = {:.2f}'.format(key, value)
print(formatted)

>>>
my_var     = 1.23
```

위치 지정자 중괄호에 위치 인덱스, 즉 format 메서드에 전달된 인자의 순서를 표현하는 위치 인덱스를 전달할 수도 있다.

```python
formatted = '{1} = {0}'.format(key, value)
print(formatted)

>>>
1.234 = my_var
```

이를 통해 인자의 순서가 맞는지 체킹하는 노력을 하지 않아도 되고(1번 문제 해결), 같은 값을 여러번 쓰고 싶을 때는 그 인덱스를 여러 번 쓰면 된다(3번 문제 해결). 하지만 가독성(2번 문제)은 해결 불가능.

따라서 웬만하면 str.format 메서드는 사용하지 말 것!

**인터폴레이션을 통한 형식 문자열(f-문자열)**

파이썬 3.6부터 도입. format 함수의 형식 지정자 안에서 콜론 뒤에 사용할 수 있는 내장 미니 언어를 f-문자열에도 사용할 수 있다. 또한 위치 지정자 중괄호 안에 완전한 파이썬 식을 넣을 수 있다.

```python
for i, (item, count) in enumerate(pantry):
    old_style = '#%d: %-10s = %d' % (
        i + 1,
        item.title(),
        round(count))
        
    new_style = '#{}: {:<10s} = {}'.format(
        i + 1,
        item.title(),
        round(count))
        
    f_string = f'#{i+1}: {item.title():<10s} = {round(count)}'
```

**기억해야 할 내용**
- %연산자를 사용하는 C스타일 형식화 문자열은 여러 가지 단점과 번잡성이라는 문제가 있다.
- str.format 메서드는 형식 지정자 미니 언어에서 유용한 개념 및 몇 가지를 새로 제공했다. 하지만 이를 제외하면 str.format 메서드도 C스타일 형식 문자열의 문제점을 그대로 가지고 있으므로, 가능하면 str.format사용을 피해야 한다.
- f-문자열은 값을 문자열 안에 넣는 새로운 구문으로, C 스타일 형식화 문자열의 가장 큰 문제점을 해결해준다.
- f-문자열은 간결하지만, 위치 지정자 안에 임의의 파이썬 식을 직접 포함시킬 수 있으므로 매우 강력하다.