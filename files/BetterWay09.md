## Better way 9. for나 while 루프 뒤에 else 블록을 사용하지 말라

파이썬에서는 루프가 반복 수행하는 내부 블록 바로 다음에 else 블록을 추가할 수 있다.

```python
for i in range(3):
    print('Loop', i)
else:
    print('Else block!')

>>>
Loop 0
Loop 1
Loop 2
Else block!
```

놀랍게도 else 블록은 루프가 끝나자마자 실행된다. if/else, try/except, try/finally를 배운 프로그래머는 for/else의 else부분을 '루프가 정상적으로 완료되지 않으면 이 블록을 실행하라'는 뜻으로 가정하기 쉽다. 하지만 실제로는 완전히 반대로 행동한다. 

루프 안에서 break문을 만나 루프를 빠져나왔을 때 else블록 실행되지 않음, 빈 시퀀스에 대해 루프를 실행하면 else블록이 바로 실행됨, while 루프의 조건이 처음부터 False인 경우(루프가 한 번도 실행되지 못하는 경우)에도 else 블록이 바로 실행됨.

**기억해야 할 내용**
- 파이썬은 for나 while 루프에 속한 블록 바로 뒤에 else 블록을 허용하는 특별한 문법을 제공한다.
- 루프 뒤에 오는 else블록은 루프가 반복되는 도중 break를 만나지 않은 경우에만 실행된다.
- 동작이 직관적이지 않고 혼동을 야기할 수 있으므로 루프 뒤에 else 블록을 사용하지 말라.