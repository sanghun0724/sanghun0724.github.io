---
layout: post

---
<br/>

최근 내 iTime프로젝트에서 쓰고있던  swift-clocks 라이브러리의 Testclock을 사용한 clock unit test들이 random하게 터지는 일이 발생을 했다. 
원인을 찾아보니 TestClock에서 제공하는 advance(to:) 함수가 flaky한것이 원인이었다.  
 당시 내 생각은 Test에서 쓰이는 Clock의 인터페이스가 deterministic하지 못한건 잘못 되었다 생각하고 swift-concurrency-extra의 `withMainSerialExecutor` 함수를 사용해, TestClock의 advance(to:)  구현부를 감싸 Test시에 동작을 좀 더 deterministically 하게 바꾸어 PR을 올렸다.  [PR 코드 수정 부분](https://github.com/pointfreeco/swift-clocks/pull/27/files)

  <br><br>

그러다 몇시간 만에 바로 poinfreeco 메인테이너인 [**stephencelis**](https://github.com/stephencelis) 에게서 답변이 달렸다. (기본적으로 PR 피드백이 굉장히 빠르신거 같다**👍**)

<br>
<br>
  

![stephencelis1](/assets/stephenceils1.png "stephencelis1")
  
<br>
<br>

요약:

-   너의 코드가 Test를 less flaky하게 만드는건 맞다.

- 하지만, 실제 환경을 테스트하려는 사용자들이 TestClock을 사용하는 것을 방해 또는 예상치못한 effect를 발생시킬수 있다.

  
<br>

Test의 Flakiness에만 신경을 쓰고 있던터라, Test의 실제 환경 반영 여부는 잘 고려하지 못했었다. 또한 executor를 바꾸게 되면 기존 사용자들이 영향을 받을 수 있다는 점도 간과하였다.

그럼에도 불구하고, less flaky한 코드를 내버려두는것에 의구심이 남아 좀 더 디테일한 설명요청과 함께 질문을 하였더니 금방 정성스런 답변을 받을 수 있었다.

<br>
<br>

![stephencelis2](/assets/stephenceils2.png "stephencelis2")

<br>
<br>

요약:

-   사용자들은 default executor와 함께 TestClock을 사용하여 실제 환경과 유사하게 동작하도록 해야 한다.
-   withMainSerialExecutor를 사용하면 비동기 코드가 메인 스레드에서 예측 가능하게 연속적으로 실행되어 TestClock을 사용하는 코드가 현실과 다르게 동작하게 된다. 
-   따라 실제 환경을 테스트하려는 사용자들은 여전히 default excecutor에서 테스트가 가능해야 하며, flaky한 현상들은 다른 방법으로 각자 handling 되어야 하는게 적절하다.

  
 <br>
 
즉, 애초에 이 라이브러리에서 TestClock이 설계 될때, 우선적으로 고려했던 것은 “Clock이라는 도메인의 실제환경을 반영하는 TestClock”인 것이다. 
그렇기에 내가 수정한 PR 코드의 변경으로 설계자가 의도한 “실제 환경 반영” 이라는 의도를 비틀 수 있는 것이다.
 따라서 이 PR이 반영 되려면, TestClock이 실제 환경을 반영하는 특징을 깨뜨리지 않고 기존 사용자들에게 의도되지 않는 영향을 주지 않으며 기존 개선하고자 했던 부분인  테스트 시의 less flaky한 ` advance(to:)` 구현체를 구현하면 된다.

<br>
<br>

다만 현재 처럼, Test 환경에서 **Stability vs Real-World Conditions을 비교해보아야 하는 상황이면 어떻게 해야할까?**

 <br>

둘의 정의를 통해 맥락을 비교해보자.

 <br>

**Stability:**

-   테스트 시의 안정성은 동일한 조건에서 여러 번 실행될 때 동일한 결과를 얻을 수 있는 특성을 의미함. 즉, 동일한 테스트 데이터와 환경 설정에서 실행될 때 일관된 결과를 제공하는 것이 중요.
-   테스트 시의 안정성은 주로 개발자들이 코드 변경사항을 확인하고 버그를 찾는 데 도움이 된다. 안정적인 테스트는 개발 주기를 빠르게 만들고, 코드 변경에 대한 신뢰성을 높일 수 있다.

 <br>
 
 **Real-world Conditions:**

-   테스트가 실제 조건에서 어떻게 동작하는지 확인하는 것은 애플리케이션이 실제 사용 환경에서 어떻게 동작할지에 대한 자신감을 제공.
-   현실적인 조건에서 테스트를 실행하면 미처 고려하지 못한 환경에서 발생할 수 있는 문제를 미리 감지할 수 있다.

 <br>

두 측면은 프로젝트의 성격에 따라 어떤 측면이 더 중요한지 나뉘게 된다. 만약, 프로젝트가 안정성과 신뢰성을 중시한다면 `Stability`가 중요할 수 있다.

반면에 실제 사용 환경에서 발생하는 다양한 조건을 반영해야 하는 경우 `Real-world Conditions`이 중요할 수 있다. 또는 개발 단계와 라이프 사이클 단계에 따라 우선시해야 하는 측면이 다를 수 있다.
 예를 들어,안정성 있고 빠른 개발을 위해 초기에는 테스트 시의 안정성을 고려하고, 추후 개발 프로덕트가 실제 사용 환경에 가까워 질수록 테스트 시의 실제 환경을 구축하여 다양한 조건들을 테스트 하는게 중요할 수 있다. 종합적으로 프로젝트의 특성과 단계에 따라 두 측면을 균형 있게 고려하는 것이 중요.

<br>
<br>

#### 결론

-   정답은 없다. “ Flakiness한 테스트? 잘못된거 아니야? “ 라고 바로 도달하기 보다는 한발자국 떨어져 관점을 바꾼 후 문제를 다각도로 관찰 해보도록 해보자.
-   라이브러리의  코드를  변경  시  기존  사용자들을에게  어떤  영향을  미치는지  늘  고려하도록  하자. 

<br>
<br>
-자세한 PR 내용은 하단 참조-
<br>
Ref: <br>
 
[https://github.com/pointfreeco/swift-clocks/pull/27](https://github.com/pointfreeco/swift-clocks/pull/27) <br>
