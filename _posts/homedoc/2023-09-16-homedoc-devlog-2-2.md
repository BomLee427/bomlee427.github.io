---
title: "2-2. 도메인 계층 - Measurement, Glucose, Pressure"
date: 2023-09-16T00:00:00+09:00
categories: ["HomeDoc 개발 회고록"]
tags: ["엔티티"]
classes: wide
toc: true
---

이번 포스팅에서는 HomeDoc의 핵심 도메인 중 하나인 **건강 측정 기록** 엔티티들을 살펴보기로 한다.


### 개요
현재 HomeDoc이 기록할 수 있는 건강 정보에는 **혈당**, **혈압**의 두 종류가 있다.
해당 기능들의 간략한 요구사항은 아래와 같다.

---
1. 유저 플로우
    1) 모바일에 연결이 가능한 측정기기, 혹은 외부 건강 앱 연동을 통해 모바일 앱에 건강 측정 데이터를 입력한다.
    2) 모바일 앱은 입력된 데이터를 가공해 백엔드 서버에 전달한다.

2. 모든 건강 측정 기록은 **회원, 수기입력 여부, 메모, 측정일시**라는 공통 필드를 갖고 있고, 추가로 각각의 건강기록에 해당하는 측정치 필드를 갖는다.

3. 기록 조회 시, 단순 조회뿐 아니라 통계 조회도 필요하다.

---

우선 공통 필드가 다수 있고, 측정기록이라는 같은 속성을 갖고 있으므로 상속 관계로 설계하기로 하였다.
먼저 Measurement라는 추상 클래스를 작성하고, 실제 측정 기록 엔티티는 그것을 상속받는 자손 클래스로 작성한다.


### 현재 코드
#### Measurement
```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE) // 상속관계 매핑
@DiscriminatorColumn(name = "dtype")
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
public abstract class Measurement extends BaseAuditingEntity {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "measurement_id")
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "member_id")
    private Member member;

    @Enumerated(EnumType.STRING)
    @Column(columnDefinition = "varchar(64) default 'MANUAL'")
    private Manual manual; //MANUAL, AUTOMATIC

    private String memo;
    private LocalDateTime measuredAt;

    protected void setMeasurement(Member member, Manual manual, Normality normality, String memo) {
        this.member = member;
        this.manual = manual;
        this.memo = memo;
        this.measuredAt = LocalDateTime.now();
    }

    public void updateToManual() {
        this.manual = Manual.MANUAL;
    }

    public void updateMemo(String memo) {
        this.memo = memo;
    }

    public void deleteMeasurement() {
        super.delete();
    }

    public boolean isValidAuthor(Long memberId) {
        return (this.getMember() != null && (this.getMember().getId() == memberId));
    }
}
```

상속관계 매핑 전략은 `SINGLE_TABLE`로 설정했다. 이에 대한 자세한 내용은 이전 포스트 [1. 문서와 설계](https://bomlee427.github.io/homedoc-devlog-1)에 작성했기에 여기서는 리뷰를 생략하기로 한다.

우선 공통 필드를 작성하고 다른 엔티티(Member)와의 연관관계를 설정했다. 수기입력 여부는 두 가지 값만 들어갈 수 있도록 Enum으로 정했다. 값 수정 메서드의 네이밍 규칙은 `updateXXX()` 형태로 정했기 때문에 규칙에 맞게 작성한다. 
단 수기입력 여부 변경의 경우 자동측정->수기입력 방향의 변경만 가능하도록 했다. 이미 수기입력한 정보를 자동측정 기록으로 변경할 일은 없기 때문이다. 해당 메서드는 자동측정 값이 잘못되거나 했을 경우 측정값을 수정하면서 수기입력 여부를 업데이트하는 경우만을 위해 작성하였다.

`isValidAuthor()` 메서드는 불필요해 보인다. member가 null인 경우는 데이터 입력 단계에서 이미 검증할 수 있다. 입력 단계에서 검증했음에도 member가 null이 된다는 것은 db에서의 데이터 정합성 자체가 깨졌다는 것이므로 예외처리 단계에서 이 부분까지 체크하는 것은 오버엔지니어링에 가깝다. 또 member의 id를 체크하는 부분은 Repository 레벨에서 where 절을 추가함으로써 처리할 수 있다. 따라서 `@Deprecated` 어노테이션을 달아준다. 추후 Service 코드를 리팩터링한 뒤 삭제하기로 한다.

코드를 보다 보니 한 가지 더 생각나는 점이 있다. manual 필드의 `@Column(columnDefinition = "varchar(64) default 'MANUAL'")` 부분이다. 모든 필드에 올바르게 스키마를 설정했는지 확인하지 않았기 때문에, 앞으로 리뷰하는 엔티티들에서는 해당 부분도 체크할 것이다.

우선 **manual** 필드의 `@Column` 어노테이션에 `nullable=false` 속성을 추가하고, `@NotNull` 어노테이션을 붙인다. 객체의 필드 값 역시 `Manual.MANUAL`로 초기화되도록 하고, 따라서 불필요해진 `default 'MANUAL'` 부분은 삭제한다.

또 **member** 필드에도 `nullable=false`를 설정해야겠다는 생각이 들었다. 그러나 비즈니스 로직에 따라 해당 필드가 일시적으로 null인 경우도 있을 수 있으니 무턱대고 설정했다간 예기치 못한 오류를 마주칠 수도 있다. 작성자가 null일 수 있는 순간으로는 엔티티 생성 시점 등을 생각해 볼 수 있다. 생성 시점의 비즈니스 로직을 살펴보자.

```java
public Glucose toEntity(Member member) {
    return Glucose.builder()
            .member(member)
            .manual(valueOfOrNull(Manual.class, manual))
            .fasted(valueOfOrNull(Fasted.class, fasted))
            .meal(valueOfOrNull(Meal.class, meal))
            .value(value)
            .memo(memo)
            .build();
}
```

Measurement를 상속받는 Glucose 클래스는 위와 같이 엔티티 생성에 **빌더 패턴**을 이용하고 있다. 따라서 객체 생성 시점에 곧바로 member 값이 전달되므로 작성자가 null일 수 있는 순간은 존재하지 않는다. `@NotNull` 어노테이션과 `nullable=false` 옵션을 추가한다.

# 
`setMeasurement()` 는 유지보수성 향상과 보일러 플레이트 코드 삭제를 위해 작성한 메서드다. Measurement를 상속받는 자식 클래스들은 모두 생성자에서 부모 클래스가 갖고 있는 공통 필드의 값을 초기화하게 되는데, 자식 클래스의 갯수가 늘어날수록 이 부분의 중복 코드가 늘어나고, 부모 클래스에 변경이 생길 경우 리팩터링이 어려워질 수 있다. 따라서 Measurement 클래스의 필드를 초기화하는 메서드를 만들고, 접근제어자를 `protected`로 설정해 자식 클래스에서 해당 메서드에 접근할 수 있도록 하였다.

### 리팩터링

#### Measurement
```java

@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE) // 상속관계 매핑
@DiscriminatorColumn(name = "dtype")
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
public abstract class Measurement extends BaseAuditingEntity {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "measurement_id")
    private Long id;

    @NotNull
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "member_id", nullable = false)
    private Member member;

    @NotNull
    @Enumerated(EnumType.STRING)
    @Column(columnDefinition = "varchar(64)", nullable = false)
    private Manual manual = Manual.MANUAL; //MANUAL, AUTOMATIC

    private String memo;
    private LocalDateTime measuredAt;

    protected void setMeasurement(Member member, Manual manual, Normality normality, String memo) {
        this.member = member;
        this.manual = manual;
        this.normality = normality;
        this.memo = memo;
        this.measuredAt = LocalDateTime.now();
    }

    public void updateToManual() {
        this.manual = Manual.MANUAL;
    }

    public void updateMemo(String memo) {
        this.memo = memo;
    }

    public void deleteMeasurement() {
        super.delete();
    }

    @Deprecated
    public boolean isValidAuthor(Long memberId) {
        return (this.getMember() != null && (this.getMember().getId() == memberId));
    }
}
```
리팩터링한 코드의 모습은 위와 같다.

추가로, manual 필드의 `columnDefinition`을 설정하며 애플리케이션 단의 예외처리는 어느 정도가 적당할까에 대한 궁금증이 생겼다. 사실 늘 고민하는 부분이지만 이것 역시 답은 없는 것 같다. 서비스의 특성과 상황, 세부 비즈니스 로직에 따라 최적의 선은 늘 달라질 것이다. 나는 평소 오버엔지니어링을 하는 경향이 있어 그런 점을 조금 더 신경쓰고 있다.

### Glucose와 Pressure
이번에는 Measurement를 상속받아 실제 엔티티 클래스를 작성할 차례다. **Glucose**와 **Pressure**는 각각 혈당과 혈압 측정에 대한 엔티티이다. 포함해야 하는 항목은 아래와 같다.

---

### 혈당
- 끼니(아침, 점심, 저녁)
- 공복 여부(공복, 식후)
- 혈당 수치(실수 값)

### 혈압
- 이완기 혈압(정수 값)
- 수축기 혈압(정수 값)

---

그 외 공통 필드는 모두 Measurement에서 상속받는다.


#### Glucose
```java
@Entity
@Getter
@DiscriminatorValue("GLU")
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Glucose extends Measurement {

    @Builder
    public Glucose(Member member, Manual manual, String memo, Meal meal, Fasted fasted, Double value) {
        this.meal = meal;
        this.fasted = fasted;
        this.value = value;
        this.setMeasurement(member, manual, memo);
    }

    @Enumerated(EnumType.STRING)
    @Column(name = "glucose_meal")
    private Meal meal; //BREAKFAST, LUNCH, DINNER
    @Enumerated(EnumType.STRING)
    @Column(name = "glucose_fasted")
    private Fasted fasted; //FASTED, AFTER_MEAL
    @Column(name = "glucose_value")
    private Double value;

    public void updateMeal(Meal meal) {
        this.meal = meal;
    }

    public void updateFasted(Fasted fasted) {
        this.fasted = fasted;
    }

    public void updateValue(Double value) {
        this.value = value;
    }
}

```

#### Pressure
```java
@Entity
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@DiscriminatorValue("PRE")
@Getter
public class Pressure extends Measurement {

    @Builder
    public Pressure(Member member, Manual manual, String memo, Integer diastolic, Integer systolic) {
        this.diastolic = diastolic;
        this.systolic = systolic;
        this.setMeasurement(member, manual, memo);
    }

    @Column(name = "pressure_dias")
    private Integer diastolic;
    @Column(name = "pressure_sys")
    private Integer systolic;

    public void updateDiastolic(Integer diastolic) {
        this.diastolic = diastolic;
    }

    public void updateSystolic(Integer systolic) {
        this.systolic = systolic;
    }

}

```

필드 갯수가 많으므로 객체 생성에 빌더 패턴을 사용하고 있고, 상위 클래스인 Measurement에서 상속받는 필드에는 Measurement의 protected 메서드인 `setMeasurement()`를 사용해 값을 초기화하도록 했다.


### 마치며
이것으로 서비스의 핵심 도메인 두 가지를 모두 살펴보았다. 이후의 코드리뷰 순서를 조금 고민해 보았는데, 계층별 리뷰보다는 도메인별 리뷰가 흐름을 살펴보기에는 더 좋을지도 모르겠다는 생각이 들었다. 이후의 방향은 다음번 포스트를 작성할 때 결정하기로 하고 오늘은 이만 마친다. :-)