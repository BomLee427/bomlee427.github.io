---
title: "E2E 테스트 프레임워크, 무엇이 있을까?"
date: 2024-05-28T19:00:00+09:00
categories: ['Test']
tags: ['E2E', 'Selenium', 'Cypress', 'Playwright']
classes: wide
toc: true
---


최근 기능 테스트(E2E 테스트) 도구에 대해 조사할 일이 있었다.
그 과정에서 정리한 내용을 블로그에 기록한다.

---

## 기능 테스트(E2E 테스트)란?
- 어플리케이션을 동작시킴으로써 실제 사용자 경험과 동일한 테스트를 수행
- 어플리케이션 전체를 사용해 테스트
- 사용자 시나리오 또는 전체 기능이 올바르게 작동하는지를 확인하기 위해 작성
- 장점: 실제 사용자의 사용 방식과 가장 유사하므로 높은 테스트 신뢰도를 보장
- 단점: 테스트 실행속도가 느리고 테스트 구축/유지보수 비용이 높음

---
## 1. For Web
### Selenium IDE
- 오픈 소스 (Apache 2.0 License)
- GUI 기반 브라우저 확장 프로그램
    - 프로그래밍 언어를 모르더라도 사용 가능
    - CLI 지원
- Command, Target, Value로 구성된 간단한 테스트 케이스를 작성할 수 있음
    - Command(행위)는 [open, click, type check, run script] 등의 단순 동작으로 구성
    - Target은 Command의 대상, Value는 Command의 결과로 기대되는 값
    - 루프, 디테일한 테스트 환경 설정, 외부 시스템과의 상호작용 등 유연한 테스트는 불가능
- 낮은 학습 난이도와 테스트 케이스 작성 비용
- 작성된 테스트를 WebDriver 코드로 내보낼 수 있음

### Selenium WebDriver
- 오픈 소스 (Apache 2.0 License)
- 브라우저를 보다 더 유연하게 제어하기 위한 API 라이브러리
- 코드 기반 테스트
    - Java, C#, Python, Ruby, JavaScript 지원
    - 소스코드와 함께 형상 관리 및 배포 가능
    - 외부 시스템(데이터베이스 쿼리, 모듈 또는 컴포넌트, API)과 쉽게 통합 가능
- 브라우저 드라이버를 직접 구현하는 방식으로 실행됨
    - 브라우저를 객체(Object)로 취급하여 객체지향적 테스트 코드 작성 가능
    - 다양한 브라우저를 지원하므로 Cross Browser Test에 적합
    - 테스트 속도가 느림
    - 병렬 테스팅 미지원(별도 라이브러리 등을 이용하여 구현 시 간접적으로 가능)
- 테스트 구축 및 테스트 케이스 작성 비용이 높음
- 성숙하고 풍부한 오픈소스 생태계

### Cypress
- 핵심 테스트 도구인 Test Runner는 오픈 소스(MIT License)
- 클라우드 기반으로 다양한 부가기능을 제공하는 Cypress Cloud는 부분 유료
- All in One Package로 구축과 사용이 쉬움
- Test Runner
    - 오픈 소스
    - Cypress 테스트 실행 도구
    - Javascript만 지원
    - 브라우저 상에서 직접 테스트 실행
        - Javascript를 통해 iframe 내의 DOM, Window, LocalStorage 등의 객체에 접근
        - Selenium에 비해서는 빠른 실행 시간, Playwright에 비해서는 느린 실행 시간
        - 컴퓨팅 자원을 많이 사용
- Cypress Cloud
    - 부분 유료 : Free 플랜을 포함해 총 4단계의 요금 플랜
    - 공식 문서: https://docs.cypress.io/guides/cloud/introduction
    - 클라우드 기반 테스트 환경 제공
    - 테스트 결과는 모두 클라우드에 저장
    - 로그 확인/디버깅/분석/진단 등 추가 기능을 포함한 대시보드
    - 테스트 병렬 처리, 테스팅 머신 간 로드 밸런싱이나 스마트 오케스트레이션 등의 테스트 속도 및 자원 관리 최적화 도구 제공

### Playwright
- 오픈 소스 (Apache 2.0 License)
- Playwright 클라이언트와 서버는 Single WebSocket을 통해 연결
- Headless Browser를 통해 테스트 실행
    - DevTools Protocol을 이용해 브라우저와 통신
    - Selenium에 비해 매우 빠른 실행속도
    - Cypress에 비해 약간 빠른 실행속도
- 테스트 결과 상세 리포트, 비디오 녹화 기능, 병렬 테스트 기능 기본 내장
- 안드로이드 네이티브 테스트를 **실험적으로** 지원
    - 실제 디바이스가 아닌 에뮬레이터 기반
    - 안드로이드용 크롬, 안드로이드 웹뷰 지원

---

## 2. For API
### Postman
- Javascript로 Pre-request script, Post-request script 작성
- 코드 기반으로 Request를 생성하고 Response 검증 가능
- 테스트 관련 기능은 대부분 무료 또는 부분 유료이나, 플랜에 따라 Workspace 액세스 인원이 제한됨

---

## 3. For App
### Appium
- 오픈 소스(Apache 2.0 License)
- 안드로이드, iOS 지원
    - 단일 코드베이스 크로스 플랫폼에서의 높은 코드 재사용성
    - 안드로이드 4.2 이하 버전은 지원하지 않음
- 네이티브 지원, 하이브리드 앱 제한적 지원
- 실제 디바이스 기반 및 에뮬레이터 기반 테스트 모두 가능
- Selenium WebDriver와 호환되는 모든 언어 지원

### Selendroid
- 오픈 소스(Apache 2.0 License)
- 안드로이드만 지원
- 네이티브, 하이브리드 모두 제공
- Java, C#, Perl, Haskell, PHP, Javascript, PHP, Python, Ruby 지원

### Calabash
- 오픈 소스(Eclipse Public License 1.0)
- UI 상호작용 기반(Behavior-Driven) 테스트 프레임워크
    - 안드로이드, iOS 모두 접근 가능
- Ruby만 지원, 다만 Ruby 대신 Cucumber 사용 가능

---
### Cucumber?
- 시나리오 기반의 Test Runner
- 자연어에 가까운 Ghrekin 문법을 사용해 테스트 시나리오 작성
    - 개발자: 별도의 학습이 필요하고, 숙련되기 전까지는 생산성이 낮을 수 있음
    - 테스트 이해관계자: 프로그래밍 지식이 없더라도 쉽게 테스트 시나리오를 이해할 수 있음
- 프로그래밍 언어로 실제 테스트 코드를 작성해 시나리오 검증
    - Calabash 단독 사용 시 Ruby로만 테스트 코드 작성 가능
    - Cucumber는 Java, Javascript, Ruby 지원
- 테스트 시나리오와 테스트 코드 사이에 데이터를 주고받을 수 있음
- Calabash 외에도 Cypress, Playwright 등 다른 테스트 프레임워크와 통합 가능
- 테스트 자동화(Progressive automation)보다는 인수 테스트(Acceptance test)에 적합
    - 주로 개발자만 테스트에 접근하는 상황에서 얻는 이점이 거의 없음
    - 프로그래밍 언어가 아닌 별도의 시나리오 문법으로 테스트 케이스를 작성해야 하는 것이 부담일 수 있음


---

### References
- [테스트의 종류 - 기능테스트(E2E), 통합테스트, 단위테스트](https://shorttrack.tistory.com/9)
- [서버 사이드 테스트 자동화 여정](https://engineering.linecorp.com/ko/blog/server-side-test-automation-journey-1)
- [테스트 자동화 VS 자동화 테스트](https://blog.naver.com/wisestone2007/221848534889)
- [What is test automation?](https://www.zaptest.com/ko/%ED%85%8C%EC%8A%A4%ED%8A%B8-%EC%9E%90%EB%8F%99%ED%99%94%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9E%85%EB%8B%88%EA%B9%8C-%EC%A0%84%EB%AC%B8-%EC%9A%A9%EC%96%B4-%EC%97%86%EC%9D%8C-%EA%B0%84%EB%8B%A8%ED%95%9C)
- [포스트맨에서 API 테스트 자동화하기](https://velog.io/@galaxy/Postman%EC%97%90%EC%84%9C-API-%ED%85%8C%EC%8A%A4%ED%8A%B8-%EC%9E%90%EB%8F%99%ED%99%94%ED%95%98%EA%B8%B0)
- [What Flaky Tests Can Tell You](https://www.stickyminds.com/article/what-flaky-tests-can-tell-you)
- [Test Practice using Selenium - 1](https://velog.io/@dahunyoo/Test-Practice-using-Selenium-1)
- [End-to-end testing of API endpoints with Postman and Postman BDD](https://medium.com/nixplay/end-to-end-testing-of-api-endpoints-with-postman-and-postman-bdd-f2ec27fcaf75)
- [Streamline your API workflow with Postman](https://medium.com/nixplay/streamline-your-api-workflow-with-postman-d2b15e1cf605)
- [Playwright OVERVIEW](https://ipex.tistory.com/entry/Playwright-OVERVIEW)
- [How Cypress Makes Testing Fun](https://medium.com/angular-in-depth/how-cypress-makes-testing-fun-a56da1294285)
- [Cypress vs Playwright: A Detailed Comparison](https://www.lambdatest.com/blog/cypress-vs-playwright/)
- [A Selenium IDE vs. WebDriver comparison](https://www.techtarget.com/searchsoftwarequality/tip/A-Selenium-IDE-vs-WebDriver-comparison)
- [Selenium IDE vs WebDriver: Main Differences for Testers and Developers](https://www.blazemeter.com/blog/selenium-ide-vs-webdriver)
- [Cypress vs. Selenium: Compare test automation frameworks](https://www.techtarget.com/searchsoftwarequality/tip/Cypress-vs-Selenium-Compare-test-automation-frameworks)
- [Do you use Cucumber to write integration tests?](https://www.reddit.com/r/rails/comments/132217t/do_you_use_cucumber_to_write_integration_tests/?rdt=64758)