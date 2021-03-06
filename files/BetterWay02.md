## Better way 2. PEP 8 스타일 가이드를 따르라

PEP 8은 파이썬 코드를 어떤 형식으로 작성할지 알려주는 스타일 가이드이다.

다음은 여러분이 꼭 따라야 하는 규칙이다.

**공백**

다음에서 설명하는 공백은 탭, 스페이스, 새 줄(new line) 등의 문자를 모두 합한 말이다.

- 탭 대신 스페이스를 사용해 들여쓰기
- 문법적으로 중요한 들여쓰기에는 4칸 스페이스
- 라인 길이는 79개 문자 이하(한글 한 글자는 시각적으로 영문 두 글자에 해당한다고 계산하기)
- 긴 식을 다음 줄에 이어서 쓸 경우에는 일반적인 들여쓰기보다 4 스페이스를 더 덜여쓰기
- 파일 안에서 각 함수와 클래스 사이에는 빈 줄을 두 줄 넣기
- 클래스 안에서 메서드와 메서드 사이에는 빈 줄을 한 줄 넣기
- 딕셔너리에서 키와 콜론(:) 사이에는 공백을 넣지 않고, 한 줄 안에 키와 값을 같이 넣는 경우에는 콜론 다음에 스페이스를 하나 넣는다.
- 변수 대입에서 = 전후에는 스페이스를 하나씩만 넣는다.
- 타입 표기를 덧붙이는 경우에는 변수 이름과 클론 사이에 공백을 넣지 않도록 주의하고, 콜론과 타입 정보 사이에는 스페이스를 하나 넣어라.

**명명 규약**

PEP 8은 파이썬 언어의 여러 부분에 사용하는 이름을 어떻게 붙일지에 대한 고유 스타일을 제공.

- 함수, 변수, 애트리뷰트(Attribute; 은 클래스 내부에 포함돼 있는 메서드나 변수를 의미)는 lowercase_underscore처럼 소문자와 밑줄을 사용한다.
- 보호돼야 하는 인스턴스 애트리뷰트는 일반적인 애트리뷰트 이름 규칙을 따르되, _leading_underscore처럼 밑줄로 시작한다.
- 비공개(한 클래스 안에서만 쓰이고 다른 곳에서는 쓰면 안 되는 경우) 인스턴스 애트리뷰트는 일반적인 애트리뷰트 이름 규칙을 따르되, __leading_underscore처럼 밑줄 두 개로 시작한다. 
참고: [언더스코어 하나와 두개의 차이](https://www.datacamp.com/community/tutorials/role-underscore-python)
- 클래스는 CapitalizedWord처럼 여러 단어를 이어 붙이되, 각 단어의 첫 글자를 대문자로 만든다.
- 모듈 수준의 상수는 ALL_CAPS처럼 모든 글자를 대문자로 하고 단어와 단어 사이를 밑줄로 연결한 형태를 사용한다.
- 클래스에 들어 있는 인스턴스 메서드는 호출 대상 객체를 가리키는 첫 번째 인자의 이름으로 반드시 self를 사용해야 한다.
- 클래스 메서드는 클래스를 가리키는 첫 번째 인자의 이름으로 반드시 cls를 사용해야 한다.
참고: [Static Method, Class Method](https://wikidocs.net/21054)

**식과 문**

- 긍정적인 식을 부정하지 말고(if not a is b) 부정을 내부에 넣어라(if a is not b).
- 빈 컨테이너나 시퀀스([]나 ''등)를 검사할 때는 길이를 0과 비교(if len(something) == 0)하지 말라. 빈 컨테이너나 시퀀스 값이 암묵적으로 False로 취급된다는 사실을 활용해 'if not 컨테이너'라는 조건문을 써라.
- 마찬가지로 비어 있지 않은 컨테이너나 시퀀스([1, 5]이나 'hi' 등)를 검사할 때도 길이가 0보다 큰지 비교하지 말라. 대신 if 컨테이너가 비어 있지 않은 경우 암묵적으로 True로 평가된다는 사실을 활용하라.
- 한 줄짜리 if문이나 한 줄짜리 for, while 루프, 한 줄짜리 except 복합문을 사용하지 말라. 명확성을 위해 각 부분을 여러 줄에 나눠 배치하라.
- 식을 한 줄 안에 다 쓸 수 없는 경우, 식을 괄호로 둘러싸고 줄바꿈과 들여쓰기를 추가해서 읽기 쉽게 만들라.
- 여러 줄에 걸쳐 식을 쓸 때는 줄이 계속된다는 표시를 하는 \ 문자보다는 괄호를 사용하라.

**임포트**

- import 문을 항상 파일 맨 앞에 위치시켜라.
- 모듈을 임포트할 때는 절대적인 이름을 사용하고, 현 모듈의 경로에 상대적인 이름은 사용하지 말라. 예를 들어 bar 패키지로부터 foo 모듈을 임포트한다면 from bar import foo라고 해야 하며, 단지 import foo라고 하면 안 된다.
- 반드시 상대적인 경로로 임포트해야 하는 경우에는 from . import foo처럼 명시적인 구문을 사용하라.
- 임포트를 적을 때는 표준 라이브러리 모듈, 서드 파티 모듈, 여러분이 만든 모듈 순서로 섹션을 나눠라. 각 섹션에서는 알파벳 순서로 모듈을 임포트하라. 
참고: [모듈과 패키지](https://offbyone.tistory.com/106)

**기억해야 할 내용**
- 파이썬 코드를 작성할 때는 항상 파이썬 개선 제안 (PEP 8) 스타일 가이드를 따르라.
- 큰 파이썬 커뮤니티와 공통된 스타일을 공유하면 다른 사람과 협력할 때 도움이 된다.
- 일관성 있는 스타일을 사용하면 나중에 자신이 작성한 코드를 직접 수정할 때도 더 수월해진다.