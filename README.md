# Unimo: Beyond (VR Multiplayer)

Unity 기반 VR 멀티플레이 프로젝트입니다.  
Firebase(Authentication / Realtime Database)로 **계정·유저데이터·세션 제어**를 구성했고,  
Photon PUN2로 **매칭/룸 시스템**을 구현했습니다.

---

## 🎬 Links
- 📺 포트폴리오 영상: [추가 예정]
- 📄 PPT: [추가 예정]
- 🔗 Repository: [추가 예정]

---

## 📋 목차
- [게임 소개](#game-intro)
- [프로젝트 정보](#project-info)
- [내 역할](#my-role)
- [내가 구현한 핵심](#what-i-built)
  - [Firebase (Auth / Realtime DB)](#firebase)
  - [Photon PUN2 (매칭 / 룸)](#photon)
  - [VR 최적화 (서버 최소 요청 + 로컬 참조)](#vr-opt)
  - [스테이지 인게임 (CSV/BPM 탄막 패턴)](#stage)
- [기술 스택](#tech-stack)

---

<a name="game-intro"></a>
## 🎯 게임 소개
Unimo: Beyond는 **우주선을 조종해 광물을 채굴하며 적 탄막을 회피**하는 VR 멀티플레이 닷지 액션 게임입니다.  
본 README는 “게임 설명”보다 **제가 직접 개발한 시스템(코드) 중심**으로 정리한 포트폴리오 문서입니다.

---

<a name="project-info"></a>
## 🧾 프로젝트 정보
- 개발 기간: 2025.05.02 - 2025.07.04
- 개발 인원: 8명 (클라이언트 4 / 기획 4)
- 플랫폼: PC VR / Quest VR
- 엔진/버전: Unity 2022.3.21f1
- 네트워크: Photon PUN2
- 백엔드: Firebase Authentication, Realtime Database

---

<a name="my-role"></a>
## 👤 내 역할 (개발 팀 리더 / 클라이언트)
기획 팀 리더와 분리된 **개발 팀 리더(클라이언트 파트 리드)** 로서 아래 시스템을 주도적으로 구현했습니다.

- Firebase 기반 **인증/유저 데이터 파이프라인 + 세션 제어**
- Photon PUN2 기반 **랜덤 매칭/사설방(코드) 룸 시스템**
- Stage/Skin **ScriptableObject 로컬 참조 최적화 구조**
- CSV/BPM 기반 **스테이지 탄막 스케줄 시스템**

---

<a name="what-i-built"></a>
## ✅ 내가 구현한 핵심

<a name="firebase"></a>
### 1) Firebase (Auth / Realtime DB)
- 회원가입/로그인/계정찾기(아이디/비밀번호) **UI 및 로직 구현**
- 회원가입 성공 시 UID 기준으로 **초기 유저 데이터 생성**  
  - “가입 성공”에서 끝나지 않고, **즉시 플레이 가능한 상태**로 초기화
- 닉네임 정책: **Trim + 길이 제한(2~8) + 중복 체크**
- **동시 접속 방지(세션 토큰)**
  - 로그인 시 토큰 생성
  - Session을 트랜잭션으로 원자적 기록하여 중복 로그인 차단
  - 비정상 종료 대비 OnDisconnect 기반 세션 정리 흐름 구성

<a name="photon"></a>
### 2) Photon PUN2 (매칭 / 룸)
- **랜덤 매칭(Quick Match)**
  - `JoinRandomRoom` -> 실패 시 `OnJoinRandomFailed`에서 즉시 방 생성  
  - 대기 시간을 줄이고 “입장 시도 -> 실패 시 생성 -> 성공 시 동기화”를 한 흐름으로 구성
  - 입장/퇴장 콜백으로 룸 UI를 실시간 갱신
  - PlayerCount가 MaxPlayers(4) 충족 시 **자동 시작 트리거**
- **사설방(친구와 / RoomCode)**
  - RoomCode 생성/참가 흐름 구성
  - `CustomRoomProperties`에 RoomCode 기록
  - 코드 오류/입장 실패 시 즉시 안내 UI로 사용자 흐름 제어
- **룸 상태 동기화**
  - `CustomProperties`로 닉네임/프로필/Ready 상태를 실시간 반영
  - 마스터 변경(`OnMasterClientSwitched`) 대응
  - 매칭 취소/이탈 시 UI/네트워크 상태 정리로 잔여 상태 방지

<a name="vr-opt"></a>
### 3) VR 최적화 (서버 최소 요청 + 로컬 참조)
- StageData(1~50), SkinData를 **ScriptableObject 카탈로그로 로컬 참조**
- 선택/프리뷰 과정은 서버 요청 없이 즉시 반영 (로컬 처리)
- **저장(확정) 시점에만 DB 업데이트**하여 불필요한 통신 최소화  
  - VR 환경에서 네트워크 지연/프레임 드랍 리스크를 줄이도록 설계

<a name="stage"></a>
### 4) 스테이지 인게임 (CSV/BPM 탄막 패턴)
- CSV를 파싱해 탄막 스케줄을 구성하고, BPM 타이밍 기반으로 실행하는 파이프라인 구현
  - Loader: CSV 파싱
  - Executor: 비트 타이밍 실행
- 탄막 3종으로 난이도/리듬을 설계
  - 기본 탄막: 베이스라인(방향/각도 분산)
  - 유도 탄막: 스폰 시점 플레이어 위치를 샘플링해 해당 방향으로 발사
  - 프리셋 패턴 탄막: 각도/범위 프리셋으로 패턴화된 탄막 연출

---

<a name="tech-stack"></a>
## 🧩 기술 스택
- Unity 2022.3.21f1
- Photon PUN2
- Firebase Authentication / Realtime Database
- XR Interaction Toolkit
- URP
- C#
