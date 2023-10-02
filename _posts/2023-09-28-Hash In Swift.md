---
layout: post

---
<br/>

## Direct-Address-Table

<br>

> Key값으로  k를 갖는  원소는 index k에  저장하는 방식의 테이블이다.(array 사용)

<br>

하지만 이렇게되면, 크게 두가지 문제가 발생한다.
<br><br>

**1. 불필요한 공간 낭비**

- Ex) key가 만약 2000번대 부터 시작한다면 0~1999의 인덱스의 공간은 낭비하게 된다.
<br>
<br>

**2. key가 다양한 자료형 지원 x**

- Ex) key값이 만약 String이라면, index에 어떤 기준으로 저장해야할지 방법이 전무 -> key값이 String이면 저장을 할수 없게 됨.

이와 같은 문제들을 해결한 버전(?)이 바로 **Hash Table** 개념이다.
<br>
<br>
## Hash Definition<br>
<br>
해쉬테이블을 제대로 설명하기전, 해싱,해쉬, 해쉬 함수 등의 용어정리를 먼저 정리해보자.
<br>
<br>

#### **- Hash**

<br>

> “해시(hash)”는 일반적으로 무언가를 잘게 쪼갠 후에 결과물을 생성하는 과정을 의미
Ex) 해쉬 브라운, 해쉬 태그
<br>
<br>

#### **- Hash Function**

<br>
위의 해쉬 개념을 컴퓨터 과학에 적용시켜 해쉬 함수를 다음과 같이 서술합니다.<br>
> 임의의 길이를 가진 데이터를 입력받아 고정된 길이의 값, 즉 해시값을 출력하는 함수
> 해시 함수를 사용하면 입력값에 대해 항상 동일한 규칙에 따라 고정된 길이의 해시값을 생성
<br>
<br>

#### **- Hash Value**

<br>

> 해시 값(해시 또는 체크섬이라고도 함)은 특정 길이의 문자열 값으로, 해싱 함수의 계산 결과
<br>
<br>

#### **- Hashing**

<br>

> 매핑 전 원래 데이터의 값을 키(key), 매핑 후 데이터의 값을 해시값(hash value), 매핑하는 과정 자체를 해싱(hashing)
<br>
<br>

## Hash Table
<br>

앞서 말한, Direct-Address-Table의 문제점을 해결하기 위해 **Hash Table은 hash function 'h' 를 사용하여 (key: value)를 인덱스 h(k) 에 저장한다.<br>
** **주로 키 k값을 갖는 원소가 위치 h(k)에 해쉬된다.** 라고 표현하기도 한다.  그리고 여기서 전제조건은 key가 무조건 존재해야 하며 중복되서도 안된다
<br>
<br>

> slot, bucket: hash table을 구성하는 (key:value)를 저장하는 각각의 공간을 칭함.

<br>
<br>

좀 더 구체적으로, Hash Table을 사용해 Direct-Address-Table의 문제점을 어떻게 해결할까?<br>

1. 공간 낭비 문제 
방금과 같이 2000의 값을 가지는 key가 있다고 가정하면, 해쉬 테이블은 Hash Function을 활용해 키값을 넣어 해쉬값을 만들어 내고 그값을 인덱스로 사용하여 효율성을 높인다.<br> 예를들어, k modular 9 이라는 식의 해쉬 함수가 있다면 2000을 9로 나눈 나머지니깐 2가 나온다. 그럼 index0 ~ 2000이 아닌 index 0 ~ 2만 써도 되니 불필요한 공간 논리 상, 1998을 아낄 수 가 있다. (원리가 이렇다는 것)
<br>
<br>

2. Key 자료형 지원 문제
문제예시와 같이 key값이 String이 오게 된다면,  마찬가지로 해쉬 함수를 통해 이 스트링 값을 hash value로 치환해준다.(나름의 알고리즘 존재) 그리고는 그 값을 인덱싱하는데 쓸 수 있게 되는 것 이다. 
<br>
<br>


### 그러면 Hash Table은 마냥 해피해피한 자료구조일까?<br> 
그렇지 않다. Hash Table도 여러가지 문제점을 갖고 있지만 가장 대표적인 문제점은 Colision 문제이다.
<br>
<br>

## Colision
 
<br>

서로 다른 Key의 해쉬값이 값을 때를 칭함. ( 중복되는 key는 없지만 hash value는 중복될 수 있다.)<br>
그러므로,  colision이 최대한 적게 나도록 해쉬 함수를 잘 설계해야하고, 발생하는 경우는 chaning 또는 open addressing등의 방법을 고려한다. (이외에도 많음)
<br>
<br>

## Colision을 해결하는 방법
<br>

여러가지 방법이 있지만, 이 섹터에서 소개할건 **open addressing**방식과  **chaning** 방식이다.

<br>
### - Open Addressing
<br>

Collision이 발생하면 미리 정한 규칙에 따라 hash table의 비어있는 slot을 찾는다. (추가적인 메모리 사용 x) <br>
slot을 찾는 방식에 따라 **Linear Probing, Quadratic Probing, Double Hashing** 등이 있음
<br>
<br>

#### 1. Linear Probing(선형 조사법) & Quadratic Probing(이차 조사법)
<br>
colision발생시, 발생한 해시값으로 부터 일정한 값만큼 뛰어 넘어, 비어있는 slot에 데이터를 저장
Linear(+1,2,3) Quadratic (1^2, 2^2,3^2)
colision이 많아지면 특정 영역에 데이터가 몰리는 클러스터링이 발생 할수도 있음. 클러스터링이 발생하면 
평균 탐색 시간이 증가

<br>
#### 2. Double Hasing(이중 해싱)
<br>

앞서말한 클러스터링 문제가 발생하지 않도록, 2개의 해쉬 함수를 사용함
첫번째 해쉬함수: 최초의 해쉬값을 얻을때
두번째 해쉬함수: colision발생 시, 탐사 이동폭을 얻기 위해
<br>
<br>

### - Separate Chaning
<br>

Linked List 또는 Tree를 사용한다. Collision 발생 시, Linked List에 노드(slot)을 추가하여 데이터를 저장
(Worst case로 N개의 모든 key가 동일한 해쉬값을 가지는 경우 n개의 길이를 가진 Linked List가 만들어짐 <br>
-> 검색 및 삭제 O(n) (기본은 O(1))) 
<br>
<br>

## Swift Hashable Protocol

<br>
Hashable 프로토콜은 해싱에 사용할 수 있는 해시 값을 생성하는 기능을 정의한다. 또한 여기서 해시 값은 고정 길이의 값을 가지며, 주로 컬렉션에서 빠른 검색을 위해 사용된다. Hashable 프로토콜을 준수하는 타입은
` func hash(into hasher: inout Hasher)`
함수를 구현해야 한다.
(Swift의 기본 타입은 기본적으로 Hasable하다.)
<br>
<br>

#### - Hasher(?)

<br>
주석을 읽어보니 **hashValue**가 **deprecated** 되고 **hash**함수가 그 자리를 메운거 같음<br>
Hasher는 **Set**과 **Dictionary**에서 사용되는 범용 해시 객체<br>
**Hasher**를 사용하여 임의의 바이트 시퀀스를 정수 해시 값으로 매핑할 수 있다고 함.<br> 또한 **combine** 메서드를 사용하여 데이터를 해시에 추가할 수 있으며, 해시 작업을 완료한 후 **finalize()**를 호출하여 해시 값을 검색하는 것도 가능하다고 한다.<br>
**warning: Hasher의 구현 알고리즘이 라이브러리의 버전 간에 변경될 수 있으므로, 프로그램 실행 간에 해시 값을 저장하거나 재사용하지 말것.**<br>

<br>
<br>

#### 한번 Hashable프로토콜로 커스텀 Key값을 만들어 보자.<br>
<br>

```swift
struct MyMacBook: Hashable {
    var name: String
    var price: Int
    
    func hash(into hasher: inout Hasher) {
        hasher.combine(name) // 데이터 추가
        hasher.combine(price) // 데이터 추가
                print(hasher.finalize()) // ex) -7858459044973498536
    }
}
let M1 = MyMacBook(name: "M1", price: 100)
let M2 = MyMacBook(name: "M2", price: 200)
// 구조체를 키값으로~
let macBookDict: [MyMacBook: String] = [M1: "M1 구입했구나", M2: "M2 구입했구나"]
let BuyM1 = macBookDict[MyMacBook(name: "M1", price: 100)] // -> "M1 구입했구나"
```
<br>

추가로 Hasable, Equatable 프로토콜을 채택해주면 컴파일러가 자동으로 hash(into:) 함수를 구현해준다. 혹은 위 예제처럼 직접 함수를 구현해줄 수 있다.
Swift 4.1 이전까지는 자동으로 구현되지 않아 반드시 사용자가 hash(into:) 함수와 == 연산자를 구현해주어야 했다고 함
<br>
<br>
재밌는건 이 부분이 Set의 Contain함수가 O(1)인 이유를 설명해 준다는 것이다. 
Set의 구현체를 보면 
<br>
```swift
@frozen
@_eagerMove
public struct Set<Element: Hashable> {
  @usableFromInline
  internal var _variant: _Variant
```  
<br>
이처럼 원소를 hashable 타입으로 받고있다. 그래서 일반적으로 Set은 일반적으로 키와 값이 동일한 해시맵으로 구현되거나 한다.(아닌 경우가 있을수 있지만) <br>
결국 이런 각 고유한 해쉬 값을 통해 값의 조회가 가능하니 O(1)로 조회가 가능한 것이다.<br>
또한 같은 이유로 Set에 어떤 커스텀 타입을 넣고 싶다면 해당 타입은 꼭 Hashable을 준수해야 한다.

<br>
<br>


### Swift는 어떻게 Colision을 처리할까?
<br>
더 나아가 이런 궁금증도 생길 수 있다., Swift의 colision이 발생하면 어떤 방식을 사용할까? <br>
Swift는 기본적으로 Cahning을 사용하고, 해시 충돌이 자주 발생할 것으로 예상되는경우, Linear Probing또는 Double hashing을 고려한다고 한다.<br>
더불어, 해쉬 충돌을 예방하기 위해 (충돌 저항성을 높이기 위해) SipHash라는 방법을 사용한다고 한다.
<br>
<br>

### SipHash & Seed
<br>

Swift뿐만 아니라 Radis에서도 사용하고 있는 해쉬 알고리즘이다. 
임의의 값인 Seed를 활용해 동일한 입력 데이터에 대해  다른 해쉬 값이 생성되도록 하여 해쉬값을 분산시키고 충돌 저항성을 높힌다.
<br>
<br>
```swift
let value = "hello, world"
let seed = arc4random()

// SipHash를 사용하여 hash값을 생성
let hash = value.hash(into: &seed)

// hash값 출력
print(hash)

```
<br>
<br>
<br>
<br>
<br>
<br>

+) Ref: <br>
https://jeong9216.tistory.com/523
https://ratsgo.github.io/data%20structure&algorithm/2017/10/25/hash/
