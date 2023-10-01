---
layout: post

---
<br/>

이번 글에서는, **Generic, Protocol, any, some** 이 4개의 것들이 Swift 타입 시스템에서 어떠한 역할을 하고 있는지 서술하려고 한다. 
<br>
<br>

## Generic & Protocol
<br>
제네릭스를 이용하면 여러 타입에 공통적으로 사용할 수 있는 함수와 자료형을 정의할 수 있다.  뿐만 아니라 Protocol을 사용하여 타입에 제약조건을 걸어 사용할 수도 있다. 
<br>
<br>

```swift
// 보일러 코드들
protocol Fixable {
    func fix() -> String
}

struct LapTop: Fixable {
    func fix() -> String {
        return "fix labtop"
    }
}

struct DeskTop: Fixable {
    func fix() -> String {
        return ""
    }
}

func isFixing<Target: Fixable>(of computer: Target) -> Bool {
    return computer.fix() != ""
}
```
<br>
이런식으로 각 Target타입이 Fixable만을 채택후 구현을 해주면 코드를 공유할 수 있다. 

<br>
<br>

## 타입으로서의 Protocol & Generic 
<br>
위의 제네릭 방법뿐만 아니라 protocol 자체를  타입으로도 지정해 줄수 있다.
<br>
<br>

```swift
func isFixing(computer: Fixable) -> Bool {
     return computer.fix() != ""
}
```
<br>
<br>

### 그러면 Generic과 Protocol의 타입으로서의 차이점은?
<br>

```swift
struct QueueExistential {
    var queue: [Fixable]
}

struct QueueGeneric<Target: Fixable> {
    var queue: [Target]
}

let laptop = LapTop()
let desktop = DeskTop()

// 1 - use protocol
//✅ Compile Success
let queueExistential = QueueExistential(queue: [laptop, desktop])

// 1 - use generic
// 🔴 Error: type 'any Fixable' cannot conform to 'Fixable', note: only concrete types such as structs, enums and classes can conform to protocols
let queueGeneric = QueueGeneric(queue: [laptop, desktop])
```
<br>
이 예시에서는 protocol을 사용한 Queue는 컴파일이 되고 Generic은 Error가 난다. 왜 이런 차이가 벌어지게 되는 것일까?<br>

Generic은 컴파일시 프로토콜을 준수하는 구체적인 타입이 필요하다. 단순히 Fixable protocol을 이용해 어떠한 타입의 인자를 받도록 **“제한”** 하는 것이지 Fixable한 다양한 타입을 받도록 하는것은 아니다. 이어 제네릭을 **타입단위의 추상화** 라고도 한다.<br>
 반면 Protocol은 프로토콜을 준수하는 모든 타입을 참조할 수 있다. 해당 프로토콜을 만족하기만 한다면 어떤 값이든 들어갈 수 있기 때문에 **값 단위의 추상화** 라고도 부른다.  이 차이를 잘 이해하는 것 이 중요하다.

<br>
<br>

## Existential any <br>
<br>
any를 설명하기에 앞서, 해당 코드를 보자
<br>

```swift
let num: Equatable = 1 
let text: Equatable = "st"
// 🔴 error: use of protocol 'Equatable' as a type must be written 'any Equatable'
```
<br>

Equatable를 타입으로는 지정할 수 없고, 타입으로 지정하려면 **‘any Equatable’**를 쓰라고 에러가 나온다. <br>
`protocol Equatable` 가 num,text 의 `type` 으로 사용되었으므로, 이 맥락에서 `Equatable` 는 existential type 이다.<br>
(**existential type**: protocol 이 type 으로 사용될 때를 칭함)
<br>
<br>

보통 구체적인 타입을 프로토콜 타입으로 안전하게 치환할 수 있는 경우를 `‘covariant’`하다고 한다. 보통 associatedtype을 가지고 있거나(구현이 필요함), equatable의 연산자와 같이 
<br>

```swift 
static func == (lhs: Self, rhs: Self) -> Bool
```
<br>

lhs와 rhs의 구체적인 타입이 일치하지 않을 경우 Equatable에서 안정성 문제가 발생한다. (`Non-covariant`)
이렇게 제약 조건으로서의 프로토콜과 타입으로서의 프토토콜은 다른 개념으로 볼수가 있기 때문에 any Protocol이라는 개념이 등장하게 된다. <br>
프로토콜 ‘타입’에는 any를 붙힘으로써 제약조건으로서의 프로토콜과 구분을 두는 것이다. 
이뿐 아니라 existential type 은 concrete type의 비용 차이 문제와 더불어 (dynamic vs static) 가시성에서 두 타입(existential vs concrete)에 대한 구분이 가지 않는 점을 근거로 swift 5.6 에 Existential any
가 release 하게 된다.
<br>
<br>

## Opauqe Type
<br>

직역하자면 **‘불분명한 타입’** 이라 한다. <br>
다른 이름으로는 **역 제네릭 타입(reverse generic type)** 이라고 불르기도 한다. 
그에 대한 이유는 Opauqe Type의 특징과 연관이 있다. 
<br>

기존 generic으로 작성된 함수는 
<br>

```swift 
struct Stack<Element> {
    var items: [Element] = []
    
    mutating func push(_ item: Element) {
        items.append(item)
    }
    
    mutating func pop() -> Element {
        return items.removeLast()
    }
}
```
<br>
<br>

### 구현부: 추상화 하여 작성

<br>

### 호출부: 구체적인 타입 지정( 여기서 타입 지정 ) 
<br>
<br>

와 같은 특징을 지닌다.

이제 `opeque type`을 살펴보자. 
opeque type some 키워드 + 프로토콜로 사용한다.
<br>
<br>
```swift
func makeArray() -> some Collection {
    return [1, 2, 3]
}
```
<br>

제네릭과 반대로 makeArray 함수 안에서 [Int]라는 구체타입을 알게 된다.
그리고 호출부 입장에서는 추상적인 타입인 ‘Collection’ 타입 밖에 알지 못한다. 

그래서 정리하자면 
<br>
<br>

### 구현부: 구체적인 타입 지정 (여기서 타입 지정)
<br>

### 호출부: 추상타입
<br>
<br>

해서 이러한 특성으로 인해 `"Opaque Type을 사용하여 리턴 타입의 정보를 숨길 수 있다”`라고도 말함
또한 Opaque Type을 사용하게 되면 호출부에게 타입을 숨기면서 컴파일러는 구체타입을 알기 때문에 Type Identity를 preserve(보존)할 수 있다라고도 말할수 있습니다. 

<br>
<br>
+) Opeque Type은 해당 용도로 사용될 수 있습니다.
<br>

**- 함수(메소드)의 리턴타입**
 
<br>

**- stored property 타입**

<br>

**- computed property 타입**

<br>

**- subscripts** 

<br> 

**- (Swift 5.7 ~) 함수 파라미터 타입**

<br>
<br>
<br>
<br>

+) Ref<br>
https://forums.swift.org/t/improving-the-ui-of-generics/22814#heading--clarifying-existentials<br>
https://developer.apple.com/videos/play/wwdc2022/110352/<br>
https://docs.swift.org/swift-book/documentation/the-swift-programming-language/protocols/<br>
https://developer.apple.com/videos/play/wwdc2022/110353/<br>
https://zeddios.tistory.com/1348<br>
https://github.com/apple/swift-evolution/blob/main/proposals/0335-existential-any.md<br>
https://engineering.linecorp.com/ko/blog/about-swift-type-system#1<br>
<br>

