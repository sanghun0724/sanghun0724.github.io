---
layout: post

---
<br/>

## Mutation

<br>
> : 어떤 것의 형태나 구조의 변화

<br>

**흔히들 프로그래밍에서 Mutation을 피하라고 한다. 이렇게 말하는 근거가 무엇일까?**
<br>

<br>
- Mutation으로 인해 예상치 못한 디버그하기 어려운 문제가 발생할 수 있다. 즉, 데이터가 어딘가에서 부정확해지며 어디서 발생하는지 알 수 없을 수 있다.<br>
- 변형은 코드를 이해하기 어렵게 만든다. 언제든지 배열이나 객체가 다른 값을 가질 수 있으므로 코드를 읽을 때 주의해야 한다.<br>
- 함수 매개변수를 Mutation하게 만든다면,  함수의 행동은 복잡해질수 있다.<br>
- Mutation은 예측 하기 힘들다. 프로그래밍 중, 어떤 메소드가 원본 데이터를 수정하고 어떤 메소드가 수정하지 않는지 단순 기억력으로는 기억 하기 쉽지 않다. <br>
<br>
<br>

**그렇다면 Mutation이 무조건 나쁜걸까?** <br>

<br>

**아니다.**  Mutation은 프로그래밍 세계에서 동작을 위해 꼭 필요한 기능이다. 또한 다음과 같은 이유로 활용을 고려해볼수 있다. <br>
- 값을 Mutation을 사용하여 변경하면, 값 타입을 복사하는 비용을 줄일 수 있다. 이는 성능을 향상시킬 수 있다.<br>
- 사용 하기에 따라 유지보수성을 올릴 여지도 있다. 한곳에 모아두고 관리해야 하는 상태가 존재한다면 Mutation을 활용해봄직 하다.<br> 
그럼에도 불구하고, Mutation 사용시 Side-Effect가 나오는 경우가 정말x4 많기 때문에 Mutation에 대한 정확한 이해 및 주의가 필요하다.<br>
<br>

## 타입에 따른 변화 관찰 

<br>
<br>

Swift에서는 참조 타입과 값 타입으로 구분하여 Mutation의 동작을 다르게 처리한다.
그러므로 참조타입과 값타입의 행동 원리를 정확히 이해 하는게 중요하다. 이와 관련하여 크게 두가지로 나누어 코드 실행을 통해 살펴보자.
<br>
<br>

**테스트 Class, Struct 작성**
<br>
<br>

```swift
// MARK: - class

class PersonClass {
    var name: String
    var jacket = JacketClass(color: "pulple")
    
    init(name: String) {
        self.name = name
    }
}

class JacketClass {
    var color: String
    
    init(color: String) {
        self.color = color
    }
}

// MARK: - struct

struct PersonStruct {
    var name: String
    var jacket =  JacketClass(color: "pulple")
    
    init(name: String) {
        self.name = name
    }
    
    mutating func setName(_ name: String) {
        self.name = name
    }
}

struct JacketStruct {
    var color: String
    
    init(color: String) {
        self.color = color
    }
}

extension PersonStruct: Equatable {
    static func == (lhs: PersonStruct, rhs: PersonStruct) -> Bool {
        lhs.name == rhs.name
    }
}

```
<br>


### **MISSON(1) - 변수에 할당된 인스턴스를 다른 인스턴스로 바꾸기 (let, var 각각 경우)**

<br>
<br>

#### **Class**

<br>
<br>

```swift
let person1 = PersonClass(name: "tony")
let person2 = PersonClass(name: "mathew")

// var
var man1 = person1
print("--------Class-------")
print(man1.name)
print("------------------")
man1 = person2
print(man1.name)
print("------------------")

// let
let man2 = person1
print("--------Class-------")
print(man2.name)
print("------------------")
man2 = person2 // Error: Cannot assign to value: 'man2' is a 'let' constant, Change 'let' to 'var' to make it mutable
print(man2.name)
print("------------------")
```

<br>

#### **Struct**

<br>
<br>

```swift
/// PersonStruct에 대한 값들이 stack에 저장
// var
var woman1: PersonStruct = person3
print("--------Struct-------")
print(woman1.name)
print("------------------")
woman1 = person4
print(woman1.name)
print("------------------")

// let
let woman2: PersonStruct = person3
print("--------Struct-------")
print(woman2.name)
print("------------------")
woman2 = person4 // Error: Cannot assign to value: 'man2' is a 'let' constant, Change 'let' to 'var' to make it mutable
print(woman2.name)
print("------------------")
```
<br>

let은 재할당이 안되고 단 한번만 값이 할당된다. 그러기에 변할 수 없다. <br>

```swift
weak let weakMan = PersonClass(name: "weakMan")
// Error: 'weak' must be a mutable variable, because it may change at runtime
```
 <br>
 
 이것이 우리가 `'weak let'`을 쓸수 없는 이유이기도 하다.
 (weak로 선언된 참조는 인스턴스가 해제되면 자동으로 nil로 설정됨)
<br>
<br>

### **MISSON(2) - 할당된 인스턴스의 프로퍼티 값 바꾸기 (let, var 각각 경우)**

<br>
<br>

#### **Class**

<br>

```swift
// var
var white1 = PersonClass(name: "oliver")
white1.name = "tom"
white1.jacket = JacketClass(color: "red")
print("------------------")
print(white1.name)
print(white1.jacket.color)


print("------------------")

// let
let white2 = PersonClass(name: "oliver")
white2.name = "tom"
white2.jacket = JacketClass(color: "red")
print(white2.name)
print(white2.jacket.color)
print("------------------")
```
<br>

#### **결과:**
<br>

```
------------------
tom
red
------------------
tom
red
------------------
```
<br>

Class의 경우, 프로퍼티를 let과 var 둘다 변경이 가능하다. 어째서 일까? 
`let white2` 변수가 stack에 `person1: personClass`를 가르키는 주소값을 가지고 있다.
여기서 let이 가지고 있는 상수는 주소값이므로 그 주소값만이 let의 특성인 Immutable해진다. 그러므로 인스턴스의 프로퍼티는 var로 설정했다면 변경이 가능하다.
<br>
<br>

#### **Struct**

<br>
```swift
print("------------------")
// var
var black1 = PersonStruct(name: "jamal")
black1.name = "darnell"
black1.jacket = JacketClass(color: "green")
print(black1.name)
print(black1.jacket.color)

print("------------------")

// let
let black2 = PersonStruct(name: "jalen")
black2.name = "darnell" // Error: Cannot assign to property: 'black2' is a 'let' constant, Change 'let' to 'var' to make it mutable
black2.jacket = JacketClass(color: "green") // Error: Cannot assign to property: 'black2' is a 'let' constant, Change 'let' to 'var' to make it mutable
black2.setName("darnell") // Error: Cannot assign to property: 'black2' is a 'let' constant, Change 'let' to 'var' to make it mutable
print(black2.name)
print(black2.jacket.color)
print("------------------")
```
<br>

Struct의 경우, let으로 선언이 되면 자신은 물론 프로퍼티수정이 불가능 하다. 어째서 일까?
Class와 다르게 참조형태가 아닌, 값 자체들을 (주로) Stack에 저장해두고 있기 때문이다. 그러므로 let에 해당하는것들은 struct값들 그자체이므로 Immutable 해진다.
물론 mutable func도 사용하지 못한다.
<br>
<br>

## Struct with var<br>
<br>
`var struct` 같은 경우는 해당 값들이 정말 변경 되는 것일까? 결론을 먼저 말하자면 "논리적으로 값타입은 Immutable하다"가 맞다. 
<br>

```swift
struct Bar {
    var x: Int
    var y: Int
}
struct Foo {
    var bar: Bar {
        didSet(oldValue)
        {
            print("Got a new bar!")
            print(oldValue)
        }
    }
}
var foo = Foo(bar: Bar(x: 1, y: 1))
foo.bar.x = 10
```
<br>

이코드를 실행해보면 `foo.bar = something` 을 안했는데도 `"Got a new bar!"` 가 프린팅 된다. 이로써 알수 있는건 "아.. 변경을 시도하면 값이 안에서 변경되는게 아니라 새로운 값을 복사해서 넣는구나"를 알수 있다. 이를 더 명확하게 증명하는것은 우리가 `didSet`에서  `OldValue`를 가져올수 있다는거다. 사용자가 두 버전(old, current)의 값을 동시에 참조할 수 있으므로 컴파일러는 값의 전체 복사본을 만들어야 할거다.
<br>
<br>
물론 컴파일러의 깊은곳을 들어가면 메모리 값의 실제 수정이 내부에서 발생할 가능성이 있더라도 우리가 코드 쓰는 의미 수준에서 전체 Bar 구조체는 완전히 새로운 값으로 대체 된다고 이해하는 것이 적절하다.
<br>
<br>

## 결론<br>
* Swift에서 Mutation은 참조 타입과 값 타입에 따라 동작이 다르다.
* 참조 타입은 인스턴스의 참조를 변경하는 것이고, 값 타입은 인스턴스의 값을 직접 변경하는 것이다.
* 하지만 논리적으로는 값타입은 "변경이 아닌 새로운 값을 복사한다"라고 이해해야 적절하다
<br>
<br>
<br>
<br>
<br>

+)Ref: <br>
https://forums.swift.org/t/why-are-structs-in-swift-said-to-be-immutable/55319/15


