# 시나리오로 배우는 프론트엔드 실무

## 아키텍처 리뷰 파트에서 알면 좋을 것들 메모

1. backend

   - b2c와 backoffice(즉, 관리자용?) api 서버는 따로 분리하는 것이 좋다.

   * 여기서 cdn 디렉토리는 실제 서버 아니고 강의용(실제 cdn 서비스 사용하면 비용문제있으니까)으로 간단한 파일 시스템 만들어 논 것, s3같은 클라우드 서비스로 대체해서 생각하면 됨.
     - 과거에는 보통 이미지들은 물리적인 서버의 파일서버에다가 올렸음, 요즘은 클라우드 서비스를 많이 사용함
   * graph-ql-server : 프로토콜 표준화를 위한 bff

   - remote-config : msa가 됐을 때 백단의 도메인들이 많아졌을 때 엔드포인트들이 다양하게 바뀔 수도 있기 때문에 config를 하나의 서비스로 모아두고 이 서비스가 api로 config를 제공하는 것
     ```
      // config.develop.js
       module.exports = {
         server: {
           cdn: 'http://cdn.12shop.com:1201',
           b2c: 'http://api.12shop.com:1202',
           web0: 'http://api.12shop.com:1203',
           web1: 'http://api.12shop.com:1203',
           webview1: 'http://api.12shop.com:1204',
           applog: 'http://api.12shop.com:1205',
         },
       }
     ```

2. client
   - react-natvie 앱 코드 들어있음
3. web/web
   - 웹임 ! 얘는 SSR로 만들어짐 그 이유는 아래 **기술 스택 선택** 항목 참고
   * web0 : 기존에 앱다운로드만 있는 웹
   * web1 : 앱 내용똑같이 보이는 버전
4. web/webview
   - client의 react-native 앱 내에 사용되는 webview임. 이 친구는 CSR로 채택되었음. 그 이유는 webview니까 SEO가 필요하지 않음, 앱 내에 webview 화면이 여러 개일 경우 ssr이면 서버가 필요하니까 관리비용이 많이듬, 그리고 만약 tab이 있고 tab마다 webview라고 하면 로딩이 빨리되어야하는 데, 첫 로딩만 느리고 그뒤에 빠른 csr이 적당
   * webview1 : 쇼케이스화면
     - webview1.1 : 리팩토링 전, 상태관리 redux
     - webview1.2 : 리팩토링 후, 상태관리 recoil
   * webview2 : 충전 화면
     - webveiw2.1 : 리팩토링 전, class 컴포넌트, 다른 웹뷰와 같이 사용하는 공통 header 컴포넌트가 해당 프로젝트에 또 구현되어 있음.
     - webview2.2 : 리팩토링 후, hook으로 바꾸고, 공통 header 12shop/components에서 가져와서 사용

## 일잘러 개발자가 되기 위해서 어떻게 프로젝트에 접근해야할까?

### RFC라는 문서를 항상 작성하자

- RFC : 여러 사람들의 의견을 청취하고 싶을 때 작성하는 문서, 의견을 듣고 싶은 내용을 기술하고 동료들에게 공개적으로 의견을 받고 수정하기 위한 그런 목적의 문서

* RFC 템플릿(https://www.notion.so/RFC-04730273a5fe42d2b2f9390ba3fa6172)

### Poc VS. Prototype

- 어떤 제품을 만들 때 RFC를 작성하면서 PoC가 필요한지, Prototype이 필요한 지 둘다 필요한 지 고민해보고 제안하라

#### PoC

- Proof of Concept
- 아이디어가 있을 때 기술적인 관점에서 실제 **실현 가능한 건가 확인하는 작업**

* PoC는 어떻게 하는 걸까 ?
  - 최우선 목표 : 프로젝트의 불확실한 요소를 제거하는 것
  * 불확실한 요소 어떻게 제거할 건데 ? => 구현 가능한 기술과 구현 가능성이 [미지수]인 알수 없는 기술을 분리한다.
    > - 단순히 안해봤으니까 미지수로 분리 X, 이렇게 분리하기 위한 기준이 필요
    > - PoC 미지수의 예
    >   - 실현 가능한 기술인가?
    >   - 성능이 필요한 수준인가?
  * 예시
    > - 이미지 서버 개발시 이미지 리사이징 퍼포먼스
    > - N개의 리사이징 이미지를 생성하는 성능 문제와 이미지 최적화 옵션 범위가 요구되는 만큼의 수준인지 확인

#### Prototype

- 아이디어를 실체화 하는 것

* 실체를 만들어 아이디어의 정합성을 확인하는 것이 목표이기 때문에 구현 방법과 기술이 중요하지 않다.
* 최대한 빠르게 만들 수 있는 그런 기술을 채택해서 유사한 형태로 만들고 사용자한테 보여준다. 이게 매력적으로 보여? 안 매력적으로 보여 ?
* 즉, **아이디어가 실제로 시장에 먹히는 아이디어인지 빠르게 확인하기 위한 것**
* Prototype의 가장 중요한건 시간.

### 기술 스택 선택

- 이 항목에서 예시는 12shop앱을 웹으로 구축하는 경우의 RFC를 바탕으로 작성된 내용임을 참고하라.

* 항상 프로젝트 요구사항에서부터 출발하라.
* 어디서부터 출발해야할까 ?

  - RFC/개발/핵심 목표 항목에서 중요 키워드들 추출해서 연관된 기술들 찾아본다.

    > - 핵심목표 예시
    >
    >   1.  서비스가 더 많은 사용자에게 노출될 수 있도록 한다.
    >   2.  사용자에게 더 많은 서비스 정보를 제공한다.
    >   3.  사용자에게 서비스의 매력을 증진시킨다.

    > - 추출 키워드 : 더 많은 노출, 매력증진

  * 위의 예시에서 더 많은 노출, 즉 서비스를 노출시키기 위해선 어떤 기술을 써야 할까 ? => SEO
  * 매력증진 => 성능
  * 여러가지 기술들을 살펴보고 본질적인 목표로 돌아와서 선택

- 선택

  - _선택된 거 이탤릭체_

  1.  핵심 구현 방식 CSR VS. **SSR**

      - 각각의 장단점을 고려해서 결정

      * 예제 프로젝트에서는 스타트업에 사용자가 많지 않고 초기단계임. 그래서 CSR보다 검색 엔진 최적화 돼서 사용자한테 많이 노출되는 중요하다는 판단을 근거로 SSR 선택함.

  2.  React SSR VS. **Next.js**

      - React SSR : 리액트만으로 SSR 가능함.

      * 이 둘도 장단점을 고려해보자.

      * 항상 어떤 솔루션을 선택할때는 장단점이 분명히 존재한다.
        - 직접 만드는 경우(React SSR)
          > - 처음 만들 땐 굉장히 많은 비용이 들어간다. 아키텍처도 만들어야 하고 만드는 과정에 여러가지 시행착오를 거쳐야되는 게 큰 단점(즉, 시작이 느릴 수 있다)
          >
          > * 완벽하게 되었다라고 하는 시점부터는 엄청난 장점, 새로운걸 해보려고 할 때 지원되는 가 안되는 가 검토하는 게 아니라 그냥 하면되니까
        * 외부 솔루션을 쓰는 경우(Next.js)
          > - 처음엔 굉장히 빠르게 시작 가능
          >
          > * 새로운 걸 시도할 때 지원안하는 문제 생길 수 있음
      * 엔지니어 입장에선 위와 같은 장단점을 고려해서, 내가 하는 서비스의 유형을 보고 판단.
      * 강사님이 내린 결론

        - React SSR : 학습용(=> _학습용이라고 정의한 이유는 SSR이 어떻게 이루어 지는가 동작방식에 대해 깊이 알고 있는 건 프론트엔드 엔지이어로서 굉장히 중요함. Next에서 제공하는 아키텍처의 이해도를 높이는 것 보단 Reat에서 제공하는 아주 로어한 api들을 조합해서 구현해보는게 SSR의 밑바닥을 익히기 좋다. 그래서 반드시 해보라고 학습용_)
        - Next.js : 실전용

        * 강의에 예제 프로젝트인 12# 서비스 자체가 앱으로 사용자한테 전달되는 게 핵심가치, 웹은 보조적인 역할 수행, 당장 1~2년 안에 새로운 기능을 계속 추가하는 식의 서비스가 안될 것 같음. 그래서 위와 같은 결론 도출

### 효율적인 레거시 리팩토링을 위한 문서 작성

- 레거시를 가시화 하는 것이 중요하다.

* 레거시 제안 문서 템플릿(https://www.notion.so/decd42ae86a0499a9f81eb3d1b34637e?v=089a5740fe19417ebf72569eaebcd101&p=3a0484bc957847a5a1810e16d92783ef&pm=s)

## 예제 프로젝트로 SSR Poc

- step-by-step/poc-ssr-showcase 프로젝트

#### Poc 계획

- 무엇을 확인할 것인가? => 목표
  > - Next.js SSR 라우트 구조 확인
  > - 페이지별 Meta Taging => 검색엔진한테 적절하게 잘 설명해주는 것, openGraph
  > - API Fetching
  >   - 어플리케이션을 큰 덩어리로 구분 짓는 다면?
  >     - 데이터, 데이터를 UI로 만든느 로직, 데이터를 가져오는 Fetching(데이터가 큰 경우, 여러 호출 요청 등 다양한 로직이 들어감)
  >     - 위 3가지 항목 중 가장 중요한 Fetching, CSR은 무조건 클라이언트에서 Fetching이니까 심플한 편, SSR은 상황에 따라 클라이언트에서 Fetching, 서버에서 Fetching 이렇게 두가지 상황 생길 수 있음.
  > - Next.js 기능 목록 체크

#### Poc 프로젝트 생성 후 위의 사항들 체크

- Next.js 보일러 플레이트로 생성

* 위의 목표 순서대로 라우트 구조부터 확인(강사님의 개인 취향이라고 함, 라우터부터 확인하면 전체를 파악한 느낌)

  - 공식문서를 보면서 라우트 어떻게 제공하는 지 보고 우려사항 생각.
  - 강사님의 Next.js SSR 라우트 구조 확인
    > - File-system routing
    > - Dynamic routing
    > - Shallow routing
    > - Routing & **Lifecyle**
    - 공식 문서들을 쫙 보고 위와 같은 키워드들 도출, 라우팅에 너무 많은 기능들을 제공하는 것 같다는 동물적 불안감을 가짐. 특히 Lifecycle이라는 측면까지 엮이면 굉장히 복잡해질 수 있는 상황들이 발생하겠다. 생각한다.
    * 위와 같은 걸 발견하고 더 리서치를 해봐야겠다고 생각한다. 위에서 말한대로 ssr은 서버, 클라이언트 둘다 fetching이 일어날 수 있는 데, react 렌더링이 반복적으로 계속 일어나서 성능 저하되는 현상들이 발생할 수 있음.
    * 우리가 충분히 이해하지 못하고 위와 같은 일이 생길 가능성이 충분히 있어보임. 위와 같은 복잡도가 딱 보이니까. 그래서 **저런 키워드들을 보고 불안감을 느끼고 그런 부분에 더 시간을 투자해야겠구나 생각해야함.**
    * 이런 것을 하려고 Poc, 프로토 타입을 하는 것이다. 문제점을 발견하고 해결책이 있는지 틀렸는 지 빨리 결론을 내리기 위해

* 페이지별 Meta Taging
  - next/head
* API Fetching

  - Server Side => Server Side fetching에는 어떤 것들이 있는 지 체크
    - getServerSideProps
    - getStaticProps
  - Client side => Client side에는 어떤 것들이 있는 지
  - API gateway => API gateway 역할도 할 수 있는 지

* 위와 같은 과정 지름길 => 좋은 Boilerplate 찾아서 공식문서랑 같이 보면 좋다

#### npm start로 실행

- 위와 같은 과정 거치면서 실제로 작성해서 확인할 항목 선택하고, 해당 poc에 작성해본다.

* step-by-step/poc-ssr-showcase 에선 product 버튼 하나 생성하고 'next/image'와 html 기본 img 태그 비교 테스트했음.

## 개발자가 챙겨야 하는 것

- 테스트 - 단위테스트 vs. E2E 테스트
  - Cypress
- 로그 - 사용자 로그, 앱 로그
  - Sentry IO 같은 서비스 연동, 유료 서비스를 못쓴다면 유사한 형태의 무료 서비스를 제안 해보기
- 예외 상황 처리 - 404, 500, etc..
  - 전역적으로 최상위에서 에러핸들링 할 수 있게끔, 404, 500 페이지를 만들어 두기
- 코드에 매몰되지 않기

## QA

- 단위테스트 vs. E2E 테스트
- 강사님의 개인 의견으로 프론트단에서 수많은 단위테스트는 오히려 득보단 실인 것 같다. 프론트단은 화면이 수시로 수정사항이 발생 할 수 있는 데 이때마다 테스트도 업데이트 해주어야하고, 로직은 잘짯는 데 브라우저 환경때문에 화질이 깨진다던가 이런건 어차피 못잡음, 완벽한 테스트가 될 수 있을까라는 생각.
- 강사님은 **기본적으로 E2E테스트(즉, 통합테스트)를 기반으로 해야한다고 생각한다.(통합이라고해서 실제 API 데이터말고 MOCK 데이터로), 단위테스트는 핵심 로직 중심의 단위테스트, 이 두가지를 적절히 구축**
- 프론트앱의 특성상 최종적으로 사용자한테 렌더링 되는 앱의 결과, 앱들이 갖고 있는 기능들 단위로 테스트가 이루어지고, 시작적인 측면에서도 잘 렌더링 되는가 하는 부분들을 E2E테스트를 해볼 수 있는 부분이 많다.

* 물론 TDD가 안중요하다는 건 아니다. 사람마다 의견차이가 다르니 직접 겪어보아라.
* **단위테스트는 안하더라도 꼭 E2E테스트는 하라 !**

## 배포 후 운영 & 모니터링

- 사용자 활동과 앱 활동 로그를 남기자 !
  - Sentry IO : 강사님이 자주 사용하는 거라고 함, 여러 에러 수집과 성능관련 지표도 제공
  * 사용자 활동 : Google analytics (무료, 사용자 이탈률과 같은 정보 제공함. 마케팅쪽에서 주로 사용하는 데 개발자가 미리 알고 심어놓으면 더 좋다 !)

## 웹뷰

- 딥링크 : 네이티브 개발자가 딥링크를 만들어서 프론트엔드 개발자에게 줄텐데, 이때 주는 그대로 사용하는 것이 아닌, 미래 확장성을 고려해서, option(json) 추가하는 것 제안. 이런건 프론트엔드 개발자가 챙겨야한다.
  - 어떤 웹뷰는 header 부분이 네이티브앱 그대로 보이고, 어떤 웹뷰는 전체화면이 webview로 보여야해서 header가 안보여아하고 이러한 옵션들, 그리고 주문프로세스라고하면 주문이 완료되고 웹뷰가 종료될 때 모든 웹뷰가 종료되어야하는 그런 플로우를 관리하는 옵션과 같은 것이 필요하기 때문에

* 화면간 데이터 교환을 위한 아키텍처
  - 이것도 네이티브 개발자는 잘 모르니 프론트 엔드 개발자가 신경써야함. 서로 다른 프로젝트인 웹뷰간에 데이터를 공유해야하는 경우, 딥링크에 데이터를 실어서 공유하는 거, 웹 스토리지를 이용하는 것 이런건 한계가 있음.
  - 네이티브 쪽에 저장을 해야함. 그럴러면 네이티브 쪽에 데이터 저장을 위한 스펙이 필요, 이건 네이티브 쪽에서 제공을 해줘야 함.
  * **여러 멀티 웹뷰간 데이터 교환을 위해선 네이티브에서 제공되는 자바스크립트 인터페이스가 필요하고, 필요할 때마다 만들면 안되고 한번 만들어지면 웹앱 내에서 모든 서비스의 웹뷰가 사용할 수 있도록 초기에 프론트엔드 개발자가 요청**

## 컴포넌트 저장소 분리하기

- 한 서비스 안에 여러 웹뷰 프로젝트가 존재할 경우, 공통 컴포넌트는 반드시 존재한다. 이럴 경우, 각 프로젝트마다 각각 component폴더를 생성해서 똑같은 컴포넌트를 각자 구현해야할까 ?
- 한 서비스에서 이러한 **공통 컴포넌트를 공유하기 위해 npm private를 이용하는 것이 좋다고 추천 !**
  - verdaccio 사용

* web/12shop-components 폴더가 컴포넌트를 분리해놓은 프로젝트이다.
* 해당 프로젝트는 rollup으로 빌드함.
  - 빌드가 더 빠르다.
  - 번들링에서 배포하는 타입을 다양하게 아웃풋을 만들어 낼수 있다. webpack은 commonjs로만 아웃풋 만듬, rollup은 esm방식으로도 아웃풋 생성가능
  - 코드 스플릿팅과 관련돼서도 훨씬 더 강력한 기능들을 많이 제공
  - 그래서 프로덕 관점에선 webpack이 많이 사용되지만, 라이브러리 관점에선 rollup 번들러 많이 쓴다고 함

## 라이브러리 만들 때 꿀팁

- log 남기는 라이브러리로 library/app-logger을 만들었다고 하자. 이를 web/webview1.1에서 사용하고 싶을 때 어떻게 사용해야할까?
- npm을 이용할 수 있다 !
  - 라이브러리를 배포 안했을 경우 사용방법
    - 초기에 라이브러리 개발하고 테스트해볼 때 이렇게 사용하면 됨.
      > - npm install 해당 라이브러리 폴더
      >
      > * 이렇게 설치해도 실제 npm으로 설치한 것 처럼 사용 가능!
      >   ```
      >     npm install ../../library/app-logger
      >   ```

## 프로토콜 표준화

- MSA 구조의 아키텍처로 서버가 여러개이고 데이터를 받을 때 response 형태가 다 다를 수 있다.
  - reponse 데이터안에 실제 데이터를 넘겨줄 때 data, result,.. 이렇게 다양한 객체이름으로 넘겨올 수 있음.
  - error를 넘겨줄 때도 statusCode는 200이고, 내용이 error 이렇게 만들어진 api도 있을 수 있음.
- 이렇게 프로토콜이 다양할 경우 프론트에서 그거에 맞게 다양하게 처리해야하는 문제 발생
- 프로토콜 표준화를 개발 초기라면 이거부터 정하는게 중요함!
- 그치만 이미 크기가 큰 서비스라면 ? 어떻게 표준화해야 할 까? 서버 표준 도입 vs. **클라이언트 표준 도입**
- 클라이언트 표준 도입 => BFF를 이용한다 !
  - 강의에선 graphql로 BFF 구성 => Rest API의 단점 규격이 없음. graphql은 schema로 규격만들 수 있음.
    - graphql 응답의 내용물을 클라이언트 사이드에서 결정할 수 있다.
  * BFF 구축하면 보안적으로도 더 안전함, BFF도 서버에 있으니까 서버에서 서버로 api 요청하는 거니까 더 private함. 그리고 cors 걱정은 client와 bff사이에서만 신경쓰면 됨.
