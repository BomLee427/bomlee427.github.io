---
title: "3-1. 회원 및 건강정보 - Controller"
date: 2023-09-18T00:00:00+09:00
categories: ["HomeDoc 개발 회고록"]
tags: 
classes: wide
---

이번 포스트에서는 **회원** 및 **건강정보** 도메인의 Controller 계층을 살펴본다.

**회원**은 서비스를 이용하는 일반 고객을 의미하며, **건강정보**는 각각의 회원이 가입 후 입력하는 성별, 키, 몸무게, 질환, 가족력 등의 기초적인 건강정보를 의미한다.

회원과 관련된 인증 및 인가 서비스에 관해서는 이후에 다루기로 했다.



### 요구사항

- 이메일 주소 하나 당 한 명의 회원만 가입 가능하다(이메일은 중복되지 않는다).
- 회원과 건강정보는 반드시 1:1의 관계이다.
- 회원은 일부 개인정보 및 건강정보를 수정할 수 있다.
- 회원이 탈퇴할 경우 해당 회원의 건강정보, 측정기록, 병원 등록 등의 연관 정보는 모두 삭제되어야 한다.


### 현재 코드

#### MemberController
```java
@RestController
@RequiredArgsConstructor
@RequestMapping("/v1")
public class MemberController {

    private final MemberService memberService;

    /**
     * 이메일 회원가입
     */
    @PostMapping("/member")
    public ResponseEntity<CommonResponseDto<Map<String, Long>>> directJoinV1(
            @Valid @RequestBody final MemberCreateRequestDto dto
    ) {
        return ResponseEntity.ok(CommonResponseDto.getResponse(Map.of("id", memberService.directJoin(dto))));
    }

    /**
     * 내 정보 조회
     */
    @PreAuthorize("hasAnyRole('USER')")
    @GetMapping("/myinfo")
    public ResponseEntity<CommonResponseDto<MemberResponseDto>> getMyInfoV1() {
        return ResponseEntity.ok(CommonResponseDto.getResponse(memberService.getMemberById(getCurrentUserPK().orElse(null))));
    }

    /**
     * 내 정보 수정
     */
    @PreAuthorize("hasAnyRole('USER')")
    @PatchMapping("/myinfo")
    public ResponseEntity<CommonResponseDto<MemberResponseDto>> updateMyInfoV1(@RequestBody MemberUpdateRequestDto dto) {
        return ResponseEntity.ok(CommonResponseDto.getResponse(memberService.defaultInfoUpdate(getCurrentUserPK().orElse(null), dto)));
    }

    /**
     * 회원 탈퇴
     */
    @PreAuthorize("hasAnyRole('USER')")
    @DeleteMapping("/member")
    public ResponseEntity<CommonResponseDto<Map<String, Long>>> resignV1() {
        return ResponseEntity.ok(CommonResponseDto.getResponse(Map.of("id", memberService.resign(getCurrentUserPK().orElse(null)))));
    }
}

```

#### HealthProfileController
```java
@RestController
@RequiredArgsConstructor
@RequestMapping("/v1/healthprofile")
@PreAuthorize("hasAnyRole('USER')")
public class HealthProfileController {

    private final HealthProfileService healthProfileService;

    /**
     * 자신의 건강정보 조회
     */
    @GetMapping("/my")
    public ResponseEntity<CommonResponseDto<HealthProfileResponseDto>> getHealthProfileV1() {
        return ResponseEntity.ok(CommonResponseDto.getResponse(healthProfileService.getHealthProfile(SecurityUtil.getCurrentUserPK().orElse(null))));
    }

    /**
     * 자신의 건강정보 수정
     */
    @PutMapping("/my")
    public ResponseEntity<CommonResponseDto<HealthProfileResponseDto>> updateHealthProfileV1(
            @Valid @RequestBody final HealthProfileUpdateRequestDto dto
    ) {
        return ResponseEntity.ok(CommonResponseDto.getResponse(healthProfileService.updateHealthProfile(SecurityUtil.getCurrentUserPK().orElse(null), dto)));
    }
}

```

우선 코드를 보자마자 가장 신경쓰이는 것은 한 줄당 코드 길이이다...
보아하니 메소드나 지네릭이 여러 번 중첩되어 있는 부분이 많은 것 같다. 메소드 중첩은 변수를 추출하거나 ResponseEntity의 빌더 패턴을 사용하면 깔끔하게 해결할 수 있을 것 같은데, 지네릭 중첩은 지금 당장 해결할 방안이 잘 생각나지 않았고, 자연스럽게 **ResponseEntity**의 필요성에 대해 고민하게 되었다.

### ResponseEntity
**ResponseEntity**는 HTTP 응답을 디테일하게 설정하기 위한 객체로, 스프링 프레임워크에서 제공하고 있다. 

그러나 ResponseEntity를 사용하지 않더라도, `@RestController`나 `@ResponseBody` 어노테이션을 쓴다면 자바 객체를 return하는 것만으로도 간단히 HTTP Response에 직렬화된 형태의 HTTP Body를 실어 보낼 수 있다. 그렇다면 ResponseEntity를 써야 하는 이유는 무엇일까? 그 답을 알기 위해 ResponseEntity의 코드를 살펴보자.

```java
public class ResponseEntity<T> extends HttpEntity<T> {
	private final Object status;
}
```

ResponseEntity를 살펴보면 생성자나 빌더, getter 등을 제외하고는 오직 상태를 나타내는 **status** 필드만을 멤버변수로 갖는 것을 알 수 있다. 그렇다면 HttpEntity로부터는 무엇을 상속받고 있을까.

```java
public class HttpEntity<T> {
	private final HttpHeaders headers;

	@Nullable
	private final T body;
}
```

HttpEntity는 HttpHeaders 타입의 **headers**와 T타입의 **body**를 멤버변수로 갖는다. ResponseEntity의 T타입은 상속받은 body의 타입이다.

ResponseEntity는 body 외에도 status와 headers를 추가로 포함하고 있고, 또 이를 설정하기 위한 수많은 빌더 메소드를 제공하고 있다. 반면 `@ResponseBody`를 통해 객체를 리턴하는 방식은 HTTP body 외에 다른 것은 설정할 수 없다. 즉 ResponseEntity를 사용하는 이유는 **조금 더 디테일하게 HTTP 응답을 설정할 수 있어서**인 것이다.

현재 HomeDoc에서 `200 OK` 외의 응답은 모두 ExceptionHandler로 처리하고 있다. 따라서 별도의 status 설정은 필요하지 않다. 또 HTTP header에 관한 스펙도 현재는 존재하지 않는다.

그래서 **ResponseEntity를 현재 시점에서 사용할 필요는 없다**는 결론을 내렸다. 추후 필요해진다 하더라도 만일 전역에서 요구하는 헤더라면 컨트롤러에서 일일이 설정하는 것이 아니라 다른 방식으로 제공해야만 하고, 메서드마다 개별로 요구하는 헤더라면 해당 메서드를 수정하면 된다.

그 외에 SecurityContext에서 memberId를 구하는 부분은 변수로 추출하는 것이 깔끔할 것 같다.

### CommonResponseDto
```java
@Getter
public class CommonResponseDto<T> {

    @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd HH:mm:ss")
    private final LocalDateTime time = LocalDateTime.now();
    private String message;
    private T data;

    public static <T> CommonResponseDto<T> getResponse() {
        CommonResponseDto<T> response = new CommonResponseDto<>();
        response.message = "SUCCESS";
        return response;
    }

    public static <T> CommonResponseDto<T> getResponse(T data) {
        CommonResponseDto<T> response = new CommonResponseDto<>();
        response.message = "SUCCESS";
        response.data = data;
        return response;
    }

    public static <T> CommonResponseDto<T> getResponse(String message, T data) {
        CommonResponseDto<T> response = new CommonResponseDto<>();
        response.message = message;
        response.data = data;
        return response;
    }
}
```

CommonResponseDto는 API의 응답 형태를 일관성있게 하기 위해 작성한 클래스다. 데이터 응답 시간, 메시지, 요청한 데이터 등을 포함한다.
메서드 중첩에 대해서도 생각하다 보니 자연스럽게 이 클래스에도 생각이 미치게 되었다. 여기서는 `getResponse()`라는 세 개의 팩토리 메서드를 통해 객체를 생성하고 있는데, 굳이 여러 가지로 오버로드할 필요 없이 빌더 패턴을 이용하면 충분히 해결이 가능하다는 생각이 들었다. 또 해당 클래스 역시 dto의 일종이긴 하지만 CommonResponse라는 이름에서 충분히 기능을 유추할 수 있다고 판단해 클래스명도 줄이기로 했다.

### 리팩터링

#### MemberController
```java
@RestController
@RequiredArgsConstructor
@RequestMapping("/v1")
public class MemberController {

    private final MemberService memberService;

    /**
     * 이메일 회원가입
     */
    @PostMapping("/member")
    public CommonResponse<Map<String, Long>> directJoinV1(
            @Valid @RequestBody final MemberCreateRequestDto dto
    ) {
    
        return CommonResponse.<Map<String, Long>>builder()
                .data(Map.of("id", memberService.directJoin(dto)))
                .build();
    }

    /**
     * 내 정보 조회
     */
    @PreAuthorize("hasAnyRole('USER')")
    @GetMapping("/myinfo")
    public CommonResponse<MemberResponseDto> getMyInfoV1() {
    
        Long memberId = SecurityUtil.getCurrentUserPK().orElseThrow(NotAuthenticatedException::new);
        return CommonResponse.<MemberResponseDto>builder()
                .data(memberService.getMemberById(memberId))
                .build();
    }

    /**
     * 내 정보 수정
     */
    @PreAuthorize("hasAnyRole('USER')")
    @PatchMapping("/myinfo")
    public CommonResponse<MemberResponseDto> updateMyInfoV1(@Valid @RequestBody final MemberUpdateRequestDto dto) {
    
        Long memberId = SecurityUtil.getCurrentUserPK().orElseThrow(NotAuthenticatedException::new);
        return CommonResponse.<MemberResponseDto>builder()
                .data(memberService.defaultInfoUpdate(memberId, dto))
                .build();
    }

    /**
     * 회원 탈퇴
     */
    @PreAuthorize("hasAnyRole('USER')")
    @DeleteMapping("/member")
    public CommonResponse<Map<String, Long>> resignV1() {
    
        Long memberId = SecurityUtil.getCurrentUserPK().orElseThrow(NotAuthenticatedException::new);
        return CommonResponse.<Map<String, Long>>builder()
                .data(Map.of("id", memberService.resign(memberId)))
                .build();
    }
}
```

#### HealthProfileController
```java
@RestController
@RequiredArgsConstructor
@RequestMapping("/v1/healthprofile")
@PreAuthorize("hasAnyRole('USER')")
public class HealthProfileController {

    private final HealthProfileService healthProfileService;

    /**
     * 자신의 건강정보 조회
     */
    @GetMapping("/my")
    public CommonResponse<HealthProfileResponseDto> getHealthProfileV1() {

        Long memberId = SecurityUtil.getCurrentUserPK().orElseThrow(NotAuthenticatedException::new);
        return CommonResponse.<HealthProfileResponseDto>builder()
                .data(healthProfileService.getHealthProfile(memberId))
                .build();
    }

    /**
     * 자신의 건강정보 수정
     */
    @PutMapping("/my")
    public CommonResponse<HealthProfileResponseDto> updateHealthProfileV1(
            @Valid @RequestBody final HealthProfileUpdateRequestDto dto
    ) {

        Long memberId = SecurityUtil.getCurrentUserPK().orElseThrow(NotAuthenticatedException::new);
        return CommonResponse.<HealthProfileResponseDto>builder()
                .data(healthProfileService.updateHealthProfile(memberId, dto))
                .build();
    }
}

```

#### CommonResponse
```java
@Getter
public class CommonResponse<T> {

    @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd HH:mm:ss")
    private final LocalDateTime time = LocalDateTime.now();
    private String message;
    private T data;

    @Builder
    public CommonResponse(String message, T data) {
        this.message = message == null ? "SUCCESS" : message;
        this.data = data;
    }
}
```

컨트롤러 전체적으로 ResponseEntity 대신 CommonResponse를 리턴하도록 했고, memberId는 변수로 추출한 다음 null일 경우에는 인증 예외를 던지도록 변경했다. 또 MemberController의 `updateMyInfoV1()`의 매개변수에 `@Valid` 어노테이션과 final 키워드가 누락된 것을 발견해 고쳐주었다.
CommonResponse는 빌더 패턴으로 변경했고, 별도의 메시지가 입력되지 않을 경우 SUCCESS로 초기화되도록 했다. 이렇게 하니 코드가 훨씬 보기 좋아졌다. 이후 유지보수할 일이 생기더라도 쉽게 고칠 수 있을 것이다.

### 마치며
지난 포스트를 작성하며 다음 리뷰를 어떤 순서로 해야 할지 고민이 있었는데, 도메인 별로 리뷰하기로 결정한 뒤 포스트를 작성하다 보니 생각보다 내용이 길어져 결국 도메인-계층별 포스트가 되었다. 앞선 도메인에서 미리 고친 부분들은 이후의 포스트에서는 생략하면 되니 나쁘지 않은 방식인 것 같아 계속 이렇게 리뷰하기로 했다.

또 오늘 코드를 살펴보니 예전에 작업하며 적어놓은 TODO 주석이 굉장히 많았는데, 이것도 리뷰를 하면서 해결하지 않으면 영원히 손대지 않을 것 같아서 다음 포스트부터는 그 주석들도 해결하면서 진행하려고 한다. 이번 포스트의 주석들은 다음 포스트를 작성하며 해결한 뒤 여기에 덧붙일 예정이다.

오늘은 유독 코드가 깔끔해진 것을 보니 참 속이 시원했다. 커밋 내역을 보니 마지막 커밋은 11월이고, 아마 실제 코드를 작성한 시기는 이보다 훨씬 앞서였을 것이다. 거의 1년에 가까운 시간을 거슬러 과거의 내가 쓴 코드를 보니 부족한 점이 정말 많이 보인다. 성장의 증거라고 생각하니 더할 나위 없이 기쁘다. 미래의 나도 지금의 나에게서 부족한 점을 찾을 수 있도록 계속해서 열심히 기록하자. :-)

### 참고
https://dev-splin.github.io/spring/Spring-ResponseEntity/#responsebody
https://tecoble.techcourse.co.kr/post/2021-05-10-response-entity/
