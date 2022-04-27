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



#### 1.3 LinkedList

- 배열은 가장 기본적인 형태의 자료구조로 데이터 읽는데 걸리는 시간이 가장 빠르지만 크기를 변경할 수 없고 큰 배열을 생성하면 메모리가 낭비되며 데이터 추가 삭제에 시간이 많이 걸리는 단점을 가지고 있다. 
- 링크드 리스트: 불연속적으로 존재하는 데이터를 서로 연결한 형태. 다음 요소 접근은 쉽지만 이전 요소 접근은 어려워 이를 보완한 **더블 링크드 리스트**도 있다. 일반 링크드 리스트보다 많이 사용된다. 따라서 링크드 리스트 클래스 역시 더블 링크드 리스트로 구현되어 있다.
- 더블 써큘러 링크드 리스트(이중 원형 연결리스트, doubly circular linked list): 더블 링크드 리스트의 접근성 보다 향상. 더블 링크드 리스트의 첫 요소와 마지막 요소를 연결시키면 된다.
- 순차적으로 추가/삭제하는 경우에는 ArrayList가 더 빠르다. 또한 get(i)로 요소 값을 얻어올 때 배열은 각 요소가 메모리상에 연속적으로 존재해 주소를 곧바로 읽어 값을 가져오지만 링크드리스트는 처음부터 i번째 데이터까지 차례대로 따라가야 한다. 그래서 저장할 개수가 많아질수록 읽어오는 시간이 길어진다.
- 두 클래스를 조합해 사용할 수도 있다. 데이터를 저장할 때 ArrayList를 사용하고 작업할 때 LinkedList로 옮기면 좋은 효율을 얻을 수 있다.



#### 1.4 Stack과 Queue

- 자바에서 Stack은 클래스를 구현해 제공하지만 Queue는 인터페이스만 정의해두었기 때문에 이를 구현한 클래스를 선택해 사용한다. LinkedList가 해당한다.
- 스택의 활용: 워드 undo/redo, 웹브라우저 뒤로/앞으로
- 큐의 활용: 인쇄대기목록, 버퍼
- PriorityQueue: 저장한 순서에 관계없이 우선순위가 높은 것부터 꺼낸다. null은 저장 불가능. 저장 공간으로 배열을 사용하며 각 요소를 힙이라는 자료구조 형태로 저장한다. (힙은 이진트리의 한 종류로 가장 큰 값이나 가장 작은 값을 빠르게 찾을 수 있다.) 숫자의 경우엔 작을수록 우선순위가 높아 가장 먼저 꺼내진다. 객체를 저장할 경우 객체의 크기를 비교할 수 있는 방법을 제공해야 한다.
- Deque: 양쪽 끝으로 추가/삭제가 가능해 stack, queue를 하나로 합친 것과 같다.



#### 1.5 Iterator, ListIterator, Enumeration

- 컬렉션에 저장된 요소를 접근하는데 사용하는 인터페이스. Enumeration이 Iterator의 구버전이고 Listiterator가 기능이 향상된 버전이다.
- Iterator: 컬렉션에 저장된 요소들을 읽어오는 방법 표준화. 컬렉션 요소에 접근하는 기능을 가진 Iterator인터페이스를 정의하고 Collection인터페이스에서는 Iterator를 반환하는 iterator()를 정의하고 있다.
- Iterator인터페이스의 메서드
  - boolean hasNext(): 읽어올 요소가 남아있는지 확인
  - Object next(): 다음 요소 읽어오기
  - void remove(): next()로 읽어온 요소 삭제. 선택적 기능으로 Iterator인터페이스를 구현하는 크랠스에서 구현하지 않아도 된다. 메서드 몸통에서 throw new UnsupportedOperationException(); 을 발생시킨다. 단순히 빈 몸통을 유지하는 것보다 호출하는쪽에서 기능이 동작하지 않는 이유를 빨리 알 수 있게 한다.
- Iterator로 컬렉션 요소 읽어오는 방법 표준화, **공통 인터페이스 정의로 표준 정의 & 구현해 코드 일관성 유지, 재사용성 극대화**
- ArrayList선언시 참조변수를 Collection으로 하면 Collection인터페이스를 구현한 다른 클래스, LinkedList같은 클래스로 바꾸려면 선언문 하나만 변경하면 된다. 
- Map인터페이스를 구현한 컬렉션 클래스는 쌍으로 저장하기 때문에 keySet()이나 entrySet()과 같은 메서드로 각각 Set형태로 얻어온 후 다시 iterator()를 호출해야 얻을 수 있다.

```java
Iterator it = map.entrySet().iterator();

// 유사한 코드. append가 StringBuffer를 return하기에 가능하다.
StringBuffer sb = new StringBuffer();
sb.append("a").append("b").append("c");
```



- Enumeration: 컬렉션 프레임웍이 만들어지기 전에 사용하던것. 구버전
- ListIterator: Iterator를 상속받아 기능 추가. 양방향 이동 가능. 다만 List인터페이스를 구현한 컬렉션만 사용.



#### 1.6 Arrays

- 배열을 다루는데 유용한 메서드 정의.
- setAll(): 배열을 채우는데 사용할 함수형 인터페이스를 구현한 객체나 람다식을 매개변수로 지정해야 한다.
- binarySearch(): 저장된 요소 검색. 반드시 배열이 정렬된 상태여야 한다. 일치하는 요소가 여러개라면 어떤 것의 위치가 반환될지 알 수 없다.
- deepToString(): 다차원 배열의 모든 요소 문자열로 출력
- deepEquals(): 다차원 배열의 비교.equlas()로 다차원 배열을 비교하면 저장된 내용이 같아도 false를 얻는데, 배열에 저장된 배열의 주소를 비교하게 되기 때문이다.
- asList(): 배열을 List에 담아서 반환. 매개변수 타입이 가변인수라서 배열 생성 없이 저장할 요소만 담아도 된다. 주의할 점은 asList()가 반환한 List의 크기는 변경할 수 없다. 추가, 삭제는 불가능하지만 저장된 내용 변경은 가능하다.
- parallelXXX(): parallel로 시작하는 메서드는 빠른 결과를 얻기 위해 여러 쓰레드가 작업을 나누어 처리하게 한다.
- spliterator(): 여러 쓰레드가 처리하도록 하나의 작업을 여러 작업으로 나누는 Spliterator를 반환한다.
- stream(): 컬렉션을 스트림으로 변환한다.



#### 1.7 Comparator와 Comparable

- Arrays.sort()는 Character클래스의 Comparable의 구현에 의해 정렬되는 것이다. 
- Comparable, Comparator는 **인터페이스**로, 컬렉션을 정렬하는데 필요한 메서드를 정의하고 있다.
- Comparable을 구현한 클래스: 같은 타입 인스턴스끼리 서로 비교할 수 있는 클래스들, Integer같은 wrapper클래스와 String, Date, File같은 것들. 기본적으로 **오름차순**으로 정렬된다. 내림차순 또는 **다른 기준으로 정렬**하고 싶다면 **Comparator를 구현**해서 정렬기준을 제공할 수 있다.

```java
// Comparator와 Comparable의 실제 소스
// compare과 compareTo는 두 객체가 같으면 0, 비교하는 값보다 작으면 음수, 크면 양수를 반환한다.

public interface Comparator {
    int compare(Object o1, Object o2);
    boolean equals(Object obj);
}

public interface Comparable {
    public int compareTo(Object o);
}


// 일반 문자열 오름차순 정렬은 공백, 숫자, 대문자, 소문자 순이다. 문자 유니코드 순서에 따라 정렬된다.
// 다른 기준
Arrays.sort(strArr, String.CASE_INSENSITIVE_ORDER); // 대소문자 구분안하고 정렬
Arrays.sort(strArr, new Descending()); // 역순 정렬

// 내림차순 정렬. compare의 매개변수가 Object 타입이기 때문에 compareTo를 바로 호출할 수 없어 먼저 Comparable로 형변환을 한다.
class Descending implements Comparator {
    public int compare(Object o1, Object o2) {
        if (o1 instanceof Comparable && o2 instanceof Comparable) {
            Comparable c1 = (Comparable)o1;
            Comparable c2 = (Comparable)o2;
            return c1.compareTo(c2) * -1; // 또는 c2.compareTo(c1)
        }
        return -1;
    }
}
```



#### 1.8 HashSet

- Set인터페이스를 구현한 가장 대표적인 컬렉션. add나 addAll 메서드를 사용해 새로운 요소를 추가하는데 이미 저장된 요소라면 false를 반환한다. 저장순서를 유지하지 않기 때문에 유지하길 원한다면 LinkedHashSet을 사용해야한다.
- HastSet()에 초기용량을 설정한 HashSet객체를 생성할 수 있는데 이 때 float loadFactor를 설정할 수도 있다. 이는 컬렉션 클래스에 저장공간이 가득차기 전 미리 용량을 확보하기 위한 것으로 0.8로 지정했다면 80%가 채워졌을 때 용량이 두 배로 늘어난다. 기본값은 0.75다.
- 새로운 클래스를 만들어 HashSet에 해당객체를 추가할 때 두 인스턴스가 같은지 제대로 인식하게 하려면 add메서드가 새로운 요소 추가 전 호출하는 equals()와 hashCode()를 목적에 맞게 오버라이딩 해야 한다.

```java
class Person {
    String name;
    int age;
    
    Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    boolean equals(Object obj) {
        if (obj instanceof Person) {
            Person tmp = (Person) obj;
            return name.equals(tmp.name) && age == tmp.age;
        }  
        return false;
    }
    
    public int hashCode() {
        return (name+age).hashCode();
        // JDK 1.8부터 추가된 java.util.Objects클래스의 hash()를 이용해도 된다. 이 쪽을 추천
        // Objects.hash(name, age);
    }
    
    public String toString() {
        return name + ":" + age;
    }
}
```

- 오버라이딩된 hashCode()는 세 조건을 만족시켜야 한다.
  - 동일한 객체에 여러번 호출해도 동일한 int를 반환한다. Object클래스는 객체 주소로 해시코드를 만들어 실행할 때마다 해시코드값이 달라질 수 있다.
  - equals()로 true를 얻은 두 객체에 대해 각각 hashCode()를 호출해서 얻은 결과는 반드시 같아야한다.
  - equals()호출 시 false를 반환하는 두 객체는 hashCode()를 호출했을 때 같은 int를 반환하는 경우가 있어도 괜찮지만 해싱을 사용하는 컬렉션의 성능을 향상시키기 위해 다른 int를 반환하는게 좋다.



#### 1.9 TreeSet

- 이진 검색 트리라는 자료구조 형태로 데이터를 저장하는 컬렉션 클래스. 정렬, 검색, 범위검색에 높은 성능을 보인다.
- TreeSet은 이진 검색 트리 성능을 향상시킨 **레드-블랙 트리**로 구현되어 있다.
- Set인터페이스를 구현했으므로 중복된 데이터 저장X, 정렬된 위치에 저장되므로 저장순서 유지X. 정렬된 상태를 유지하므로 검색, 범위 검색이 빠르다. 하지만 저장위치를 찾아서 저장하고 삭제하면 트리 일부를 재구성해야해서 링크드 리스트보다 추가, 삭제 시간이 더 걸린다.

```java
class TreeNode {
    TreeNode left;
    Object element;
    TreeNode right;
}
```

- 왼쪽 자식노드는 부모노드보다 작은 값, 오른쪽 자식 노드에는 부모 노드보다 큰 값을 저장한다.
- TreeSet에 저장되는 객체가 Comparable을 구현하던가 Comparator를 제공해 두 객체를 비교할 방법을 알려줘야 한다.
- 메서드
  - Object ceiling(Object o): 같은 객체 반환. 없으면 큰 값 중 가장 가까운 객체 반환. 없으면 null
  - Comparator comparator(): 정렬기준을 반환
  - NavigableSet descendingSet(): TreeSet에 저장된 요소들을 역순으로 정렬해서 반환
  - Object floor(Object o): 같은 객체 반환. 없으면 작은 값 중 가장 가까운 객체 반환. 없으면 null
  - SortedSet headSet(Object toElement, boolean inclusive): 지정된 객체보다 작은 값의 객체들 반환. inclusive가 true면 같은 값도 포함
  - Object higher(Object o) / lower(Object o): 큰 값 / 작은 값 중 가장 가까운 객체 반환. 없으면 null
  - Object pollFirst() / pollLast(): 첫 요소 / 마지막 요소 반환
  - boolean retainAll(Collection c): 컬렉션과 공통된 요소만 남기고 삭제(교집합)
  - SortedSet subSet(Object fromElement, Object toElement): 범위 검색 결과 반환(끝범위 포함X), from Inclusive와 toInclusive로 조정가능
  - SortedSet tailSet(Object fromElement): 지정 객체보다 큰 값의 객체들 반환
  - Object[] toArray(): 지정된 객체를 객체배열로 반환
- 문자열 정렬순서는 코드값이 기준: 공백, 숫자, 대문자, 소문자으로 정렬된다.



#### 1.10 HashMap과 Hashtable

- Vector와 ArrayList처럼 구-신 버전 관계다. 마찬가지로 새로운 버전인 HashMap사용을 권장한다.
- 해싱을 사용해 많은 양 데이터 검색에 뛰어나다. 
- 실제소스를 살펴보면 HashMap은 Entry라는 내부 클래스를 정의해 key, value를 가진다. 그리고 Entry타입의 배열을 선언해 서로 관련된 값인 키와 값을 각각 배열이 아닌 하나의 클래스로 정의해 하나의 배열로 다룬다. (객체지향적인 코드)
- HashMap은 키, 값을 각각 Object 타입으로 저장한다. 따라서 어떤 객체도 저장할 수 있지만 키는 주로 String대문자나 소문자를 통일해서 사용한다.
- Set entrySet(): HashMap에 저장된 키와 값을 엔트리(키-값의 결합) 형태로 Set에 저장해서 반환
- Hashtable은 키, 값으로 null을 허용하지 않지만 HashMap은 허용
- 해싱: 해시함수를 이용해 데이터를 해시테이블에 저장하고 검색하는 기법. 해시함수는 데이터가 저장되어있는 곳을 알려줘 원하는 데이터를 빠르게 찾을 수 있다. 해싱에서 사용하는 자료구조는 **배열과 링크드 리스트의 조합**으로 되어 있다. 저장할 데이터 키를 해시 함수에 넣으면 배열의 한 요소를 얻게 되고 다시 그곳에 연결된 링크드 리스트에 저장하게 된다. 다만 링크드 리스트는 크기가 커질수록 검색에 시간이 걸리므로 많은 서랍에 하나의 데이터만 저장되어있을 때 빨라진다. 반면 배열은 크기가 커져도 원하는 요소가 몇번째에 있는지만 알면 배열 인덱스가 n인 요소 주소 = 배열의 시작주소 + type의 size * n이라는 공식에 의해 빠르게 찾을 수 있다.
- 해싱을 구현한 컬렉션 클래스는 Object클래스에 정의된 hashCode()를 해시함수로 사용한다. 객체의 주소를 이용해 알고리즘으로 해시코드를 만들어낸다. String클래스는 Object의 hashCode()를 오버라이딩해 문자열 내용으로 해시코드를 만들어낸다. 그래서 같은 내용이면 같은 해시코드를 얻는다.
- 새로운 클래스를 정의할 때 equals()를 재정의 오버라이딩해야하면 hashCode()도 재정의해서 equals가 true면 hashCode()도 결과값이 같게 해줘야한다.그렇지 않으면 다른 객체로 인식한다.



#### 1.11 TreeMap

- 이진검색트리 형태로 키-값 쌍 데이터를 저장해 검색과 정렬에 적합하다.
- 검색은 대부분 HashMap이 TreeMap보다 뛰어나지만 범위검색, 정렬은 TreeMap을 사용하자.



#### 1.12 Properties

- HashMap 구버전인 Hashtable을 상속받아 구현. (String, String)형태로 저장하는 단순화된 컬렉션 클래스
- 주로 애플리케이션 환경설정과 관련된 속성을 저장하는데 사용하며 데이터를 파일로부터 읽고 쓰는 편리한 기능 제공. 
- 메서드
  - String getProperty(String key, String defaultValue): 지정된 키의 값 반환. 디폴트 값 설정도 가능
  - void list(PrintStream/PrintWriter out): 지정된 PrintStream/PrintWriter에 저장된 목록 출력
  - void load(InputStream inStream / Reader reader): 지정된 InputStream/Reader로부터 목록을 읽어서 저장
  - Enumeration propertyNames(): 목록의 모든 키가 담긴 Enumeration 반환(컬렉션 프레임웤 이전의 구버전이라 Iterator를 사용하지 않는다.)
  - Object setProperty(String key, String value): 지정된 키와 값 저장. 기존에 같은 키로 저장된 값이 있으면 그 값을 Object로 반환하며 아니면 null반환
  - void store(OutptStream out, String comments): 지정된 목록을 지정된 OutputStream에 출력(저장). comments는 목록에 대한 주석.



#### 1.13 Collections

- 컬렉션과 관련된 메서드 제공.
- 멀티 쓰레드 프로그래밍은 하나의 객체를 여러 쓰레드가 동시에 접근할 수 있어 데이터 일관성 유지를 위해 공유되는 객체에 **동기화**가 필요하다. Vector, Hashtable같은 구버전은 자체적으로 동기화 처리가 되어있지만 멀티쓰레드가 아닌 경우 성능을 떨어뜨리는 요인이 된다.
- 새로 추가된 ArrayList, HashMap같은 컬렉션은 필요한 경우에 java.util.Collections클래스의 동기화 메서드를 이용해 처리한다.

```java
static Collection synchronizedCollection(Collection c)
```

- 컬렉션에 저장된 데이터를 보호하기 위해 읽기전용으로 만들어야 할 때가 있다. 멀티 쓰레드 프로그래밍에서 여러 쓰레드가 하나의 컬렉션을 공유하면 데이터가 손상될 수 있는데 이를 방지할 때 사용한다.

```java
static Collection unmodifiableCollection(Collection c)
```

- 단 하나의 객체만 저장하는 싱글톤 컬렉션: 매개변수로 저장할 요소를 지정하면 해당 요소를 저장하는 컬렉션을 반환한다.

```java
static List singletonList(Object o)
static Set singleton(Object o) // singletonSet 아님
```

- 컬렉션에 지정된 종류 객체만 저장할 수 있도록 제한하고 싶을 때

```java
static Collection checkedCollection(Collection c, Class type)
```



## Chapter 12 지네릭스, 열거형, 애너테이션 generics, enumeration, annotaion

### 1. 지네릭스(Generics)

- 다양한 타입의 객체들을 다루는 메서드, 컬렉션 클래스 컴파일 시 **타입 체크**를 해주는 기능. 객체 타입 안정성 높이고(의도하지 않은 타입의 객체가 저장되는 것 막고 원래와 다른 타입으로 잘못 형변환되어 발생하는 오류 줄여줌) 형변환의 번거로움이 줄어든다. 또 타입체크와 형변환을 생략할 수 있어 코드가 간결해진다.
- ArrayList와 같은 컬렉션 클래스는 다양한 종류의 객체를 담을 수 있지만 보통 한 종류의 객체를 담는다. 그런데 꺼낼때마다 타입체크를 하고 형변환을 하면 불편하고 원하지 않는 종류 객체가 포함되는 걸 막을 수 없다.
- 지네릭 타입은 **클래스**와 **메서드**에 선언할 수 있다.

```java
// 지네릭 타입으로 변경하기 전
class Box {
    Object item;
    
    void setItem(Object item) { this.item = item; }
    Object getItem() { return item; }
}

// 클래스에 선언하는 지네릭 타입. 클래스 옆에 <T>를 붙이고 Object를 모두 T로 바꾼다.
class Box<T> {
    T item;
    
    void setItem(T item) { this.item = item; }
    T getItem() { return item; }
}
```

- T: 타입 변수(type variable). 하지만 타입 변수는 T가 아닌 다른 문자를 사용해도 된다. E를 사용한 경우 element의 첫 글자에서 따왔다. 타입 변수가 여러개면 Map<K, V>처럼 콤마를 구분자로 나열한다. 기호가 다를 뿐 **임의의 참조형 타입**을 의미하는 것은 같다. 기존에 Object 타입을 사용해 형변환이 불가피했지만 이젠 원하는 타입을 지정하기만 하면 된다.

```java
// 지네릭 클래스인 Box 객체 생성. 사용될 실제 타입을 지정한다. 이를 지네릭 타입 호출이라고 함.
// 지정된 타입 String은 매개변수화된 타입이라고 한다.
Box<String> b = new Box<String>();
b.setItem(new Object()); // String이외는 불가하므로 에러
b.setItem("ABC");
String item = b.getItem(); // 형변환 불필요
```

- 지네릭 도입되기 이전과의 호환을 위해 사용될 실제 타입을 지정하지 않고 new Box(); 처럼 기존 방식으로 객체를 생성하는 것이 허용되지만 경고가 발생한다. Box<Object>를 지정하면 지정하지 않은 것이 아니라 알고 적은 것이라 경고가 발생하지 않는다.
- Box<T>: 지네릭 클래스. T의 Box 또는 T Box라고 읽는다.
- T: 타입변수, 타입 매개변수
- Box: 원시 타입. 컴파일 후 원시 타입으로 바뀐다.

- 지네릭스의 제한
  - 모든 객체에 동일하게 동작해야 하는 static멤버에 타입 변수 T를 사용할 수 없다. T는 인스턴스 변수로 간주된다. static 멤버는 대입된 타입의 종류에 관계없이 동일한 것이어야 하기 때문이다. 
  - 지네릭 타입의 배열을 생성하는 것도 허용되지 않는다. 지네릭 배열 타입의 참조변수 선언은 가능하지만 new T[10]처럼 배열 생성은 안된다. new연산자는 컴파일 시점에 타입 T를 정확히 알아야하는데 클래스 컴파일 당시에는 알 수 없다. newInstance()와 같이 동적으로 객체를 생성하는 메서드를 사용하거나 Object배열을 생성하고 복사해 T[]로 형변환하는 방법 등을 사용한다.



#### 1.3 지네릭 클래스의 객체 생성과 사용

```java
class Box<T> {
    ArrayList<T> list = new ArrayList<T>();
    
    // 만들어진 객체에 사용할 때 대입된 타입과 다른 타입 객체는 추가할 수 없지만 참조변수가 Fruit라면 Apple같은 자손들은 이 메서드 매개변수가 될 수 있다.
    void add(T item)         { list.add(item); }
    T get(int i)             { return list.get(i); }
    ArrayList<T> getList()   { return list; }
    int size()               { return list.size(); }
    public String toString() { return list.toString(); }
}
```

- 지네릭 클래스의 객체를 생성할 때 참조변수와 생성자에 대입된 타입이 일치하지 않으면 상속관계여도 에러가 발생한다. 
- 단, 두 지네릭 클래스 **타입**이 상속관계에 있고 **대입된 타입**이 같은 것은 괜찮다.

```java
Box<Apple> appleBox = new FruitBox<Apple>();
```

- JDK 1.7부터 추정이 가능한 경우 타입 생략이 가능하다. 참조변수 타입에 나와있어 생성자에 반복해서 지정해주지 않아도 되는 것이다.

```java
Box<Apple> appleBox = new Box<>();
```



#### 1.4 제한된 지네릭 클래스

- 지네릭 타입에 extends를 사용하면 특정 타입의 자손들만 대입할 수 있게 제한할 수 있다. 인터페이스를 구현해야할 때도 implements가 아닌 extends를 사용하며 인터페이스와 클래스를 동시에 구현하면 &를 사용한다.

```java
class FruitBox<T extends Fruit & Eatable> {
    ...
}
```



#### 1.5 와일드 카드

```java
class Juicer {
    // 매개변수 타입을 고정해두면 다른 매개변수가 올 수 없어 오버로딩을 해야하지만 지네릭 타입은 컴파일할 때만 사용하고 제거되어 지네릭 타입이 다르다고 오버로딩이 성립하지 않아 메서드 중복 정의가 된다.
    static Juice makeJuice(FruitBox<Fruit> box) {  
        String tmp = "";
        for(Fruit f : box.getList()) temp += f + " ";
        return new Juice(tmp);
    }
}
```

- 이럴 때 사용하는 게 **와일드 카드**다. 기호 ?로 표현하며 어떤 타입도 될 수 있다. &를 사용할 수 없다.
  - <? extends T>: 상한 제한. T와 그 자손들만 가능
  - <? super T>: 하한 제한. T와 그 조상들만 가능
  - <?>: 제한 없이 모든 타입 가능. <? extends Object>와 동일
- 따라서 앞선 makeJuice 메서드 매개변수를 FruitBox<? extends Fruit>로 바꿀 수 있다. 만약 extends Object로 바꾼다면 아래 for문의 Fruit객체가 보장되지 않는다. 하지만 FruitBox 클래스 자체가 이미 <T extends Fruit>처럼 사용되어 FruitBox는 Fruit의 자손이란걸 알고 있어 에러가 발생하지 않는다.
- comparator에 와일드카드를 사용하지 않으면 T에 Apple을 대입했을 경우, Grape를 대입했을 경우 마다 각각 Comparator가 필요하지만 조상 Comparator를 가능하게 하면 Comparator<Fruit>나 Comparator<Object>가 가능하다. 이런 장점 때문에 Comparator에는 항상 <? super T>가 습관적으로 따라 붙는다.

```java
static <T> void sort(List<T> list, Comparator<? super T> c)
```



#### 1.6 지네릭 메서드

- 메서드 선언부에 지네릭 타입이 선언된 메서드. 지네릭 타입 선언 위치는 반환 타입 바로 앞이다.
- 지네릭 클래스에 정의된 타입 매개변수와 지네릭 메서드에 정의된 타입 매개변수는 **전혀 별개의 것**이다.
- 지네릭 메서드는 지네릭 클래스가 아닌 클래스에도 정의될 수 있다.
- static 멤버는 타입 매개변수를 사용할 수 없지만 메서드에 지네릭 타입을 선언하고 사용하는 것은 가능하다.

```java
// 위에 나온 makeJuice()를 지네릭 메서드로 바꾸면
    static <T extends Fruit> Juice makeJuice(FruitBox<T> box) {  
        String tmp = "";
        for(Fruit f : box.getList()) temp += f + " ";
        return new Juice(tmp);
    }

// 메서드 호출할 때 이렇게 해야하지만
Juicer.<Fruit>makeJuice(fruitBox)
    
// fruitBox 선언부에서 컴파일러가 추정가능하기 때문에 생략 가능하다.
Juicer.makeJuice(fruitBox)
```



- 매개변수 타입이 복잡할 때 유용하다.

```java
public static void printAll(ArrayList<? extends product> list,
                           ArrayList<? extends product> list2) {
    for (Unit u: list) {
        System.out.println(u);
    }
}

// 지네릭 메서드
public static <T extends Product> void printAll(ArrayList<T> list,
                                                ArrayList<T> list2) {
    for (Unit u: list) {
        System.out.println(u);
    }
}
```



#### 1.7 지네릭 타입의 형변환

- 지네릭 타입과 넌지네릭 타입간의 형변환은 경고가 발생하지만 가능하다. 하지만 대입된 타입이 다른 지네릭 타입간의 형변환은 불가능하다. 
- Box<? extends Object> wBox = new Box<String>(); 는 매개변수에 다형성이 적용되어 형변환 가능하다.
- 그 외 내용은 다시 읽어보고 이해하기



#### 1.8 지네릭 타입의 제거

- 컴파일러는 지네릭 타입으로 소스파일을 체크하고 필요한 곳에 형변환을 넣어준 다음 지네릭 타입을 제거한다. 이전 소스와의 호환성을 위해서다. 
- <T extends Fruit>는 Fruit가 되고 <T>인 경우는 Object가 된다.
- 지네릭 타입 제거 후 타입이 일치하지 않으면 형변환을 추가한다.
