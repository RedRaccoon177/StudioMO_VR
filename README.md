# Unimo: Beyond (VR Multiplayer)

Unity 기반 VR 멀티플레이 프로젝트입니다.  
Firebase(Authentication/Realtime Database)로 계정·유저데이터·세션 제어를 구성했고, Photon PUN2로 매칭/룸 시스템을 구현했습니다.

## Project Info
- 개발 기간: 2025.05.02 - 2025.07.04
- 개발 인원: 8명 (클라이언트 4 / 기획 4)
- 플랫폼: PC VR / QuestVR
- 엔진: Unity 2022.3.21f1
- 네트워크: Photon PUN2
- 백엔드: Firebase Authentication, Realtime Database

## My Role (Team Lead / Client)
- Firebase 기반 계정/데이터 파이프라인 + PUN2 매칭 시스템 + Stage/Skin 데이터 로컬 최적화 구조 구현

## What I Built
### Firebase (Auth / Realtime DB)
- 회원가입/로그인/계정찾기(아이디/비밀번호) UI 및 로직 구현
- 가입 성공 시 UID 기준 초기 유저 데이터 생성(즉시 플레이 가능 상태로 초기화)
- 동시 접속 방지를 위한 세션 토큰 생성 및 트랜잭션 기록(RunTransaction) 처리
- 비정상 종료 대비 OnDisconnect 기반 세션 정리 흐름 구성

### Photon PUN2 (Match / Room)
- 랜덤 매칭: JoinRandomRoom -> 실패 시 즉시 방 생성(OnJoinRandomFailed)으로 대기 시간 최소화
- 사설방(친구와): RoomCode 생성/참가 + 룸 옵션/예외 처리 UI 구성
- CustomProperties 기반 룸 UI 동기화(닉네임/프로필/Ready 상태 실시간 반영)
- 마스터 변경 대응(OnMasterClientSwitched), 매칭 취소/이탈 시 상태 정리

### Data Optimization (VR 고려)
- StageData(1~50), SkinData를 ScriptableObject로 로컬 참조해 서버 요청 최소화
- 선택/프리뷰 단계는 로컬 처리, 확정 시점에만 DB 업데이트하여 네트워크 부하 절감

### Stage Ingame (Bullet Pattern)
- CSV/BPM 기반 탄막 패턴 스케줄 시스템(BulletPatternLoader/Executor) 구현
- 3종 탄막(기본/유도/프리셋 패턴)으로 난이도와 리듬을 구성
