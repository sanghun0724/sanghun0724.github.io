---
layout: post

---
<br/>

<br>

## Naming
<br>

#### 애매함을 피해라. 
```swift
extension List {
    public mutating func remove(at position: Index) -> Element
}
```
<br>

#### 필요없는 말은 생략해라

```swift
public mutating func removeElement(_ member: Element) -> Element
```

<br>

#### Name variables, parameters, and associated types 등 각자의 책임에 따라 네이밍, Not Type.. 

```swift
protocol ViewControlller {
    associatedtype ContentView: View
}
class ProductionLine {
    func restock(from supplier: WidgetFactory)
}
```
<br>

하지만, 만약 타입이 역할인경우? -> associated typed , Protocol 
```swift
protocol Sequence {
    associatedtype Iterator: IteratorProtocol
}

protocol IteratorProtocol {}
```
이런경우는 뒤에 Protocol이라 명시 

<br>

#### 매개 변수의 역할을 명확히 하기 위해 약한 타입 설명 추가 

```swift
func addObserver(_ observer: NSObject, forKeyPath path: String)
grid.addObserver(self, forKeyPath: graphics) // clear
```
<br>
<br>

## 유창한 사용을 위한 노력
<br>

#### 영어 문법대로 말이되게 method명, 파라메터명 만들기

```swift
x.insert(y, at: z)          “x, insert y at z”
x.subViews(havingColor: y)  “x's subviews having color y”
x.capitalizingNouns()       “x, capitalizing nouns”
```
<br>

#### 해당 메소드가 호출 중심이 아닌경우, 하나 또는 두개의 매개변수 이후 유창성이 떨어지는것은 괜찮다.

```swift
AudioUnit.instantiate(
  with: description,
  options: [.inProcess], completionHandler: stopProgressBar)
```
<br>

#### 팩토리 메소드의 시작은 `make` 붙이기 
(ex) foctory.makeInterator())

<br>
<br>

#### Side-Effect에 따라 함수와 메소드 이름 지정

    1. Side-Effect 없으면 명사구로 (ex) x.distance(to: y), i.successor())
    2. Side-Effect 있으면 명령형 동사로 (ex) x.sort(), x.append())
     더 나아가,해당 함수가 Nonmutating인 경우 “ed” or “ing” 붙이기
    3. 하지만, 메소드가 자연스럽게 명사구로 설명이 될때 --> 
    y.union(z)-> y.formUnion(z) j,
    c.successor(i)-> c.formSuccessor(&i)
    와 같이 “form” prefix해주기

<br>

#### Boolean 메소드나 다른 프로퍼티는 Nonmutating일 때, 수신하는 쪽의 Assertion 처럼 읽혀야 함.
(Ex) x.isEmpty, line1.intersects(line2))

<br>

#### 무엇인가 설명하는 Protocol은 명사로 읽혀야 함.

<br>

#### 기능을 설명하는 Protocol은 “able”, “ible”, “ing” 등을 뒤에 붙힘

<br>

#### Other types, properties, variables, and constants의 네임들은 명사들로 읽혀야 함.

<br>
<br>


## 용어를 잘 사용하기
<br>

> Term of Art :  특정 분야나 직업 내에서 정확하고 전문적인 의미를 갖는 단어나 문구.
<br>

#### 애매모호한 말들 피하기. Ex) 화려하고 겉만 번지르르한 단어들 말고 핵심적이고 본질적인 언어 써라.. 
<br>

#### 정해진 의미를 지켜라.
<br>
    1. Don’t surprise an expert. : 신조어 처럼 있던것 개조해서 말 만들지 말기 <br>
    2. Don’t confuse a beginner: 새로운 용어를 배우려는 사람은 본래 용어의 traditional한 의미를 먼저 배움. 그러니깐 건들이지 마.. 
<br>
<br>
#### 정해진 의미를 지켜라.약어, 축약어 피해라 

선례를 살펴보고 수용하기. 초보자에게 더 쉬운 용어를 제공하겠다고 선례 무시하고 신조어 만들지 말기.
Ex) not list -> Array OK 
(list가 초보자에게 더 쉬워보일수는 있어도 no no)

+ 특정 도메인에서는 sin(x) 처럼 쓰이는것들이 있는데 이걸 위와 같은 이유로 verticalPositionOnUnitCircleAtOriginOfEndOfRadiusWithAngle(x).와 같이 만들지 말기.

<br>
<br>

## Convention
<br>

### - General Conventions

<br>

#### O(1)이 아닌 Computed Property의 접근 시 Complexity을 문서화합니다. (오.. 이건 새롭다)

<br>

#### 그냥 생 free functions 보다 메소드나 프로퍼티를 선호하자.  (Free func는 거의 안써서.. 공감이 안됨)

<br>

#### 카멜 케이스 컨벤션 따르기

<br> 

#### 같은 의미 이거나 같은 도메인일경우, Method들은 base name을 끼리끼리 공유 할수 있다.
```swift
extension Shape {
  /// Returns `true` if `other` is within the area of `self`;
  /// otherwise, `false`.
  func contains(_ other: Point) -> Bool { ... }

  /// Returns `true` if `other` is entirely within the area of `self`;
  /// otherwise, `false`.
  func contains(_ other: Shape) -> Bool { ... }

  /// Returns `true` if `other` is within the area of `self`;
  /// otherwise, `false`.
  func contains(_ other: LineSegment) -> Bool { ... }
}
```
<br>

❌❌ 이 케이스는 different sementic이므로 리네이밍 해야함.
```swift
extension Database {
  /// Rebuilds the database's search index
  func index() { ... }

  /// Returns the `n`th row in the given table.
  func index(_ n: Int, inTable: TableID) -> TableRow { ... }
}
```
<br>

#### “overloading on return type”를 피해라 
❌❌ 타입 추론에 있어 모호성을 유발
```swift 
extension Box {
  /// Returns the `Int` stored in `self`, if any, and
  /// `nil` otherwise.
  func value() -> Int? { ... }

  /// Returns the `String` stored in `self`, if any, and
  /// `nil` otherwise.
  func value() -> String? { ... }
}
```
<br>
<br>
 
## Parameters
<br> 

#### Documentation에 알맞은 (어울릴만한) 파라메터 명 채택하기 

```swift
/// Return an `Array` containing the elements of `self`
/// that satisfy `predicate`.
func filter(_ predicate: (Element) -> Bool) -> [Generator.Element]

/// Replace the given `subRange` of elements with `newElements`.
mutating func replaceRange(_ subRange: Range, with newElements: [E])
```
<br>

❌❌ 어색하고 문법도 틀림
```swift
/// Return an `Array` containing the elements of `self`
/// that satisfy `includedInResult`.
func filter(_ includedInResult: (Element) -> Bool) -> [Generator.Element]

/// Replace the range of elements indicated by `r` with
/// the contents of `with`.
mutating func replaceRange(_ r: Range, with: [E])
```
<br>

Default Parameter 활용 하기 
(Simple을 위해)

```swift
extension String {
  /// ...description...
  public func compare(
     _ other: String, options: CompareOptions = [],
     range: Range? = nil, locale: Locale? = nil
  ) -> Ordering
}
```
<br>

매개 변수 리스트 마지막에 보통 default 파라메터 값을 넣어주면 좋다. 
Default 값이 없는 초기 파라메터가 보통 sementic 측면에서 중요하며, 안정적임

`#fileID` 쓰는거 선호하기. (Save space & protects developers privacy)

<br>
<br>

## Agument Labels 
<br> 

#### 인수를 유용하게 구분할수 없는 경우 라벨들을 다 없애라. <br>
(Ex)  min(number1, number2), zip(sequence1, sequence2) )

<br> 

#### 상위 타입 캐스팅 하는 경우 첫번째 라벨 없애기 
Ex)Int64(someUInt32), String(veryLargeNumber, radix: 16)

<br> 

#### 하위 타입 캐스팅 하는 경우는 라벨 살리기

```swift
extension UInt32 {
  /// Creates an instance having the specified `value`.
  init(_ value: Int16)            ← Widening, so no label
  /// Creates an instance having the lowest 32 bits of `source`.
  init(truncating source: UInt64)
  /// Creates an instance having the nearest representable
  /// approximation of `valueToApproximate`.
  init(saturating valueToApproximate: UInt64)
}
```

<br>

#### 첫번째 인자가 전치사구인 경우, 인자 라벨을 제공하기 
Ex) x.removeBoxes(havingLength: 12).

<br> 
<br> 

❌❌  하지만 여러개의 인수들이 한가지의 추상화를 하고 있다면 Exeption
```swift
a.move(toX: b, y: c)
a.fade(fromRed: b, green: c, blue: d)
```
<br>
-> into
<br>

```swift
a.moveTo(x: b, y: c)
a.fadeFrom(red: b, green: c, blue: d)
```
<br>

첫번째 인자가 문법적인 표현을 구사하는 경우,  라벨 생략 -> 대신 func base name에 포함시키기 
Ex) x.addSubview(y)

<br>
\
**반대로 말하면 첫번째 인자가 문법적으로 말하고 있지 않으면 라벨이 필요하단거임 <중요>**
```swift 
view.dismiss(animated: false)
let text = words.split(maxSplits: 12)
let studentsByName = students.sorted(isOrderedBefore: Student.namePrecedes)
```
<br>

말했던 라벨 안붙이는 특이케이스 제외하고 나머지는 모두 다 인자에 라벨 붙여라. 

<br>
<br>

## Special Instructions
<br>

#### 튜플이나 클로져 파라메터에 라벨을 지정해라 

```swift 
/// - Returns:
///   - reallocated: `true` if a new block of memory
///     was allocated; otherwise, `false`.
///   - capacityChanged: `true` if `capacity` was updated;
///     otherwise, `false`.
mutating func ensureUniqueStorage(
  minimumCapacity requestedCapacity: Int,
  allocate: (_ byteCount: Int) -> UnsafePointer<Void>
) -> (reallocated: Bool, capacityChanged: Bool)
```
<br>
#### top-level-func가 있으면 그 함수의 parameter 네이밍 하듯이 지어라.

<br>
<br>

#### Unconstrained 한 다형성을 다룰때는  오버로드 묶음들 중에서 애매함을 피하기 위해  좀더 신경써라

<br>


❌❌  뭐가 실행되는지 애매함
```swift 
struct Array {
  /// Inserts `newElement` at `self.endIndex`.
  public mutating func append(_ newElement: Element)

  /// Inserts the contents of `newElements`, in order, at
  /// `self.endIndex`.
  public mutating func append(_ newElements: S)
    where S.Generator.Element == Element
}


var values: [Any] = [1, "a"]
values.append([2, 3, 4]) // [1, "a", [2, 3, 4]] or [1, "a", 2, 3, 4]?
```
<br>

#### 명확하게 메소드 시그니쳐 추가 
```swift 
struct Array {
  /// Inserts `newElement` at `self.endIndex`.
  public mutating func append(_ newElement: Element)

  /// Inserts the contents of `newElements`, in order, at
  /// `self.endIndex`.
  public mutating func append(contentsOf newElements: S)
    where S.Generator.Element == Element
}
```
<br>
<br>
<br>

# 코코아 터치 기반 Bool 케이스 정리 
<br>

is + 명사 
“(무엇)인가?” 
```swift 
func isDescendant(of view: UIView) -> Bool //UIView: "view의 자식인가?"
```
<br>

#### is +  현재진행형(~ing)
“~하는 중인가?”
```swift 
var isExecuting: Bool { get } // operation: 오퍼레이션이 진행중인가?
var isPending: Bool { get } // MSMessage: “메세지가 보내기전 대기중인가?”
```
<br>

#### is + 형용사
case 1: 단어 자체가 형용사인것 : opaque, readable, visible 등
case 2: 과거분사 형태 (수동태라고 생각): hidden, selected, highlighted, completed
```swift
var isOpaque: Bool { get set }
var isSelected: Bool { get set }
var  isHighlighted: Bool { get set }
var isHidden: Bool { get set }
```
<br>

#### ❌ is + 동사원형은 피하기 ❌

```swift
// 모호함
var isEdit 

// 명확함
var isEditable: Bool // 편집 가능한가?
var isEditing: Bool // 편집 중인가?
var canEdit: Bool // 편집할 수 있는가?
```
<br>
(다만 상황과 컨벤션에 따라 조정 가능)

<br>
<br>

### 조동사 

#### can
“~ 할수 있는가?”
```swift
var canBecomeFirstResponder: Bool { get }
```
<br>

#### should, will
“~해야 하는가?” or “~ 할 것인가?”
```swift
var shouldRefreshRefetchedObjects: Bool { get }
```
<br>
<br>

### has

#### has로 시작하는 Bool 유형 2가지, 서로 뜻이 다름

#### has + 명사 
“~를 가지고 있는가?”
```swift
var hasiCloudAccount: Bool { get }
var hasVideo: Bool { get }
```
<br>

#### has + 과거분사 
“~가 현재까지 유지되고 있는가?”
```swift
var hasConnected: Bool { get }
var hasEnded: Bool { get }
```
<br>
<br>

### 동사원형

#### 3인칭 단수 써라.
<br>
- supports: ~을 지원하는가?
- includes: ~을 포함하는가?
- shows: ~을 보여줄것인가?
- accepts: ~을 받아 주는가?
- contains: ~을 포함하고 있는가?
- returns: 리턴하는가?
- preserves: 보존하는가?
<br>

```swift
var supportsVideo: Bool //CXProviderConfiguration: 비디오를 지원하는가?
var includesCallsInRecents: Bool //CXProviderConfiguration
var showsBackgroundLocationIndicator: Bool //CLLocationManager
var allowsEditing: Bool //CNContactViewController: 편집을 허용하는가?
var acceptsFirstResponder: Bool //NSResponder
var preservesSuperviewLayoutMargins: Bool //UIView
var returnsObjectsAsFaults: Bool //NSFetchRequest
```
<br>
(예외적으로 3인칭 단수 안 쓰는 경우도 있음)
-> 주체에 따라 
<br>
<br>
<br>

# 변수 네이밍
<br>

#### swift getter 
어떤 인스턴스를 리턴하는 함수나 메소드에 get 안씀
-> 바로 타입이름(명사)로 
```swift
func distance(from location: CLLocation) -> CLLocationDistance 
```
<br>

#### 결과를 바로 리턴하는 ’fetch’
(오래 걸리지 않는 동기적 작업, 요청이 실패하지 않음)
```swift
class func fetchAssets(withLocalIdentifiers identifiers: [String], options: PHFetchOptions?) -> PHFetchResult<PHAsset>
```
<br>

#### 유저에게 요청하거나 작업이 실패할 수 있을 때 ‘Request’
```swift
func requestData(for resource: PHAssetResource, options: PHAssetResourceRequestOptions?, dataReceivedHandler handler: @escaping (Data) -> Void, completionHandler: @escaping (Error?) -> Void) -> PHAssetResourceDataRequestID
```
<br>

#### 작업의 단위가 클로져나 Request로 래필 되어있으면 ‘Perform’ or ‘excute’
```swift
//NSManagedObjectContext
func perform(_ block: @escaping () -> Void)

//CNContactStore
func execute(_ saveRequest: CNSaveRequest) throws
```
<br>

#### XXXError
```swift
// public error인경우
enum [모듈이름]Error {} 

// internal error인경우
[구체적]error
```

<br>
<br>
<br>

# 전치사

#### 전치사 어떻게 사용할까
<br>

| 전치사 | 의미 | 예시 |
|---|---|---|
| to | 입력으로 제공된 데이터를 출력으로 반환합니다. | `add(x: Int, y: Int) -> Int` |
| from | 입력으로 제공된 데이터를 가져옵니다. | `load(data: Data, from: URL)` |
| with | 입력으로 제공된 데이터를 사용하여 작업을 수행합니다. | `sort(numbers: [Int], with: <#SortAlgorithm#>)` |
| on | 입력으로 제공된 데이터에 작업을 수행합니다. | `draw(shape: Shape, on: Canvas)` |
| in | 입력으로 제공된 데이터 내에서 작업을 수행합니다. | `iterate(over: [Int], in: { (number: Int) in ... })` |
| into | 입력으로 제공된 데이터를 다른 데이터로 변환합니다. | `convert(number: Int, into: String)` |
| as | 입력으로 제공된 데이터의 타입을 변경합니다. | `cast(value: Any, as: Int)` |
| by | 입력으로 제공된 데이터를 사용하여 작업을 수행합니다. | `filter(numbers: [Int], by: { (number: Int) in number % 2 == 0 })` |
| for | 입력으로 제공된 데이터의 모든 항목에 대해 작업을 수행합니다. | `forEach(numbers: [Int], for: { (number: Int) in ... })` |

<br>
<br>


#### 전치사 사용 전에 끊어라
```swift
func chapterMetadataGroups(withTitleLocale locale: Locale, containingItemsWithCommonKeys)
func determineCompatibleFileTypes(withCompletionHandler handler: ([String]) -> Void)
```

<br>
<br>
<br>
<br>
Ref)<br>
(개인 블로그 내용은 원작자의 허락을 구했습니다.)<br>
https://soojin.ro/blog/naming-boolean-variables<br>
https://soojin.ro/blog/english-for-developers-swift<br>
https://forums.swift.org/t/guidelines-first-argument-labels-prepositions-inside-the-parens/1369/18<br>
https://github.com/apple/swift-3-api-guidelines-review/commit/da7e512cf75688e6da148dd2a8b27ae9efcb8821?diff=split
