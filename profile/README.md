# Z-igma iOS
👉 https://www.youtube.com/shorts/boQhE3Xd6c4

> 약속 장소 후보를 함께 추가하고 투표로 장소를 확정하는 iOS 네이티브 앱입니다.

사용자는 로그인 후 약속을 만들고, 지도에서 후보 장소를 추가하며, 참여자들과 투표를 통해 최종 장소를 결정할 수 있습니다. 또한 초대 링크, 실시간 접속자 표시, 지도 댓글 기능을 통해 약속 참여자들이 같은 화면에서 장소를 논의할 수 있도록 구현했습니다.

---

## 프로젝트 개요

| 구분 | 내용 |
| --- | --- |
| 프로젝트명 | Z-igma iOS |
| 앱 형태 | iOS 네이티브 앱 |
| 주요 목적 | 약속 생성, 장소 후보 추가, 장소 투표, 장소 확정 |
| 연동 서버 | Z-igma Spring Boot 백엔드 |
| 실행 방식 | Xcode에서 `Zigma.xcodeproj` 실행 |

---

## 주요 기능

| 기능 | 설명 |
| --- | --- |
| 로그인 | 카카오 로그인과 로컬 시연용 테스트 계정 로그인을 지원합니다. |
| 홈 화면 | 사용자가 참여 중인 약속 목록을 진행 중/확정 완료 상태로 확인합니다. |
| 약속 생성 | 약속명, 날짜, 카테고리, 복수 투표 여부를 입력해 새 약속을 만듭니다. |
| 지도 화면 | 약속별 후보 장소를 지도 위에 표시하고 새 후보지를 추가합니다. |
| 장소 투표 | 후보 장소에 투표하거나 투표를 취소할 수 있습니다. |
| 장소 확정 | 방장은 투표 결과를 확인하고 최종 장소를 확정할 수 있습니다. |
| 재투표 | 동점이 발생하면 동점 후보만 대상으로 재투표를 진행합니다. |
| 댓글 | 지도 위 특정 위치에 댓글을 남기고, 댓글 마커를 눌러 내용을 확인합니다. |
| 실시간 접속자 | WebSocket을 통해 현재 같은 약속 지도에 접속한 사용자를 표시합니다. |
| 초대 링크 | 방장이 초대 링크를 복사하고, 초대받은 사용자는 링크로 약속에 참여합니다. |

---

## 화면 구성

| 화면 | 설명 |
| --- | --- |
| 로그인 화면 | 카카오 로그인 또는 테스트 계정 로그인을 선택합니다. |
| 홈 화면 | 참여 중인 약속 목록을 확인하고 약속 상세로 이동합니다. |
| 약속 생성 화면 | 새 약속 정보를 입력해 서버에 생성 요청을 보냅니다. |
| 지도 화면 | 후보 장소, 댓글, 현재 접속자를 한 화면에서 확인합니다. |
| 투표 현황 화면 | 후보지별 득표 수와 투표 상태를 확인하고 장소를 확정합니다. |
| 계정 화면 | 현재 로그인 상태를 확인하고 로그아웃할 수 있습니다. |

---

## 사용자 흐름

```text
로그인
  ↓
홈 화면에서 약속 확인
  ↓
약속 생성 또는 기존 약속 선택
  ↓
지도 화면에서 후보 장소 추가
  ↓
참여자들이 후보 장소에 투표
  ↓
방장이 투표 현황 화면에서 장소 확정
```

초대 링크를 사용하는 경우에는 다음 흐름으로 동작합니다.

```text
방장이 투표 현황 화면에서 초대 버튼 클릭
  ↓
초대 링크 복사
  ↓
다른 사용자가 링크 실행
  ↓
앱이 inviteCode를 읽어 서버에 참여 요청
  ↓
홈 화면에 초대받은 약속 표시
```

---

## 코드 구조

```text
Zigma
├── App
│   ├── ZigmaApp.swift
│   └── AppState.swift
├── Core
│   ├── APIClient.swift
│   ├── AppConfig.swift
│   └── AppEvents.swift
├── Models
│   ├── APIEnvelope.swift
│   ├── PromiseModels.swift
│   └── MapModels.swift
├── Services
│   ├── PromiseService.swift
│   ├── CandidatePlaceService.swift
│   └── PresenceService.swift
├── Views
│   ├── LoginView.swift
│   ├── HomeView.swift
│   ├── CreatePromiseView.swift
│   ├── MapEntryView.swift
│   ├── PromiseMapView.swift
│   ├── VoteResultView.swift
│   ├── MainTabView.swift
│   └── AccountView.swift
└── Resources
    ├── Info.plist
    └── Assets.xcassets
```

---

## 주요 코드 설명

### App

`ZigmaApp.swift`는 앱의 시작점입니다. `RootView`에 `AppState`를 주입하여 로그인 상태, 초대 링크 처리, 토큰 저장 상태를 앱 전체에서 공유합니다.

`AppState.swift`는 로그인 토큰을 관리합니다. 앱이 `zigma://` 초대 링크로 실행되면 inviteCode를 읽어 서버에 약속 참여 요청을 보내는 역할도 담당합니다.

### Core

`APIClient.swift`는 서버 통신을 담당하는 공통 HTTP 클라이언트입니다. JWT 토큰을 Authorization 헤더에 넣어 요청하고, 서버 에러 메시지를 앱에서 사용할 수 있는 형태로 변환합니다.

`AppConfig.swift`에는 로컬 서버 주소, WebSocket 주소, 카카오 OAuth 주소가 정의되어 있습니다.

### Models

`PromiseModels.swift`는 약속 목록, 약속 상세, 약속 멤버, 초대 코드 응답 모델을 정의합니다.

`MapModels.swift`는 후보 장소, 투표 정보, 실시간 접속자, 지도 댓글 관련 모델을 정의합니다.

### Services

`PromiseService.swift`는 약속 목록 조회, 약속 상세 조회, 약속 생성, 초대 링크 생성, 초대 코드 참여 요청을 담당합니다.

`CandidatePlaceService.swift`는 후보 장소 조회/추가/삭제, 투표/투표 취소, 장소 확정, 재투표, 댓글 조회/생성을 담당합니다.

`PresenceService.swift`는 WebSocket STOMP 연결을 통해 현재 약속 지도에 접속한 사용자 목록을 실시간으로 받아옵니다.

### Views

`LoginView.swift`는 카카오 로그인과 로컬 테스트 계정 로그인을 제공합니다. 테스트 계정 로그인도 서버에서 실제 JWT를 발급받아 사용하므로 WebSocket과 초대 기능을 함께 시연할 수 있습니다.

`HomeView.swift`는 참여 중인 약속 목록을 보여줍니다. 약속을 선택하면 투표 현황 화면으로 이동합니다.

`CreatePromiseView.swift`는 새 약속 생성 화면입니다. 약속명, 날짜, 카테고리, 복수 투표 여부를 입력받아 서버에 생성 요청을 보냅니다.

`PromiseMapView.swift`는 지도 기반 기능을 담당합니다. 후보 장소 마커, 댓글 마커, 실시간 접속자 표시, 후보지 추가, 댓글 작성 기능을 포함합니다. 지도 좌표는 `MKMapView`의 좌표 변환을 사용해 실제 터치 위치와 마커 위치가 일치하도록 구현했습니다.

`VoteResultView.swift`는 약속의 장소 투표 현황을 보여줍니다. 후보별 득표 수, 동점 상태, 재투표, 장소 확정, 초대 링크 복사 기능을 처리합니다.

---

## 백엔드 연동

앱은 로컬 백엔드 서버와 통신하도록 설정되어 있습니다.

```swift
static let apiBaseURL = URL(string: "http://localhost:8080")!
static let webSocketURL = URL(string: "ws://localhost:8080/ws")!
```

주요 API 흐름은 다음과 같습니다.

| 기능 | API |
| --- | --- |
| 약속 목록 조회 | `GET /promises` |
| 약속 상세 조회 | `GET /promises/{promiseId}` |
| 약속 생성 | `POST /promises` |
| 후보 장소 조회 | `GET /promises/{promiseId}/candidates` |
| 후보 장소 추가 | `POST /promises/{promiseId}/candidates` |
| 장소 투표 | `POST /promises/{promiseId}/votes` |
| 투표 취소 | `DELETE /promises/{promiseId}/votes/{candidateId}` |
| 장소 확정 | `POST /promises/{promiseId}/confirmed` |
| 재투표 | `POST /promises/{promiseId}/revote` |
| 댓글 조회 | `GET /promises/{promiseId}/comments` |
| 댓글 생성 | `POST /promises/{promiseId}/comments` |
| 초대 링크 생성 | `POST /promises/{promiseId}/invite` |
| 초대 참여 | `POST /promises/invite/{inviteCode}` |
| 실시간 접속자 | `/topic/promises/{promiseId}/presence` |

---

## 실행 방법

1. 백엔드 서버를 로컬에서 실행합니다.

```bash
cd /Users/cheonseongjin/Documents/zigma-BE
sh gradlew bootRun
```

2. Xcode에서 iOS 프로젝트를 엽니다.

```text
ios/Zigma/Zigma.xcodeproj
```

3. 원하는 iPhone 시뮬레이터를 선택한 뒤 실행합니다.

4. 카카오 로그인 또는 테스트 계정 로그인을 통해 앱에 진입합니다.

---

## 시연용 로그인

카카오 계정이 여러 개 없을 때도 기능을 시연할 수 있도록 로컬 테스트 계정 로그인을 제공합니다.

| 버튼 | 설명 |
| --- | --- |
| 테스트 계정 1 | 방장 역할 시연용 계정 |
| 테스트 계정 2 | 초대받는 사용자 역할 시연용 계정 |

테스트 계정도 백엔드에서 실제 JWT를 발급받기 때문에 다음 기능을 함께 시연할 수 있습니다.

- 약속 생성
- 초대 링크 복사
- 초대 링크로 약속 참여
- WebSocket 실시간 접속자 표시
- 후보지 추가 및 투표
- 댓글 작성

---

## 마무리

Z-igma iOS는 약속 장소를 함께 정하는 과정을 모바일 환경에서 자연스럽게 사용할 수 있도록 구현한 SwiftUI 기반 앱입니다. 단순히 약속 목록을 보여주는 것을 넘어, 지도 위 후보지 제안, 실시간 접속자 확인, 댓글, 투표, 초대 링크까지 하나의 사용자 흐름으로 연결하는 데 중점을 두었습니다.
