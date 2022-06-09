## Better way 56. 언제 동시성이 필요한지 인식하는 방법을 알아두라

콘웨이의 생명게임

```python
ALIVE = '*'
EMPTY = '-'

class Grid:
    def __init__(self, height, width):
        self.height = height
        self.width = width
        self.rows = []
        for _ in range(self.height):
            self.rows.append([EMPTY] * self.width)

    def get(self, y, x):
        return self.rows[y % self.height][x % self.width]

    def set(self, y, x, state):
        self.rows[y % self.height][x % self.width] = state

    def __str__(self):
        output = ''
        for row in self.rows:
            for cell in row:
                output += cell
            output += '\n'
        return output

grid = Grid(5, 9)
grid.set(0, 3, ALIVE)
grid.set(1, 4, ALIVE)
grid.set(2, 2, ALIVE)
grid.set(2, 3, ALIVE)
grid.set(2, 4, ALIVE)
print(grid)

def count_neighbors(y, x, get):
    n_ = get(y - 1, x + 0) # 북(N)
    ne = get(y - 1, x + 1) # 북동(NE)
    e_ = get(y + 0, x + 1) # 동(E)
    se = get(y + 1, x + 1) # 남동(SE)
    s_ = get(y + 1, x + 0) # 남(S)
    sw = get(y + 1, x - 1) # 남서(SW)
    w_ = get(y + 0, x - 1) # 서(W)
    nw = get(y - 1, x - 1) # 북서(NW)
    neighbor_states = [n_, ne, e_, se, s_, sw, w_, nw]
    count = 0
    for state in neighbor_states:
        if state == ALIVE:
            count += 1
    return count

def game_logic(state, neighbors):
    if state == ALIVE:
        if neighbors < 2:
            return EMPTY # 살아 있는 이웃이 너무 적음: 죽음
        elif neighbors > 3:
            return EMPTY # 살아 있는 이웃이 너무 많음: 죽음
    else:
        if neighbors == 3:
            return ALIVE # 다시 생성됨
    return state

def step_cell(y, x, get, set):
    state = get(y, x)
    neighbors = count_neighbors(y, x, get)
    next_state = game_logic(state, neighbors)
    set(y, x, next_state)

def simulate(grid):
    next_grid = Grid(grid.height, grid.width)
    for y in range(grid.height):
        for x in range(grid.width):
            step_cell(y, x, grid.get, next_grid.set)
    return next_grid

class ColumnPrinter:
    def __init__(self):
        self.columns = []

    def append(self, data):
        self.columns.append(data)

    def __str__(self):
        row_count = 1
        for data in self.columns:
            row_count = max(
                row_count, len(data.splitlines()) + 1)

        rows = [''] * row_count
        for j in range(row_count):
            for i, data in enumerate(self.columns):
                line = data.splitlines()[max(0, j - 1)]
                if j == 0:
                    padding = ' ' * (len(line) // 2)
                    rows[j] += padding + str(i) + padding
                else:
                    rows[j] += line

                if (i + 1) < len(self.columns):
                    rows[j] += ' | '

        return '\n'.join(rows)

columns = ColumnPrinter()
for i in range(5):
    columns.append(str(grid))
    grid = simulate(grid)

print(columns)

>>>
---*-----
----*----
--***----
---------
---------

    0     |     1     |     2     |     3     |     4    
---*----- | --------- | --------- | --------- | ---------
----*---- | --*-*---- | ----*---- | ---*----- | ----*----
--***---- | ---**---- | --*-*---- | ----**--- | -----*---
--------- | ---*----- | ---**---- | ---**---- | ---***---
--------- | --------- | --------- | --------- | ---------
```

단일 코어에서 한 스레드로 실행되는 프로그램의 경우에는 이런 코드가 잘 작동한다. 하지만 프로그램의 요구 사항이 바뀌어서 이제는 앞에서 살짝 암시한 바와 같이, game_logic 함수 안에서 약간의 I/O(소켓 통신 등)가 필요하다고 하자. 예를 들어 그리드 상태와 인터넷을 통한 사용자 간 통신에 의해 상태 전이가 결정되는 MMOG(Massively Multiplayer Online Game)가 이런 경우다.

해결책은 I/O를 병렬로 수행해서 그리드 크기와 관계없이 각 세대를 대략 100밀리초 안에 계산할 수 있게 만드는 것이다. 각 작업 단위에 대해 동시 실행되는 여러 실행 흐름을 만들어내는 과정을 팬아웃(fan-out)이라고 한다. 전체를 조율하는 프로세스 안에서 다음 단계로 진행하기 전에 동시 작업 단위의 작업이 모두 끝날 때까지 기다리는 과정을 팬인(fan-in)이라고 한다.

**기억해야 할 내용**
- 프로그램이 커지면서 범위와 복잡도가 증가함에 따라 동시에 실행되는 여러 실행 흐름이 필요해지는 경우가 많다.
- 동시성을 조율하는 가장 일반적인 방법으로는 팬아웃(새로운 동시성 단위들을 만들어냄)과 팬인(기존 동시성 단위들의 실행이 끝나기를 기다림)이 있다.
- 파이썬은 팬아웃과 팬인을 구현하는 다앙한 방법을 재공한다.