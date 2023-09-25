---
layout: post

---
<br/>
### 메모리 관리<br>

"**메모리 관리**(memory management)는 [컴퓨터 메모리](https://ko.wikipedia.org/wiki/%EC%BB%B4%ED%93%A8%ED%84%B0_%EB%A9%94%EB%AA%A8%EB%A6%AC)에 적용된 [리소스](https://ko.wikipedia.org/wiki/%EC%8B%9C%EC%8A%A4%ED%85%9C_%EB%A6%AC%EC%86%8C%EC%8A%A4) 관리의 일종이다. 가장 단순한 형태의 메모리 관리 방법은 프로그램의 요청이 있을 때, 메모리의 일부를 해당 프로그램에 할당하고, 더 이상 필요하지 않을 때 나중에 다시 사용할 수 있도록 할당을 해제하는 것이다" - 위키피디아<br>


#### 메모리 관리 기법이 필요한 이유 또는 배워야 하는 이유가 뭘까?<br>

여러 이유들이 있겠지만 내가 생각했을때 앱개발자로서  가장 큰 이유는 2가지가 있다.
![국가별 인기 스마트폰 기종 - 2018년 7월 기준](https://kimchunsick.me/static/f7774dfe19783a9066c75ab3dc48d231/6a068/favorite-smartphones.jpg)
1.  메모리를 최적화 한다면 너 낮은 사양의 기기들을 지원 가능. 더많은 유저 확보 가능

1.  더 나은 유저 서비스 제공 가능, 메모리 릭 또는 댕글링 포인터 등의 문제 발생 방어 가능.

<br>
### 메모리 관리 기법 종류들<br>

앱 개발 레이어에서는 메모리 관리 기법을 자동과 수동 기법으로 양분 해볼 수 있다.

-   자동 메모리 관리

    -   GC(Garbage Collector): 프로그램 실행 중에 사용되지 않는 메모리를 자동으로 해제하는 기법
    -   ARC(Automatic Reference Counting): 객체의 참조 카운트를 사용하여 객체의 생명주기를 관리하는 기법

-   수동 메모리 관리

    -   Malloc/Free: 개발자가 직접 메모리를 할당하고 해제하는 기법
    -   New/Delete: C++에서 사용하는 메모리 관리 기법

  <br>

## MRC<br>

실제로 Objective-C에서는 MRC라는 ‘수동 메모리 관리 기법’을 사용했었다.<br>

```
- (void)setName:(NSString *)newName {
[newName retain];
[name release];
name = newName;
}
```
<br>
-   retain : **retain count**(= reference count) **증가**를 통해 현재 Scope에서 **객체가 유지**되는것을 보장<br>
-   release : **retain count**(= reference count)를 **감소**시킴. retain 후에 필요 없을 때 release 함<br>
<br>
이런 식으로 retain과 relase를 사용하여 메모리 관리를 해주고는 했다. 하지만 프로그램이 커질수록 하나하나 인스턴스에 대한 메모리 관리를 해주는 것이 쉽지 않았고 그에 대한 대안으로 Apple에서는 ARC를 소개한다.

  

## ARC vs GC<br>

ARC란 자동 메모리 관리 기법에 속하는 방법 중 하나로 개발자가 신경쓸 필요없이 자동으로 Compile Time에 실행 되어 동적으로 Reference count를 세고 메모리 관리를 해주는 방법이다.
<br>
  

비슷한 방법으로 GC (**Garbage Collection**) 이라는 Jva or C#, GO 등에서 쓰이는 메모리 관리 기법 이 있는데 차이점을 서술하자면 이렇다.<br>

**GC**: 이미 할당된 **메모리에서 더 이상 사용하지 않는 메모리를 해제하는 기술**

-   런타임시, 백그라운에서 사용되지 않는 개체와 개체 그래프를 감지하지 않는 방식으로 작동
-   런타임 메모리가 부족하거나, 특정시간이 경과한 후 등의 불확실한 시점에 작동
-   retain cycle을 포함하여 전체 객체 그래프를 한번에 관리
<br>

**ARC**: 수동으로 개발자가 직접 retain/release를 통해 **reference counting을** 관리해야하는 부분을 **자동으로 관리해주는 기술**

-   객체가 사용되지 않을때, 실시간으로 메모리에서 release
-   컴파일 타임에 작동, 백그라운드 처리가 없으므로 메모리에 더 효과적
-   하지만 retain cycle을 자동으로 해결 x<br>


### + ARC가 Retain Cycle을 해결 못하는 이유<br>

GC같은 경우 reachable 객체를 실시간으로 살펴보며 작동한다. 이 객체의 외부참조가 없는 것을 감지하면, 서로를 참조하는 객체 그래프 전체를 버리기 때문에 retain cycle이 발생하지 않는다.<br>

반면 ARC는 GC보다 더 낮은 수준에서 작동하며, 참조 카운트를 기반으로 생명 주기를 관리하기 때문에 설계 상, retain cycle을 자동으로 처리할 수는 없다.

이와 같은 문제를 해결하기 위해, strong, weak, unowned과 같은 Storage Modifier가 도입된 것이다.

** Reachable: 사용하고 있는 메모리

  

## ARC Proposal

잠깐, 자동 메모리 관리 방법에는 ARC방법 뿐만 아니라 GC도 존재한다. 근데 왜 애플은 GC가 아닌 ARC를 도입 했을까?

  

사실 애플에서는 GC를 도입한적이 있다. (ios는 아니고 os x 에서만)

하지만 금방 os x에서도 deprecated 되었다. (10.5 ~. 10.8 OS X Mountain Lion )

사실 이문제는, 효율성에 대한 문제이다. 두 시스템 모두 장단점이 있으며 애플은 단지 선택했을 뿐이다. GC보다는 ARC가 시스템에 더 효율적이라고 판단한 것이다.  두 기법의 차이점을 통해 구체적으로 그 기준을 살펴보자.

  

-   ARC는 전체적으로 GC보다 느릴수 있다. 애플은 이 문제가 크지 않다고 판단한 듯 하다.
-   앞서 말했듯이, ARC는 Retain Cycle을 자동으로 처리해주지 않지만, 이를 위한 대안으로 Storage Modifier을 제공하고, 해당 부분은 개발자의 몫으로  넘어가길 택했다.
-   GC는 ARC보다 더 많은 메모리가 필요하다 (경우에 따라 최대 2배). 애플은 모바일 디바이스의 RAM이 너무 적어서 GC에 낭비할 수없다고 판단한 듯하다. 이어 이 부분은 안드로이드 휴대폰이 기본적으로 더 많은 메모리를 갖는 이유이다. (GC가 없을 경우에 비해 1.8~2.0x정도 많은 메모리를 사용하게 됨)
-   GC는 더 빠르게 작동할 수 있지만, 트리거 되는 시점에 잠재적으로 몇 밀리초동안 프로그램이 멈춘다. 만약 중요한 일을 한다면 이 시간은 꽤나 Critical 할수도 있다. 반해 ARC는 약간 느리지만 실행은 원활하다. 애플이 판단하길 잠재적인 문제를 일으킬수 있는 리스크를 지는것 보다 느리더라도 원활한 실행 환경을 더 중요하게 생각했다고 볼 수 있다.

  
##  Reference Counting

다시 돌아와서 그럼 ARC를 통한 참조 카운팅이 어떻게 이루어질까. 레퍼런스 카운팅은 그대로 말하자면, 참조 횟수이다. 즉, 이 인스턴스를 누가 얼마나 가르키고 있느냐를 나타낸 것이다.  그 기준은 뭔지 실제 코드를 보며 알아보자

-   Reference Counting ‘플러스’ + 일때

     - **인스턴스의 주소값을 변수에 할당할 때**

        1.  인스턴스를 새로 생성하는 경우
        ```
        var pulpleCar: Car? = Car(wheel: "pulple")
        ```

        2.  기존 인스턴스를 다른 변수에 대입하는 경우
        ```
        let otherCar = pulpleCar
        ```
  <br>

-   Reference Counting ‘마이너스’ - 일때

    - Object lifetime은 use-based 이다. 마지막 참조 사용시 직후 release를 실행
    
        1.  인스턴스를 가르키던 변수가 메모리에서 해제 되었을 시

            위와 같은 이유로 마지막 참조 사용시 직후 release된다면 count - 1
            ```
            func test() {
              let traveler1 = Traverler(name: "Lily") // 초기화는 참조카운트를 1로 설정
              // traveler2에 대해 retain
              let traveler2 = traveler1
              // release - code
              // traveler1 참조를 마지막으로 사용한 직후 release 코드 삽입
              traveler2.destination = "Big Sur"
              // traveler2에 대해 release
              print("Done traveling")
            }
            ```
 
        2.  nill 지정시
        ```
        var pulpleCar = nil
        ```

        3.  변수에 다른 인스턴스 대입

            변수가 다른 인스터스를 가르키게 된거니 당연하다.
            ```
            var pulpeCar = redCar
            ```

        4.  프로퍼티의 경우, 속한 클래스 인스턴스가 해제되는 경우
        ```
        class Person {
            var name: String
            var car: Car = Car(wheel: "pulpleCar")
        }
        var person: Person? = Person(name: "tonny")
        person?.car = nil
        ```

        프로퍼티의 경우 가르키던 인스턴스의 소속되어 있기 때문에, 인스턴스가 해제된다면 같이 - 1 을 하게 됨.<br>

 
하지만, 만약 프로퍼티의 레퍼런스 카운터가 1 이상이라면 속한 클래스 인스턴스가 해제된다고 해도 프로퍼티의 인스턴스는 메모리에 남아있다. 즉, 무조건 속한 클래스의 인스턴스가 사라진다고 프로퍼티의 레퍼런스도 같이 사라진다는 것이 아니라 단순 reference count - 1을 해주는 것.<br>

  

#### +) 앞서 ARC설명 시에 ARC가 compile time에 실행되어 동적으로 실행되는 것들의 reference count를 세고 메모리 관리를 할 수있다고 하는데 그게 가능한 이유가 뭘까?<br><br>

답은 ARC의 작동방식과 연관이 있다.  ARC의 메커니즘은 Swift Runtime이라는 라이브러리에 구현되어 있다. 그리고 Swift Runtime은 HeapObject 구조체를 사용하여 동적으로 할당된 모든 개체를 나타낸다. 이때 여기서 [the SIL generation phase](https://swift.org/compiler-stdlib/#compiler-architecture) 라는 컴파일 단계가 실행되게 된다. 그리고서는 HeapObject의 initialization & destruction 을 가로채어 _swiftc_ 컴파일러가 swift_retain() & swift_release() 코드를 적절히 삽입해주고 이 삽입된 코드들이 런타임에 적절히 실행되는 것이다.
<br>
<br>
<br>
<br>
<br>
 
+) REF
https://developer.apple.com/videos/play/wwdc2021/10216/

https://www.quora.com/Why-doesnt-Apple-Swift-adopt-the-memory-management-method-of-garbage-collection-like-Java-uses

[https://velog.io/@ellyheetov/ARC-VS-GC](https://velog.io/@ellyheetov/ARC-VS-GC)

[https://sujinnaljin.medium.com/ios-arc-%EB%BF%8C%EC%8B%9C%EA%B8%B0-9b3e5dc23814](https://sujinnaljin.medium.com/ios-arc-%EB%BF%8C%EC%8B%9C%EA%B8%B0-9b3e5dc23814)

[https://medium.com/computed-comparisons/garbage-collection-vs-automatic-reference-counting-a420bd4c7c81](https://medium.com/computed-comparisons/garbage-collection-vs-automatic-reference-counting-a420bd4c7c81)

https://www.vadimbulavin.com/swift-memory-management-arc-strong-weak-and-unowned/
