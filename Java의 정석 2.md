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



### Chapter 11 컬렉션 프레임웍

- 데이터 군을 저장하는 클래스들을 표준화한 설계. JDK 1.2부터 등장. 그 전에는 컬렉션 클래스와 다수의 데이터를 저장할 수 있는 클래스를 각자 방식으로 처리해야 했다.
- 컬렉션: 다수의 데이터(데이터 그룹) / 프레임웍: 표준화된 프로그래밍 방식



#### 1.1 컬렉션 프레임웍의 핵심 인터페이스

- 컬렉션데이터 그룹을 크게 3가지 타입으로 인식해 각 컬렉션을 다루는데 필요한 기능을 가진 **3개의 인터페이스 정의**. 인터페이스 List, Set의 공통 부분을 다시 뽑아 Collection라는 새로운 인터페이스 추가
  - List와 Set을 구현한 컬렉션 클래스들은 공통부분이 많아 Collection을 정의할 수 있었지만 Map은 다른 형태로 컬렉션을 다룬다.
  - JDK 1.5부터 Iterable인터페이스 추가, 이를 Collection이 상속받게 변경됐지만 공통 메서드 iterator()를 뽑아 중복을 제거하기 위한 것에 불과하다.

- List: 순서O, 중복O, ArrayList, LinkedList, Stack, Vector 등
- Set: 순서X, 중복X, HashSet, TreeSet등
- Map: 키와 값 쌍으로 이루어진 데이터 집합. 순서X, 키 중복X, 값 중복O, HashMap, TreeMap, Hashtable, Properties등
- 컬렉션 프레임웍의 모든 클래스는 List, Set, Map 중 하나를 구현하고 있고 구현한 인터페이스 이름이 클래스 이름에 포함되어 있다. 
  - Vector, Stack, Hashtable, Properties 같은 클래스들은 컬렉션 프레임웍 이전부터 존재해 명명법을 따르지 않는다. 이런 기존 컬렉션 클래스들은 호환을 위해 설계를 변경해 남겨뒀지만 사용하지 않는 편이 좋다. 새로 추가된 ArrayList나 HashMap을 사용하자.
- Map인터페이스의 메서드에서 값을 반환하는 values() 메서드는 중복을 허용해 Collection 타입으로 반환하고 keySet()은 키 값이 중복을 허용하지 않아 Set타입으로 반환한다. 
- Map.Entry인터페이스는 Map인터페이스의 내부 인터페이스로 key-value쌍을 다루기 위해 내부적으로 정의해둔 것이다. 



#### 1.2 ArrayList

- 컬렉션 프레임웍에서 가장 많이 사용되는 컬렉션 클래스
- Object 배열을 이용해 데이터를 순차적으로 저장하는데 더 이상 저장할 공간이 없으면 보다 큰 새로운 배열을 생성해 기존에 저장된 내용을 복사한 뒤 저장한다.
- ArrayList 내부에서 Object 배열을 멤버변수로 선언하고 있어 모든 종류의 객체를 담을 수 있다.
- 메서드
  - ArrayList(): 크기가 10인 ArrayList생성
  - ArrayList(Collection c): 주어진 컬렉션이 저장된 ArrayList 생성
  - ArrayList(int initialCpacity): 지정된 초기용량 갖는 ArrayList 생성
  - boolean addAll(int index, Collection c): 주어진 컬렉션의 모든 객체를 지정 위치부터 저장(위치 필수X)
  - void ensureCapacity(int minCapacity): ArrayList 용량이 최소한 minCpacity가 되도록 한다.
  - boolean retainAll(Collection c): 컬렉션과 공통된 것들만 남기고 나머지 삭제
  - int size(): 저장된 객체 개수 반환
  - void sort(Comparator c): 정렬기준으로 정렬
  - void trimToSize(): 용량을 크기에 맞게 줄인다.
  - Collections.sort(ArrayList array): Collections 클래스의 sort 메서드를 이용해 정렬할 수 있다.
  - 한 ArrayList에서 다른 ArrayList와 공통된 요소를 삭제하고 싶을 때 for문 안에서 remove메서드를 사용한다. 하지만 for문의 변수 i가 0부터 시작하는게 아니라 list2.size()-1부터 감소하게 되는데 증가시키면서 삭제할 경우 한 요소가 삭제되면 빈 공간을 채우기 위해 나머지가 자리이동을 해 올바른 결과를 얻을 수 없다.
  - 용량(capacity)은 ArrayList에서 요소가 들어갈 수 있는 총 공간을 의미하고 크기(size)는 실제로 무언가 들어가 있는 것의 개수를 의미한다. Vector에서는 capacity가 부족하면 자동으로 2배로 증가한 새 인스턴스를 생성한다.
  - System.arraycopy(array1, startIndex, array2, elementCount): array1의 startIndex에서 array2로 elementCount개의 데이터를 복사한다. remove 메서드를 구현할 때 삭제할 요소가 맨 마지막에 위치하지 않았다면 중간에서 삭제한 후 남은 부분을 복사하기 위해 사용한다.