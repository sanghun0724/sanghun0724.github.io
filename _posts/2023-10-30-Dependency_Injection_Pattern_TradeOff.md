---
layout: post

---
<br/>

내가 iTime에서 needle을 걷어내고(라이브러리 의존성) 순수 DI를 내가 하게됨으로써 나에게 선택지는 크게 3가지가 있었다. 
- IoC Container 방식(SL): 특정 모듈(어셈블리)에 속하는 인터페이스 구현타입을 IoC 컨테이너로 등록하여, 사용자가 클래스 생성자를 직접 호출하는게 아니라, IoC 컨테이너에 의해 호출 된다. 인스턴스 생성방향이 역전되는게 특징이다.
- Pure -DI 방식: 심플하게 말하자면 IoC Container 없이 의존성을 주입하는 방법이다.
- Convention over Configuration 방식: 설정 대신 약속으로 DI를 한다는 것인데.. 사실 이번 iTime은 나혼자 하는거라 큰 의미가 없다.(지금 내상황에서의 장점도 크지 않다.) 그러므로 이번 옵션에서는 제외하겠다.

<br>
<br>

## Pure DI vs IoC Container

<br>
￼![image](https://github.com/sanghun0724/sanghun0724.github.io/assets/66512239/c6d2f291-6930-44fe-aef4-3183f27e568f)
<br>
<br>

## IoC Container

<br>
IoC 컨테이너를 사용하면 의존성 등록 비용이 적다. 하나하나 생성자를 통해 의존성을 넣어주는 코드를 작성하지 않아도 되니 말이다. 앱이 커지면 커질수록 객체들도 많아 질테니 이런 수작업을 줄이고 코드를 줄일 수 있다는건 유지보수의 용이성을 올리는 일이니 장점이 분명하다. 하지만, 단점도 명확하다. 먼저 IoC Container를 도입하려면 별도의 container에 대한 학습 비용이 발생하게 된다. 무엇보다 가장 큰 문제는 의존성이 잘못 만들어졌을 경우, 컴파일 에러를 발생시키지 않고 런타임 에러 발생이 가능하다는 것이다.(Weakly typed) 그리고 이런 에러가 발생해버리면 특정 기능이 실행되지 않거나 앱이 크래쉬가 날수도 있는 가능성을 내포한다. 이는 앱 개발자가 가장 회피해야할 유저사용경험 중 하나이다.

<br>
<br>

## Pure-DI
<br>

Pure-DI 방식의 단점은 명백하다. 바로 **High maintenance**. 즉, 일일이 손으로 의존성 주입 코드를 작성해주어야 하기 때문에 코드양이 늘어나게 되고 개발하는데 비용 또한 늘어나게 된다. 이는 결국 유지보수 비용의 증가로 연결된다. `그렇다면 IoC Container가  Pure DI보다 좋을까?` 이는 상황마다 다르다. 이 질문을 상기시키며 Pure DI의 장점을 살펴보자. 먼저 IoC Container의 학습비용이 사라지게 된다. 또한 각 모듈의 Composition Root를 찾으면 개체 그래프가 어떻게 구성되는지 분명하게 파악할 수 있다. Pure DI의 가장 큰 장점은 IoC Container의 Weakly typed 문제가 Pure DI에서는 발생되지 않는다는 것이다. 이는 런타임 에러를 피할 수 있는 매우 강력한 장점이다. 

정리를 통해 여기서 우리가 고민해야하는 방향이 좁혀졌다.  흔히들, (이 아티클을 쓰기전 나를 포함) 의존성 등록 비용을 줄인다는 이유로 IoC Container의 학습 비용과 Weakly Typed 비용을 제대로 고려하지 못하는 것 같다. 모든 비용을 고려하여 내 iTime에는 어떤 DI 방식을 쓸지 고려해보자
<br>
￼![image](https://github.com/sanghun0724/sanghun0724.github.io/assets/66512239/942ef014-19d9-4459-9575-444de64a3acb)
<br>
<br>

먼저 선택에 앞서 현황파악을 먼저 시도해보았다.
<br>
1. 현재 내 앱은 개발중이며 사용자 경험이 중요한 어플이다. <br>
2. 개인이 개발하는 앱이기 때문에 상당량 규모의 의존성 등록이 발생할 확률이 적다.<br>

<br>
<br>
이어 비용을 고려해보자, 
<br>

* 현재 내 앱의 의존성 등록비용이 Weaky typed 비용 보다  큰가? -> **NO** 
<br>

* 앱 개발자로서 crash를 내포할 코드를 작성하는 리스크를 지면서까지 의존성 등록 비용을 줄여야하는 이유가 명확한가? -> **NO** 
<br>
<br>

## Conclusion
<br>

`내 식견으로 앞의 질문들에 대한 답을 고려해보았을때, 현재 내 앱에 들어갈 DI 방식으로는 Pure-DI가 적절한것으로 판단된다.`

<br>
추후에 정말 앱이 커져 의존성 등록 비용이 기하급수적으로 커진다면 IoC Container를 고려해봄직하다. 

<br>
<br>
<br>
<br>
Ref: <br> 
[https://blog.ploeh.dk/2012/11/06/WhentouseaDIContainer/](https://blog.ploeh.dk/2012/11/06/WhentouseaDIContainer/) <br>
[https://blog.ploeh.dk/2014/06/10/pure-di/](https://blog.ploeh.dk/2014/06/10/pure-di/) <br>
[https://jwchung.github.io/DI%EB%8A%94-IoC%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EC%A7%80-%EC%95%8A%EC%95%84%EB%8F%84-%EB%90%9C%EB%8B%A4](https://jwchung.github.io/DI%EB%8A%94-IoC%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EC%A7%80-%EC%95%8A%EC%95%84%EB%8F%84-%EB%90%9C%EB%8B%A4) <br>
[https://minsone.github.io/pure-dependency-injection](https://minsone.github.io/pure-dependency-injection) <br>
