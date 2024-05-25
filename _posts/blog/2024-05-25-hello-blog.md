---
title: "HomeDoc 프로젝트 명세서"
date: 2024-05-25T14:00:00+09:00
categories: ["HomeDoc 개발 회고록"]
tags: 
classes: wide
---

# homedoc

- HomeDoc은 만성질환 환자들이 자신의 건강 측정 정보를 입력하고 측정 기록 및 통계를 열람할 수 있는 건강관리 서비스입니다.
- 병원 측에서도 자신에게 배정된 환자의 건강정보를 열람하고 실제 진료에 참고할 수 있습니다.
- 환자는 모바일 앱, 병원은 웹을 통해 접근하는 것을 가정하고 개발되었습니다.


---


### 1. 프로젝트 개요

- 프로젝트 이름 : HomeDoc
- 프로젝트 요약 : 만성질환 환자 건강관리 서비스 백엔드용 API
- 주요 기능
    - 환자 : 회원가입 기능, 건강 측정 정보 CRUD, 조회 및 페이징, 통계 기능
    - 병원 : 환자 등록 및 삭제 기능, 환자 및 환자의 건강정보 열람 및 코멘트 기능
    - 관리자 : 회원(환자) 관리 기능(환자의 개인정보 및 건강정보는 열람할 수 없음), 제휴 병원 등록 및 관리 기능
- 개발 언어 : JAVA 11
- 개발 환경 : Spring boot 2.7.x, gradle 8, Spring Data JPA, Git
- API 스펙: OAS 3.0, JSON reponse

### 2. 서비스 설명

- 만성질환 환자들이 자신의 건강 측정 정보를 입력하고 측정 기록 및 통계를 열람할 수 있는 건강관리 서비스
- 병원 측에서도 자신에게 배정된 환자의 건강정보를 열람하고 실제 진료에 참고할 수 있음
- 환자는 모바일 앱, 병원은 웹을 통해 접근하는 것을 가정하고 개발

### 3. 요구사항 분석

**앱(환자)용 API**

1. 회원가입
    - 이메일을 통해 회원 구분
    - 가입 직후 사용자명 자동 생성됨. 이후 수정 가능
    - 가입 후 건강정보 선택 입력
        - 나이, 성별, 키/몸무게 등의 신체정보
        - 가족력 등의 질병위험요소
        - 해당 정보를 바탕으로 추천 건강정보 및 측정정보 가이드라인 등을 제공
    - 일반 회원가입은 이메일 형식의 아이디를 요구
    - 비밀번호는 8~20자 이내, 숫자와 영문자, 특수문자 사용 필수
    1. 건강알리미 기능 (미구현)
    - 각 측정 정보별로 정상범위를 설정하면 해당 범위를 벗어나는 측정 데이터가 있을 때 정상범위 초과 여부를 기록
    - 회원가입시 입력받은 건강정보가 존재할 경우, 측정정보에 대한 평균적인 정상범위를 계산하여 자동 설정
    - 입력된 건강정보가 없거나, 데이터가 부족하거나, 회원이 직접 입력하기를 원하면 직접 정상범위를 설정 가능
1. 측정정보 조회
    - 파라미터로 입력받은 기간에 따라 회원의 건강 측정 데이터 통계 제공
    - 측정 기록에 대한 단순 목록 형식의 조회 가능
    - 필터링 기능
        - 입력 방식 : 수기입력 여부로 필터링
        - 측정 조건 : 건강정보 별로 측정 조건이 있는 경우 해당 조건으로 필터링(예: 공복혈당만 표시 등)
1. 측정정보 입력 및 삭제
    - 혈당, 혈압 데이터 입력
    - 기기를 통한 자동 측정인지, 수기 입력 데이터인지 입력
    - 측정 시 설정된 측정 정상범위가 존재할 경우 정상범위 여부를 함께 체크하여 입력
    - 측정 조건이 있을 경우(예: 혈당 측정 시 공복여부/끼니) 측정 시에 조건 선택
    - 측정 시 메모 입력 가능
    - 측정정보 수정/삭제 가능(측정값 수정은 수기입력을 제외하고 불가)
1. 병원 연결 기능
    - 서비스 제휴 중인 병원의 해시코드를 입력해 해당 병원의 관리 환자로 등록 가능
    - 측정 기록에 작성된 병원측 코멘트 열람 가능
    - 관리 중단을 원할 시 병원등록 해제 가능

## API 간단 명세

### 환자용

| Name | Method | Path | Description |
| --- | --- | --- | --- |
| 로그인 | POST | /signin |  |
| 토큰 리프레시 | POST | /refresh |  |
| 회원가입 | POST | /member |  |
| 개인정보조회 | GET | /myinfo |  |
| 개인정보수정 | PATCH | /myinfo |  |
| 회원탈퇴 | DELETE | /resign |  |

| Name | Method | Path | Description |
| --- | --- | --- | --- |
| 건강정보조회 | GET | /healthprofile/my |  |
| 건강정보수정 | PUT | /healthprofile/my |  |

| Name | Method | Path | Description |
| --- | --- | --- | --- |
| 혈당측정 | POST | /measure/glucose |  |
| 혈당 목록조회 | GET | /measure/glucose |  |
| 혈당 통계조회 | GET | /measure/glucose/statistic | qureyParam required |
| 혈당 개별조회 | GET | /measure/glucose/{id} |  |
| 혈당수정 | PATCH | /measure/glucose/{id} |  |
| 혈당삭제 | DELETE | /measure/glucose/{id} |  |

| Name | Method | Path | Description |
| --- | --- | --- | --- |
| 혈압측정 | POST | /measure/pressure |  |
| 혈압 목록조회 | GET | /measure/pressure |  |
| 혈압 통계조회 | GET | /measure/pressure/statistic |  |
| 혈압 개별조회 | GET | /measure/pressure/{id} |  |
| 혈압수정 | PATCH | /measure/pressure/{id} |  |
| 혈압삭제 | DELETE | /measure/pressure/{id} |  |

| Name | Method | Path | Description |
| --- | --- | --- | --- |
| 병원 연결 | POST | /hospital | qureyParam required |
| 병원해제 | DELETE | /hospital/{id} |  |
| 등록병원 목록조회 | GET | /hospital |  |

### 관리자용

| Name | Method | Path | Description |
| --- | --- | --- | --- |
| 회원 리스트 | GET | /member |  |
| 특정 회원 조회 | GET | /member/{id} |  |
| 회원정보 수정 | PATCH | /member/{id} |  |
| 특정 회원 삭제 | DELETE | /member/{id} |  |

| Name | Method | Path | Description |
| --- | --- | --- | --- |
| 제휴병원 등록 | POST | /hospital |  |
| 제휴병원 수정 | PUT | /hospital/{id} |  |
| 제휴병원 삭제 | DELETE | /hospital/{id} |  |
| 제휴병원 목록조회 | GET | /hospital |  |

### 권한관리용

| Name | Method | Path | Description |
| --- | --- | --- | --- |
| 권한 추가 | POST | /memberAuthority/{id} |  |
| 권한 삭제 | DELETE | /memberAuthority/{id} |  |

## 애플리케이션 설계

- 계층형 패키지 설계 : Controller, Service, Repository, Domain

## 도메인 설계

- 회원(환자)
- 병원
- 측정정보
    - 혈당
    - 혈압


## ERD, UML
![_HomeDoc-ERD](https://github.com/BomLee427/homedoc/assets/103720594/a92b1130-d729-46c4-9683-a8dd11bca762)
![_HomeDoc-UML](https://github.com/BomLee427/homedoc/assets/103720594/9a811408-f638-41a9-8bf0-7e29b4423592)
