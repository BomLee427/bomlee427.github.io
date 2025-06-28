---
title: "이미지 렌더링 및 CSS 애니메이션 최적화 시도"
date: 2025-06-29T00:31:00+09:00
categories: ["Frontend"]
tags: ['트러블슈팅', '웹 브라우저 렌더링', '성능 최적화']
classes: wide
toc: true
---

이번주 업무 중 발생한 트러블 슈팅 과정과 생각의 흐름을 짧게 메모한다.

### 문제
- 배경: 파일 업로드 시 썸네일이 보여야 하고, 썸네일이 로딩 중일 때는 스피너가 표시된다.
- 문제: `Image` 태그가 화면에 렌더링되기 직전의 짧은 시간 스피너가 멈춘다.

### 추론 과정
가장 처음 이 문제를 확인한 것은 내가 아니라 팀 동료였다. 해당 동료는 예전부터 PC 사양이 다른 상황에서 최적화 문제를 발견해 준 경우가 종종 있었기 때문에 이번에도 비슷한 원인일 것으로 가정하고 다음 방식으로 검증했다.

- 매우 큰 이미지(width, height가 약 9000픽셀 정도)에서 문제 발생 빈도가 급증하는 것을 확인
- 내 PC에서 크롬 개발자 도구로 CPU 쓰로틀링을 걸자 같은 현상 발생

문제의 썸네일은 js의 `FileReader`를 사용해 `Data URL`로 인코딩해 표시하고 있었다. 

그래서 처음에는 큰 이미지의 `Data URL` 인코딩 과정에서 부하가 걸리는 것이라고 생각했는데, 크롬의 성능 탭에서 확인해 보니 해당 작업은 메인 스레드가 아닌 `Web worker`가 하고 있었다.

또, 한창 로딩 중이 아닌 로딩이 끝나기 직전에 애니메이션이 멈춰버리는 점, 디버깅해 본 결과 애니메이션이 멈춰 있을 때 스피너의 표시 조건이 `false`였던 점으로부터 미루어 보아 인코딩 작업이 아닌 **렌더링 작업** 시 발생하는 문제로 추정했다.

### 해결
큰 이미지를 화면에 그리는 과정에서 생긴 문제이므로, 리사이징 과정을 비동기로 추가해 부하를 줄여보기로 했다.

```ts
if (fileIsImage) {
    // async load thumbnail
    loading = true
    imageToDataURL(file) // Data URL encode
    .pipe(
        takeUntil(destroy$),
        concatMap(url => loadImage(url)), // load HTMLImageElement from URL
        concatMap(image => { 
        if (image.width > resizeThreshold || image.height > resizeThreshold) { // resizing
            return resizeImageAsync(image)
                .pipe(
                    map(resizedURL => [image, resizedURL] as [HTMLImageElement, string])
                )
        }
        return of([image, ''] as [HTMLImageElement, string])
        }),
        finalize(() => {
            loading = false
        })
    )
    .subscribe([image, resizedURL] => {
        // ...
    })
}
```

리사이징 후, CPU 쓰로틀링을 x20까지 걸어도 스피너 애니메이션이 멈추지 않는 것을 확인할 수 있었다.

(포스팅 중 갑자기 생각나서 `<img>` 태그에 `loading="lazy"` 를 걸어봤는데, 이걸로는 해결되지 않는다.)

### 더 공부할 내용

어찌저찌 해결은 했지만, 아직 정확한 원인을 설명할 수는 없다. 심지어 똑같은 썸네일 처리 샘플 코드를 Stackblitz에서 만들어보았는데 같은 상황이 거의 발생하지 않는 것으로 보아 프로젝트 내의 어떤 요인이 관여하고 있을 가능성도 매우 크다.

글을 쓰기 전까지는 단순히 해결 과정만 기록할 계획이었는데, 쓰다 보니 사실 내가 해결한 문제는 정말 겉핥기일 뿐이었다는 것을 깨달았다. 특히 스피너의 애니메이션을 멈추게 만드는 범인이 누구인지 아직도 모른다는 점이... 심지어 `transform` 속성은 js 메인 스레드가 아니라 브라우저가 담당하고 있다는데, 왜 이런 일이 일어났던 걸까. 아직도 미스터리다.

웹 사이트 렌더링 과정에서 브라우저와 js의 메인 스레드가 어떻게 상호작용하고, 무엇을 누가 얼만큼 분담하고 있는지를 아는 것이 비슷한 문제가 발생했을 때 해결할 수 있는 핵심 지식인 듯하다. 우선 아래의 아티클부터 천천히 읽어 보기로 했다.

- [속도가 왜 중요할까요?](https://web.dev/learn/performance/why-speed-matters?hl=ko)
- [최신 브라우저의 내부 살펴보기 3 - 렌더러 프로세스의 내부 동작](https://d2.naver.com/helloworld/5237120)


알면 알수록 모르는 것만 잔뜩 생긴다.

