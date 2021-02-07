---
layout : post
title : "Javascript: 동작 원리"
date : 2021-02-07
category :
  - Javascript
elements:
  - call Stack
  - callback queue
  - v8
  - event loop
tags:
  - Call Stack
  - Callback Queue
  - v8
  - event loop
comments : true

---

# 1. 자바스크립트 엔진
 크롬의 V8엔진에 대해서는 자주 들어봤을 것이다. 그 외에 또 어떤 엔진이 있고 각각의 엔진들이 어떤 방식으로 작동 하는지도 알아보자.

## 개요

javascript를 다루는 사람들은 근 몇년 전 부터 굉장히 많다. 실제로 GitHut stats에서 볼수 있듯이 Github 의 Active Repositories 와 Total Pushes 양도 어마어마하다. javascript를 사용하는 대부분의 사람들은 이것이 *싱글 쓰레드 기반*이고 콜백 큐를 사용한다는 사실을 알고 있겠지만 명확하게 집고 넘어가 보도록 하자.

## 1) 엔진의 역할
자바스크립트 엔진의 대표적인 예는 Google V8 엔진이며, V8 은 Chrome에서 사용한다.
Chrome 브라우저는 
* Blink : Renderer 엔진(html, css)
* V8 : Javascript 엔진


V8은 Chrome이 아니더라도 독립적 실행이 가능하며 V8로 빌드된 Node.js가 대표적이다.

엔진은 2개의 구성요소로 되어 있으며
* Memory Heap : 메모리 할당

    Memory 생존주기는 프로그래밍 언어와 상관없이 비슷하다. 
    1. allocate
    1. use
    1. release

    두 번째 부분은 모든 언어에서 명시적으로 사용된다. 그러나 첫 번째 부분과 마지막 부분은 저수준 언어에서는 명시적이며, 자바스크립트와 같은 대부분의 고수준 언어에서는 암묵적으로 작동한다.

    메모리 힙은 변수, 함수 저장, 호출 등의 작업이 발생하여 위 내용들이 진행되는 공간이다.
    쉽게 생각하면 'Memory Heap'이라는 이름의 창고가 있고, 변수나 함수들은 겉에 이름이 라벨지로 붙어있는 박스들인거다.



* Call Stack : 호출 스택이 쌓이는 곳
의 역할을 맡고 있다.

## 2) V8
구글이 주도하여 C++로 작성된 고성능의 자바스크립트 & 웹 어셈블리 언어이다. ECMAScript와 Web Assembly 표준에 맞게 구현하였다.

### V8 특징
V8은 아래와 같은 특징을 지닌다.
* JavaScript소스 코드를 컴파일, 실행
* 생성하는 Object를 메모리에 할당
* 가비지 콜렉션을 이용해 사용하지 않는 Object의 메모리를 해제
* Hidden Class를 이용해 프로퍼티에 신속하게 접근
* TurboFan을 이용해 최적화


>JavaScript -> Parser -> Abstract Syntax Tree -> Interpreter Ignition -> Bytecode

1. 자바스크립스 소스코드를 가져와 *Parser*에게 넘긴다.
2. Parser는 파싱을 통해 *AST(Abstract Syntax Tree)* 로 변환시킨다.
3. AST를 인터프리터를 통해 *Bytecode*로 변환(Ignition)한다.
4. Bytecode를 실행함으로써 실제 작동하게 된다.
5. 그 중 자주 사용되는 코드는 *TurboFan*으로 보내진다.
6. TurboFan은 이 코드를 *Optimized Machine Code*로 컴파일해놓고 사용한다.

#### JIT Compiler
javascript는 보통 js파일로 배포되고 브라우저에서 이를 사용한다.  

브라우저가 Javascript를 처리하기 위해 컴파일을 하는데 보통 Interperter로 처리한다.

V8에서는 먼저 Javascript 코드를 Interpreter 방식으로 Compile하고, ByteCode를 만들어 내는데 이후 Compile속도를 높이기 위해 ByteCode를 캐싱 해두고, 자주 쓰이는 코드를 inline caching과 같은 최적화를 통해 반복적인 Compile을 할 때 참조하여 속도를 높힌다.

이런 JIT(Just-In-Time) Compiler를 통해 Interpreter의 느린 속도를 개선할 수 있다.


#### Ignition
> AST를 해석하고 최적화하는 인터프리터

컴파일러 대비 메모리 사용량이 감소하고 파싱 시 오버헤드가 적다. bytecode는 기계어보다 간결하기 때문에, 기계어를 컴파일 하는 것 보다 bytecode로 컴파일 하는 것이 메모리 사용량 뿐 아니라 파싱하기에도 유리하다. 또한 컴파일 파이프라인의 복잡성이 낮다. 

#### TurboFan
> 최적화 컴파일러

V8은 컴파일된 ByteCode를 최적화 하기 위해 Turbofan을 이용한다.
Turbofan은 자주 사용되는 Native code와 중복된 코드를 재사용하여 코드 크기, 메모리 오버헤드를 줄이고 속도를 증가시킨다. 

Turbofan은 아래 순서로 진행된다.

1. Graph Building : 바이트 코드 또는 AST 를 Graph 로 만든다.
1. Native Context Specialization & Inlining : Load / Store / Call IC 를 기반으로 기본 컨텍스트에 특화된 Simple Graph 를 생성한다.
1. Typed Optimization : Type 에 따라 Simple Graph 로 변환한다.
1. General Optimization : Graph 를 기반으로 중복 제거 같은 최적화를 진행한다.
1. Code Generation: 죽은 코드를 제거고, 레지스터에 할당한다.

이때 V8에서는 최적화를 진행하기 위해 Schedule(CFG:control flow graph) 과정을 거치게 된다.   

   
   

## 3) 런타임

### Web APIs
setTimeout과 같은 브라우저 내장 API는 JavaScript 엔진에서 제공하는 것이 아니다. 
DOM, Ajax, setTimeout과 같이 브라우저에서 제공하는 API를 Web APIs라고 한다. 

### Event Loop
> Callback Event Queue에서 순차적으로 동작시키는 Loop

자바스크립트는 단일 스레드 기반 언어이기 때문에 한번에 하나의 작업을 순차적으로 진행한다. 호출 스택이 하나라는 소리이다.
하지만 실제 자바스크립트가 사용되는 환경을 생각해보면 많은 작업이 동시에 처리되고 있는 것을 볼 수 있다.
심지어 Node.js 기반의 웹서버에서는 동시에 여러 개의 HTTP요청을 처리하기도 한다.   
분명히 한번에 하나씩 처리한다고 하였는데 자바스크립트는 어떻게 동시성을 지원하는걸까?

자바스크립트는 이벤트 루프를 통해서 비동기 방식을 지원한다. 하지만 이것은 자바스크립트 엔진에서 제공해주는 것이 아닌 브라우저나 node.js 에서 제공해 주는 것이다.

### Call Stack(호출 스택)

자바스크립트는 기본적으로 싱글 쓰레드 기반 언어입니다. 호출 스택이 하나라는 소리다. 따라서 한 번에 한 작업만 처리할 수 있다.

호출 스택은 기본적으로 현재 우리가 어느 부분을 처리 하려는지를 알려주는 자료구조다. 실행커서가 특정 함수에 있다면, 해당 함수는 호출 스택의 가장 위에 쌓여있는 것이다. 그 함수가 실행이 끝나고 리턴 값을 돌려줄때 호출 스택에서 함수가 제거된다. 함수의 적재 순서는 순차적이지만 처리 순서는 반대이다. 

### Callback queue(Task Queue)
> Event 실행 관리를 위한 임시 저장 Queue

자바스크립트의 런타임 환경의 Callback Queue는 처리할 메세지 목록과 실행할 콜백 함수들의 리스트이다. 버튼 클릭 같은 이벤트나 DOM 이벤트, http 요청, setTimeout 같은 비동기 함수는 Web API를 호출하며 Web API는 콜백 함수를 Callback Queue에 밀어 넣는다.

Callback queue는 대기하다가 Call Stack이 비는 시점에 이벤트 루프를 돌려 해당 콜백 함수를 스택에 넣는다. __이벤트 루프의 기본 역할은 큐와 스택 두 부분을 지켜보고 있다가 스택이 비는 시점에 콜백을 실행시켜 주는 것이다.__

브라우저에서는 이벤트가 발생할 때마다 메세지가 추가되고 이벤트 리스너가 첨부된다. 콜백 함수의 호출은 호출 스택의 초기 프레임으로 사용되며 자바스크립트가 싱글 스레드 이므로 스택에 대한 모든 호출이 반환될 때까지 메세지 폴링 및 처리가 중지 된다. 동기식 함수 호출은 이와 반대로 새 호출 프레임을 스택에 추가한다.

## Reference
>https://joshua1988.github.io/web-development/translation/javascript/how-js-works-inside-engine/
https://evan-moon.github.io/2019/06/28/v8-analysis/
https://meetup.toast.com/posts/78
https://developer.mozilla.org/ko/docs/Web/%EC%B0%B8%EC%A1%B0/API
https://velog.io/@imacoolgirlyo/JS-%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-%EC%97%94%EC%A7%84-Event-Loop-Event-Queue-Call-Stack#3-event-queue
https://developer.mozilla.org/ko/docs/Web/JavaScript/Memory_Management
https://velog.io/@namezin/javascript-%EB%8F%99%EC%9E%91-%EC%9B%90%EB%A6%AC#%EA%B0%9C%EC%9A%94