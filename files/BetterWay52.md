동시성(concurrency): 컴퓨터가 같은 시간에 여러 다른 작업을 처리하는 것처럼 보이는 것

병렬성(parallelism): 같은 시간에 여러 다른 작업을 실제로 처리하는 것

병렬적인 프로그램은 항상 동시성 프로그램이지만, 동시성 프로그램이 반드시 병렬적인 프로그램인 것은 아니다. 

병렬성과 동시성의 가장 핵심적인 차이는 속도 향상에 있다. 어떤 프로그램의 서로 다른 두 실행 경로가 병렬적으로 앞으로 진행되면, 전체 작업을 수행하는 데 걸리는 시간이 절반으로 줄어든다. 반대로 동시성 프로그램은 겉으로 볼 때는 병렬적으로 실행되는 것처럼 보이지만, 전체 작업에 걸리는 시간은 빨라지지 않는다.

## Better way 52. 자식 프로세스를 관리하기 위해 subprocess를 사용하라

파이썬이 하위 프로세스를 실행하는 방법은 많다(예. os.popen, os.exec 등). 하지만 자식 프로세스를 관리할 때는 subprocess 내장 모듈을 사용하는 것이 가장 좋다. 

간단하게 사용할 때는 subprocess.run이 깔끔

```python
import subprocess

# 파이썬 3.6이나 그 이전 버전에서는 제대로 작동하지 않는다.(capture_output을 사용할 수 없음)
# 윈도우에서는 echo가 없는 경우 제대로 작동하지 않을 수 있다.
result = subprocess.run(['echo', '자식프로세스가 보내는 인사!'], capture_output=True, encoding='utf-8')
# capture_output=True로 설정하면 result객체로 표준출력에 해당되는 result.stdout, 또는 표준에러에 해당되는 result.stderr를 구할 수 있다.

result.check_returncode() # 예외가 발생하지 않으면 문제 없이 잘 종료한 것이다
print(result.stdout)

>>>
자식프로세스가 보내는 인사!

# 참고
"""
capture_output= False로 하고 실행하면?
>>>
자식프로세스가 보내는 인사!
None
"""
```

세부적인 제어는 subprocess.popen을 이용해야 한다.

```python
import subprocess

# 윈도우에서는 sleep이 없는 경우 제대로 작동하지 않을 수 있다.
proc = subprocess.Popen(['sleep', '1'])
while proc.poll() is None:  # .poll(): 자식 프로세스가 종료되었는지를 확인한다.
    print('작업중')

print('종료 상태', proc.poll())

>>>
작업중
작업중
작업중
작업중
...
종료 상태 0
```

부모 프로세스가 원하는 개수만큼 많은 자식 프로세스를 병렬로 실행할 수 있다. 다음 코드는 Popen을 사용해 자식 프로세스를 한꺼번에 시작한다. 

```python
import subprocess
import time

# 윈도우에서는 sleep이 없는 경우 제대로 작동하지 않을 수 있다.
start = time.time()
sleep_procs = []
for _ in range(10):
    proc = subprocess.Popen(['sleep', '1'])
    sleep_procs.append(proc)

for proc in sleep_procs:
    proc.communicate()  # 질문: communicate()메소드가 하는 일은? 스레드의 join같은 역할(다른 프로세스)

end = time.time()
delta = end - start
print(f'{delta:.3} 초만에 끝남')

>>>
1.02 초만에 끝남
```

각 프로세스가 순차적으로 실행됐다면, 총 걸린 시간은 10초 이상이었을 것이다.

파이썬 프로그램의 데이터를 파이프(pipe)를 사용해 하위 프로세서로 보내거나, 하위 프로세서의 출력을 받을 수 있다. 이를 통해 여러 다른 프로그램을 사용해서 병렬적으로 작업을 수행할 수 있다. 

```python
import subprocess
import os

# 시스템에 openssl을 설치하지 않은 경우에는 작동하지 않을 수 있다.
def run_encrypt(data):
    env = os.environ.copy()
    env['password'] = 'zf7ShyBhZOraQDdE/FiZpm/m/8f9X+M1'
    proc = subprocess.Popen(
        ['openssl', 'enc', '-des3', '-pass', 'env:password'],
        env=env,
        stdin=subprocess.PIPE,  # 인풋 아웃풋을 받을 수 있는 인터페이스라고 이해하기
        stdout=subprocess.PIPE) 
    proc.stdin.write(data) # stdin=subprocess.PIPE로 해서 데이터를 이렇게 넣는 것임
    proc.stdin.flush()     # 자식이 입력을 받도록 보장한다. 물을 내리는 것. 버퍼라는 공간에 남아 있는 데이터를 파이프로 밀어 넣음.
    return proc

procs = []
for _ in range(3):
    data = os.urandom(10)  # return random bytes
    proc = run_encrypt(data)
    procs.append(proc)

for proc in procs:
    out, err = proc.communicate()
    print(out[-10:])

>>>
b'\xe49\xf5\xdc"\x0c\x7fn\xcf@'
b'\x8eAyv\xae\x1dM^4\xaa'
b'\xe2\x12\x97\xf2\x05\x0eH]\xca\xb7'
```

한 자식 프로세스의 출력을 다음 프로세스의 입력으로 계속 연결시켜서 여러 병렬 프로세스를 연쇄적으로 연결할 수도 있다.

```python
import subprocess
import os

# 시스템에 openssl을 설치하지 않은 경우에는 작동하지 않을 수 있다.
def run_encrypt(data):
    env = os.environ.copy()
    env['password'] = 'zf7ShyBhZOraQDdE/FiZpm/m/8f9X+M1'
    proc = subprocess.Popen(
        ['openssl', 'enc', '-des3', '-pass', 'env:password'],
        env=env,
        stdin=subprocess.PIPE,
        stdout=subprocess.PIPE)
    proc.stdin.write(data)
    proc.stdin.flush() # 자식이 입력을 받도록 보장한다
    return proc

def run_hash(input_stdin):
    return subprocess.Popen(
        ['openssl', 'dgst', '-whirlpool', '-binary'],
        stdin=input_stdin,
        stdout=subprocess.PIPE)

encrypt_procs = []
hash_procs = []

for _ in range(3):
    data = os.urandom(100)

    encrypt_proc = run_encrypt(data)
    encrypt_procs.append(encrypt_proc)

    hash_proc = run_hash(encrypt_proc.stdout)
    hash_procs.append(hash_proc)

    # 자식이 입력 스트림에 들어오는 데이터를 소비하고 communicate() 메서드가
    # 불필요하게 자식으로부터 오는 입력을 훔쳐가지 못하게 만든다.
    # 또 다운스트림 프로세스가 죽으면 SIGPIPE를 업스트림 프로세스에 전달한다.
    encrypt_proc.stdout.close()
    encrypt_proc.stdout = None

for proc in encrypt_procs:
    proc.communicate()
    assert proc.returncode == 0

for proc in hash_procs:
    out, _ = proc.communicate()
    print(out[-10:])

assert proc.returncode == 0

>>>
b'\x9c\x83\xc6lZ*)\xfc\x98I'
b'0m\x1c\xd9\x90\x1f_x\x8f\x8e'
b'\xfa\xfb\xa0_\xd27\xaci\xd0e'
```

자식 프로세스가 결코 끝나지 않는 경우, 입력이나 출력 파이프를 기다리면서 블록(block)되는 경우가 우려된다면 timeout 파라미터를 communicate 메서드에 전달할 수 있다. timeout 값을 전달하면 자식 프로세스가 주어진 시간 동안에 끝나지 않을 경우 예외가 발생한다. 

```python
import subprocess

# 윈도우에서는 sleep이 없으면 제대로 작동하지 않을 수 있다.
proc = subprocess.Popen(['sleep', '10'])
try:
    proc.communicate(timeout=0.1)
except subprocess.TimeoutExpired:
    proc.terminate()
    proc.wait()

print('종료 상태', proc.poll())

>>>
종료 상태 -15
```

**기억해야 할 내용**
- subprocess 모듈을 사용해 자식 프로세스를 실행하고 입력과 출력 스트림을 관리할 수 있다.
- 자식 프로세스는 파이썬 인터프리터와 병렬로 실행되므로 CPU 코어를 최대로 쓸 수 있다.
- 간단하게 자식 프로세스를 실행하고 싶은 경우에는 run 편의 함수를 사용하라. 유닉스 스타일의 파이프라인이 필요하다면 Popen 클래스를 사용하라.
- 자식 프로세스가 멈추는 경우나 교착 상태를 방지하려면 communicate 메서드에 대해 timeout 파라미터를 사용하라.