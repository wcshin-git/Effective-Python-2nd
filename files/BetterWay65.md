소프트웨어에서 강건성은 프로그램이 실행되는 동안에 주위 환경이 변하거나, 잘못된 입력이 들어오거나, 내부에서 오류가 발생해도 별 문제없이 계속 실행되며 (오류 보고 등을 한 후) 정상적으로 계속 실행될 수 있는 능력을 뜻한다.

## Better way 65. Try/ecxept/else/finally의 각 블록을 잘 활용하라

**finally 블록**

예외가 발생하더라도 정리 코드를 실행해야 한다면 try/finally를 사용하라. 파일 핸들을 안전하게 닫기 위해 try/finally를 사용하는 경우가 자주 있다. try 블록이 실행된 다음에는 항상 finally 블록이 실행된다.

```python
def try_finally_example(filename):
    print('* 파일 열기')
    handle = open(filename, encoding='utf-8') # OSError 발생할 수 있음
    try:
        print('* 데이터 읽기')
        return handle.read()      # UnicodeDecodeError 발생할 수 있음
    finally:
        print('* close() 호출')
        handle.close()            # try 블록이 실행된 다음에는 항상 이 블록이 실행됨

filename = 'random_data.txt'

with open(filename, 'wb') as f:
    f.write(b'\xf1\xf2\xf3\xf4\xf5')  # 잘못된 utf-8 이진 문자열

# 오류가 나는 부분. 오류를 보고 싶으면 커멘트를 해제할것
data = try_finally_example(filename)

>>>
* 파일 열기
* 데이터 읽기
* close() 호출
UnicodeDecodeError: 'utf-8' codec can't decode byte 0xf1 in position 0: invalid continuation byte
```

**else 블록**

try 블록이 예외를 발생시키지 않으면 else 블록이 실행된다. else 블록을 사용하면 try 블록 안에 들어갈 코드를 최소화할 수 있다. try 블록에 들어가는 코드가 줄어들면 발생할 여지가 있는 예외를 서로 구분할 수 있으므로 가독성이 좋아진다.

```python
import json

def load_json_key(data, key):
    try:
        print('* JSON 데이터 읽기')
        result_dict = json.loads(data)     # ValueError가 발생할 수 있음
    except ValueError as e:
        print('* ValueError 처리')
        raise KeyError(key) from e
    else:
        print('* 키 검색')
        return result_dict[key]            # KeyError가 발생할 수 있음
```

```python
load_json_key('{"foo": bad payload', 'foo')

>>>
* JSON 데이터 읽기
* ValueError 처리

JSONDecodeError: Expecting value: line 1 column 9 (char 8)

The above exception was the direct cause of the following exception:
```

```python
load_json_key('{"foo": "bar"}', 'fool')
>>>

* JSON 데이터 읽기
* 키 검색

KeyError: 'fool'
```

**모든 요소를 한꺼번에 사용하기**

```python
UNDEFINED = object()

def divide_json(path):
    print('* 파일 열기')
    handle = open(path, 'r+')             # OSError가 발생할 수 있음
    try:
        print('* 데이터 읽기')
        data = handle.read()              # UnicodeDecodeError가 발생할 수 있음
        print('* JSON 데이터 읽기')
        op = json.loads(data)             # ValueError가 발생할 수 있음
        print('* 계산 수행')
        value = (
            op['numerator'] /
            op['denominator'])            # ZeroDivisionError가 발생할 수 있음
    except ZeroDivisionError as e:        # try문에서 ZeroDivisionError가 발생할 경우 캐치하여 이 블록을 실행
        print('* ZeroDivisionError 처리')
        return UNDEFINED
    else:                                 # try 블록에서 어떤 에러도 발생하지 않았을 때 실행됨
        print('* 계산 결과 쓰기')
        op['result'] = value
        result = json.dumps(op)
        handle.seek(0)                    # OSError가 발생할 수 있음
        handle.write(result)              # OSError가 발생할 수 있음
        return value
    finally:
        print('* close() 호출')
        handle.close()                    # 어떤 경우든 실행됨. finally 안에 return이 있으면 앞의 블록에 return이 있었어도 가장 마지막에 호출되는 finally의 return을 전달한다.
```

정상적이 경우 try, else, finally 블록이 실행된다.

```python
temp_path = 'random_data.json'

with open(temp_path, 'w') as f:
    f.write('{"numerator": 1, "denominator": 10}')

assert divide_json(temp_path) == 0.1
>>>

* 파일 열기
* 데이터 읽기
* JSON 데이터 읽기
* 계산 수행
* 계산 결과 쓰기
* close() 호출
```

계산이 잘못된 경우 try, except, finally 블록은 실행되지만 else 블록은 실행되지 않는다. 

```python
with open(temp_path, 'w') as f:
    f.write('{"numerator": 1, "denominator": 0}')

assert divide_json(temp_path) is UNDEFINED
>>>

* 파일 열기
* 데이터 읽기
* JSON 데이터 읽기
* 계산 수행
* ZeroDivisionError 처리
* close() 호출
```

JSON 데이터가 잘못된 경우에는 try, finally 블록이 실행되지만 except와 else 블록은 실행되지 않는다.

```python
with open(temp_path, 'w') as f:
    f.write('{"numerator": 1 bad data')

# 오류가 나는 부분. 오류를 보고 싶으면 커멘트를 해제할것
divide_json(temp_path)

>>>
* 파일 열기
* 데이터 읽기
* JSON 데이터 읽기
* close() 호출

JSONDecodeError: Expecting ',' delimiter: line 1 column 17 (char 16)
```

**기억해야 할 내용**
- try/finally 복합문을 사용하면 try 블록이 실행되는 동안 예외가 발생하든 발생하지 않든 정리 코드를 실행할 수 있다.
- else 블록을 사용하면 try 블록 안에 넣을 코드를 최소화하고, try/except 블록과 성공적인 경우에 수행해야 할 코드를 시각적으로 구분할 수 있다.
- try 블록이 성공적으로 처리되고 finally 블록이 공통적인 정리 작업을 수행하기 전에 실행해야 하는 동작이 있는 경우 else 블록을 사용할 수 있다.