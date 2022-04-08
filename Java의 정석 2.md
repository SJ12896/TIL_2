## Java의 정석 2

### Chapter 10 날짜와 시간 & 형식화

#### 1.1 Calendar와 Date

- (java.util)Date, Calendar: 날짜와 시간 다루는 클래스
  - Calendar: 추상클래스로 직접 객채 생성 불가능. Calendar.getInstance()로 인스턴스를 얻어 사용
  - Calendar가 추가되며 Date는 대부분 메서드가 deprecated 되었지만 여전히 필요로 하는 메서드들 존재
- java.time패키지: 기존 단점들을 개선해 추가된 클래스들
- get(Calendar.MONTH) 범위는 0~11이므로 0이 1월을 의미한다. 따라서 set할때도 숫자로 입력하면 -1을 해서 적는다. 아니면 Calendar.AUGUST처럼 글자로 적어준다.
- 두 날짜간의 차이를 얻으려면 getTimeMillis()로 천분의 일초 단위로 변환한다. 계산후 다시 1000으로 나눈다.
- add나 roll을 사용해 일정기간만큼 증가된 날짜를 얻을 수 있다. 다만 roll은 날짜가 증가해도 다른 필드에 영향을 주지 않는다. 다만 일 필드가 말일인데 월필드를 변경하면 영향이 간다.



#### 2. 형식화 클래스

- java.text 패키지에 포함. 숫자, 날짜, 텍스트 데이터를 일정 형식에 맞게 표한할 수 있도록 했다.
- 숫자 형식화: DecimalFormat
- 날짜 형식화: SimpleDateFormat
- 특정 범위에 속하는 값 문자열 변환: ChoiceFormat
- 데이터를 정해진 양식에 맞게 출력: MessageFormat