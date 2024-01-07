---
layout: post

---
<br/>

<br>

optional은 swift programming에서 가장 자주 사용하는 문법중 하나이다. 하지만, (이전의 나 역시도) optional이 어떤 형태를 갖추었고 어떻게 구현되어있는지 '정확히' 알지 못하여 이번 기회를 통해 알게 된 것들을 기록하고 서술한다.

<br> 
<br> 

## Optional is Enum

<br>

optional은 단지 enum 이다. 이는 구현부를 보게되면 쉽게 확인해 볼 수 있다.

<br>

```swift 
@frozen
public enum MyOptional<Wrapped>: ExpressibleByNilLiteral {
    case none
    case some(Wrapped)
```

<br>

이를 기반으로 이런식으로 enum형태로 optinal을 사용할 수 있다.

<br>

```swift 
func returnSomeOptional(_ num: Int) -> Optional<Int> {
    if num % 2 == 0 {
        return .some(num)
    } else {
        return .none
    }
}
```

<br>

이걸 Short 버전으로 줄인 버전이 바로 우리가 흔히쓰는 옵셔널 문법이다. 

<br>

```swift 
// shorterWay

func returnSomeOptinal(_ num: Int) -> Int? {
    if num % 2 == 0 {
        return num
    } else {
        return nil
    }
}
```

<br>
<br>

## Switch Optional

<br>

이어 옵셔널이 enum이기 때문에 다음과 같이 switch문으로도 사용 가능하다. 

<br>

```swift
func switchSomeOptional(_ num: Int?) -> String {
    switch num {
    case 22?: return "hello 22?"
    case (1..<100)?: return "1~100"
    case let number?: return "number: \(number)"
    case nil: return "nil"
    }
}
```

<br>
<br>

## Nil-coalescing operator

<br>
<br>

옵셔널의 ?? 연산자를 통해 기존보다(연산자를 사용하기 전보다) 더욱 쉽게 옵셔널을 언래핑 할 수 있다. 이어,  실패할 경우 Default Value를 리턴하도록 한다. 
<br>
<br>

```swift
var name = BookStore.store?.book?.name ?? "Default Value"
print(name)
```
<br>

구현을 보게 되면, 

<br>
<br>

```swift 
public func ?? <T>(optional: T?, defaultValue: @autoclosure () throws -> T)
    rethrows -> T {
  switch optional {
  case .some(let value):
    return value
  case .none:
    return try defaultValue()
  }
}
```
<br>
<br>

사실 설정했던 Default Value가 closure라는 사실을 알 수 있다. 최적화 관점에서, 디폴트 값을 실제로 사용하지 않을시, 클로져는 실행되지 않기 때문에 효율성이 증가한다. 
<br>
표현식으로는 @autoclosure를 사용하여 (클로져임에도 불구하고) 괄호를 생략하여 사용 가능 했던 것 이였다. 
<br>
<br>

## map & flatMap
<br>
<br>

Swift에서는 collection뿐만아니라 optional도 값 변환을 위한 오퍼레이션인 map과 flatMap을 제공한다. 
<br>

### 1. Map 
<br>
```swift
let number: Int? = Int("100")
let mappedNumber = number.map { (Int("\($0 + $0)")) }
print(mappedNumber)
// prints: Optional(Optional(200))
```

<br>
<br>

### 2. flatMap
<br>
Map과 거의 같지만, flatMap은 말그대로 값을 한번 flat하게 처리한다. 
<br>

```swift
let number: Int? = Int("100")
let mappedNumber = number.flatMap { (Int("\($0 + $0)")) }
print(mappedNumber)
// prints: Optional(200)
```
<br>
<br>
좀더 자세한 차이를 살펴보려면 구현부를 찾아보면 된다. 
<br>
<br>

```swift 
 @inlinable
    public func map<U>(
      _ transform: (Wrapped) throws -> U
    ) rethrows -> U? {
        switch self {
        case .some(let y):
            return .some(try transform(y)) // some에 담아서 처리 
        case .none:
            return .none
        }
    }
    
    @inlinable
    public func flatMap<U>(
        _ transform: (Wrapped) throws -> U?
    ) rethrows -> U? {
        switch self {
        case .some(let y):
            return try transform(y)
        case .none:
            return .none
        }
    }
```
<br>
<br>
map의 경우  .some(try transform(y))와 같이 처리하여, 말 그대로 map을 수행하는 self객체(optional)을 감싸서 반환하게 된다.
하지만 flatMap의 경우, 바로 try transform(y)으로 받은 클로져값을 그대로 리턴해준다. 

<br>
<br>
<br>

### 전체 Custom Optional Implementation
<br>
<br>

``` swift 
//
//  MyOptinal.swift
//
//  Created by 이상헌 on 2023/10/02.
//

@frozen
public enum MyOptional<Wrapped>: ExpressibleByNilLiteral {
    case none
    case some(Wrapped)
    
    @_transparent
    public init(_ some: Wrapped) { self = .some(some) }
    
    @inlinable
    public func map<U>(
      _ transform: (Wrapped) throws -> U
    ) rethrows -> U? {
        switch self {
        case .some(let y):
            return .some(try transform(y))
        case .none:
            return .none
        }
    }
    
    @inlinable
    public func flatMap<U>(
        _ transform: (Wrapped) throws -> U?
    ) rethrows -> U? {
        switch self {
        case .some(let y):
            return try transform(y)
        case .none:
            return .none
        }
    }
    
    // var i: Index? = nil
    @_transparent
    public init(nilLiteral: ()) {
        self = .none
    }
    
    @inlinable
    public var unsafelyUnwrapped: Wrapped {
        @inline(__always)
        get {
            if let x = self {
                return x
            }
            fatalError("unsafelyUnwrapped of nil optional")
        }
    }
    
    @inlinable
    internal var _unsafelyUnwrappedUnchecked: Wrapped {
      @inline(__always)
      get {
        if let x = self {
          return x
        }
          fatalError("_unsafelyUnwrappedUnchecked of nil optional")
      }
    }
    
    ///     if let numberOfShoes = numberOfShoes.take() {
    ///       print(numberOfShoes)
    ///       // Prints "34"
    ///     }
    internal mutating func _take() -> Wrapped? {
        switch self {
        case .some(let wrapped):
            self = nil
            return wrapped
        case .none:
            return nil
        }
    }
    
    
}

extension MyOptional: Equatable where Wrapped: Equatable {
    public static func ==(lhs: Wrapped?, rhs: Wrapped?) -> Bool {
      switch (lhs, rhs) {
      case let (l?, r?):
        return l == r
      case (nil, nil):
        return true
      default:
        return false
      }
    }
}

// MARK: - Hashable

extension MyOptional: Hashable where Wrapped: Hashable {
    @inlinable
    public func hash(into hasher: inout Hasher) {
        switch self {
        case .none:
            hasher.combine(0 as UInt8)
        case .some(let wrapped):
            hasher.combine(1 as UInt8)
            hasher.combine(wrapped)
        }
    }
}

// MARK: - ?? operator

extension MyOptional {
    @inlinable
    static public func ?? <T>(optional: T?, defaultValue: @autoclosure () throws -> T)
    rethrows -> T {
        switch optional {
        case .some(let value):
            return value
        case .none:
            return try defaultValue()
        }
    }
}

```

<br>
<br>
<br>
<br>

+)Ref:

<br>
https://github.com/apple/swift
https://agostini.tech/2017/09/24/understanding-optionals/
https://developer.apple.com/documentation/swift/optional/flatmap(_:)
