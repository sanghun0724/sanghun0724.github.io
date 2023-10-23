---
layout: post

---
<br/>

<br>

설계 및 디자인 패턴, 아키텍쳐 패턴 등을 뭐라고 정의할까? 
보통 이에 대해 `"어떤 문제가 발생하고 다른 똑똑한 사람들이 우아하며 일관성 있게 정리하여 내놓은 해답"` 정도로 생각했다. 하지만, 아키텍쳐 영역은 거의 예술의 영역이라 볼수 있을 만큼 심오하다. 애초에 모든 문제에 대응하는 실버불릿이라고 말할 것들이 없으며 형태 또한 환경, 시기에 따라 자꾸 변화한다. 
<br>

![image](https://github.com/sanghun0724/sanghun0724.github.io/assets/66512239/7db27232-a39a-4436-ae39-38e0d6b2eb4b)

<br>
이런의미에서 아키텍쳐를  `“현재의 애플리케이션 개발을 위한 최선의 접근방식을 적용하려는 끊임없는 진화의 시도중 하나”` 라고도 정의할 수 있을 것 같다. 여기서 잊지 말아야 하는건 그 접근 방식의 시작은 항상 문제로 부터 출발한다는 거다. (설계나 패턴이 먼저가 아니라는 소리)

<br>
<br>

***

<br>

## MVC 

<br>

￼![image](https://github.com/sanghun0724/sanghun0724.github.io/assets/66512239/83e28058-5db2-4014-81e4-95c298d4b553)

<br>

### 개요
<br>
모델-뷰-컨트롤러(model–view–controller, MVC)
<br>

**Model:** 객체가 행동과 프로퍼티로 실제 세계의 Entity와 process를 모델링하는데 사용되는 도메인 모델 <br>
**View:** 모델의 시각적 표현, 주로 애플리케션의 위젯이나 스크린을 지칭<br>
**Controller:** 키보드나 마우스 같은 사용자의 입력에 응답하는 구성요소. 사람과 애플리케이션을 이어주고 사용자가 view 및 model가 상호작용 할 수 있도록 다리역할을 수행.

<br>
<br>

#### 흔한 오해
<br>

`“controller가 model과 view를 분리시킨다?”` -> NO. MVC 패턴자체가 presentation Layer으로부터 Domain layer의 concern을 분리 시키는거다. 그리고 이 분리는 controller가 분리시키는게 아니라 observer pattern통해 달성이 된다. controller는 user와 application을 분리하는것 (view와 모델이 아니라..)

<br>
<br>

### 탄생이유
<br>

1979년, 제록스 팔로 알토 연구소 <br>
라이브 린스케이지(Trygve Reenskaug)란 사람이 다이나북팀에서 GUI개발을 하던중, 생각을 떠올리게 됨. 
“사용자가 세상을 인식하는 방법(멘탈 모델)과, 컴퓨터가 정보를 인식하고 처리하는 방법이 달라보인다.
전혀 책임이 다른 이 두 부분을 잘 분리시켜서 설계하는 게 효율적이겠구나.”
그렇게 해서 MVC는 세상에 모습을 드러나게 되었고 결국 MVC의 존재여부는 다음과 같다. **"GUI를 가진 소프트웨어를 객체 지향적으로 잘 구조화하기 위해서"** 좀더 추상화 시키자면 “관심사의 분리(separation of concerns)”를 위해 탄생했다. 라고 봐도 된다.
<br>
(또는 "도메인과 presentation의 영역 분리"라고도 말할 수 있다.)

<br>
<br>

***

<br>

## MVP

<br>
MVP는 대표적으로 두가지 방식이 유래
<br>

#### 1. The Taligent Model-View-Presenter Pattern
<br>
￼
￼![image](https://github.com/sanghun0724/sanghun0724.github.io/assets/66512239/0cd93c52-456f-447c-a5b2-53d82c70193f)

<br>

### 개요
<br>
MVC의 변형으로 유사하게 애플리케이션의 도메인, 프레젠테이션 및 사용자 입력의 concern을 구성요소로 분리함. <br>

**Model:** 애플리케이션의 코어기능이나 데이터  <br>
**Command:** data에 대한 operation이 모여있는 곳. e.g) deleting, printing, saving <br>
**View:** 데이터의 시각화 <br>
**Interactor:** 사용자 이벤트가 모델에서 수행되는 작업에 매핑되는 방식을 다룸 e.g) 키보드 입력, 체크박스  <br>
**Presenter:** 어플리케이션 내 다른 구성 요소의 전반적인 상호 작용을 조정. (Coordinator 느낌..?)

<br>
<br>

### 탄생이유
<br>
MVC로부터 파생 Apple, IBM 및 Hewlett-Packard의 합작 투자 회사인 Taligent에서 1990년대 초에 시작됨 사용자 인터페이스 프레임워크의 기초로 1995년 IBM이 Taligent, Inc.를 인수한 후에 Mike Potel이 처음으로 Taligent programming model에서 발견되는 특징적인 패턴을 설명하기 위해 "Model-View-Presenter"라는 이름을 제안함.

<br>
<br>

#### 2. The Dolphin Smalltalk Model-View-Presenter Pattern
<br>

￼![image](https://github.com/sanghun0724/sanghun0724.github.io/assets/66512239/49da30ca-feee-4a54-ba00-e6dc9b71a266)
￼
<br>

### 개요 
<br>
Dolphin Smalltalk 팀은 MVP 패턴의 공식 설명에서 인터랙터, 명령 및 선택 항목을 제거하여 Taligent MVP 패턴을 단순화시킴.
Presenter의 역할을 단순화하여 하위 컨트롤러에서 뷰를 대신하여 모델 업데이트를 중재하는 구성요소로 변경시킴. (이게 현재 우리가 알고있는 MVP에 가까움)
<br>

**Model:** 애플리케이션의 코어기능이나 데이터 <br>
**View:** 데이터의 시각화 <br>
**Presenter:** 모델과 상호작용하는 presentation 로직을 포함하는 곳 

<br>
<br>

### 탄생이유
<br>

1. 복잡성 때문에 model에서 view로 다이렉트로 접근하고 싶은 유혹이 생김.. <- 이 유혹에 환멸을 느끼게 됨 <br>
2. 그렇다고 MVC 쓰기에는 최신 플랫폼은 기본 위젯에서 바로 유저이벤트를 처리하는데 이부분에서 전통적인 MVC와 맞지 않다고 느낌 (controller가 이 역할을 하니깐) <br>
3. MVC의 진화라기보단 개발 하고 있던 VisualWorks MVC내의 리팩토링 과정에서 진행하다보니 MVP 라는 패턴이 나온 느낌 <br>

> swift에서 MVP라하면 Presenter layer를 추가하여 viewModel을 만들어 view로 내려주는 용도로 사용하는것이 흔하다.

<br>
<br>

***

<br>

## MVVM
<br>
￼
￼![image](https://github.com/sanghun0724/sanghun0724.github.io/assets/66512239/3718c0ba-51ee-4c8b-8ab0-833b3620e183)

<br>

### 개요
<br>

**Model:** 애플리케이션의 코어기능이나 데이터 <br>
**View:** 데이터의 시각화 <br>
**ViewModel:**  view 속성, 명령의 추상화(결국 로직짜라는 소리), binder를 사용함. MVP의 Presenter와 다르게 viewModel은 view를 모른다. <br>
**Binder:** viewModel과 view를 동기화 해줌 <br>

<br>
<br>

### 탄생이유
<br>

MVVM은 WPF의 Data-Binding기능을 사용해서 View layer에서 거의 모든 GUI코드를 제거(or 숨김)하도록 설계되었다. 이에 Microsoft WPF 및 Silverlight 설계자인 John Gossman은 2005년 자신의 블로그에서 MVVM을 발표했다.  MVVM으로 역할 분리를 통해 당시 인터렉티브 디자이너는 비지니스 로직 프로그래밍 보다는 UX 요구사항에 집중할 수 있었다 한다. 따라 생산성 향상을 위해 애플리케이션의 계층을 나눠 여러 작업 흐름에 개발할 수 있다록 한 것이다. 설령 혼자 작업한다 해도 view 영역은 자주 변경되므로 모델과 뷰를 더 적절히 분리 한것으로 더 생산적인 결과를 낳았다.

<br>

결과적으로 MVVM의 가이드 라인이 유도하는 바는 모델과 프레임워크가 가능한 한 많은 작업을 구동하고 View를 직접 조작하는 로직을 제거하거나 최소화 하는것 이다.

<br>
<br>

***

<br>

## MVI 
<br>
￼
￼![image](https://github.com/sanghun0724/sanghun0724.github.io/assets/66512239/f773775d-8cdf-4a74-affe-76683f48c974)

<br>

### 개요
<br>

* Unidirectional cycle of data
* Non-blocking (뷰가 모델의 상태 변화에 의해 블로킹되지 않음)
* Immutable state
<br>

**View:** 사용자에게 제공되어 보여 지는 UI 부분으로 상태(state)를 입력 받아 화면에 출력 <br>
**Intent:** 앱 상태를 변경 하려는 의도를 의미하며 사용자의 상호작용으로 발생한 상태를 변경 하는 동작 <br>
**Model(State):** 앱의 상태를 의미하며 MVI에서는 하나의 상태만을 갖으며 imutable한 데이터 구조로 작성, intent를 트리거 하여 새로운 상태를 만드는 것만이 State를 바꾸는 유일한 방법

<br>
<br>

### 탄생이유
<br>
Hannes Dorfmann가 전통적인 MVP or MVVM의 문제점을 꼬집으며 내세운 리덕스기반 패턴
대표적으로는 MVVM이 가지고 있던 Mutiple Inputs (non-thread-safe) + Multiple State 특성으로 발생되는 side-effect문제와 더불어 앱의 상태관리 문제를 해결하려고 함. 

<br>

* 어떻게?
    * MVI에서 상태는 불변, 즉, View를 업데이트 하기위해선 intent()를 발생시켜 그에 따라 새로운 상태를 덧씌우는것이 유일한 방법
    * Model은 뷰 상태에 대한 Single Source Of Truth이므로 상태 중복 X + Immutable -> 따라 model자체를 copy() 함
 
<br>
<br>

***

<br>

## Viper
<br>

￼![image](https://github.com/sanghun0724/sanghun0724.github.io/assets/66512239/ec57a5b5-1b5b-4990-9792-0d182660fec5)
￼
<br>

### 개요

<br>
Clean Architecturer를 기반으로한 설계 -> 논리적 구조를 별개의 layer로 분리 -> easier to isolate dependencies & Test
<br>

**View:** Presenter에게 받은 data를 보여주고 사용자에게 받은 input action을 다시 presenter에게 전달 <br>
**Interactor:** usecase를 포함한, 비지니스 로직을 담당 <br>
**Presenter:** display하는데 필요한 뷰 로직 + user input을 view에게 받으면 반응해줌 (interactor에게 데이터를 request하던가 or view logic) <br>
**Entity:** interactor가 사용하는 basic model <br>
**Routing:** 어느 순서, 어느 screen이 보여질껀지 등의 navigation logic 담당 <br>

<br>

> 설계의 바탕은 **Single Responsibility Principle.**

<br>
<br>

### 탄생이유
<br>
글에서 말하는걸로보아.. 찾은 결과로 보아.. [objc.io-Viper](https://www.objc.io/issues/13-architecture/viper/) 가 ios진영의 viper 시작을 알리는 글로 추정이 된다. <br> 
여기 나와있듯이 iOS진영에서 테스트는 중요한 부분으로 여겨지지 않았고 이를 못마땅하게 본(?) objc.io 팀원들은 테스트를 좀 더 짜기 쉽게 하기위해 새로운 설계방법인 Viper를 만들었다. 

<br>
<br>

***

<br>

## RIBs
<br>

￼￼![image](https://github.com/sanghun0724/sanghun0724.github.io/assets/66512239/8ad75dd9-cb5b-475b-9355-c116d64fb1b6)

<br>

### 개요
<br>
Riblet이라는 단위로 어플리케이션의 RIB Tree를 만들어 설계 
<br>

**View(Controller):** 최대한 “dump”하게 디자인 됨. 일반적으로 유닛테스트가 필요한 코드가 여기 포함되지 않음 <br>
**Interactor:** 비지니스 로직을 담당, interactor lifecycle Rx Subscription이 일어나야함. <br>
**Presenter:** business Model 를 view Model로 바꿔줌.(그 반대도). 생략가능 -> 생략시 이 책임은 interactor or view <br>
**Component:** RIB dependencies를 관리함. 주로 dependencies들에 대한 엑세스제어, 제공 등을 담당,  RIB을 만들때 builder와 함께 사용된다. 

<br>
<br>

### 탄생이유
<br>
Cross-platform 아키텍쳐 프레임워크, 중첩된 상태들을 가진 거대한 모바일 어플리케이션을 위해 디자인 됨 
<br>
(여기서 부턴 사견입니다.) 
<br>
갈수록 앱들이 고도화 되고 도메인에 따라 (ex, uber) 한 화면에 많은 Nested 상태들
을 가지는 경우가 많아짐. uber 앱만 봐도 메인 맵 에서 정말 많은 일들이 일어남. 이런 앱의 상태들과 앱 내 책임들을 일관성있게 정리하려고 시도한게 바로 이 Ribs + Ribs Tree라고 생각. 추가로 이걸로도 상태관리가 부족하면 State-Machine 같은 객체를 아키텍쳐에 추가해보는 생각을 가져갈 수도 있음. 

<br>
<br>
<br>
<br>
<br>

Ref)<br>
https://hannesdorfmann.com/android/mosby3-mvi-1/<br>
https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93viewmodel<br>
https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93presenter<br>
http://aspiringcraftsman.com/2007/08/25/interactive-application-architecture/#the-model-view-presenter-pattern<br>
https://yoon-dailylife.tistory.com/117<br>
https://www.objc.io/issues/13-architecture/viper/<br>    
