This is a repo summarizing Effective Python 2nd (파이썬 코딩의 기술 개정2판) in Korean.

작성일자: 21.09.16

# Contents

## 1장. 파이썬답게 생각하기
[Better way 1. 사용 중인 파이썬의 버전을 알아두라][way1]  
[Better way 2. PEP 8 스타일 가이드를 따르라][way2]  
[Better way 3. bytes와 str의 차이를 알아두라][way3]  
[Better way 4. C스타일 형식 문자열을 str.format과 쓰기보다는 f-문자열을 통한 인터폴레이션을 사용하라][way4]  
[Better way 5. 복잡한 식을 쓰는 대신 도우미 함수를 작성하라][way5]  
[Better way 6. 인덱스를 사용하는 대신 대입을 사용해 데이터를 언패킹하라][way6]  
[Better way 7. range보다는 enumerate를 사용하라][way7]  
[Better way 8. 여러 이터레이터에 대해 나란히 루프를 수행하려면 zip을 사용하라][way8]  
[Better way 9. for나 while 루프 뒤에 else 블록을 사용하지 말라][way9]  
[Better way 10. 대입식을 사용해 반복을 피하라][way10]

## 2장. 리스트와 딕셔너리
[Better way 11. 시퀀스를 슬라이싱하는 방법을 익혀라][way11]  
[Better way 12. 스트라이드와 슬라이스를 한 식에 함께 사용하지 말라][way12]  
[Better way 13. 슬라이싱보다는 나머지를 모두 잡아내는 언패킹을 사용하라][way13]  
[Better way 14. 복잡한 기준을 사용해 정렬할 때는 key 파라미터를 사용하라][way14]  
[Better way 15. 딕셔너리 삽입 순서에 의존할 때는 조심하라][way15]  
[Better way 16. in을 사용하고 딕셔너리 키가 없을 때 KeyError를 처리하기보다는, get을 사용하라][way16]  
[Better way 17. 내부 상태에서 원소가 없는 경우를 처리할 때는 setdefault보다 defaultdict를 사용하라][way17]  
[Better way 18. \_\_missing\_\_을 사용해 키에 따라 다른 디폴트 값을 생성하는 방법을 알아두라][way18]

## 3장. 함수
[Better way 19. 함수가 여러 값을 반환하는 경우 절대로 네 값 이상을 언패킹하지 말라][way19]  
[Better way 20. None을 반환하기보다는 예외를 발생시켜라][way20]  
[Better way 21. 변수 영역과 클로저의 상호작용 방식을 이해하라][way21]  
[Better way 22. 변수 위치 인자를 사용해 시각적인 잡음을 줄여라][way22]  
[Better way 23. 키워드 인자로 선택적인 기능을 제공하라][way23]  
[Better way 24. None과 독스트링을 사용해 동적인 디폴트 인자를 지정하라][way24]  
[Better way 25. 위치로만 인자를 지정하게 하거나 키워드로만 인자를 지정하게 해서 함수 호출을 명확하게 만들라][way25]  
[Better way 26. functools.wrap을 사용해 함수 데코레이터를 정의하라][way26]

## 4장. 컴프리헨션과 제너레이터
[Better way 27. map과 filter 대신 컴프리헨션을 사용하라][way27]  
[Better way 28. 컴프리헨션 내부에 제어 하위 식을 세 개 이상 사용하지 말라][way28]  
[Better way 29. 대입식을 사용해 컴프리헨션 안에서 반복 작업을 피하라][way29]  
[Better way 30. 리스트를 반환하기보다는 제너레이터를 사용하라][way30]  
[Better way 31. 인자에 대해 이터레이션할 때는 방어적이 돼라][way31]  
[Better way 32. 긴 리스트 컴프리헨션보다는 제너레이터 식을 사용하라][way32]  
[Better way 33. yield from을 사용해 여러 제너레이터를 합성하라][way33]  
[Better way 34. send로 제너레이터에 데이터를 주입하지 말라][way34]  
[Better way 35. 제너레이터 안에서 throw로 상태를 변화시키지 말라][way35]  
[Better way 36. 이터레이터나 제너레이터를 다룰 때는 itertools를 사용하라][way36]  

## 5장. 클래스와 인터페이스
[Better way 37. 내장 타입을 여러 단계로 내포시키기보다는 클래스를 합성하라][way37]  
[Better way 38. 간단한 인터페이스의 경우 클래스 대신 함수를 받아라][way38]  
[Better way 39. 객체를 제너릭하게 구성하려면 @classmethod를 통한 다형성을 활용하라][way39]  
[Better way 40. super로 부모 클래스를 초기화하라][way40]  
[Better way 41. 기능을 합성할 때는 믹스인 클래스를 사용하라][way41]  
[Better way 42. 비공개 애트리뷰트보다는 공개 애트리뷰트를 사용하라][way42]  
[Better way 43. 커스텀 컨테이너 타입은 collections.abc를 상속하라][way43]

## 6장. 메타클래스와 애트리뷰트
[Better way 44. 세터와 게터 메서드 대신 평범한 에트리뷰트를 사용하라][way44]  
[Better way 45. 애트리뷰트를 리팩터링하는 대신 @property를 사용하라][way45]  
[Better way 46. 재사용 가능한 @property 메서드를 만들려면 디스크립터를 사용하라][way46]  
[Better way 47. 지연 계산 애트리뷰트가 필요하다면 \_\_getattr\_\_, \_\_getattribute\_\_, \_\_setattr\_\_을 사용하라][way47]  
[Better way 48. \_\_init_subclass\_\_를 사용해 하위 클래스를 검증하라][way48]  
[Better way 49. \_\_init_subclass\_\_를 사용해 클래스 확장을 등록하라][way49]  
[Better way 50. \_\_set_name\_\_으로 클래스 애트리뷰트를 표시하라][way50]
[Better way 51. 합성 가능한 클래스 확장이 필요하면 메타클래스보다는 클래스 데코레이터를 사용하라][way51]  

## 7장. 동시성과 병렬성
[Better way 52. 자식 프로세스를 관리하기 위해 subprocess를 사용하라][way52]  
[Better way 53. 블로킹 I/O의 경우 스레드를 사용하고 병렬성을 피하라][way53]  
[Better way 54. 스레드에서 데이터 경합을 피하기 위해 Lock을 사용하라][way54]  
[Better way 55. Queue를 사용해 스레드 사이의 작업을 조율하라][way55]  
[Better way 56. 언제 동시성이 필요한지 인식하는 방법을 알아두라][way56]  
[Better way 57. 요구에 따라 팬아웃을 진행하려면 새로운 스레드를 생성하지 말라][way57]  
[Better way 58. 동시성과 Queue를 사용하기 위해 코드를 어떻게 리팩터링해야 하는지 이해하라][way58]  

## 8장. 강건성과 성능
[Better way 65. Try/ecxept/else/finally의 각 블록을 잘 활용하라][way65]  
[Better way 66. 재사용 가능한 try/finally 동작을 원한다면 contextlib과 with문을 사용하라][way66]  
[Better way 67. 지역 시간에는 time보다는 datetime을 사용하라][way67]  
[Better way 68. copyreg를 사용해 pickle을 더 신뢰성 있게 만들라][way68]  
[Better way 69. 정확도가 매우 중요한 경우에는 decimal을 사용하라][way69]  
[Better way 70. 최적화하기 전에 프로파일링을 하라][way70]  
[Better way 71. 생산자-소비자 큐로 deque를 사용하라][way71]  
[Better way 72. 정렬된 시퀀스를 검색할 때는 bisect를 사용하라][way72]  
[Better way 73. 우선순위 큐로 heapq를 사용하는 방법을 알아두라][way73]  




[way1]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay01.md
[way2]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay02.md
[way3]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay03.md
[way4]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay04.md
[way5]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay05.md
[way6]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay06.md
[way7]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay07.md
[way8]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay08.md
[way9]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay09.md
[way10]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay10.md
[way11]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay11.md
[way12]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay12.md
[way13]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay13.md
[way14]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay14.md
[way15]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay15.md
[way16]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay16.md
[way17]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay17.md
[way18]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay18.md
[way19]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay19.md
[way20]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay20.md
[way21]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay21.md
[way22]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay22.md
[way23]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay23.md
[way24]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay24.md
[way25]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay25.md
[way26]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay26.md
[way27]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay27.md
[way28]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay28.md
[way29]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay29.md
[way30]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay30.md
[way31]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay31.md
[way32]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay32.md
[way33]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay33.md
[way34]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay34.md
[way35]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay35.md
[way36]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay36.md
[way37]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay37.md
[way38]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay38.md
[way39]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay39.md
[way40]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay40.md
[way41]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay41.md
[way42]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay42.md
[way43]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay43.md
[way44]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay44.md
[way45]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay45.md
[way46]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay46.md
[way47]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay47.md
[way48]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay48.md
[way49]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay49.md
[way50]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay50.md
[way51]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay51.md
[way52]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay52.md
[way53]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay53.md
[way54]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay54.md
[way55]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay55.md
[way56]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay56.md
[way57]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay57.md
[way58]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay58.md
[way65]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay65.md
[way66]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay66.md
[way67]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay67.md
[way68]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay68.md
[way69]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay69.md
[way70]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay70.md
[way71]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay71.md
[way72]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay72.md
[way73]:https://github.com/wcshin-git/Effective-Python-2nd/blob/master/files/BetterWay73.md
