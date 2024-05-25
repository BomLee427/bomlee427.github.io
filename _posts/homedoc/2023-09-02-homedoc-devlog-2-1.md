---
title: "2-1. 도메인 계층 - 계층형 패키지 구조와 Member 클래스"
date: 2023-09-02T00:00:00+09:00
categories: ["HomeDoc 개발 회고록"]
tags: ["엔티티","패키지 구조"]
classes: wide
toc: true
---

이번 포스팅에서는 **도메인 계층**의 소스코드를 리뷰하고자 한다.

도메인(엔티티) 계층은 보통 가장 먼저 작성하는 클래스이기도 하지만, 명확한 기준이나 설계 없이 작성할 경우 이후의 개발 과정에서 끊임없이 수정하게 되는 클래스이기도 하다.

그러나 객체와 테이블을 매핑하는 ORM 환경에서 엔티티 클래스의 변경은 곧 DB 구조의 변경을 의미한다. 실제 프로덕션 환경에서는 DB 설계가 자주 바뀌는 것은 결코 바람직하지 않을 것 같아, 이번 코드 리뷰를 통해 실제 변경 과정을 추적하고 리팩터링을 진행하면서 완성된 엔티티 클래스는 무엇을 포함해야 하고 무엇을 고려해야 할지 고민해 볼 것이다.

### 계층형 설계

계층별 코드 리뷰를 하기에 가장 앞서, 우선 현재의 프로젝트 패키지 구조를 살펴보자.

![프로젝트 구조](https://velog.velcdn.com/images/bomlee427/post/533e8e74-4222-44c7-86fe-766928c5f3ba/image.png)

HomeDoc은 먼저 계층별로 패키지를 나누고, 그 아래에서 필요에 따라 다시 세부 패키지를 나누고 있는 **계층형 패키지 구조**를 택하고 있다.

프로젝트의 패키지 구조는 보통 **계층형**과 **도메인형**의 두 가지 선택지가 있다. 둘 중 무엇을 골라야 할지 결론부터 이야기하자면 **정답은 없다.** 오히려 프로젝트의 규모와 도메인의 특성, 개발 환경에 따라 패키지 구조 역시 계속해서 변할 수 있도록 유연해야 한다고 한다. [(참고: 프로젝트 폴더 구조에 대해 / 인프런, 김영한님 답변)](https://www.inflearn.com/questions/16046/comment/42317)

때문에 프로젝트 시작 단계부터 패키지 구조에 대해 너무 깊게 고민하지 않으려 했다. 개발 도중 내가 미처 생각지 못한 더 나은 패키지 구조가 있다면 검토를 거쳐 변경하는 것도 나쁜 선택이 아님을 알았기 때문이다.

개발 초기에 계층형 구조를 택한 이유는 두 가지였다.
첫 번째로 **익숙하기 때문**이다. HomeDoc의 모델이 된 지난 회사의 프로젝트가 계층형 구조로 되어 있었기 때문에, 우선 익숙한 형태로 시작하면 크게 헤메지 않고 개발을 진행할 수 있을 것 같았다.
두 번째로 각 계층 내 **클래스의 책임**을 조금 더 명확히 나누고 싶었다. 지난 회사의 프로젝트가 계층형 구조로 되어있었다고는 하나 Java가 아닌 다른 언어로 작성되어 OOP의 설계 원칙들을 명확히 따르고 있지는 않았기 때문에, 첫 Java 프로젝트인 HomeDoc에서는 그 원칙들을 잘 지켜 각 클래스의 역할을 명확히 분리하고 싶었다. 명시적으로 패키지가 분리되어 있다면 각 클래스의 **단일 책임 원칙**이 더 확실히 지켜질 것이라 생각하였다.


### 기존 코드 살펴보기

패키지 구조를 결정했으니 실제 소스코드를 작성할 차례다.
우선 핵심 도메인 중 하나인 **Member** 클래스를 가장 먼저 살펴보고자 한다. ERD와 UML은 [이전 포스팅](https://bomlee427.github.io/homedoc-devlog-1)에서 다루었다.


#### Member
```java
@Entity
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
public class Member extends BaseAuditingEntity {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "member_id")
    private Long id;

    @Column(unique = true)
    private String email;

    private String password;

    private OauthType oauthType;

    private String oauthId;

    private String name;

    private String refreshToken;

    @OneToMany(mappedBy = "member", fetch = FetchType.LAZY)
    private List<MemberAuthority> memberAuthorities;

    private Member(String email, String password, String name) {
        this.email = email;
        this.name = name;
        this.password = password;
    }

    private Member(OauthType oauthType, String oauthId, String name) {
        this.oauthType = oauthType;
        this.oauthId = oauthId;
        this.name = name;
    }

    public static Member createDirectMember(String email, String password, String name) {
        return new Member(email, password, name);
    }

    public static Member createSnsMember(OauthType oauthType, String oauthId, String name) {
        return new Member(oauthType, oauthId, name);
    }

    public void updateEmail(String email) {
        this.email = email;
    }

    public void updateName(String name) {
        this.name = name;
    }

    public void updateRefreshToken(String refreshToken) {
        this.refreshToken = refreshToken;
    }

    public void updateOauthInfo(OauthType oauthType, String oauthId) {
        this.oauthType = oauthType;
        this.oauthId = oauthId;
    }

    public void updatePassword(String password) {
        this.password = password;
    }

    public void resign() {
        super.delete();
    }

}

```

현재 Member 클래스의 형태는 위와 같다. 클래스 생성에는 직접가입/SNS가입 두 종류의 팩토리 메서드를 사용하였고, 객체의 캡슐화를 위해 Setter 사용을 지양해야 한다는 원칙에 따라 updateXXX 형태의 객체 수정용 메서드를 별도로 작성하였다. 언뜻 보면 별 문제가 없어 보이지만, 시간이 꽤 지나고 다시 코드를 찬찬히 뜯어보니 고쳐야 할 부분들이 몇 군데 눈에 보인다.

첫 번째로 **객체 생성 방식**이다. 변수의 갯수가 많거나 동일한 타입의 변수가 여러개일 때는 생성자나 팩토리 메서드로 객체를 생성하기 어려워진다. 매개변수에 값을 잘못 할당할 가능성도 있으며, 어떤 매개변수에 어떤 값이 할당되는지에 대한 가독성도 떨어지기 때문이다(코틀린에는 이를 보완하기 위한 named arguments라는 기능이 있다).

특히 두 개의 팩토리 메서드 중 createDirectMember()의 경우 String 변수 세 개를 인자로 받는다. 만일 실수로 변수의 순서가 뒤바뀌어 들어가도 검증할 방법이 없다. 현재는 필드 수가 많지 않아 위험도가 낮을지 모르지만 Member 클래스는 애플리케이션의 핵심 엔티티인 만큼 추후 요구사항이 어떤 방식으로 확장될지 모른다. 따라서 빌더 패턴 등으로 리팩터링이 필요해 보인다.

두 번째로 oauthType과 oauthId가 개별 필드로 흩어져 있는 점이다. 이 두 필드는 데이터베이스 상에서 **논리적 복합키** 역할을 하므로 변경이 발생한다면 반드시 동시에 다루어져야 한다. 따라서 두 필드를 묶어 하나의 **임베디드 타입**으로 관리하는 것이 더 안정적일 것이다.

### 리팩터링

#### Member
```java
@Entity
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
public class Member extends BaseAuditingEntity {

	...
    
    @Embedded
    private OauthInfo oauthInfo;

	...
    
    @Builder(builderMethodName = "defaultBuilder")
    private Member(String email, String password, String name) {
        this.email = email;
        this.name = name;
        this.password = password;
    }

    @Builder(builderMethodName = "oauthBuilder")
    private Member(OauthType oauthType, String oauthId, String name) {
        this.oauthInfo = OauthInfo.of(oauthType, oauthId);
        this.name = name;
    }

	...
    
    public void updateOauthInfo(OauthType oauthType, String oauthId) {
        this.oauthInfo = OauthInfo.of(oauthType, oauthId);
    }
}
```

#### OauthInfo
```java
@Embeddable
public class OauthInfo {

    @Enumerated(EnumType.STRING)
    private OauthType oauthType;

    private String oauthId;

    public static OauthInfo of(OauthType oauthType, String oauthId) {
        OauthInfo oauthInfo = new OauthInfo();
        oauthInfo.oauthType = oauthType;
        oauthInfo.oauthId = oauthId;

        return oauthInfo;
    }
}

```

리팩터링한 코드의 모습은 위와 같다. 롬복 어노테이션으로 두 가지의 빌더 패턴을 적용하고, OauthInfo라는 임베디드 객체를 만들어 값 변경 시 두 필드가 반드시 하나의 객체로 다루어지도록 했다.

여담으로 객체 생성 방식을 선택하는 기준에 대해 고민해 본 적이 있는데 이것 역시 정답은 없는 것 같다. 가장 최근 진행했던 팀 프로젝트에서는 1) 매개변수가 네 개 이상이거나 2) 같은 타입의 매개변수가 세 개 이상이거나 3) 필드에 값이 할당될 수도, 그렇지 않을 수도 있는 경우가 잦은 객체의 경우 빌더 패턴을 선택하기로 합의한 뒤 개발을 진행하였는데, 이처럼 기준을 명확하게 정하니 여러 명이 코드를 작성해도 일관성이 깨지지 않아 만족스럽게 개발을 진행한 기억이 있다.

### 마치며

개발 회고를 위한 리팩터링을 진행하긴 했지만, 사실 프로젝트를 한창 개발하던 도중에도 Member 클래스는 수도 없이 바뀌었다. 가장 처음 작성한 Member 클래스에는 리프레시 토큰이나 유저 권한 테이블과의 연관관계도 없었고, Auditing 기능을 위한 상속이나 soft delete 메서드도 개발 도중에 필요를 느껴 뒤늦게 추가한 것이다.

이처럼 몇 번씩 수정을 거친 경험 덕분에 다음 프로젝트에서는 초반부터 필요한 것들을 빠르게 생각해낼 수 있었다. 이번 리팩터링도 예전에는 보이지 않던 것이 보이게 되었다는 의미일 테니, 그동안의 공부가 헛되지는 않았다는 증거일 것이다. :-)

다음 시간에는 이 프로젝트의 또 다른 핵심 도메인 중 하나인 건강 측정 기록에 관련된 엔티티들을 살펴보기로 하자.
