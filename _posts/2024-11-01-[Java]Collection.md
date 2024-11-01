---
title: "Collection(List, Set, Map) and Array"
excerpt: "[Java] About Collection(List, Set, Map) and Array"

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2024-10-30
---


### 들어가면서

  - CompletableFuture를 정리하면서 Java8에서 지원하는 stream()에 대해서 정리해보고자 합니다.
  - 그 전에 Collection에 대해 요약 정리해보고 다음 포스트에서 stream()에 대해서 다뤄보겠습니다.

<br />

---

> ### Array vs ArrayList(Collection)

  - 
  - Array는 객체로 Heap 메모리에 저장하며 고정된 크기로 동일한 타입을 저장할 수 있다.
    - Array의 길이 조절이 불가능하다.
  - Collection은 Array와 다르게 배열의 길이 조절에 용이하다.
    - Collection은 기존 배열 개념을 향상시켜 데이터 처리에 용이하도록 만든 Interface
  
  - #### Performance Test
    ```java
    public class ArrayWithList {

      public static void main(String[] args){

        //Array
        String[] array = {"1", "2", "3", "4", "5", "6", "7", "8"};

        Time arrayTime =new Time();
        Time listTime =new Time();

        arrayTime.setStartTime();
        String arrayData = array[7];
        arrayTime.setEndTime();
        
        //List
        List<String> list = Arrays.asList(array);

        listTime.setStartTime();
        String listData = list.get(7);
        listTime.setEndTime();

        arrayTime.printNanoSec();
        listTime.printNanoSec();
      }
    }

    ```
    ![image info](/assets/img/Collection5.png)
    <img src="/assets/img/Collection5.png" alt="" width="0" height="0">

    - Array 조회 시간 : 1300ns
    - List 조회 시간 : 4500ns

  - #### get()

    ```java
    public E get(int index) {
      Objects.checkIndex(index, size);
      return elementData(index);
    }
    ```
    - ArrayList / LinkedList의 get()은 index까지 연결된 데이터를 타고 들어가야 되기 때문에 Array보다 조회가 오래 걸린다.
    - 하지만 조회에서 시간적으로 엄청 큰 차이가 나는 것은 아니다.


<br />

---


> ### Collection

  ![image info](/assets/img/Collection.png)
  <img src="/assets/img/Collection.png" alt="" width="0" height="0">


  - 다수의 데이터를 쉽고 효과적으로 처리할 수 있는 표준화된 방법을 제공하는 클래스의 집합
  - 데이터를 저장하는 자료구조 & 데이터를 처리하는 알고리즘을 구조화하여 클래스로 구현함
  - Collection 핵심 인터페이스
    - List 인터페이스
      - 순서가 있는 데이터의 집합
      - 중복 허용함
    - Set 인터페이스
      - 순서를 유지하지 않는 데이터의 집합
      - 중복 허용하지 않음
    - Map 인터페이스
      - Key-Value의 쌍으로 이루어진 데이터의 집합
      - 순서를 유지하지 않음
      - 키의 중복을 허용하지 않고 값을 중복은 허용함

<br />

---


> ### List

  - 객체를 Index로 관리하며 객체를 저장하면 자동 Index가 부여된다
  - 객체 자체를 저장하는 것이 아닌 객체의 번지를 참조한다
  - **null**이 저장될 경우에는 해당 Index는 객체를 참조하지 않는다
  - Vector, ArrayList, LinkedList

  - #### Vector

    - List<E> list = new Vector<E>();
    - Thread-safe
      - 쓰레드의 개수와 상관없이 동기화 처리를 하므로 Thread-safe하지만 Single Thread 환경에서도 동기화 처리를 하므로 성능이 좋지 않다
      - MultiThread 환경에서 Thread가 동시에 이 메소드를 실행할 수 없어서 안전하게 객체를 추가, 삭제할 수 있다


  - #### ArrayList

    - ArrayList<String> list = new ArrayList<String>();
    - 가장 많이 사용되는 Collection 클래스
    - 배열에 더 이상 저장할 공간이 없으면 보다 큰 새로운 배열을 생성해 기존의 배열에 저장된 내용을 새로운 배열로 복사한 다음에 저장한다(Array와 다른 점)
    - 추가/제거가 많을 경우 오버헤드가 많이 발생한다
      - 중간에 삽입될 때 데이터들이 뒤로 밀리면서 성능저하 발생
    - 동기화처리가 되는 Collection을 생성하면 MultiThread환경에서 사용 가능하다
      - Collections.synchronizedList
      - Collections.synchronizedMap
      - Collections.synchronizedCollection

  - #### LinkedList

    - LinkedList<String> list = new LinkedList<String>();
    - ArrayList와 비슷하지만 노드(객체)끼리의 주소 포인터를 서로 가리키며 링크(참조)함으로써 이어지는 구조
    - List, Stack, Queue 자료 구조로 사용
    - LinkedList 종류
      - 단방향 연결 리스트(Node next, data)
      - 양방향 연결 리스트(Node next, Node previous, data)
      - 양방향 원형 연결 리스트(마지막 요소를 만나면 다시 처음 요소로 되돌아가는 방식)
    - 공간의 제약이 존재하지 않으며 삽입 / 삭제 역시 노드가 가리키는 포인터만 바꿔주면 된다
      - 삽입 / 삭제 처리가 빠르다
      - 불연속적인 단위로 저장되어 있어서 ArrayList보다 느리다

    - ArrayList vs LinkedList(데이터 조회, 수정 / 단위 ns)
    
    ![image info](/assets/img/Collection2.png)
    <img src="/assets/img/Collection2.png" alt="" width="0" height="0">
    ![image info](/assets/img/Collection1.png)
    <img src="/assets/img/Collection1.png" alt="" width="0" height="0">
      - 수정(삽입/삭제) >> 조회 : LinkedList
      - 조회 >> 수정(삽입/삭제) : ArrayList
      - 실제 성능면에서 둘은 큰 차이는 없다(둘을 굳이 비교하자면 미세한 성능 차이가 존재)


  - #### Summary(Vector, ArrayList, LinkedList)
    - Vector : 과거에 사용하던 대용량 처리 Collection이며 잘 사용 안하고 동기화처리가 자동적으로 내부에서 일어나므로 비교적 성능이 좋지 않다.
    - ArrayList : 배열의 복사에 의한 데이터 저장처리를 내부적으로 수행하므로 많은 데이터의 수정시에는 성능이 떨어지나 각 데이터에 대한 인덱스를 가지고 있기 때문에 조회에 있어서 빠르다. 
    - LinkedList : 다음 자료의 위치 정보를 가지며, 인덱스는 가지고 있지 않다. 데이터의 수정에는 다음 노드가 가르키는 포인터만 수정하면 되므로 많은 정보의 수정이 일어날 때 유용하다. 대신 데이터 조회시 처음 자료부터 찾아 나가야 하므로 느려진다.

<br />


---

> ### Set

  - 데이터 중복을 허용하지 않고 순서가 없는 데이터의 집합
  - 데이터 순회(순서도 없고 Index가 존재하지 않음)
    - iterator 인터페이스
      ```java
      Iterator<String> iterator = set.iterator();
      while(iterator.hasNext()){
        String str = iterator.next();
      }
      ```
    - for-each문
      ```java
      for(String str : set)
      ```
    
  - HashSet, LinkedHashSet, TreeSet
  
  - #### HashSet

    - Set<E> set = new HashSet<E>();
    - 객체를 저장하는 과정(중복 허용x)
    ![image info](/assets/img/Collection3.png)
    <img src="/assets/img/Collection3.png" alt="" width="0" height="0">


  - #### LinkedHashSet
    - 데이터가 저장되는 순서대로 순서를 유지한다

  - #### TreeSet
    - 데이터 추가 시 오름차순 정렬로 자동 저장된다

<br />

---

> ### Map

  - Key-Value 쌍으로 구성된 Entry 객체를 저장하는 구조
  - Key, Value는 모두 객체
  - Key 중복 허용x
    ![image info](/assets/img/Collection4.png)
    <img src="/assets/img/Collection4.png" alt="" width="0" height="0">
    
  - HashMap, TreeMap
  
  - #### HashMap

    - HashMap<String, Object> hashMap = new HashMap<String, Object>();
    - Map 인터페이스를 구현한 대표적인 Map 컬렉션
    - 담을 데이터의 개수가 많은 경우에 초기 크기를 지정해주는 것을 권장


  - #### LinkedHashSet
    - TreeMap<String, Object> treeMap = new TreeMap<String, Object>();
    - 이진 트리 기반의 Map 컬렉션
    - TreeSet과 비슷하지만 MapEntry를 저장한다
      - 저장하는 동시에 Key에 대하여 정렬한다
      - 숫자 > 알파벳 대문자 > 알파벳 소문자 > 한글

<br />

---

### Reference

- https://inpa.tistory.com/entry/JAVA-%E2%98%95-LinkedList-%EA%B5%AC%EC%A1%B0-%EC%82%AC%EC%9A%A9%EB%B2%95-%EC%99%84%EB%B2%BD-%EC%A0%95%EB%B3%B5%ED%95%98%EA%B8%B0
- https://hoon26.tistory.com/25
- https://inpa.tistory.com/entry/JAVA-%E2%98%95-ArraysasList-%EC%99%80-Listof-%EC%B0%A8%EC%9D%B4-%ED%95%9C%EB%B0%A9-%EC%A0%95%EB%A6%AC?category=890834
- https://codemanmulsang.tistory.com/41