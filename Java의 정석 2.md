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



### 2. 열거형(enums)

- 서로 관련된 상수를 편리하게 선언하기 위한 것. 여러 상수를 정의할 때 사용하면 유용하다. JDK 1.5부터 새로 추가되었다. 열거형이 갖는 값뿐만 아니라 타입도 관리해 논리적인 오류를 줄인다. `타입에 안전한 열거형(typesafe enum)`이라서 실제 값이 같아도 타입이 다르면 컴파일 에러가 발생한다.  또 상수 값이 바껴도 기존의 소스를 다시 컴파일하지 않아도 된다.

```java
class Card {
    enum Kind { CLOVER, HEART, DIAMOND, SPADE } // 열거형 Kind를 정의
    enum Value { TWO, THREE, FOUR } // 열거형 Value를 정의
    
    final Kind kind;   // 타입이 int가 아니라 Kind
    final Value value;
}

// 체크
if (Card.Kind.CLOVER == Card.Value.Two) // 컴파일 에러
```

- 열거형 상수간 비교는 ==을 사용가능하다. equals()가 아니라 그만큼 빠른 성능을 제공한다. 하지만 비교 연산자는 사용 불가능하고 compareTo()는 사용가능하다.(두 비교대상이 같으면 0, 왼쪽이 크면 양수, 오른쪽이 크면 음수)

```java
enum Direction { EAST, SOUTH, WEST, NORTH }

// switch문의 조건식에도 사용 가능
void move() {
    switch (dir) {
        case EAST: x++;    // Direction.EAST라고 쓰면 안된다. 상수 이름만 적는다.
            break;
        case WEST: x--;
            break;
        case SOUTH: y++;
            break;
        case NORTH: y--;
            break;
    }
}
```

- .values(): 열거형에 정의된 모든 상수를 배열에 담아 반환한다.
- .ordinal(): 모든 열거형 조상인 java.lang.Enum 클래스에 정의된 것으로 열거형 상수가 정의된 순서(0부터 시작)를 정수로 반환한다. 이 값은 내부적 용도로만 사용되기 위한 것으로 열거형 상수의 값으로 사용하지 않는 것이 좋다. 
- class<E> getDeclaringClass(): 열거형 Class 객체 반환
- name(): 열거형 상수 이름을 문자열로 반환
- T valueOf(Class<T> enumType, String name): 지정된 열거형에서 name과 일치하는 열거형 상수를 반환.

```java
// 열거형 상수의 이름으로 문자열 상수에 대한 참조를 얻게 해준다.
Direction d = Direction.valueOf("WEST");
System.out.println(Direction.WEST == Direction.valueOf("WEST")); // true
```

- 열거형 상수의 값이 불연속적이면 열거형 상수 이름 옆에 원하는 값을 ()와 함께 적어주면 된다. 그리고 지정된 값을 저장할 수 있는 인스턴스 변수와 생성자를 새로 추가해주어야 된다. 주의할 점은 `열거형 상수를 모두 정의한 후 다른 멤버를 추가한다. 끝에 ;도 잊지 않는다.`
- 필요하면 열거형 상수에 여러 값을 지정할 수 있으며 맞는 인스턴스 변수와 생성자도 새로 추가하면 된다.

```java
enum Direction {
    EAST(1), SOUTH(5), WEST(-1), NORTH(10); // 끝에 ;필요
    
    private final int value; // 정수 저장할 필드(인스턴스 변수) 추가
    Direction(int value) { this.value = value; } // 생성자 추가. 제어자가 private이기 때문에 외부에서 열거형 객체 생성 불가능.
    
    public int getValue() { return vlaue; }
}
```

- 열거형에 추상 메서드 추가하기

```java

enum Transportation {
    // 운송 수단 종류별로 상수를 정의하고 있다. 
    BUS(100) {
        int fare(int distance) { return distance * BASIC_FARE; }
    },
    TRAIN(150) { int fare(int distance) { return distance * BASIC_FARE; }},
    TRAIN(150) { int fare(int distance) { return distance * BASIC_FARE; }};
    
    // 운송수단 별로 요금을 계산하는 방식이 다르므로 추상 메서드를 선언 후 각 열거형 상수가 구현하게 했다.
    abstract int fare(int distance);
    
    // private이 아닌 protected로 설정해 각 상수에서 접근할 수 있게 만들었다.
    protected final int BASIC_FARE;
    
    Transportation(int basicFare) {
        BASIC_FARE = basicFare;
    }
}
```

- 열거형에 들어가는 상수 하나하나는 열거형의 객체이다. 클래스로 표현한다면

```java
class Direction {
    static final Direction EAST = new Direction("EAST");
    static final Direction SOUTH = new Direction("SOUTH");
    static final Direction WEST = new Direction("WEST");
    static final Direction NORTH = new Direction("NORTH");
    
    private String name;
    
    private Direction(String name) {
        this.name = name;
    }
}
```

- 모든 열거형은 추상 클래스 Enum의 자손이다.

```java
// 그냥 MyEnum<T>로 선언했다면 compareTo를 사용할 수 없다. 타입 T에 ordinal이 정의되어 있는지 확인할 수 없기 때문이다. 타입 T가 MyEnum<T>의 자손이어야 한다는 의미로 타입 T가 MyEnum의 자손이므로 ordinal이 정의되어있는건 분명해 형변환 없이도 에러가 나지 않는다. -> 좀 더 이해가 필요할듯
abstract class MyEnum<T extends MyEnum<T>> implements Comparable<T> {
    static int id = 0;
    
    int ordinal;
    String name;
    
    public int ordinal() { return ordinal; }
    
    MyEnum(String name) {
        this.name = name;
        ordinal = id++;
    }
    
    // Comparable 인터페이스를 구현해 열거형 상수간 비교가 가능하다.
    public int compareTo(T t) {
        return ordinal - t.ordinal();
    }
}
```



### 3. 애너테이션(annotation)

- 프로그램의 소스코드 안에 다른 프로그램을 위한 정보를 미리 약속된 형식으로 포함시킨 것.  주석처럼 프로그래밍 언어에 영향을 미치지 않으면서도 다른 프로그램에게 유용한 정보를 제공할 수 있다. JDK에서 기본적으로 제공하는 것과 다른 프로그램에서 제공하는 것들이 있는데 뭐든간에 `약속된 형식으로 정보를 제공하기만 하면 될 뿐이다.` JDK에서 제공하는 표준 애너테이션은 주로 컴파일러를 위한 것이다. 새로운 애너테이션을 정의할 때 사용하는 `메타 애너테이션`(애너테이션 적용대상(target)이나 유지기간(retention)등을 지정하는데 사용)도 있다.

- 표준 애너테이션

  - @Override: 컴파일러에게 오버라이딩하는 메서드라는 것을 알린다. 오버라이딩하면서 조상의 메서드 이름을 잘못 적으면 새로운 메서드로 인식해 오류가 발생하지 않는다. 그래서 @Override를 붙이면 조상에 있는지 확인해 없으면 오류가 발생한다.
  - @Deprecated: 앞으로 사용하지 않을 것을 권장하는 대상에 붙인다. 다른 것으로 대체되어 더 이상 사용하지 않을 것을 권한다. 이게 붙은 대상을 사용하는 코드를 작성하면 컴파일시 메세지가 나타난다.
  - @SuppressWarnings: 컴파일러의 특정 경고메시지가 나타나지 않게 해준다. 주로 deprecastion, unchecked(지네릭스 타입 지정X), rawtypes(지네릭스 사용X), vararags(가변인자 타입이 지네릭 타입) 같은 경고 메세지를 억제한다. 억제하려는 경고 메세지를 애너테이션 뒤의 괄호에 지정한다. 배열처럼 {}에 여러개를 지정할 수 있다. -Xlint 옵션으로 컴파일해 나오는 경고 내용 중 []안에 있는 것이 종류다.
  - @SafeVrags: 지네릭스 타입의 가변인자에 사용한다.(JDK 1.7) 메서드에 선언된 가변인자 타입이 non-reifiable일 경우 해당 메서드 선언과 호출에서 unchecked가 발생한다. 이 경고를 억제하기 위해 사용한다. static, final이 붙은 메서드와 생성자에만 붙일 수 있다. `오버라이드될 수 있는 메서드에는 사용할 수 없다.` 컴파일 후에도 제거되지 않는 타입이 reifiable타입이고 제거되는 타입이 non-reifiable이다. 지네릭 타입은 대부분 제거된다. SuppressWarnings는 메서드가 호출되는 곳에서도 애너테이션을 붙여야하지만 SafeVarargs는 호출되는 곳도 자동으로 억제되게 한다. 또 varargs경고는 억제할 수 없어 @SafeVarargs와 @SuppressWarnings("varargs")를 습관적으로 같이 사용하는게 좋다. - > 이 부분도 이해가 더 필요하다.
  - @FunctionalInterface: 함수형 인터페이스라는 것을 알린다(1.8) 함수형 인터페이스를 올바르게 선언했는지(예를 들어 추상 메서드가 하나여야 한다는 제약이 있다.) 확인해준다. 실수를 방지하게 붙이도록 하자.
  - @Native: native메서드에서 참조되는 상수 앞에 붙인다.(1.8) 네이티브 메서드는 JVM이 설치된 os의 메서드다. 보통 C언어로 작성되어 있는데 자바는 메서드 선언부만 정의하고 구현은 하지 않는다. 모든 클래스 조상인 Object의 메서드들은 대부분 네이티브 메서드다. 호출하는 방법은 자바 일반 메서드와 같지만 실제 호출되는 것은 os의 메서드다. 내용이 없어 자바에 정의된 네이티브 메서드와 os의 메서드를 연결해주는 작업이 추가로 필요하다. `JNI(Java Native Interface)`가 그 역할을 하는데 이 책에서는 다루지 않는다.
  - @Target: 애너테이션이 적용가능한 대상을 지정하는데 사용(메타) 지정가능한 애너테이션 적용대상 종류는 ANNOTATION_TYPE, CONSTRUCTOR, FIELD(멤버변수, enum상수, 기본형에 사용됨.), LOCAL_VARIABLE, METHOD, PACKAGE, PARAMETER, TYPE(클래스, 인터페이스, enum), TYPE_PARAMETER(타입 매개변수-1.8), TYPE_USE(타입이 사용되는 모든 곳,  참조형에 사용됨, 1.8)
  - @Documented: 애너테이션 정보가 javadoc으로 작성된 문서에 포함되게 한다.(메타) 기본 애너테이션 중 @Override, @SuppressWarnings를 제외하고는 모두 붙어 있다.
  - @Inherited: 애너테이션이 자손 클래스에 상속되도록 한다.(메타) 
  - @Retention: 애너테이션이 유지되는 범위 지정에 사용(메타) 유지정책으로 SOURCE(소스파일에만 존재, Override처럼 컴파일러가 사용하는 애너테이션의 유지정책.), CLASS(클래스 파일에 존재, 실행시에 사용불가, 기본값), RUNTIME(클래스 파일에 존재, 실행 시 사용가능, reflection을 통해 클래스 파일에 저장된 애너테이션 정보를 읽어서 처리 가능. @FunctionalInterface는 컴파일러가 체크하지만 실행 시에도 사용되므로 RUNTIME)
  - @Repeatable: 애너테이션을 반복해서 적용할 수 있게 한다.(1.8, 메타) 일반 애너테이션과 달리 같은 이름의 애너테이션이 여러 개가 하나의 대상에 적용될 수 있어 하나로 묶어 다룰 수 있는 애너테이션도 추가 정의해야한다.

  ```java
  @interface ToDos {  // 여러 개의 ToDo애너테이션을 담을 컨테이너 애너테이션
      ToDo[] value(); // 이름이 반드시 value
  }
  
  @Repeatable(ToDos.class)  // 과롷 안에 컨테이너 애너테이션을 지정
  @interface ToDo {
      String value();
  }
  ```



- @기호를 앞에 붙이는 걸 제외하면 인터페이스를 정의하는 것과 동일하게 정의한다. @Override는 애너테이션이고 Override는 애너테이션의 타입이다. 애너테이션 내에 선언된 메서드를 애너테이션 요소라고 한다. 애니테이션도 인터페이스처럼 상수를 정의할 수 있지만 디폴트 메서드는 정의할 수 없다. 메서드는 다른 애너테이션을 포함할 수 있다. 요소들은 반환값이 있고 매개변수는 없는 추상메서드 형태며 상속을 통해 구현하지 않아도 된다. 다만 애너테이션 적용시 이 요소들 값을 빠짐없이 지정해줘야 한다. 요소 이름도 같이 적어 지정하므로 순서는 상관없다. 기본값을 가질 수 있으며 null을 제외한 모든 리터럴이 가능하다.

```java
@interface TestInfo {
    // 요소가 하나고 이름이 value면 애너테이션을 적용할 때 요소 이름을 생략하고 값만 적어도 된다.(요소 타입이 배열이도 가능하다.)
    int count() defualt 1;
    String[] info() defualt {"aaa", "bbb"};  // 기본값이 여러개
    String testedBy();
}
```

- 모든 애너테이션 조상은 Annotation이지만 상속이 허용되지 않는다. 또 이건 애너테이션이 아닌 일반 인터페이스로 정의되어 있다. 따라서 모든 애너테이션 객체는 조상의 equals(), hashCode(), toString() 메서드를 호출할 수 있다.

- 마커 애너테이션: 값을 지정할 필요 없으면 요소를 정의하지 않아도 된다. Serializable이나 Cloneable인터페이스처럼 요소가 하나도 정의되지 않은 애너테이션이다.
- 애너테이션 요소의 규칙
  - 요소 타입: 기본형, String, enum, 애너테이션, Class
  - ()안에 매개변수 선언 불가능
  - 예외 선언 불가능
  - 요소를 타입 매개변수로 정의 불가능

- 클래스에 적용된 애너테이션을 실행시간에 얻으려면 

```java
// 클래스 객체를 의미하는 리터럴. 모든 클래스파일은 클래스로더에 의해 메모리에 올라갈 때 클래스 정보가 담긴 객체를 생성하는데 이 객체를 클래스 객체라고 한다. 이 객체를 참조할 때 클래스이름.class 형식 사용
// 클래스 객체는 해당 클래스에 대한 정보르 ㄹ모두 가지며 애너테이션 정보도 포함된다. 
// 클래스 객체의 getAnnotation()으로 메서드에 매개변수 정보를 얻고자하는 애너테이션을 지정하거나 getAnnotations()로 배열로 받는다. 
Class<AnnotationEx5> cls = AnnotationEx5.class;  
TestInfo anno = (TestInfo)cls.getAnnotation(TestInfo.class);
```



### Chapter 13 쓰레드

- 프로세스: 실행 중인 프로그램. 프로그램을 실행하려면 OS로부터 실행에 필요한 자원(메모리)을 할당받아 프로세스가 된다. 프로그램을 수행하는데 필요한 `데이터, 메모리 같은 자원과 쓰레드`로 구성되어 있다.
- 쓰레드: 프로세스의 자원을 이용해 실제로 작업을 수행하는 것
- 모든 프로세느는 하나 이상의 쓰레드가 존재하며 둘 이상의 쓰레드를 가진 프로세스를 `멀티쓰레드 프로세스`라고 한다. 하나의 프로세스가 가지는 쓰레드 개수는 제한되어 있지 않으나 쓰레드의 작업 수행에 개별적인 메모리 공간(호출스택)이 필요하므로 프로세스의 메모리 한계에 따라 생성할 수 있는 쓰레드 수가 결정된다. 실제로 프로세스의 메모리 한게에 다다를 정도로 많은 쓰레드를 생성하는 일은 없으니 걱정하지 않아도 된다.
- 대부분의 OS는 멀티태스킹을 지원하므로 여러 프로세스가 동시에 실행될 수 있다. 
- CPU의 코어가 한 번에 단 하나의 작업만 수행할 수 있으므로 동시 처리 작업 수는 코어의 개수와 일치한다. 쓰레드 개수는 늘 코어 수보다 많으므로 각 코어는 아주 짧은 시간 동안 여러 작업을 번갈아 수행해 여러 작업들이 모두 동시에 수행되는 것처럼 보이게 한다.
- 멀티쓰레딩의 장점
  - cpu 사용률 향상
  - 자원 보다 효율적으로 사용
  - 사용자에 대한 응답성 향상
  - 작업 분리로 코드 간결
- 여러 사용자에게 서비스 하는 서버 프로그램의 경우 하나의 서버 프로세스가 여러 쓰레드를 생성해 쓰레드와 사용자의 요청이 일대일로 처리되도록 프로그래밍 해야 한다. 만일 싱글쓰레드로 작성한다면 사용자 요청마다 새로운 프로세스를 생성해야 하는데 프로세스 생성은 쓰레드 생성보다 더 많은 시간과 메모리 공간이 필요해 많은 수의 사용자 요청을 서비스하기 어렵다.
  - 쓰레드는 가벼운 프로세스, 경량 프로세스(LWP, light-weight process)라고 부르기도 한다.
  - 멀티 쓰레드 프로세스는 여러 쓰레드가 같은 프로세스 내에서 자원을 공유하며 작업을 하므로 동기화(synchronization), 교착상태(deadlock, 두 쓰레드가 자원을 점유한 상태에서 서로 점유한 자원을 사용하려고 기다리느라 진행이 멈춘 상태)같은 문제들을 고려해야 한다.



#### 2. 쓰레드의 구현과 실행

- Thread 클래스 상속받는 방법: 다른 클래스를 상속받을 수 없다.

```java
class MyThread extends Thread {
    public void run() { } // Thread클래스의 run()을 오버라이딩
}
```

- Runnable 인터페이스를 구현하는 방법: 일반적으로 사용하는 방법. `재사용성`이 높고 `코드의 일관성`을 유지할 수 있어 `객체지향적인 방법`. 오직 run()만 정의된 간단한 인터페이스다. 이를 구현하기 위해선 추상메서드인 run()의 몸통만 만들어주면 된다.

```java
class MyThread implements Runnable {
    public void run() { } // Runnable 인터페이스의 run()을 구현
}
```



- Runnable 인터페이스를 구현한 경우는 `Runnable 인터페이스를 구현한 클래스의 인스턴스를 생성한 후 이 인스턴스를 Thread 클래스의 생성자의 매개변수로 제공`해야 한다. 실제 Thread클래스에서 인스턴스 변수로 Runnable 타입의 변수를 선언한 후 생성자로 Runnable 인터페이스를 구현한 인스턴스를 참조하도록 되어있다. 그리고 run()을 호출하면 참조변수를 통해 `Runnable 인터페이스를 구현한 인스턴스의 run()이 호출`된다. 상속을 통해 run()을 오버라이딩 하지 않고 외부에서 run()을 제공받게 된다. 
- 쓰레드는 start()로 호출해야만 실행된다. 호출 후 바로 실행되지 않고 실행대기 상태에 있다가 자신의 차례가 되어야 실행된다. 대기중인 쓰레드가 없다면 바로 실행. 한 번 실행이 종료된 쓰레드는 다시 실행할 수 없다. 하나의 쓰레드는 한 번의 start()만 호출할 수 있다. 따라서 한 번 더 수행해야 한다면 새로운 쓰레드를 생성해 start()를 호출해야 한다.

```java
class ThreadEX1 {
    public static void main(String args[]) {
        ThreadEX1_1 t1 = new ThreadEx1_1();  // Thread 자손 클래스의 인스턴스 생성
        
        // Thread t2 = new Thread(new ThreadEX1_2());
        Runnable r = new ThreadEX1_2();  // Runnable을 구현한 클래스의 인스턴스 생성
        Thread t2 = new Thread(r);  // 생성자 Thread(Runnable target)
        
        t1.start();
        t2.start();
    }
}

class ThreadEX1_1 extends Thread {
    public void run() {
        for (int i = 0; i < 5; i++) {
            // 조상 클래스 메서드를 직접 호출가능
            System.out.println(getName());
        }
    }
}

class ThreadEx1_2 implements Runnable {
    public void run() {
        for (int i = 0; i < 5; i++) {
            //  Thread의 static 메서드인 currentThread()를 호출해 쓰레드에 대한 참조를 얻어 와야만 호출이 가능
            System.out.println(Thread.currentThread().getName());
        }
    }
}
```

- 쓰레드의 이름 지정, 변경: 지정하지 않으면 Thread-번호의 형식으로 정해진다.
  - Thread(Runnable target, String name)
  - Thread(String name)
  - void setName(String name)



#### 3. start()와 run()

- 앞서 작성한 코드에서 쓰레드를 실행시킬 때 run()이 아닌 start()를 호출했다. run()은 생성된 쓰레드를 실행시키는 것이 아니라 단순히 클래스에 선언된 메서드를 호출하는 것일 뿐이다. start()는 새로운 쓰레드가 작업을 실행하는데 필요한 호출 스택을 생성하고 run()을 호출해 생성된 호출스택에 run()이 첫 번째로 올라가게 한다.
- 모든 쓰레드는 독립적인 작업을 수행하기 위해 자신만의 호출스택을 필요로 한다. 새로운 쓰레드를 생성하고 실행시킬 때마다 새로운 호출스택이 생성되고 쓰레드가 종료되면 호출스택이 소멸한다.
- 호출스택의 가장 위의 메서드가 현재 실행중인 메서드지만 쓰레드가 여러개가 되면 호출스택의 가장 위에 있어도 대기상태일수 있다.
- 스케줄러가 쓰레들의 우선순위를 고려해 실행순서와 시간을 정하고 쓰레드들은 스케줄에 따라 자신의 순서가 되면 지정 시간동안 작업을 수행한다.
- 쓰레드가 작업을 마치게되면 종료된 쓰레드는 호출스택이 비워지고 사라진다.
- main메서드 역시 쓰레드가 작업을 수행한다. 지금까지 main메서드가 수행을 마치면 프로그램이 종료되었지만 쓰레드가 여러개라면 종료되지 않는다. 실행 중인 사용자 쓰레드가 하나도 없을 때만 프로그램이 종료된다. 한 쓰레드가 예외가 발생해 종료되어도 다른 쓰레드의 실행에는 영향을 미치지 않는다.
- 쓰레드는 사용자 쓰레드(user thread)와 데몬 쓰레드(daemon thread) 두 종류가 있다.



#### 4. 싱글쓰레드와 멀티쓰레드

- 하나의 쓰레드로 작업을 할 때는 한 작업을 마친 후 다른 작업을 시작한다. 두 개의 쓰레드로 작업하는 경우에는 짧은 시간동안 2개의 쓰레드가 번갈아 가면서 작업을 수행해 동시에 두 작업이 처리되는 것과 같다. 
- 두 개의 쓰레드로 작업한 시간이 더 걸릴 때도 있는데 쓰레드 간의 `작업 전환(context switching)`에 시간이 걸리기 때문이다. 현재 진행 중인 작업 상태, 다음에 실행해야할 위치(PC, 프로그램 카운터)등의 정보를 저장하고 읽어 오는 시간이 소요된다. 그래서 단순 cpu를 사용하는 계산작업은 싱글쓰레드 프로그래밍이 효율적이다. 
- 싱글 코어일 경우 멀티쓰레드라도 하나의 코어가 작업하기 때문에 두 작업이 절대 겹치지 않지만 멀티 코어인 경우 동시에 두 쓰레드가 수행될 수 있어 두 작업이 겹치는 부분이 발생한다. 그래서 화면(console)이라는 자원을 두고 두 쓰레드가 경쟁하게 되는 것이다.
  - 여러 쓰레드가 여러 작업 동시에 진행: `병행(concurrent)`
  - 한 작업을 여러 쓰레드가 나눠서 처리: `병렬(parallel)`
- 쓰레드 실행 시간은 할 때마다 달라질 수 있는데 OS의 프로세스 스케줄러 영향을 받기 때문이다. 매 순간 프로세스에게 할당되는 실행시간이 일정하지 않아 쓰레드에게 할당되는 시간 역시 일정하지 않다. 쓰레드는 이런 불확실성을 가진다. 
- 자바는 OS독립적이지만 실제로 쓰레드처럼 종속적인 부분이 몇가지 있다.
- 두 쓰레드가 서로 다른 자원을 사용하는 작업은 싱글쓰레드 프로세스보다 멀티쓰레드 프로세스가 효율적이다. 예를 들어 사용자로부터 데이터를 입력받거나, 네트워크로 파일을 주고받거나, 프린터로 출력하는 작업 같이 `외부기기와의 입출력을 필요로 하는 경우`가 이에 해당한다. 사용자에게 데이터를 입력받기 위해 기다리는 구간에서 다른 일을 할 수 있어 cpu를 효율적으로 사용할 수 있다.



#### 5. 쓰레드의 우선순위

- 쓰레드는 우선순위(priority)라는 속성(멤버변수)을 가져서 우선순위 값에 따라 쓰레드가 얻는 실행시간이 달라진다. 예를 들면 파일 전송기능을 가지는 메신저는 파일 다운로드보다 채팅 내용 전송 쓰레드의 우선순위가 더 높아야 한다. 시각적인 부분, 사용자에게 빠르게 반응해야하는 작업은 우선순위가 높아야 한다.

- void setPriority(int newPriority): 쓰레드의 우선순위를 지정한 값으로 변경

  int getPriority(): 쓰레드의 우선순위 반환

- 쓰레드의 우선순위 범위는 1~10으로 숫자가 클수록 우선순위가 높다. 

- 쓰레드의 우선순위는 쓰레드를 생성한 쓰레드의 상속받으며 main쓰레드의 우선순위는 5다.

- 실험결과 싱글 코어에서는 우선순위가 높은 작업이 확실히 빨리 끝나지만 멀티코어인 경우 우선수위에 따른 차이가 거의 없다. 하지만 OS마다 다른 방식으로 스케쥴링하기 때문에 어떤 OS에서 실행하느냐에 따라 결과가 다를 수 있다. 특정 OS의 스케줄링 정책과 JVM 구현을 직접 확인해야 한다. 

- 차라리 쓰레드에 우선순위 부여 대신 작업에 우선순위를 두어 PriorityQueue에 저장하고 먼저 처리하게 하는 것이 나을 수 있다.



#### 6. 쓰레드 그룹

- 서로 관련된 쓰레드를 그룹으로 다루기 위한 것. 쓰레드 그룹에 다른 쓰레드 그룹을 포함시킬 수 있고 보안상 이유로 도입된 개념이다. 자신이 속한 쓰레드 그룹, 하위 쓰레드 그룹은 변경가능하나 다른 쓰레드 그룹의 쓰레드는 변경 불가능. 
- 메서드는 741page 참고
- 쓰레드 그룹과 쓰레드 생성자(쓰레드를 쓰레드 그룹에 포함시키려면 쓰레드 생성자 이용)
  - ThreadGroup(String name): 지정된 이름의 쓰레드 그룹 생성
  - ThreadGroup(ThreadGroup parent, String name): 지정된 쓰레드 그룹에 포함되는 새로운 쓰레드 그룹 생성
  - Thread(ThreadGroup group, String name)
  - Thread(ThreadGroup group, Runnable target)
  - Thread(ThreadGroup group, Runnable target, String name)
  - Thread(ThreadGroup group, Runnable target, String name, long stackSize)
- 모든 쓰레드는 반드시 쓰레드 그룹에 포함되어 있어야 하므로 그룹을 지정하지 않으면 자신을 생성한 쓰레드와 같은 그룹에 속한다.
- 자바 어플리케이션이 실행되면 JVM은 main, system이라는 쓰레드 그룹을 만든다. 가비지 컬렉션을 수행하는 Finalizer쓰레드 같은 건 system쓰레드 그룹에 속한다.
- 쓰레드그룹.list()를 호출해 쓰레드 그룹의 정보를 출력할 수 있다. 하위 쓰레드 그룹이나 쓰레드는 들여쓰기로 구분해서 보여준다.
- 그룹이름.setMaxPriority(int num)같은 경우는 쓰레드가 그룹에 추가되기 전에 호출되어야 하고 후에 속하게 된 쓰레드 그룹과 쓰레드는 여기에 영향을 받는다. 



#### 7. 데몬 쓰레드(daemon thread)

- 다른 일반쓰레드의 작업을 돕는 보조적인 역할 수행. 일반 쓰레드가 종료되면 강제적으로 자동종료된다. 예로는 가비지 컬렉터, 워드포로세서 자동저장, 화면자동갱신이 있다.
- 무한루프와 조건문을 이용해 실행 후 대기하다가 특정조건이 만족되면 작업을 수행하고 다시 대기한다.
- 일반 쓰레드의 작성방법, 실행방법과 같지만 쓰레드 생성 후 setDaemon(true)를 호출해야 한다. 
- getAllStackTraces()를 통해 실행중, 대기상태의 작업이 완료되지 않은 모든 쓰레드의 호출스택을 출력할 수 있는데 이를 통해 모든 쓰레드를 살펴보면 JVM이 가비지 컬렉션, 이벤트처리, 그래픽처리같은 보조작업을 위한 데몬 쓰레드를 자동적으로 생성한 것을 볼 수 있다. GUI를 가진 프로그램은 더 많은 데몬 쓰레드가 생성된다.



#### 8. 쓰레드의 실행제어

- 쓰레드 프로그래밍이 어려운 이유: 동기화(synchronization), 스케줄링(scheduling)

- 정교한 스케줄링을 통해 프로세스에게 주어진 자원, 시간을 여러 쓰레드가 낭비없이 잘 사용하도록 한다.

- 쓰레드의 생성 ~ 소멸

  - 생성 후 start()를 호출하면 실행대기열에 저장되어 차례를 기다린다. 실행대기율은 큐(queue) 구조
  - 자신의 차례가 오면 실행된다.
  - 실행시간이 다되거나, yield(주어진 실행시간을 다른 쓰레드에게 양보하고 실행대기 상태가 된다.)를 만나면 다시 실행대기상태가 되고 다음 차례의 쓰레드가 실행상태가 된다.
  - suspend(), sleep(), wait(), join(), I/O block에 의해 일시정지될 수 있음. I/O block은 입출력에서 발생하는 지연 상태로 사용자 입력을 기다리는 것 같은 경우가 있다.
  - 일시정지 시간이 끝나거나, notify(), resume(), interrupt()가 호출되면 다시 실행대기열에 저장된다.
  - 실행을 모두 마치거나 stop()이 호출되면 쓰레드는 소멸된다.

- sleep(long millis): 일정시간동안 쓰레드를 멈추게 한다. 지정된 시간이 다 되거나 interrupt()가 호출되면 잠에서 깨어나 실행대기 상태가 된다.

  - 항상 try-catch문으로 예외 처리를 해줘야한다. 매번 예외 처리하기 번거로워 try-catch를 포함하는 새로운 메서드를 만들어 사용하기도 한다.
  - 아래의 경우 th1.sleep(2000);을 호출했지만 결과를 보면 th1이 가장 먼저 끝난다. sleep()은 **현재 실행 중인 쓰레드**에 대해 작동하기 때문에 실제 영향을 받는 것은 main쓰레드이다. 그래서 sleep()은 static으로 선언되어 있고 참조변수를 이용한 호출보다 Thread.sleep(2000);과 같이 해야 한다. 

  ```java
  public static void main(String args[]) {
      ...
     	th1.start();
  	th2.start();
      
      try {
          th1.sleep(2000);
      } catch (InterruptException e) {}
  }
  ```

- interrupt()와 interrupted(): 쓰레드 작업 취소 요청을 한다. 강제로 종료시키지는 못한다. interrupted상태(인스턴스 변수)를 바꿀 뿐이다. interrupted()는 interrupt()가 호출되었는지 상태 반환 후, false로 변경한다. isInterrupted()는 상태만 반환하고 false로 초기화하지 않는다.

  - 아래 예제에서 Thread.sleep(1000); 타이밍에 interrupt()를 호출해 InterruptedException이 발생하면 쓰레드의 interrupted상태는 자동으로 false로 초기화된다.  catch블럭에 interrupt()를 추가로 넣어주면 interrupted상태가 true로 다시 바꿔진다.  -> 실행상태가 아니라 대기상태에서 interrupt()때문에 실행대기가 된 상황이라 interrupted()는 false라는 말인 것 같다(?)

  ```java
  public static void main(String args[]) {
      ...
     	th1.start();
  	th2.start();
      
      String input = JOptionPane.showInputDiaolog("입력하세요");
      System.out.println("입력한 값은" + input + "입니다.");
      th1.interrupt();
      System.out.println(th1.isInterrupted());
  }
  
  class ThreadEx14_1 extends Thread {
      public void run() {
          int i = 10;
          
          while (i != 0 && isInterrupted()) {
              System.out.println(i--);
              try {
                  Thread.sleep(1000);
              } catch (InterruptException e) {}
          }
      }
  }
  ```

- suspend(): sleep()처럼 쓰레드를 멈춘다. resume(): suspend()에 의해 정지된 쓰레드를 실행대기 상태로 만든다, stop(): 호출 즉시 쓰레드가 종료된다.

  - suspend(), stop()은 교착상태(deadlock)을 일으키기 쉬워 deprecated(전에는 사용되었지만 앞으로 사용을 권장하지 않는다)되었다.
  - 755페이지 다시 보기

- yield(): 다른 쓰레드에게 양보한다. yield()와 interrupt()를 적절히 사용하면 프로그램 응답성을 높이고 효율적인 실행을 할 수 있다.

  - 아래 예제에서 else { Thread.yield(); }가 없다면 suspended가 true일 때 남은 실행시간동안 while문을 반복하는 `바쁜 대기상태(busy-waiting)`지만 yield()로 양보하면 효율적이다.
  - 2번 예제에서는 stop()이 호출되었을 때 Thread.sleep(1000); 에 의한 일시정지 상태라면 쓰레드가 정지될 때 까지 1초의 시간지연이 생긴다. 하지만 interrupt()를 호출하면 InterruptedException이 즉시 발생해 일시정지상태에서 벗어나 응답성이 좋아진다.

  ```java
  while (!stopped) {
      if (!suspended) {
          ...
      } try {
          Thread.sleep(1000);
      } catch (InterruptedException e) {}
  } else {
      Thread.yield();
  }
  
  // 2
  public void suspend() {
      suspended = true;
      th.interrupt();
  }
  
  public void stop() {
      stopped = true;
      th.interrupt();
  }
  ```

- join(): 자신이 하던 작업을 멈추고 다른 쓰레드가 지정된 시간동안 작업을 수행하도록 기다린다. 시간을 지정하지 않으면 작업을 마칠 때까지 기다린다. interrupt()에 의해 대기 상태에서 벗어날 수 있고 try-catch문으로 감싸야 한다. sleep()과의 다른 점은 join()은 현재 쓰레드가 아닌 특정 쓰레드에 대해 동작하므로 static메서드가 아니라는 것이다.

  - 가비지 컬렉션을 수행하는 쓰레드를 만들어 데몬 쓰레드로 설정하면 설정한 메모리 기준치 이상이 사용되어 gc를 interrupt()로 깨웠지만 수행되기 전에 main쓰레드 작업이 수행되어 메모리를 사용해 한도 메모리를 넘어서게 된다. 따라서 gc를 깨운 후 join()을 사용해 gc가 작업할 시간을 주고 main을 기다리게 한다.
  - 데몬쓰레드는 우선순위를 낮추기보다 sleep()을 이용해 주기적으로 실행되다가 필요하면 interrupt()를 호출해 즉시 가비지 컬렉션이 이루어지도록 하는 것이 좋다.

  ```java
  if (gc.freeMemory() < requiredMemory) {
      gc.interrupt();
      // 추가된 부분
      try {
          gc.join(100);
      } catch (InterruptedException e) {}
  }
  ```



### 9. 쓰레드의 동기화

- 멀티쓰레드 프로세스에서는 여러 쓰레드가 같은 프로세스 내의 자원을 공유해 작업하는데 만약 a 쓰레드가 작업하던 중 b 쓰레드로 제어권이 넘어가 a가 작업하던 공유데이터를 b가 변경했다면 예상과 다른 결과가 생기게 된다. 이 때문에 한 쓰레드가 특정 작업을 마치기 전까지 다른 쓰레드에 의해 방해받지 않도록 `임계 영역(critical section)`과 `잠금(락, lock)`이 도입되었다.
- 공유 데이터를 사용하는 코드 영역을 임계 영역으로 지정하고 공유 데이터(객체)가 가진 lock을 획득한 단 하나의 쓰레드만 코드를 수행할 수 있게 한다. 해당 쓰레드가 임계 영역 내에서의 모든 코드를 수행하고 lock을 반납하면 다른 쓰레드가 이를 획득해 임계 영역에서 코드를 수행한다.
- **한 쓰레드가 진행 중인 작업을 다른 쓰레드가 간섭하지 못하도록 막는 것을 쓰레드의 동기화(synchronization)**이라고 한다.
- 자바에서 synchronized블럭과 JDK1.5부터 지원하는 java.util.concurrent.locks, java.util.concurrent.atomic 패키지를 통해 다양한 동기화를 구현할 수 있다.

#### 9.1 synchronized를 이용한 동기화

- 메서드 전체를 임계영역으로 지정: 메서드 전체를 임계 영역으로 설정. 메서드 호출 시점에 해당 메서드가 포함된 객체의 lock을 얻어 작업을 수행하고 메서드가 종료되면 반환한다.

```java
public synchronized void calcSum() {
    
}
```

- 특정한 영역을 임계 영역으로 지정: 메서드 내의 코드 일부를 블럭으로 감싸고 블럭 앞에 synchronized (참조변수)를 붙인다. `참조변수`는 락을 걸고자하는 객체를 참조하는 것이어야 한다. 이 블럭 영역 안으로 들어가며 지정 객체의 lock을 얻고 벗어나면 반납한다. 

```java
synchronized (객체의 참조변수) {
    
}
```

- 두 방법모두 lock의 획득, 반납이 자동적으로 이루어진다. 우리는 임계 영역만 설정하면 된다. 모든 객체는 lock을 하나씩 가지고 있고, 해당 객체의 lock을 가진 쓰레드만 임계 영역의 코드를 수행할 수 있다. 
- 임계 영역을 멀티쓰레드 프로그램의 성능을 좌우하므로 메서드 전체보다 후자의 방법으로 효율적인 프로그램을 만들어야 한다.

```java
class Account {
    // private이 아니면 외부에서 접근 가능해 동기화를 적용해도 balance값을 변경할 수 있다. 
    private int balance = 1000;
    
    public void withdraw(int money) {
        if (balance >= money) {
            try { Thread.sleep(1000);} catch (InterruptedException e) {}
            balance -= money;
        }
    }
}

// 메서드에 synchronized를 붙이거나
public void withdraw(int money) {
    synchronized(this) {
        if (balance >= money) {
            try { Thread.sleep(1000); } catch (InterruptedException e) {}
            balance -= money;
        }
    }
}
```

- 쓰레드를 2개 만들어 랜덤한 숫자를 출금하도록 한다면 if문에서 설정한 내용과 다르게 잔고가 음수가 될 때까지 실행된다. Thread.sleep(1000);에서 앞선 쓰레드가 출금직전 잠들어있고 두 번째 쓰레드가 if문을 통과하기 때문이다. 따라서 if문과 출금은 하나의 임계 영역으로 묶여야 한다. 

#### 9.2 wait()와 notify()

- 특정 쓰레드가 락을 가진 상태로 오랜 시간을 보낸다면, 예를 들어 출금할 돈이 부족해 입금될 때까지 기다린다면 다른 쓰레드도 모두 락을 기다려야 한다. 이때문에 wait, notify가 고안되었다. 작업을 더 이상 진행할 상황이 아니면 wait()를 호출해 락을 반납하고 기다린다. 다른 쓰레드가 락을 얻어 작업을 수행한후 다시 작업을 진행할 수 있는 상황이 되면 notify()를 호출해 작업이 진행된다.
- 오래 기다린 쓰레드가 락을 얻는 보장은 없다. wait()가 실행되면 쓰레드는 해당 객체의 대기실(waiting pool)에서 통지를 기다린다. notify()가 호출되면 대기실의 모든 쓰레드 중 임의의 쓰레드만 통지를 받는다. notifyAll()은 모든 쓰레드에게 통지하지만 lock을 얻는 것은 하나뿐이다.
- wait와 notify는 특정 객체에 대한 것이므로 Object 클래스에 정의되어 있다. 매개변수를 가진 wait는 지정 시간만 기다린 후 자동적으로  notify()가 호출된다.
- 대기실은 객체마다 존재한다. 
- 이들은 동기화 블록 내에서만 사용할 수 있다.
- `772페이지`부터 나오는 식당에서 음식을 만들어 테이블에 추가하는 요리사와 음식을 소비하는 손님 쓰레드 구현을 참고
- 손님과 요리사가 공유하는 테이블에 동기화를 적용하면 없는 음식을 가져가려 하는 등의 예외는 발생하지 않지만 음식이 없는 손님은 무한정 기다리게 된다. 손님 쓰레드가 테이블 객체의 lock을 가졌기 때문이다. 요리사는 lock이 없어 요리를 추가할 수 없다. 
- 테이블이 꽉차면 요리사를 기다리게 하고 요리가 추가한 후에는 손님을 깨우며 요리가 없으면 손님을 기다리게 하고 요리가 소비된 후에는 요리사를 깨우는 흐름을 추가하면 정상적으로 돌아간다. 하지만 테이블 객체 waiting pool에 요리사, 손님이 함께 기다려 notify()가 호출되면 누가 통지를 받을지 알 수 없다. 운나쁘게 계속 잘못된 통지가 내려지면 오랫동안 기다려 `기아 현상(starvation)`이 일어난다. 이 현상을 막기 위해 notifyAll()을 사용해 일단 모든 쓰레드에게 통지해 원하는 쓰레드를 작업하게 할 수 있다. 하지만 모든 쓰레드가 통지를 받아 여러 쓰레드가 lock을 얻기 위해 `경쟁 상태(race condition)`가 일어난다.  결국 손님과 요리사를 구분해 통지할 필요가 있다.



#### 9.3 Lock과 Condition을 이용한 동기화

-  java.util.concurrent.locks 패키지가 제공하는 lock클래스들을 이용한 동기화

- JDK 1.5부터 추가

- synchronized블럭은 자동으로 lock을 가지고 반환하며 블럭 내에서 예외가 발생해도 자동으로 풀린다. 하지만 같은 메서드 내에서만 lock을 걸 수 있어 불편하다. 이럴 때 lock을 사용

  - ReentrantLock: 재진입 가능, 가장 일반적인 배타 lock / 재진입할 수 있는이라는 뜻(Reentrant) / 특정 조건에서 lock을 풀고 다시 lock을 얻어 작업 수행이 가능하다.
  - ReentrantReadWriteLock: 읽기에 공유적, 쓰기에 배타적인 lock / 읽기는 내용을 변경하지 않으므로 읽기 lock이 걸리면 다른 쓰레드가 읽기 lock을 중복해서 걸고 읽기를 수행할 수 있다. 하지만 `읽기 lock이 걸린 상태에서 쓰기 lock은 허용되지 않으며 반대도 마찬가지다.` 
  - StampedLock: ReentrantReadWriteLock에 낙관적이 lock 기능 추가, JDK 1.8추가. 다른 lock과 달리 ㅣLock 인터페이스 구현하지 않음 / lock을 걸거나 해지할 때 `스탬프(long 타입 정수값)`을 사용하며 읽기, 쓰기 lock외에 `낙관적 읽기(optimistic reading lock)`이 추가된 것이다. `낙관적 읽기 lock은 쓰기 lock에 의해 바로 풀린다. 낙관적 읽기에 실패하면 읽기 lock을 다시 얻어 읽어야 하므로 무조건 읽기 lock을 걸지 않고 쓰기와 읽기가 충돌할 때만 쓰기가 끝난 후 읽기 lock을 거는 것이다.`

- ReentrantLock(boolean fair): 매개변수를 안줄수도 있지만 true로 주면 lock이 풀렸을 때 `가장 오래 기다린 쓰레드`가 lock을 획득할 수 있게 공정하게 처리한다. 하지만 확인절차를 거치기 때문에 성능은 떨어진다.

- lock(), unlock(), isLocked(): 수동으로 lock을 잠금하고 해제해야 한다. lock을 걸고 푸는 것을 잊지말자. 예외가 발생하거나 return문으로 빠져 나가면 lock이 풀리지 않을 수 있으므로 unlock()은 try-finally문으로 감싸는 것이 일반적이다.

- boolean tryLock(long timeout, TimeUnit unit) throws InterruptedException: 다른 쓰레드에 의해 lock이 걸려 있으면 lock을 얻으려고 기다리지 않는다. 시간을 지정하면 지정시간만큼만 기다린다. lock을 얻으면 true를 반환한다. lock()은 얻을 때까지 쓰레드를 block하므로 쓰레드 응답성이 나빠질 수 있지만 tryLock()을 사용해 지정시간동안 얻지 못하면 다시 작업할지 포기할지 결정하는게 좋다. InterruptedException을 사용해 기다리는 중 interrupt()에 의해 작업이 취소될 수 있도록 코드를 작성할 수 있다.

- 9.2에서 손님과 요리사를 구분하지 않고 한 waiting pool에 넣는 문제를 해결하기 위해 각각의 쓰레드를 위한 Condition을 만들어 따로 기다리게 할 수 있다. condition은 이미 생성된 lock에서 newCondition()을 호출해 생성한다.

  ```java
  private Condition forCook = lock.newCondition();
  private Condition forCust = lock.newCondition();
  ```

  - wait() & notify()대신 await() & signal()을 사용한다.
  - 783페이지에서 Condition을 적용한 손님과 요리사 코드 확인

#### 9.4 volatile

- 요즘 대부분 멀티 코어 프로세서를 사용하고 코어마다 별도의 캐시를 가져 문제가 발생할 수 있다.
- `코어`는 메모리에서 읽어온 값을 `캐시`에 저장하고 캐시에서 값을 읽어 작업한다. 다시 같은 값을 읽어올 때는 캐시에 있는지 확인하고 없을 때만 메모리에서 읽어온다. 도중에 메모리에 저장된 변수 값이 저장되어도 캐시에 저장된 값이 갱신되지 않아 메모리에 저장된 값이 다른 경우가 발생한다. 
- 변수 앞에 **volatile**을 붙이면 코어가 변수 값을 읽어올 때 캐시가 아닌 메모리에서 읽어와 캐시와 메모리 간 값의 불일치가 해결된다. volatile대신 synchronized블럭을 사용하면 쓰레드가 들어갈 때와 나올 때 캐시와 메모리간 동기화가 이루어져 값의 불일치가 해소된다.
- JVM은 4 byte(=32bit)단위로 처리한다. 따라서 int나 그보다 작은 타입들은 나눌 수 없는 최소 작업단위인 하나의 명령어로 읽거나 쓰기가 가능하다. 하지만 8byte인 long, double같은 변수는 다른 쓰레드가 끼어들 여지가 있어 읽고 쓰는 모든 과정에서 synchronized 블럭으로 감싸거나 volatile을 변수 앞에 붙일 수도 있다. 
  - 상수에는 volatile을 붙일 수 없지만 어차피 변하지 않는 값이라 멀티 쓰레드에 안전하다.
- volatile은 해당 변수에 대한 읽기, 쓰기가 `원자화`된다. 원자화란 `작업을 더 이상 나눌 수 없게 한다`는 의미다. synchronized 블럭은 여러 문장을 원자화해 쓰레드의 동기화를 구현한 것이다.
- 하지만 volatile은 변수의 읽기, 쓰기를 원자화한 것이지 동기화는 아니다. 예를 들어 잔고 금액을 원자화해도 잔고 조회 메서드를 동기화하지 않으면 출금중에도 잔고 조회가 가능해진다. 출금이 끝난 후에 조회하게 하려면 잔고 조회 메서드에 동기화가 필요하다. 

#### 9.5 fork & join 프레임웍

- 최근 코어의 개수를 늘려 CPU의 성능이 향상되고 있어 이런 멀티 코어를 잘 활용할 수 있는 멀티쓰레드 프로그래밍이 점점 더 중요해지고 있다. 
- JDK 1.7부터 fork & join 프레임웍이 추가되었고 이는 하나의 작업을 작은 단위로 나눠 여러 쓰레드가 동시에 처리하는 것을 쉽게 만들어준다.
- 수행할 작업에 따라 RecursiveAction(반환값 없는 작업), RecursiveTask(반환값 있는 작업) 두 클래스 중 하나를 상속받아 구현한다. 두 클래스 모두 compute()라는 추상 메서드를 가지므로 상속을 통해 이를 구현하면 된다.
- 쓰레드를 작업할 때 start()를 호출하는 것처럼 compute()가 아닌 `invoke()`를 호출해 시작한다.

-  ForkJoinPool은 쓰레드 풀로 지정된 수의 쓰레드를 생성해 미리 만들어 놓고 반복해서 재사용할 수 있게 한다. 반복해서 생성하지 않아도 된다는 장점과 너무 많은 쓰레드로 성능 저하를 막아주는 장점이 있다. 기본적으로는 코어 수와 동일한 수의 쓰레드를 생성한다. 쓰레드 풀은 수행할 작업이 담긴 큐를 제공한다. 

```java
public Long compute() {
    long size = to - from + 1;
    if (size <= 5)
        return sum();
    
    long half = (from+to)/2;
    sumTask leftSum = newSumTask(from, half);
    sumTask rightSum = newSumTask(half+1, to);
    
    leftSum.fork();  // 작업(leftSum)을 작업 큐에 넣는다.
    
    return rightSum.compute() + leftSum.join(); // compute를 재귀호출하면서 작업을 계속한다.
}
```

- fork()가 호출되어 작업 큐에 추가된 작업은 compute()에 의해 더 이상 나눌 수 없을때까지 반복해서 나뉘고 작업 큐가 비어있는 쓰레드는 다른 쓰레드의 작업 큐에서 작업을 가져와서 수행한다. 이것을 작업 훔쳐오기(work stealing)라고 한다. 이 과정은 쓰레드풀에 의해 자동으로 이루어진다. 이 과정으로 한 쓰레드에 작업이 몰리지 않고 여러 쓰레드가 작업을 나누어 처리하게 된다. 작업의 크기를 충분히 작게 해야 골고루 나눠가질 수 있다.
- fork()는 작업을 쓰레드의 작업 큐에 넣는 것. 작업 큐에 들어간 작업은 나뉠 수 없을 때까지 나뉜다. compute()로 나누거 fork()로 작업큐에 넣는 것을 반복한 후 작업 결과는 join()을 호출해서 얻는다.
- fork()는 비동기 메서드고 join()은 동기 메서드다. 따라서 fork()는 호출하면 결과를 기다리지 않고 다음 문장으로 넘어가고 compute()가 재귀호출될 때 join()은 호출되지 않다가 작업을 더 이상 나눌 수 없으면 재귀호출이 끝나고 join()의 결과를 받아서 반환한다.
- 작업을 나누고 합치는데 걸리는 시간때문에 항상 멀티쓰레드가 빠를 수 없다. for문이 재귀보다 빠른 것과 같은 이유다.



### Chapter 14 람다와 스트림

- JDK1.5부터 추가된 지네릭스와 더불어 JDK 1.8부터 추가된 람다식(lambda expression)의 등장은 자바에 큰 변화를 만들었다.
- 람다식의 도입으로 자바는 객체지향언어인 동시에 `함수형 언어`가 되었다. 
- 람다식: 메서드를 하나의 식(expression)으로 표현한 것. 함수를 간략하면서도 명확한 식으로 표현할 수 있게 해준다. 메서드의 이름과 반환값이 없어지므로 `익명 함수(anonymous function)`이라고도 한다. 메서드는 클래스에 포함되야 하므로 클래스를 만들고, 객체도 생성해야 호출가능하지만 람다식은 그 자체만으로 메서드 역할을 대신할 수 있다. 또 메서드 매개변수로 전달되어지는게 가능하고 결과로 반환될 수도 있어 `메서드를 변수처럼` 다룰 수 있게 되었다.
- 객체지향개념에서는 함수(function)대신 객체의 행위나 동작을 의미하는(method)라는 용어를 사용한다. 메서드는 함수와 같은 의미지만 특정 클래스에 속해야 한다는 제약이 있어 다른 용어를 사용한 것이다. 그러나 람다식을 통해 메서드가 `하나의 독립적인 기능`을 하기 때문에 함수라는 용어를 사용하게 되었다. 
- 람다식은 메서드에서 이름과 반환타입을 제거하고 매개변수 선언부와 몸통 사이에 ->를 추가한다. 반환값이 있는 메서드는 return문 대신 식으로 대신한다. 식의 연산결과가 자동적으로 반환값이 된다. 이 때는 문장이 아닌 식이므로 끝에 ;를 붙이지 않는다.

```java
(int a, int b) -> a > b ? a : b
```

- 선언된 매개변수 타입은 추론이 가능한 경우 생략가능하며 대부분의 경우 생략 가능하다. 반환타입이 없는 이유도 추론이 가능하기 때문이다.

```java
(a, b) -> a > b ? a : b
```

- 선언된 매개변수가 하나면 괄호()를 생략할 수 있다. 단 매개변수 타입이 있으면 불가능하다.
- 괄호{} 안의 문장이 하나면 생략 가능하다. 이 때 문장의 끝에 ;를 붙이지 않는다. 단 괄호 안의 문장이 return문일 경우 생략 불가능하다.

```java
(String name, int i) -> System.out.println(name + "=" + i)
```



#### 1.3 함수형 인터페이스(Functional Interface)

- 자바에서 모든 메서드는 클래스 내에 포함되어야 한다. 람다식은 `익명 클래스의 객체`와 동등하다.

```java
(int a, int b) -> a > b ? a : b

// 위와 동일하다. max는 임의로 지은 이름일 뿐이다.
new Object() {
    int max(int a, int b) {
        return a > b ? a : b;
    }
}
```

- **참조변수**가 있어야 객체의 메서드를 호출할 수 있다. 익명 객체의 메서드 역시 마찬가지다. 이 때 참조변수의 타입은 `클래스, 인터페이스가 가능`하며 `람다식과 동등한 메서드가 정의되어 있는 것`이어야 한다. 그래야 참조변수로 익명 객체의 메서드를 호출할 수 있다. 

```java
@FunctionalInterface  // 컴파일러가 함수형 인터페이스를 올바르게 정의했는지 확인해주므로 꼭 붙이기
interface MyFunction {
    public abstract int max(int a, int b);
}

// 인터페이스를 구현한 익명 클래스의 객체
My function f = new MyFunction() {
    public int max(int a, int b) {
        return a > b ? a : b;
    }
}

int big = f.max(5, 3);

// MyFunction에서 정의된 메서드는 람다식의 메서드의 선언부가 일치한다. 그래서 익명 객체를 람다식으로 대체할 수 있다.
MyFunction f = (int a, int b) -> a > b ? a : b;
```

- 람다식도 실제로는 익명 객체이고, MyFunction를 구현한 익명 객체의 메서드와 람다식의 매개변수 타입, 개수, 반환값이 일치하므로 대체가능하다.
- `하나의 메서드가 선언된 인터페이스를 정의`해서 람다식을 다루기로 결정되었고 이를 `함수형 인터페이스(functional interface)`라고 한다. 단 **오직 하나의 추상 메서드**가 정의되어 있어야 한다는 제약이 존재한다. 그래야 람다식과 인터페이스가 1:1로 연결될 수 있기 때문이다. static, default 메서드 개수는 제약이 없다.

```java
// 또 다른 예시. 인터페이스의 메서드 구현 복잡
List<String> list = Arrays.asList("abc", "aaa", "bbb");

Collections.sort(list, new Comparator<String>() {
    public int compare(String s1, String s2) {
        return s2.compareTo(s1);
    }
})
    
// 람다식으로 간단히 처리하기
Collections.sort(list, (s1, s2) -> s2.compareTo(s1));
```

- 메서드의 매개변수가 MyFunction타입이면 이 메서드를 호출할 때 람다식을 참조하는 참조변수를 매개변수로 지정해야 한다. 혹은 참조변수 없이 직접 람다식을 매개변수로 지정해도 된다.

```java
void aMethod(MyFunction f) {
    f.myMethod();
}

MyFunction f = () -> System.out.println("MyMethod()");
aMethod(f);

// 참조변수 없이
aMethod(() -> System.out.println("MyMethod()"));

// 메서드 반환타입이 함수형 인터페이스라면
MyFunction myMethod() {
    return () -> {};
}
```

- 람다식을 참조변수로 다룰 수 있으므로 메서드를 통해 람다식을 주고받을 수 있다. `변수처럼 메서드를 주고받는 것이 가능`해졌다. 사실상 메서드가 아닌 `객체`를 주고받는 것으로 코드가 훨씬 간결해졌다.
- 함수형 인터페이스로 람다식을 참조할 수 있는 것일 뿐, `람다식의 타입이 함수형 인터페이스 타입과 일치하는 것은 아니다.` 람다식은 익명 객체고, 익명 객체는 타입이 없다. (정확히는 있지만 컴파일러가 임의로 이름을 정한다.) 그래서 대입 연산자의 양변의 타입을 일치시키기 위해 `형변환이 필요`하다. 람다식은 MyFunction 인터페이스를 직접 구현한건 아니지만 이를 구현한 클래스의 객체와 완전히 동일해 이를 허용한다. 또 이 형변환은 생략 가능하다. 
- 람다식은 이름이 없을 뿐 객체이지만 Object타입으로 형변환은 불가능하다. 오직 `함수형 인터페이스`로만 형변환이 가능하다. 굳이 Object로 변환하려면 먼저 함수형 인터페이스로 변환해야 한다.

```java
Object obj = (Object) (MyFunction)(() -> {});
String str = ((Object) (MyFunction)(() -> {})).toString();
```

- 외부변수를 참조하는 람다식

```java
class Outer {
    int val = 10;  // Outer.this.val
    
    class Inner {
        int val = 20;  // this.val
        
        // 람다식 내에서 참조하는 지역변수는 final이 없어도 상수로 간주한다.
        // Inner 클래스, Outer 클래스의 인스턴스 변수는 상수가 아니므로 변경 가능하다.
        void method(int i) { // final int i
            int val = 30;    // final int val = 30;
            // i = 10;          상수의 값 변경 불가능
            
            MyFunction f = () -> {  // 매개변수로 i 불가능. 외부 지역변수와 같은 이름은 허용X
                System.out.println("i : " + i);
                System.out.println("val : " + val);
                System.out.println("this.val : " + ++this.val);
                System.out.println("Outer.this.val : " + ++Outer.this.val);
            }
        }
    }
}
```



#### 1.4 java.util.function패키지

- 대부분 메서드는 매개변수 한 두개, 반환 값도 없거나 한 개뿐이며 지네릭 메서드로 정의하면 타입이 달라도 된다. 그래서 java.util.function패키지에 일반적으로 자주 쓰이는 형식의 메서드를 함수형 인터페이스로 정의해 두었다. 매번 새로운 함수형 인터페이스를 정의하기보다 `이 패키지의 인터페이스를 활용`하는 것이 좋다. 그래야 메서드 이름도 통일되고 재사용성, 유지보수 측면에서 좋다.
- 803페이지(매개변수 없거나 하나, 반환 타입 없거나 하나, 매개변수 두 개인 함수형 인터페이스)
- 두 개 이상의 매개변수를 갖는 함수형 인터페이스가 필요하면 직접 만들어서 써야한다.
- 804페이지(매개변수와 반환타입이 일치, 컬렉션 프레임웍의 함수형 인터페이스)
- 806페이지(기본형을 사용하는 함수형 인터페이스 - 위는 지네릭 타입으로 기본형을 처리할 때도 래퍼 클래스를 사용해 비효율적이었다.)
- 매개변수와 반환타입이 모두 Integer라면 IntFunction을 사용할 수도 있지만(입력타입은 정해져있고 출력이 지네릭 타입) 매개변수와 반환타입이 일치하는 IntUnaryOperator를 사용해야 오토박싱&언박싱 횟수가 줄어들어 성능에 좋다. `UnaryOperator`잘 활용하기



#### 1.5 Function의 합성과 Predicate의 결합

- 위 패키지의 함수형 인터페이스는 추상메서드 외에 디폴트 메서드와 static메서드가 정의되어 있다. Function, Predicate에 정의된 메서드를 보면 다른 함수형 인터페이스 메서드도 유사해 응용 가능하다.
- Funciton의 합성: 두 람다식을 합성해 새로운 람다식을 만들 수 있다. f.andThen(g)는 f를 적용하고 g를 적용하는 함수이고 f.compose(g)는 g를 먼저 적용하고 f를 적용한다.

```java
Function<String, Integer> f = (s) -> Integer.parseInt(s, 16);
Function<Integer, String> g = (i) -> Integer.toBinaryString(i);
Function<String, String> h = f.andThen(g);
```

- identity()는 함수를 적용하기 전과 후가 동일한 `항등 함수`가 필요할 때 사용하며 람다식으로는 x->x다. 잘 사용되진 않으며 map()으로 변환작업할 때 변환없이 그대로 처리하기 위해 사용한다. 

- Predicate의 결합: 여러 Predicate를 and(), or(), negate()로 연결해 하나의 새로운 Predicate로 결합할 수 있다.

```java
Predicate<Integer> p = i -> i < 100;
Predicate<Integer> q = i -> i < 200;
Predicate<Integer> r = i -> i%2 == 0;
Predicate<Integer> notP = p.negate();

Predicate<Integer> all = notP.and(q.or(r));
// 람다식을 직접 넣어도 된다.
Predicate<Integer> all = notP.and(i -> i < 100).or(i -> i%2 == 0);
```

- static 메서드인 isEqual()은 두 대상을 비교하는 Predicate를 만들 때 사용한다. isEqual()의 매개변수로 비교 대상을 지정하고 또 다른 비교대상은 test()의 매개변수로 지정한다.

```java
Predicate<String> p = Predicate.isEqual(str1);
boolean result = p.test(str2);

// 합치면
boolean result = Predicate.isEqual(str1).test(str2);
```



#### 1.6 메서드 참조

- 람다식이 단 하나의 메서드만 호출하는 경우에 `메서드 참조(method reference)`라는 방법으로 람다식을 간략히 할 수 있다. 

- 람다식 일부가 생략되었지만 컴파일러는 생략된 부분을 parseInt메서드의 선언부, Function 인터페이스에 지정된 지네릭 타입으로 쉽게 알아낸다.

```java
Function<String, Integer> f = (String s) -> Integer.parseInt(s);

// 메서드 참조
Function<String, Integer> f = Integer::parseInt;

BiFunction<String, String, Boolean> f = (s1, s2) -> s1.equals(s2);

// 메서드 참조. Boolean을 반환하는 equals메서드는 다른 클래스에도 있을 수 있기 때문에 클래스 이름이 필요하다.
BiFunction<String, String, Boolean> f = String::equals;

// 이미 생성된 객체의 메서드를 람다식에서 사용해다면 클래스 이름 대신 객체의 참조변수를 적는다.
MyClass obj = new MyClass();
Function<String, Boolean> f = (x) -> obj.equals(x);
Function<String, Boolean> f2 = obj::equals;

BiFunction<Integer, String, Myclass> bf = (i, x) -> MyClass(i, s);

// 메서드 참조. 매개변수 개수에 따라 알맞은 함수형 인터페이스 사용.
BiFunction<Integer, String, Myclass> bf = MyClass::new;

Function<Integer, int[]> f = x -> new int[x];

// 메서드 참조. 배열을 생성할 때
Function<Integer, int[]> f2 = int[]::new;
```



#### 2. 스트림(stream)

- 지금까지 데이터를 다룰 때 컬렉션이나 배열에 담아 데이터 소스마다 다른 방식으로 다뤘다. Collection과 Iterator와 같은 인터페이스를 이용해 컬렉션을 다루는 방식이 표준화됐지만 컬렉션 클래스마다 같은 기능 메서드들이 중복해서 정의되어있다. List는 Collections.sort(), 배열은 Arrays.sort()를 사용해 정렬한다. 
- 이런 문제를 해결하기 위해 `스트림(Stream)`을 사용한다. `데이터 소스를 추상화(데이터 소스가 무엇이든 간에 같은 방식으로 다룰 수 있어 재사용성을 높인다.)`하고 데이터를 다룰 때 자주 사용되는 메서드를 정의해두었다. 배열, 컬렉션, 파일 데이터도 같은 방식으로 다룰 수 있다.
- 스트림은 Iterator처럼 일회용이라 한 번 사용하면 닫혀서 다시 사용할 수 없다. 필요하면 재생성한다.
- 내부 반복(반복문을 메서드 내부에 숨김)때문에 간결하다. forEach()는 스트림에 정의된 메서드로 매개변수에 대입된 람다식을 데이터 소스의 모든 요소에 적용한다. forEach는 스트림 요소를 소모하며 작업을 수행하므로 같은 스트림에 두 번 호출할 수 없다.

```java
String[] strArr = { "aaa", "bbb", "ccc" };    // 문자열 배열
List<String> strList = Arrays.asList(strArr); // 문자열 list

// 스트림 생성
Stream<String> strStream1 = strList.stream();
Stream<String> strStream2 = Arrays.stream(strArr);

// 스트림으로 데이터 소스의 데이터를 읽어 출력하기
// 람다식으로 표현하면 (str) -> System.out.println(str)
strStream1.sorted().forEach(System.out::println);

// 스트림은 데이터 소스로부터 데이터를 읽기만하고 변경하지 않기 때문에 필요하면 결과를 담아 반환
List<String> sortedList = strStream2.sorted().collect(Collectors.toList());
```

- 스트림의 메서드 중 데이터 소스를 다루는 작업을 `연산(operation)`이라고 한다. 
  - 중간 연산: 연산 결과가 스트림. 연속해서 중간 연산 가능
  - 최종 연산: 연산 결과가 스트림이 아닌 연산. 스트림 요소를 소모하므로 `단 한번만 가능`
  - 816쪽 연산 목록
  - 중간 연산은 map, flatMap, 최종 연산은 reduce, collect가 핵심이다.
  - **최종 연산이 수행되기 전까지 중간 연산이 수행되지 않는다.**
  
- 요소 타입이 T인 스트림은 기본적으로 Stream<T>지만 오토박싱과 언박싱으로 인한 비효율을 줄이기 위해 데이터 소스의 요소를 기본형으로 다루는 스트림(IntStream등)이 제공된다. 보다 효율적이며 int로 작업하는데 유용한 메서드가 포함된다.

- 병렬 스트림: 내부적으로 fork&join프레임웍을 사용해 연산의 병렬 수행이 가능하다. parallel() 메서드를 호출하면된다. sequential()은 반대지만 parallel() 호출을 취소할 때만 사용하면 된다.

- 스트림만들기

  - 컬렉션: Collection에 stream()이 정의되어 있다. 
  - 배열: Stream에 of()와 Arrays에 stream()이 정의되어 있다. 기본형 배열을 소스로 하는 스트림을 생성하는 메서드도 있다.(IntStream.of(int.. values))
  - 특정 범위의 정수: IntStream과 LongStream에 지정된 범위의 연속된 정수를 스트림으로 생성하는 range(), rangeClosed()가 있다. range는 끝범위가 포함되지 않고 rangeClosed에는 포함된다. 
  - 임의의 수: Random클래스에 ints(), longs(), doubles()같은 인스턴스 메서드들이 포함되어 있다. 크기가 정해지지 않은 `무한 스트림`으로 limit()를 사용해 크기를 제한해야 한다. 또는 매개변수로 스트림 크기를 지정하면 유한 스트림을 생성하며 지정된 범위의 난수를 발생시키는 메서드도 있다. 단 end는 범위에 포함하지 않는다. 
  - 람다식 - iterate(), generate(): 람다식을 매개변수로 받아 계산 값을 요소로 무한 스트림을 생성한다. iterate는 seed값부터 시작해 람다식에 의해 계산된 결과를 다시 seed값으로 해서 계산을 반복한다. generate()도 계산되는 값을 요소로 하는 무한 스트림을 생성하나 이전 결과를 이용해 계산하지 않는다. 다만 generate에 정의된 매개변수 타입은 Supplier<T>이므로 매개변수 없는 람다식만 허용된다. 그리고 두 메서드에 의해 생성된 스트림은 기본형 스트림 타입의 참조변수로 다룰 수 없다. (기본형 스트림을 Stream<>로 변경하려면 boxed()를 사용한다.)

  ```java
  Stream<Integer> evenStream = Stream.iterate(0, n->n+2);
  Stream<Double> randomStream = Stream.generate(Math::random);
  ```

  - 파일: list()는 지정된 디렉토리에 있는 파일 목록을 소스로 하는 스트림을 생성해 반환한다. 또 한 행을 요소로 하는 메서드도 있으며 파일 뿐 아니라 다른 입력대상에서도 행단위로 읽는 lines()가 있다. 
  - 빈 스트림: 연산 수행결과가 없을 때 null보다 빈 스트림 반환이 좋다.
  - 두 스트림의 연결: concat()을 사용한다.

- 823 ~ 835페이지에서 스트림의 중간연산 메서드와 사용방법

- sorted(): 지정된 Comparator로 스트림을 정렬하거나 int값을 반환하는 람다식을 사용할 수 있다. Comparator를 지정하지 않으면 스트린 요소의 기본 정렬 기준으로 정렬한다. 단, 스트림 요소가 Comparable을 구현한 클래스가 아니면 예외가 발생한다.

  - JDK 1.8부터 Comparator인터페이스에 static메서드, default 메서드가 많이 추가됐는데 이를 이용하면 정렬이 쉬워진다.

```java
strStream.sorted(Comparator.naturalOrder());     // 기본정렬
strStream.sorted((s1, s2) -> s1.compareTo(s2));  // 기본정렬
strStream.sorted(String::compareTo); // 기본정렬
strStream.sorted(String.CASE_INSENSITIVE_ORDER);  // 대소문자 구분안함
strStream.sorted(Comparator.comparing(String::length));     // 길이 순 정렬
strStream.sorted(Comparator.comparintInt(String::length));  // no오토박싱

// Comparator 인터페이스의 메서드 중 가장 기본적인건 comparing()이다. 정렬조건을 추가할 때는 thenComparaing()을 사용한다.
studentStream.sorted(Comparator.comparing(Student::getBan)
                    	.thenComparing(Student::getTotalScore))
                    	.forEach(System.out::println);
```

- map(): 스트림 요소 값 중 원하는 필드만 뽑아내거나 특정 형태로 변환해야할 때 사용한다. filter()처럼 하나의 스트림에 여러 번 적용할 수 있다.

```java
fileStream.map(File::getName)           // Stream<File>에서 Stream<String>으로
    .filter(s -> s.indexOf('.') != -1)  // 확장자 없으면 제외
    .map(s -> s.substring(s.indexOf('.')+1))
    .map(String::toUpperCase)  // 모두 대문자 변환
    .distinct()                // 중복 제거
    .forEach(System.out::print);
```

- peek(): 연산과 연산 사이에 올바르게 처리되었는지 확인하고 싶을 때 사용한다. forEach()와 다르게 스트림의 요소를 소모하지 않으므로 여러 번 끼워 넣어도 된다. filter(), map() 결과 확인할 때 유용하다.

```java
fileStream.map(File::getName)           // Stream<File>에서 Stream<String>으로
    .filter(s -> s.indexOf('.') != -1)  // 확장자 없으면 제외
    .peek(s -> System.out.printf("filename =%s%n", s))  // 파일명 출력
    .map(s -> s.substring(s.indexOf('.')+1))
    .peek(s -> System.out.printf("extension = %s%n", s))  // 확장자 출력
    .forEach(System.out::print);
```

- mapToInt(), mapToLong(), mapToDouble(): map은 연산결과로 Stream<T>타입의 스트림을 반환하는데, 숫자 반환일 경우 IntStream같은 기본형으로 변환하는 것이 더 유용할 수 있다. 
  - 기본형 스트림은 숫자를 다루는데 편리한 메서드를 제공한다.
  - sum(), average(), max(), min(): sum을 제외하고는 OptionalDouble, OptionalInt로 제공하는데 이는 일종의 `래퍼 클래스`로 int, Double을 내부적으로 가지고 있다. sum은 스트림 요소가 없을 때 0을 반환하면 그만이지만 그 외는 평균값이 0이 나온다거나 할 수 있기 때문이다. 
  - 이 메서드들은 최종연산으로 호출 후에 스트림이 닫힌다. 따라서 연속호출은 불가능하다.
  - 따라서 모두 호출할 때 불편하기 때문에 `summaryStatistics()`라는 메서드가 따로 제공된다.
  - 반대로 IntStream을 Stream<T>로 변환할 떄는 mapToObj()를, Stream<Integer>로 변환할 때는 boxed()를 제공한다.

```java
IntStream studentScoreStream = studentStream.mapToInt(Comparator.comparing(Student::getTotalScore);

IntSummaryStatistics stat = scoreStream.summaryStatistics();
long totalCount = stat.getCount();
double avgScore = stat.getAverage();
                                                      
// CharSequence에 정의된 chars()는 String이나 StringBuffer의 문자들을 IntStream으로 다룰 수 있게 해준다.
IntStream charStream = "12345".chars();                                                   int charSum = charStream.map(ch -> ch-'0').sum();
```

- flatMap(): 스트림의 요소가 배열이거나 map()의 연산결과가 배열인 경우, 즉 스트림의 타입이 Stream<T[]>인 경우, Stream<T>로 다루기 위해 사용한다. 

```java
Stream<String[]> strArrStrm = Stream.of(
	new String[]{"abc", "def", "ghi"},
    new String[]{"ABC", "DEF", "GHI"}
);

// Stream<String>으로 변환하려 했으나 원래의 문자열 배열이 각각 스트림 문자열 배열이 되어 스트림의 스트림 형태가 된다.
Stream<Stream<String>> strStream = strArrStrm.map(Arrays::stream);

// flatMap을 사용하면 각 요소가 하나의 스트림에 담긴 문자열 요소가 되어 하나의 스트림만 존재한다.
Stream<String> strStream = strArrStrm.flatMap(Arrays::stream);
```



#### 2.4 Optional<T>와 OptionalInt

- Optional<T>은 지네릭 클래스로 `T타입의 객체를 감싸는 래퍼 클래스`이다. 그래서 Optional타입의 객체는 모든 타입의 참조변수를 담을 수 있다. JDK 1.8부터 추가되었다. 최종 연산 결과를 Optional객체에 담아 반환하면 반환 결과가 null인지 if로 체크하지 않고 메서드를 통해 처리할 수 있다.

```java
Optional<String> optVal = Optional.of("abc");
```

- 참조변수 값이 null일 가능성이 있으면 of()대신 ofNullable()을 사용해야한다. of()는 NullPointerException이 발생할 수 있다. 
- Optional<T>타입의 참조변수를 기본값으로 초기화할 때는` empty()`를 사용한다. null보다 바람직하다.

```java
Optional<String> optVal = Optional.<String>empty();
```

- 값을 가져올 때는 get()을 사용하지만 null일 때는 exception이 발생하므로 orElse()로 대체할 값을 지정할 수 있다. orElseGet()으로 람다식을 지정하거나 지정된 예외를 발생시키는 orElseThrow()도 존재한다.
- Stream처럼 filter(), map(), flatMap()을 사용할 수 있다. 
- isPresent()
- ifPresent(): Optional<T>를 반환하는 findAny()나 findFirst(), reduce() 같은 최종 연산과 잘어울린다.
- 기본형 스트림으로 OptionalInt등이 존재한다. OptionalInt에 0을 저장했을 때와 empty()를 저장했을 때 int의 기본값인 0이 저장되지만 isPresent(), getAsInt(), equasl()로 보면 다르다. 하지만 empty()와 null이 저장됐으면 동일하게 취급된다.



#### 2.5 스트림의 최종 연산

- 스트림 요소를 소모해서 결과를 만든다. 단일 값 또는 스트림 요소가 담긴 배열, 컬렉션일 수 있다.
- forEach(): 스트림 요소의 출력 용도
- allMatch(), anyMatch(), noneMatch(), findFirst(), findAny(): 매개변수로 Predicate요구. findFirst()는 주로 filter와 함께 사용되어 조건에 맞는 스트림 요소가 있는지 확인하는데 사용되고 병렬스트림인 경우에는 findAny()를 사용해야 한다. 이 둘의 반환 타입은 Optional<T>이며 스트림 요소가 없으면 비어있는 Optional객체를 반환
- count(), sum(), average(), max(), min(): 기본형 스트림의 통계정보. 기본형이 아닐경우 count(), max(), min()만 메서드로 사용. 이를 사용하기보다는 기본형으로 변환하거나 reduce(), collect()를 사용해 정보를 얻는다.
- 
