## Better way 16. in을 사용하고 딕셔너리 키가 없을 때 KeyError를 처리하기보다는, get을 사용하라

get의 두 번째 인자는 첫 번째 인자인 키가 딕셔너리에 들어 있지 않을 때 돌려줄 디폴트 값이다. 

```python
count = counters.get(key, 0)
counters[key] = count + 1
```

딕셔너리에 저장된 값이 리스트철머 더 복잡한 값이라면 어떻게 해야 할까?

```python
votes = {
    '바게트': ['철수', '순이'],
    '치아바타': ['하니', '유리'],
}
key = '브리오슈'
who = '단이'

# 개선 전
if key in votes:
    names = votes[key]
else:
    votes[key] = names = []

names.append(who)

# 개선 후
if (names := votes.get(key)) is None:
    votes[key] = names = []

names.append(who)

# 참고: setdefault 사용시
names = votes.setdefault(key, [])  # 단점: votesd에 key가 있든 없든 []를 먼저 만들어놓아야 함. 따라서 defaultdict를 써라.
names.append(who)
```

**기억해야할 내용**
- 딕셔너리 키가 없는 경우를 처리하는 방법으로는 in 식을 사용하는 방법, KeyError 예외를 사용하는 방법, get 메서드를 사용하는 방법, setdefault를 사용하는 방법이 있다.
- 카운터와 같이 기본적인 타입의 값이 들어가는 딕셔너리를 다룰 때는 get 메서드가 가장 좋고, 딕셔너리에 넣을 값을 만드는 비용이 비싸거나 만드는 과정에 예외가 발생할 수 있는 경우에도 get 메서드를 사용하는 편이 낫다.
- 해결하려는 문제에 dict의 setdefault 메서드를 사용하는 방법이 가장 적합해 보인다면 setdefault 대신 defaultdict를 사용할지 고려해보라.