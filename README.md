# Unimo: Beyond (VR Multiplayer)

[프로젝트 한줄 소개]
- VR 멀티플레이 게임. Firebase(Auth/Realtime DB) 기반 계정/데이터, Photon PUN2 기반 매칭/룸 시스템을 구현했습니다.

## Demo
- Trailer: [링크]
- Portfolio Video: [링크]
- Screenshots: [이미지 경로]

## Project Info
- 개발 기간: [2025.05.02 - 2025.07.04] (43일)
- 개발 인원: [총 8명] (클라이언트: 4, 기획: 4)
- 플랫폼: [PC VR / QuestVR 등]
- 엔진/버전: Unity 2022.3.21f1
- 네트워크: Photon PUN2
- 백엔드: Firebase Authentication, Realtime Database

## My Role (개발 팀장 / 담당)
[한 줄 요약]
- 예: 계정/데이터 파이프라인(Firebase) + 멀티 매칭 시스템(PUN2) + 스테이지 데이터 최적화(ScriptableObject) 구현

[상세]
- Firebase
  - 회원가입/로그인/계정찾기(아이디/비밀번호 복구) UI 및 로직
  - UID 기반 Users/{UID} 데이터 모델 설계 및 초기 데이터 생성
  - 세션 토큰(동시 접속 방지) 트랜잭션 처리 및 비정상 종료 정리(OnDisconnect) 흐름
- PUN2
  - 랜덤 매칭(JoinRandomRoom) + 실패 시 즉시 방 생성(OnJoinRandomFailed)
  - 사설방(친구와): RoomCode 생성/참가, CustomRoomProperties 기록, 예외 처리 UI
  - 룸 UI 동기화: CustomProperties(닉네임/프로필/Ready) 기반 실시간 반영
  - 마스터 변경 대응(OnMasterClientSwitched), 취소/이탈 상태 정리(Cancel/Leave)
- 데이터 최적화/구조
  - StageData(1~50), SkinData ScriptableObject 기반 로컬 참조 구조로 서버 요청 최소화
  - 확정 시점에만 DB 업데이트(프리뷰/선택 단계는 로컬 처리)
- 인게임(스테이지)
  - CSV/BPM 기반 탄막 스케줄 시스템(BulletPatternLoader/Executor) 구현
  - 패턴 3종: Normal / Guided(스폰 시점 플레이어 방향 타겟팅) / Pattern(Angle/Range 프리셋)

## Key Features (핵심 구현 포인트)

### 1) 인증/계정 관리 (Firebase Auth + Realtime DB)
- 회원가입: CreateUserWithEmailAndPasswordAsync
- 가입 성공 시 UID 기준 초기 유저 데이터 생성(즉시 플레이 가능 상태로 초기화)
- 닉네임 정책: Trim + 길이 제한(2~8) + 중복 체크(OrderByChild("Nickname").EqualTo())
- 계정찾기
  - 아이디 찾기: 힌트 기반 Users 조회
  - 비밀번호 찾기: ID + 힌트 검증 후 비밀번호 재설정 메일 발송

### 2) 세션 제어 (동시 접속 방지)
- 로그인 성공 시 세션 토큰 생성(Guid.NewGuid())
- Users/{UID}/Session을 트랜잭션으로 원자적 기록(RunTransaction)
  - 기존 토큰 없음: 토큰/타임스탬프 저장
  - 기존 토큰 존재: Abort (중복 로그인 차단)
- 비정상 종료 대비: OnDisconnect로 세션 정리
- 토큰 변경 감지 시 즉시 SignOut 처리(중복 접속 차단)

### 3) 스테이지 데이터 최적화 (서버 최소 요청 + ScriptableObject)
- StageData ScriptableObject(1~50)에 스테이지 메타데이터(아이콘/설명/BGM/난이도 등) 고정
- 서버(Firebase)에서는 진행도(하이스코어 등) 최소 데이터만 로드
- 별/언락 UI는 하이스코어 기반 로컬 규칙으로 즉시 계산/표시
- VR 환경에서 불필요한 네트워크 요청을 줄여 지연/프레임 드랍 리스크를 낮춤

### 4) 상점/스킨 시스템 (로컬 카탈로그 + 확정 저장)
- SkinData ScriptableObject로 스킨 카탈로그 로컬 참조
- 클릭 시 즉시 프리뷰 반영(서버 요청 없이 로컬)
- 구매/해금 조건(코인/별) 검증 및 안내 UI
- 실제 DB 업데이트는 확정 시점에만 수행(불필요한 업데이트 최소화)
- 상점 종료 시 미확정 프리뷰 롤백 처리

### 5) 옵션/설정
- 닉네임/프로필: 서버 동기화(Users/{UID}) + 멀티 표기 동기화(PhotonNetwork.NickName)
- 오디오: AudioMixer(BGM/SFX) + PlayerPrefs 저장
- 언어: PlayerPrefs 저장 후 Translation/데이터셋 갱신으로 즉시 UI 반영

### 6) 대전 모드(멀티 매칭)
- 랜덤 매칭: JoinRandomRoom -> 실패 시 즉시 방 생성 -> 룸 상태 동기화
- 친구와(사설방): RoomCode 기반 생성/참가 + 에러 처리 UI + 마스터 변경 대응
- 룸 UI 동기화: CustomProperties(닉네임/프로필/Ready) 기반 실시간 반영
- 시작 조건 검증: 인원/Ready 미충족 시 시작 차단

## Tech Stack
- Unity 2022.3.21f1
- Photon PUN2
- Firebase Auth / Realtime Database
- [추가: XR Interaction Toolkit, URP 등 실제 사용 항목]

## Project Structure (예시)
Assets/
  Scripts/
    Firebase/
      Authentication.cs
      FirebaseManager.cs
      UserGameData.cs
    Network/
      MatchingSystem.cs
    Stage/
      StageManager.cs
      StageData.cs
      StageInfoDataSet.cs
    Shop/
      SkinData.cs
      ShopCanvasCtrl.cs
    Bullet/
      BulletPatternLoader.cs
      BulletPatternExecutor.cs
      BulletSpawnerManager.cs
  Resources/
    StageDatas/ (1..50)
    [기타 ScriptableObject 경로]

## Setup (로컬 실행)
> 아래는 README에 흔히 들어가는 “초기 세팅” 섹션 틀입니다. PUN2/Firebase 프로젝트별 키는 저장소에 올리지 말고 별도 설정 파일로 관리 권장.

### Prerequisites
- Unity Hub + Unity 2022.3.21f1
- Photon AppId (PUN2)
- Firebase 프로젝트 설정(google-services.json 등 플랫폼별 파일)

### Firebase Config
1. Firebase Console에서 프로젝트 생성
2. Authentication 활성화(Email/Password)
3. Realtime Database 생성 및 Rules 설정
4. Unity 프로젝트에 Firebase SDK 적용
5. [프로젝트 내 설정 위치/방법 작성]

### Photon Config
1. Photon Dashboard에서 App 생성
2. AppId 발급
3. Unity PUN2 설정에 AppId 입력
4. [리전/룸 옵션/네임 정책 등 작성]

## Data Model (Realtime DB 예시)
Users/
  {UID}/
    Nickname: string
    Coins: int
    Stars: int
    UnlockedSkins: [...]
    EquippedSkin: string
    EquippedProfile: string
    MapHighScore: { "1": int, ... "50": int }
    Session:
      Token: string
      Timestamp: long

## Troubleshooting
- Firebase 권한 오류(Permission denied): Rules/인증 상태 확인
- Photon 입장 실패: AppId/리전/네트워크 상태 확인
- [프로젝트에서 실제로 자주 나온 이슈 추가]
