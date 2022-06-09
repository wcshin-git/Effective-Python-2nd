## Better way 39. 객체를 제너릭하게 구성하려면 @classmethod를 통한 다형성을 활용하라

다형성(polymorphism)이란? 같은 모양의 코드가 다른 동작을 하는 것. Method override도 다형성의 한 예이다.

```python
### GenericInputData와 PathInputData를 사용한 방법
from threading import Thread

class GenericInputData:
    def read(self):
        raise NotImplementedError

    @classmethod
    def generate_inputs(cls, config):
        raise NotImplementedError

class PathInputData(GenericInputData):
    def __init__(self, path):
        super().__init__()
        self.path = path

    def read(self):
        with open(self.path) as f:
            return f.read()

    @classmethod
    def generate_inputs(cls, config):
        data_dir = config['data_dir']
        for name in os.listdir(data_dir):
            yield cls(os.path.join(data_dir, name)) # PathInputData(path)를 return하는 것임.

class GenericWorker:
    def __init__(self, input_data):
        self.input_data = input_data
        self.result = None

    def map(self):
        raise NotImplementedError

    def reduce(self, other):
        raise NotImplementedError

    @classmethod
    def create_workers(cls, input_class, config):
        workers = []
        for input_data in input_class.generate_inputs(config): # input_class.generate_inputs가 @classmethod 덕분에 가능한 것.
            workers.append(cls(input_data)) # cls를 쓸 수 있다는 게 장점!
        return workers

class LineCountWorker(GenericWorker):
    def map(self):
        data = self.input_data.read()
        self.result = data.count('\n')

    def reduce(self, other):
        self.result += other.result

def mapreduce(worker_class, input_class, config):
    workers = worker_class.create_workers(input_class, config)  # worker_class.create_workers가 @classmethod 덕분에 가능한 것.
    return execute(workers)

def execute(workers):
    threads = [Thread(target=w.map) for w in workers]
    for thread in threads: thread.start()
    for thread in threads: thread.join() # thread끼리 sync를 맞추는 것. 먼저 끝난 게 기다리고 있음.

    first, *rest = workers
    for worker in rest:
        first.reduce(worker)
    return first.result

tmpdir = '/Users/shin/Desktop/Effective_Python/Chapter5/test_inputs'
config = {'data_dir': tmpdir}
result = mapreduce(LineCountWorker, PathInputData, config)
print(f'총 {result} 줄이 있습니다.')

>>>
총 4709 줄이 있습니다.
```

**기억해야 할 내용**
- 파이썬의 클래스에는 생성자가 \_\_init\_\_ 메서드뿐이다.
- @classmethod를 사용하면 클래스에 다른 생성자를 정의할 수 있다.
- 클래스 메서드 다형성을 활용하면 여러 구체적인 하위 클래스의 객체를 만들고 연결하는 제너릭한 방법을 제공할 수 있다.