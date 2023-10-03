---
layout: post

---
<br/>

<br>

Swift의 Dictionary, Set 등 HashCollection이 실제로 어떻게 구현되어있을까? 
내가 알고있던 cs 지식들과 똑같이 구현되어 있을까 아니면 다르게 구현되어 있을까. 좀 더 Swift의 정확한 이해를 위해 오늘은 실제 Swift 구현부들을 조사해 보았다.
조사한 방법은 내 평소 코드에서 단순 HashCollection에 대해 읽고 쓰는 부분들이 어떻게 타고 타고 가서 실행되는지 구현 코드들을 살펴보고 이해했다. 

<br>
<br>

# Dictionary

<br>
<br>

## 1. updateValue()를 하면 무슨일이 일어날까 

<br>
<br>

```swift
var dict: [Int: String] = [1: "2"]

let oldValue = dict.updateValue("3", forKey: 1)
print(oldValue) // Optional("2")
```
<br>

평소에 updateValue 함수를 쓰지 않더라도 dictionary를 업데이트 하는 경우는 많을 것이다. 첫번째로는 이부분 들이 어떻게 실행되고 있는지 한번 이해해보자.

<br>
<br>
-> Dictionary._Variant로 이동
<br>
( _Variant는 Dictionary가 가지고 있는 구조체를 뜻한다. 안에 변수로 _NativeDictionary를 가지고 있다. )
<br>
-> _NativeDictionary.updateValue()로 이동
<br>
<br>

```swift
 @inlinable
  internal mutating func updateValue(
    _ value: __owned Value,
    forKey key: Key,
    isUnique: Bool
  ) -> Value? {
    let (bucket, found) = mutatingFind(key, isUnique: isUnique) // 키에 해당하는 버킷을 찾기
    if found {
      let oldValue = (_values + bucket.offset).move() // 예전값은 deinitialize 후
      (_values + bucket.offset).initialize(to: value) // 새로운 값을 업데이트
      return oldValue // 이전 값 반환
    }
    _insert(at: bucket, key: key, value: value) // 존재하지 않을 시 해당 버킷 삽입
    return nil
  }
```
<br>
<br>

그리고서는 위의 주석과 같이 코드를 실행 해준다. If found {}  부분을 보면 왜 Dictionary에서 쓰던 updateValue() 리턴값이 OldValue였는지 이해할 수 있다. 
지금은 key에 해당하는 버킷을 못찾았다 가정하고 _insert 구현부로 바로 이동해 보겠다. 
<br>
<br>

```swift
 @inlinable
  internal func _insert(
    at bucket: Bucket,
    key: __owned Key,
    value: __owned Value) {
    _internalInvariant(count < capacity)
    hashTable.insert(bucket) // hash table에 butcket bit 삽입
    uncheckedInitialize(at: bucket, toKey: key, value: value) // 메모리에 key value 저장
    _storage._count += 1 // storage count 하나 늘리기
  }
```
<br>
<br>

_insert에서는 hashTable에 bucket bit를 하단 코드와 같이 uncheckedInsert()해주어 hashTable이 가지고 있는 words라는 메모리값 array에 bit를 삽입한다. 

<br>
<br>

-> HashTable
```swift 
 @inlinable
  @inline(__always)
  internal func insert(_ bucket: Bucket) {
    _internalInvariant(!isOccupied(bucket)) // 버켓이 이미 테이블에 있는지 확인
    words[bucket.word].uncheckedInsert(bucket.bit) // 버켓의 비트위치에 해당하는 비트를 해시 테이블에 삽입
  }
```
<br>

> word: 8바이트(64비트)로 이루어진 행

<br>

그리고서는  uncheckedInitialize()를 통해 Bucket의 offset에 key 또는 value의 값을 더한 주소에 값을 넣어준다. 
<br>

```swift 
@inlinable // FIXME(inline-always) was usableFromInline
  @inline(__always)
  internal func uncheckedInitialize(
    at bucket: Bucket,
    toKey key: __owned Key,
    value: __owned Value) {
    defer { _fixLifetime(self) }
    _internalInvariant(hashTable.isValid(bucket))
    (_keys + bucket.offset).initialize(to: key)
    (_values + bucket.offset).initialize(to: value)
  }
```
<br>
<br>
<br>

## 2. 그럼 우리가 저장한 이 값을 어떻게 찾을까?

<br>


먼저 Dictionary에 subscript로 get {}이 발생했을 경우를 제목과 같다고 가정하고 살펴보겠다. <br>
앞에서와 같이 _variant란 객체를 통해 lookup을 실행한다.
<br>

```swift
@inlinable
  public subscript(key: Key) -> Value? {
    get {
      return _variant.lookup(key)
    }
    set(newValue) {
      if let x = newValue {
        _variant.setValue(x, forKey: key)
      } else {
        removeValue(forKey: key)
      }
    }
    _modify {
      defer { _fixLifetime(self) }
      yield &_variant[key]
    }
  }
```
<br>
<br>

-> Dictionary._Varian 
<br>
-> _NativeDictionary : DictionaryBuffer
<br>
<br>

이부분도 역시 방금봤던 케이스와 같이 _NativeDictionary 이동한다. 여기서 눈여겨봐야할점은 _NativeDictionary가 DictionaryBuffer라는 프토코콜을 채택하고 있다는 점이다. 
DictionaryBuffer에는 lookup()이라는 함수가 정의되어있어. _NativeDictionary에 구현부가 있다. 

<br>
<br>

- Lookup implementation

<br>

```swift
 @inlinable
  @inline(__always)
  func lookup(_ key: Key) -> Value? {
    if count == 0 {
      // Fast path that avoids computing the hash of the key.
      return nil
    }
    let (bucket, found) = self.find(key) // << find key 로 이동 
    guard found else { return nil }
    return self.uncheckedValue(at: bucket)
  }
```
<br>
<br>

```swift
@inlinable
  @inline(__always)
  internal func find(_ key: Key) -> (bucket: Bucket, found: Bool) {
    return _storage.find(key)
  }
```
<br>
<br>
-> __RawDictionaryStorage
<br>

타고타고 들어온 __RawDictionaryStorage 에서는 key와 key가 가지고 있는 hash value에 이 딕셔너리 인스턴스에 사용되는 seed를 넣어 find를 돌린다.

<br>
 
```swift
@_alwaysEmitIntoClient
  @inline(never)
  internal final func find<Key: Hashable>(_ key: Key) -> (bucket: _HashTable.Bucket, found: Bool) {
    return find(key, hashValue: key._rawHashValue(seed: _seed))
  }
```
<br>

그리고 드디어 여기서 임시 버켓을 하나 만들어 이 버켓이 hashTable에 속하는지 + bucket의 key가 내가 찾으려고 하는 key와 일치하는지 판별한 후 값을 결과에 따라 버켓과 함께 리턴한다.

<br>
<br>

```swift
@_alwaysEmitIntoClient
  @inline(never)
  internal final func find<Key: Hashable>(_ key: Key, hashValue: Int) -> (bucket: _HashTable.Bucket, found: Bool) {
      let hashTable = _hashTable
      var bucket = hashTable.idealBucket(forHashValue: hashValue) /// hashvalue를 가지고 있는 bucket하나를 임시로 만들고
      while hashTable._isOccupied(bucket) { // hashtable이 이 버켓을 점유하고 있다면 while문을 타게 한다.
        if uncheckedKey(at: bucket) == key { // 메모리에서 버켓을 통해 동일한 키값을 찾는다면 리턴한다.
          return (bucket, true)
        }
        bucket = hashTable.bucket(wrappedAfter: bucket) // &+ 1로 버켓 돌려주기
      }
      return (bucket, false)
  }
```
<br>
<br>


++)  uncheckedKey(): 구현부 keys 저장되어 있는 메모리 지역 접근 후 offset으로 key 리턴
```swift
@_alwaysEmitIntoClient
  @inline(__always)
  internal final func uncheckedKey<Key: Hashable>(at bucket: _HashTable.Bucket) -> Key {
    defer { _fixLifetime(self) }
    _internalInvariant(_hashTable.isOccupied(bucket))
    let keys = _rawKeys.assumingMemoryBound(to: Key.self)
    return keys[bucket.offset]
  }

```

<br>
<br>
<br>

# Set

<br>
<br>

## Set에는 어떻게 구현이 되어있을까?
<br>
<br>

신기한 점은 Set 또한 hashTable을 사용하여 lookup을 하기 때문에 유사하게 구현되어 있는걸 발견할 수 있다.
<br>

```swift
  @inlinable
  public func contains(_ member: Element) -> Bool {
    return _variant.contains(member)
  }
```
<br>

 여기서는 해당 메소드를 정의하는  (DictionaryBuffer 대신) _SetBuffer가 존재하고 (_NativeDictionary 대신)  _NativeSet이 이걸 구현하고 있다. 
<br>

```swift
@inlinable
  @inline(__always)
  internal func find(_ element: Element) -> (bucket: Bucket, found: Bool) {
    return find(element, hashValue: self.hashValue(for: element))
  }
```
<br>
하단 메소드를 보면, 어디서 많이본 메소드 형태이다. 방금 Dictionary에서 쓰던 find함수 구현 내용과 동일하게 _NativeSet에서도 구현할 수 있는것을 알 수가 있다. <br>
여기서 알 수 있는건 set도 이렇게 “Hash lookup을 통해  값을 find할 수있구나” 와 그래서 “contain이 O(1)에 실행이 되었구나.” 를 알 수가 있다. 
<br>
<br>

```swift 
 @inlinable
  @inline(__always)
  internal func find(
    _ element: Element,
    hashValue: Int
  ) -> (bucket: Bucket, found: Bool) {
    let hashTable = self.hashTable
    var bucket = hashTable.idealBucket(forHashValue: hashValue)
    while hashTable._isOccupied(bucket) {
      if uncheckedElement(at: bucket) == element {
        return (bucket, true)
      }
      bucket = hashTable.bucket(wrappedAfter: bucket)
    }
    return (bucket, false)
  }
```
<br>
<br>
<br>

## +) Hash Table , hash value 저장
<br>

해쉬테이블을 검색하면 해당 그림이 가장 먼저 나온다. 개인적으로 이 그림은 조금 오해의 소지가 있다고 본다. 
<br>

<img src=“https://github.com/sanghun0724/sanghun0724.github.io/assets/66512239/5d5a0aab-efff-42da-a7df-0389eec4e4a3”>
![image](https://github.com/sanghun0724/sanghun0724.github.io/assets/66512239/5d5a0aab-efff-42da-a7df-0389eec4e4a3)
<br>
<br>

당연히 이론적으로는 해쉬테이블의 인덱싱원리를 잘 표현하였다. 하지만, 이번에 Hash Collection들을 까보면서 느끼는 것은 조금 다르다. 
이전에는 그림과 같이  내가 dictionary에 key를 대입하면 hash func가 돌아서 hash value가 나와서 그값으로 인덱싱을 하는걸로 이해했었지만, 애초에 swift에서는 Hashable한 값들만을 key로 받는다. 
즉, 애초에 HashValue를 가진 키를 가지고 인덱싱을 하는거다. 

<br>
<br>

그리고 좀 더 구체적으로 보면 `hash value`를 만들때도 `해쉬 함수`자체 로만 값을 만드는 것이 아니라 swift에서는 충돌성을 낮춰주기 위해 `seed`라는 값을 받아 xor을 돌려 엄청난 난수를 만든 후  그 값으로 해쉬함수를 돌린다. 

<br>

```swift
internal struct _State {
    // "somepseudorandomlygeneratedbytes"
    private var v0: UInt64 = 0x736f6d6570736575
    private var v1: UInt64 = 0x646f72616e646f6d
    private var v2: UInt64 = 0x6c7967656e657261
    private var v3: UInt64 = 0x7465646279746573
    // The fields below are reserved for future use. They aren't currently used.
    private var v4: UInt64 = 0
    private var v5: UInt64 = 0
    private var v6: UInt64 = 0
    private var v7: UInt64 = 0

    @inline(__always)
    internal init(rawSeed: (UInt64, UInt64)) {
      v3 ^= rawSeed.1
      v2 ^= rawSeed.0
      v1 ^= rawSeed.1
      v0 ^= rawSeed.0
    }
  }
```

<br>
<br>
<br>
+)Ref
<br>
https://github.com/apple/swift

