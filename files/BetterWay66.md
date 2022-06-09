## Better way 66. 재사용 가능한 try/finally 동작을 원한다면 contextlib과 with문을 사용하라

contextlib 내장 모듈을 사용하면 여러분이 만든 객체나 함수를 with 문에서 쉽게 쓸 수 있다. contextlib 내장 모듈은 with 문에 쓸 수 있는 함수를 간단히 만들 수 있는 contextmanager 데코레이터를 제공한다. 

```python
import logging

def my_function():
    logging.debug('디버깅 데이터')
    logging.error('이 부분은 오류 로그')
    logging.debug('추가 디버깅 데이터')

my_function()

>>>
ERROR:root:이 부분은 오류 로그
```

컨텍스트 매니저를 정의하면 이 함수의 로그 수준을 일시적으로 높일 수 있다. 이 도우미 함수는 with 블록을 실행하기 직전에 로그 심각성 수준을 높이고, 블록을 실행한 직후에 심각성 수준을 이전 수준으로 회복시켜준다. 

```python
from contextlib import contextmanager

@contextmanager
def debug_logging(level):
    logger = logging.getLogger()
    old_level = logger.getEffectiveLevel()
    logger.setLevel(level)
    try:
        yield
    finally:
        logger.setLevel(old_level)

with debug_logging(logging.DEBUG):
    print('* 내부:')
    my_function()

print('* 외부:')
my_function()

>>>
* 내부:
DEBUG:root:디버깅 데이터
ERROR:root:이 부분은 오류 로그
DEBUG:root:추가 디버깅 데이터
* 외부:
ERROR:root:이 부분은 오류 로그
```

**with와 대상 변수 함께 사용하기**

```python
@contextmanager
def log_level(level, name):
    logger = logging.getLogger(name)
    old_level = logger.getEffectiveLevel()
    logger.setLevel(level)
    try:
        yield logger
    finally:
        logger.setLevel(old_level)

with log_level(logging.DEBUG, 'my-log') as logger:
    logger.debug(f'대상: {logger.name}!')
    logging.debug('이 메시지는 출력되지 않습니다')

>>>
DEBUG:my-log:대상: my-log!
```

with문이 끝날 때 로그 심각성 수준이 원래대로 복구되므로, with문 밖에서 my-log라는 로거에 대해 debug를 통해 메시지를 출력해도 아무 메시지가 표시되지 않는다.

```python
logger = logging.getLogger('my-log')
logger.debug('디버그 메시지는 출력되지 않습니다')
logger.error('오류 메시지는 출력됩니다')

>>>
ERROR:my-log:오류 메시지는 출력됩니다
```

**기억해야 할 내용**
- with문을 사용하면 try/finally 블록을 통해 사용해야 하는 로직을 재활용하면서 시각적인 잡음도 줄일 수 있다.
- contextlib 내장 모듈이 제공하는 contextmanager 데코레이터를 사용하면 여러분이 만든 함수를 with문에 사용할 수 있다.
- 컨텍스트 매니저가 yield하는 값은 with 문의 as 부분에 전달된다. 이를 활용하면 특별한 컨텍스트 내부에서 실행되는 코드 안에서 직접 그 컨텍스트에 접근할 수 있다.