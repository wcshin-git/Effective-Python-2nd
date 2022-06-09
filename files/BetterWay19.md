## Better way 19. 함수가 여러 값을 반환하는 경우 절대로 네 값 이상을 언패킹하지 말라

**기억해야 할 내용**
- 함수가 여러 값을 반환하기 위해 값들을 튜플에 넣어서 반환하고, 호출하는 쪽에서는 파이썬 언패킹 구문을 쓸 수 있다.
- 함수가 반환한 여러 값을 처리하는 별표 식을 사용해 언패킹할 수도 있다.
- 언패킹 구문에 변수가 네 개 이상 나오면 실수하기 쉬우므로 변수를 네 개 이상 사용하면 안 된다. 대신 작은 클래스를 반환하거나 namedtuple 인스턴스를 반환하라.