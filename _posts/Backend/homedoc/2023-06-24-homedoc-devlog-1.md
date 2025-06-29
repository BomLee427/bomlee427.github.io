---
title: "1․ 문서와 설계"
date: 2023-06-24T00:00:00+09:00
categories: ['Backend', "HomeDoc 개발 회고록"]
tags: ["문서화","설계"]
classes: wide
toc: true
---


이 시리즈는 포트폴리오용 개인 프로젝트인 [HomeDoc](https://github.com/BomLee427/homedoc)의 개발 회고록이다.

당초 목적은 회고록이 아닌 개발기를 쓰는 것이었다.
개발을 진행하며 체계적으로 그날그날 개발한 기록을 남기려 했는데, 온통 처음 해보는 것들 뿐이라 결국 흐지부지되고 말았다. 아무리 계획을 세워도 나중에 가서 이게 필요했구나, 저게 필요했구나, 하고 뒤늦게 깨닫는 일의 연속.
내 개발 자체가 체계적이지 않은데, 어떻게 기록을 체계적으로 하겠는가...

결국 기록 없이 프로젝트를 얼추 완성시켰지만, 여전히 그 시간들을 모두 내 것으로 체득한 것은 아닌 것 같았다.
그래서 회고록 겸 코드 리뷰의 형태로 뒤늦게나마 프로젝트의 개발기를 남겨보기로 결심했다.

첫 포스트의 주제는 역시 개발 전 필요한 산출물,  **프로젝트 명세와 설계**에 관한 것이다.
당시에는 별 생각 없이 의무적으로 작성했던 것들을 현재 시점에서 되짚으며 이런저런 생각을 적어보려고 한다.


### 헬스케어 서비스, HomeDoc

[HomeDoc 프로젝트 명세서](https://bomlee427.github.io/homedoc-project-spec)
HomeDoc의 프로젝트 명세 전문은 위 글에서 확인할 수 있다.


> HomeDoc은 만성질환 환자들이 자신의 건강 측정 정보를 입력하고 측정 기록 및 통계를 열람할 수 있는 건강관리 서비스입니다.
병원 측에서도 자신에게 배정된 환자의 건강정보를 열람하고 실제 진료에 참고할 수 있습니다.
환자는 모바일 앱, 병원은 웹을 통해 접근하는 것을 가정하고 개발되었습니다.


HomeDoc의 간단한 서비스 개요는 위와 같다. 요컨대 환자와 의사가 모두 사용할 수 있는 건강측정 앱의 백엔드 서버 애플리케이션이다.

헬스케어 도메인을 선정한 이유는 개발 경험이 있어서이다. HomeDoc은 재직 당시 개발하던 백엔드 서버와 거의 비슷한 기능 구성을 갖고 있다. 당시 개발하며 느꼈던 고민이나 아쉬운 점, 배운 점 모두를 프로젝트에 녹여내고 싶었다.

서버는 REST API 형태로 작성하기로 했다. 재직 당시 문서가 없는 서버의 API 스펙을 손수(...) yml 파일로 작성한 경험이 있었기 때문에, OPEN API 3.x의 스펙이나 구조 자체는 어느 정도 이해하고 있었다. 서비스를 이용하는 실제 클라이언트의 요구사항이 없는 상황이기 때문에 API 구성은 최대한 RESTful하게 작성하고자 했다.



### 문서화의 중요성

이때까지는 블로그에 개발일지를 써 보겠다는 야심찬 계획이 있었기 때문에 순서대로 차근차근 단계를 밟아나갈 생각이었다.
가장 먼저 프로젝트 명세서를 작성해야 했다. 경험이 없었기 때문에 구글링을 통해 다양한 양식을 참고하며 나름의 명세를 써 나갔다.
쓰는 것 자체는 어렵지 않았다. 그러나 **왜** 명세를 써야 하는지에 대해서는 스스로에게 질문하지 않았다. 작성한 후에도 어차피 내 머릿속에 다 들어있으니까, 라는 이유로 문서를 거의 열어보지 않았고, 변경사항이 생기더라도 문서상에는 거의 반영이 되지 않았다.

**프로젝트 명세서를 왜 작성해야 할까?**
당시의 나라면 이렇게 답했을 것이다. 1. 프로젝트를 타인에게 소개하기 위해서, 2. 개발 과정을 기록하기 위해서.
그러나 이제는 다르게 답할 수 있다. 문서화를 하는 가장 중요한 이유는 **커뮤니케이션**을 위해서이다.

몇 주 전부터 포트폴리오용 팀 프로젝트에 참여하게 되었는데, 팀 작업을 하는 과정에서 문서화의 중요성을 절실히 느꼈다. 맹인모상(盲人摸象), 즉 장님 코끼리 만지기라는 우화처럼, 문서화가 되어 있지 않다면 설령 똑같은 작업을 하는 팀원들끼리도 머릿속에 같은 그림을 그릴 수 없다. 그러다 보면 정말 간단한 설명을 할 때에도 그것을 이해시키기 위해 한참이 걸리고, 한참 돌고 돌아 이해한 뒤에는 "아, 그 말씀이셨어요?" 라며 서로 허탈감을 느낀다.

물론 문서화를 _완벽하게_ 하는 데에는 생각보다 엄청난 시간이 소요된다. 그럼에도 _최소한의_ 문서화는 효율적인 커뮤니케이션을 위해 반드시 필요하다.

그래서 HomeDoc의 명세를 작성했던 경험이 이후 팀 프로젝트에서도 어느 정도 도움이 되었다. 첫 회의 때 의결해야 할 안건들이 프로젝트 명세에 그대로 녹아 있었고, 차례차례 짚어가며 수월하게 이야기를 진행할 수 있었다.

### 현실과 이상
기능 측면의 요구사항은 나름대로 상세히 작성했지만 기술 측면의 요구사항은 볼품없었다. 처음 해보는 Java/Spring/JPA 프로젝트라 아는 게 거의 없었기 때문이다. 실제 프로젝트는 책이나 강의와는 전혀 달랐고, 고심해서 작성했던 ERD와 UML도 개발이 어느 정도 완료된 지금과 비교해 보면 다른 부분이 많다.

**설계 당시 ERD**
![설계 당시 ERD](https://velog.velcdn.com/images/bomlee427/post/88b3bcb5-89ec-404c-bfcd-605865bc5702/image.png)


**개발 후 ERD**
![개발 후 ERD](https://velog.velcdn.com/images/bomlee427/post/02492cb1-6cd6-4209-942d-34e34e434a1a/image.png)

ERD에서 가장 눈에 띄는 차이로는 상속관계 매핑 전략의 변경이 있다. 현재는 **SINGLE_TABLE 전략**을 사용하고 있는데, 자식 객체의 필드 갯수나 복잡도가 크지 않고, 각 필드의 NULLABLE을 허용하는 것이 큰 문제가 되지 않기 때문에(건강 측정 기록이므로 오히려 허용해야 한다!) 해당 전략을 택했다.

설계 당시의 ERD는 **TABLE_PER_CLASS 전략**을 사용하고 있는데, ORM 관점에서 효율적이지 못하다는 사실을 알지 못하고 DB 측면에서만 생각해 습관적으로 작성한 것이다.

만일 지금 다시 DB를 설계한다면 **JOIN 전략**을 택할 것 같은데, 건강 측정 기록은 이 서비스의 핵심 비즈니스 로직이기 때문이다. 만일 서비스가 확장되어 현재처럼 단순 측정치 이외의 필드가 추가된다면 단일 테이블 전략으로는 대응하기 좋지 못하다. 최악의 상황에는 서비스 상황에서 DB 구조를 변경하게 될지도 모르니 미리미리 설계 당시부터 확장 가능성을 생각하는 것이 좋겠다.


**설계 당시 UML**
![설계 당시 UML](https://velog.velcdn.com/images/bomlee427/post/aae116bd-042e-4e76-9f15-ab5ad5fcd071/image.png)

**개발 후 UML**
![개발 후 UML](https://velog.velcdn.com/images/bomlee427/post/a7a0370b-351b-4f08-b789-62121299cc8b/image.png)

UML에서는 Member와 HealthProfile의 연관관계 방향이 완전히 달라졌다. 설계 당시에는 각 객체가 서로를 참조할 일이 있다면 양방향으로 설정해야 한다는 사실을 알지 못했다. 작업을 진행하며 HealthProfile을 통해 Member에 접근하는 경우가 있음을 알았고 양방향 1:1 연관관계로 수정했다. 그 후 쿼리 최적화 과정에서 설계와 정 반대인 단방향으로 변경했는데, **@OneToOne 연관관계에서는 주인 쪽의 LAZY 로딩만 정상 동작**한다는 것을 뒤늦게 알았기 때문이다. (https://thorben-janssen.com/hibernate-tips-same-primary-key-one-to-one-association/)

만일 지금 설계를 다시 한다면 해당 부분을 분명히 고려했을 것이다. 연관관계를 설정함에 있어 단/양방향을 결정하는 기준은 **객체가 객체를 참조할 필요가 있느냐**인데, 객체 지향 언어로 프로젝트를 진행하는 것 자체가 처음이다 보니 그 부분에 대한 감이 없었던 것이다.

그런 이유로 양방향으로 연관관계를 변경했다가, 1:1 연관관계에서는 주인 쪽만 LAZY 로딩이 이루어진다는 점 때문에 초기 설계와는 완전히 반대방향 연관관계로 변경하였다. Member는 단독으로 다룰 일이 훨씬 더 많고 조회도 가장 빈번히 일어나는 객체이므로 참조가 많을수록 연관관계가 복잡해지고 예상치 못한 쿼리가 나갈 가능성이 높아진다. Member를 통해 HealthProfile에 접근할 일이 있다면 **Shared PK(@MapsId)** 를 통해 찾기로 했다. 적어도 로그인할 때마다 HealthProfile을 EAGER 조회하는 쿼리를 날리는 것보다는 뭐든 나을 것이다.

### 마무리
혼자 하는 작업이다 보니 문서가 없더라도 별 문제 없이 프로젝트를 구성할 수 있었겠지만, 팀 작업이라면 이야기가 달라진다. 문서화를 하거나 이미 작성된 문서를 이해하는 훈련이 미리미리 되어있지 않으면 협업을 수월히 해낼 수 없다. 습관적으로 작성한 것이긴 했지만 좋은 훈련이 되었고, 앞으로의 작업에도 계속해서 참고할 수 있는 양분이 될 것 같다. :-)

다음 포스트부터는 계층별로 실제 코드 리뷰를 해보려 한다.
