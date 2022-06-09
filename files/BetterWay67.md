## Better way 67. 지역 시간에는 time보다는 datetime을 사용하라

파이썬에서 시간대를 변환하는 방법은 두 가지이다. 왜 datetime이 최선이고 time을 피해야 하는지 이해하려면 time과 datetime을 제대로 알아야 한다.

```python
import time

now = 1598523184 # UTC기준, 유닉스 기준 시간으로부터 몇 초가 흘렀는지
local_tuple = time.localtime(now) # time.struct_time(tm_year=2020, tm_mon=8, tm_mday=27, tm_hour=19, tm_min=13, tm_sec=4, tm_wday=3, tm_yday=240, tm_isdst=0)
time_format = '%Y-%m-%d %H:%M:%S'
time_str = time.strftime(time_format, local_tuple)  # 2020-08-27 19:13:04
print(time_str)

>>>
2020-08-27 19:13:04
```

```python
time_tuple = time.strptime(time_str, time_format) # time.struct_time(tm_year=2020, tm_mon=8, tm_mday=27, tm_hour=19, tm_min=13, tm_sec=4, tm_wday=3, tm_yday=240, tm_isdst=-1)
utc_now = time.mktime(time_tuple)
print(utc_now)

>>>
1598523184.0
```

```python
parse_format = '%Y-%m-%d %H:%M:%S %Z'    # %Z는 시간대를 뜻함
depart_icn = '2020-08-27 19:13:04 KST'
time_tuple = time.strptime(depart_icn, parse_format) # time.struct_time(tm_year=2020, tm_mon=8, tm_mday=27, tm_hour=19, tm_min=13, tm_sec=4, tm_wday=3, tm_yday=240, tm_isdst=0)
time_str = time.strftime(time_format, time_tuple)
print(time_str)

>>>
2020-08-27 19:13:04
```

KST로 설정한 시간이 제대로 작동하므로 다른 시간대로 잘 처리할 것으로 예상할 수 있다. 하지만 실제로는 그렇지 않다. strptime에 PDT(미국 태평양 시간대이며 샌프란시스코가 속한 캘리포니아 등에서 사용함)를 사용하면 다음과 같은 오류가 난다.

```python
arrival_sfo = '2020-08-28 04:13:04 PDT'
# 오류가 나는 부분. 오류를 보고 싶으면 커멘트를 해제할것
time_tuple = time.strptime(arrival_sfo, parse_format)

>>>
ValueError: time data '2020-08-28 04:13:04 PDT' does not match format '%Y-%m-%d %H:%M:%S %Z'
```

여기서 문제는 time 모듈이 플랫폼에 따라 다르게 작동한다는 데 있다. time 모듈의 동작은 호스트 운영체제의 C함수가 어떻게 작동하는지에 따라 달라진다. 이로 인해 파이썬에서는 time 모듈의 동작을 신뢰할 수 없다. 따라서 여러 시간대 사이의 변환을 다룬다면 datetime모듈을 사용하라.

다음 코드는 UTC로 된 시간을 컴퓨터의 지역 시간인 KST로 바꾼다.

```python
from datetime import datetime, timezone

now = datetime(2020, 8, 27, 10, 13, 4)      # 시간대 설정이 안된 시간을 만듦. datetime.datetime(2020, 8, 27, 10, 13, 4)
now_utc = now.replace(tzinfo=timezone.utc)   # 시간대를 UTC로 강제 지정. datetime.datetime(2020, 8, 27, 10, 13, 4, tzinfo=datetime.timezone.utc)
now_local = now_utc.astimezone()             # UTC 시간을 디폴트 시간대로 변환. datetime.datetime(2020, 8, 27, 19, 13, 4, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400), 'KST'))
print(now_local)

>>>
2020-08-27 19:13:04+09:00
```

또한 datetime 모듈을 사용하면 지역 시간을 UTC로 된 유닉스 타임스탬프로 쉽게 바꿀 수 있다. 

```python
time_format = '%Y-%m-%d %H:%M:%S'
time_str = '2020-08-27 19:13:04'
now = datetime.strptime(time_str, time_format)   # 시간대 설정이 안된 시간으로 문자열을 구문분석. datetime.datetime(2020, 8, 27, 19, 13, 4)
time_tuple = now.timetuple()                     # 유닉스 시간 구조체로 변환. time.struct_time(tm_year=2020, tm_mon=8, tm_mday=27, tm_hour=19, tm_min=13, tm_sec=4, tm_wday=3, tm_yday=240, tm_isdst=-1)
utc_now = time.mktime(time_tuple)                # 구조체로부터 유닉스 타임스탬프 생성
print(utc_now)

>>>
1598523184.0
```

datetime 모듈은 한 지역 시간을 다른 지역 시간으로 바꾸는 신뢰할 수 있는 기능을 제공한다. 하지만 파이썬 기본 설치에는 UTC를 제외한 시간대 정의가 들어 있지 않다.

다행히 PyPI에서 pytz 모듈을 내려받아 기본  설치가 제공하지 않는 시간대 정보를 추가할 수 있다.

pytz를 효과적으로 사용하려면 항상 지역 시간을 UTC로 먼저 바꿔야 한다. 그 후 오프셋 연산을 수행하고, 마지막으로 다시 UTC를 지역 시간으로 바꿔라.

예를 들어 다음 코드는 샌프란시스코의 비행기 도착 시간을 UTC datetime으로 변경한다. 

```python
import pytz

arrival_sfo = '2020-08-28 04:13:04'
time_format = '%Y-%m-%d %H:%M:%S'
sfo_dt_naive = datetime.strptime(arrival_sfo, time_format)   # 시간대가 설정되지 않은 시간. datetime.datetime(2020, 8, 28, 4, 13, 4)
eastern = pytz.timezone('US/Pacific')                        # 샌프란시스코의 시간대. <DstTzInfo 'US/Pacific' LMT-1 day, 16:07:00 STD>
sfo_dt = eastern.localize(sfo_dt_naive)                      # 시간대를 샌프란시스코 시간대로 변경. datetime.datetime(2020, 8, 28, 4, 13, 4, tzinfo=<DstTzInfo 'US/Pacific' PDT-1 day, 17:00:00 DST>)
utc_dt = pytz.utc.normalize(sfo_dt.astimezone(pytz.utc))     # UTC로 변경
# sfo_dt.astimezone(pytz.utc) => datetime.datetime(2020, 8, 28, 11, 13, 4, tzinfo=<UTC>)
# utc_dt => datetime.datetime(2020, 8, 28, 11, 13, 4, tzinfo=<UTC>)
print(utc_dt)

>>>
2020-08-28 11:13:04+00:00
```

UTC datetime을 얻었으니, 이를 한국 지역 시간으로 변환할 수 있다. 

```python
korea = pytz.timezone('Asia/Seoul')
korea_dt = korea.normalize(utc_dt.astimezone(korea))
# utc_dt.astimezone(korea) => datetime.datetime(2020, 8, 28, 20, 13, 4, tzinfo=<DstTzInfo 'Asia/Seoul' KST+9:00:00 STD>)
# korea_dt => datetime.datetime(2020, 8, 28, 20, 13, 4, tzinfo=<DstTzInfo 'Asia/Seoul' KST+9:00:00 STD>)
print(korea_dt)

>>>
2020-08-28 20:13:04+09:00
```

**기억해야 할 내용**
- 여러 다른 시간대를 변환할 때는 time 모듈을 쓰지 말라.
- 여러 다른 시간대를 신뢰할 수 있게 변환하고 싶으면 datetime과 pytz 모듈을 함께 사용하라.
- 항상 시간을 UTC로 표시하고, 최종적으로 표현하기 직전에 지역 시간으로 변환하라.