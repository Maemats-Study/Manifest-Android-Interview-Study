# 안드로이드 프레임워크

## 0. 안드로이드란 무엇인가요?

- 안드로이드는 스마트폰 및 태블릿과 같은 모바일 기기를 위해 설계된 오픈소스 운영 체제
- 구글이 개발 및 유지 관리하며 리눅스 커널을 기반으로 함

### 계층 및 역할
- 리눅스 커널 (Linux Kernel)
    - 메모리, 프로세스 관리, 하드웨어 장치 드라이버 관리 등 OS의 기반
- HAL (Hardware Abstraction Layer)
    - 상위 수준의 JAVA API 프레임워크에 기기 하드웨어 기능을 노출하는 표준 인터페이스 제공
- 안드로이드 런타임 (ART) 및 코어 라이브러리
    - 앱을 실행하는 관리형 런타임 환경 (Kotlin/Java 바이트코드를 실행)
- 네이티브 C/C++ 라이브러리 모음
    - OpenGL, SQLite, WebKit 등 성능 집약적 기능을 위한 라이브러리
- 안드로이드 프레임워크 (Android Framework)
    - 앱 개발을 위한 고수준 서비스 및 API 제공 (ActivityManager, Content Providers 등)
- 애플리케이션 (Apps)
    - 시스템 앱 및 서드파티 앱을 포함한 사용자 기반 앱

## 1. 인텐트(Intent)란 무엇인가요?

- Intent는 수행될 작업에 대한 추상적인 설명이며, Activity, Service, BroadcastReceiver가 통신할 수 있도록 하는 메시징 객체
- 컴포넌트 간에 데이터를 전달하는 데 사용

### Intent 유형

- 명시적 Intent (Explicit Intent)
    - 호출할 컴포넌트의 클래스 이름을 직접 지정
    - 동일 앱 내에서 특정 Activity 시작할 때 사용
- 암시적 Intent (Implicit Intent)
    - 특정 컴포넌트를 지정하지 않고 수행할 일반적인 작업을 선언 (Action, Category, Data 기반)
    - 다른 앱이나 시스템 컴포넌트가 처리할 수 있는 작업에 사용 (웹페이지 열기, 콘텐츠 공유 등)

## 2. PendingIntent의 목적은 무엇인가요?

- PendingIntent는 다른 애플리케이션이나 시스템 컴포넌트가 미리 정의된 Intent를 나중에 애플리케이션을 대신하여 실행할 수 있는 권한을 부여하는 Intent의 래퍼(wrapper)

- 주요 사용 사례: 앱의 수명 주기를 벗어나 트리거되어야 하는 작업
    - 노티피케이션 (Notifications): 사용자가 알림을 탭했을 때 Activity 등을 열도록
    - 알람 (Alarms): AlarmManager를 사용하여 작업을 예약할 때
    - 앱 위젯 (App Widgets): 사용자가 홈 화면 위젯으로 작업을 수행할 때

## 3. Serializable과 Parcelable의 차이점은 무엇인가요?

- 두 인터페이스 모두 안드로이드 컴포넌트 간에 데이터를 전달하는 데 사용되지만, 성능과 구현 방식에 차이가 있음

### Serializable

- 유형: 표준 Java 인터페이스
- 성능: 느림 (Java 리플렉션 사용)
- 가비지 생성: 더 많음 (임시 객체 다수 생성)
- 주요 사용처: 일반적인 Java 사용에 적합

### Parcelable

- 유형: 안드로이드에 특화된 인터페이스
- 성능: 빠름 (안드로이드에 최적화)
- 가비지 생성: 더 적음 (효율적)
- 주요 사용처: 안드로이드 데이터 전달 (특히 IPC)에 선호

👉🏻 일반적으로 Parcelable이 더 나은 성능 때문에 안드로이드 애플리케이션에서 선호됨
👉🏻 Kotlin에서는 @Parcelize 플러그인을 사용하여 구현을 단순화할 수 있음

## 4. Context란 무엇이며 어떤 유형의 Context가 있나요?

- Context는 애플리케이션의 환경 또는 상태를 나타내며, 애플리케이션별 리소스 및 클래스에 대한 접근을 제공
- 앱과 안드로이드 시스템 간의 브릿지 역할을 수행

### Context 유형

- Application Context
    - 전역 애플리케이션 생명주기
    - 앱 전체 리소스에 접근하거나 오래 지속되는 `BroadcastReceiver`를 등록할 때 사용
- Activity Context
    - Activity 생명주기
    - UI 컴포넌트 생성/업데이트, 다른 Activity 실행, 테마/리소스 접근 등에 사용
- Service Context
    - Service 생명주기
    - 백그라운드 작업 실행 (네트워크, 음악 재생 등)
- BroadcastContext
    - `BroadcastReceiver`가 호출될 때 제공
    - 수명이 짧고 특정 브로드캐스트에 응답하는데 사용

📌 주의 사항: Activity 또는 Fragment Context에 대한 오래 지속되는 참조를 유지하면 메모리 누수를 유발할 수 있으므로, 장기 참조가 필요하면 `ApplicationContext`를 사용해야 함

## 5. Application 클래스란 무엇인가요?

- Application 클래스는 전역 애플리케이션 상태와 생명주기를 유지하기 위한 역할을 함
    - 다른 컴포넌트보다 가장 먼저 초기화되는 앱 프로세스의 진입점 역할을 수행
    - 앱의 전체 생명주기 동안 사용 가능한 Context를 제공

- 주요 목적은 전역 상태 유지 및 애플리케이션 전체 초기화
    - `onCreate()`: 앱 프로세스가 생성될 때 단 한 번 호출되며, 데이터베이스, 네트워크 라이브러리, 분석 도구 등 전역 의존성 초기화에 사용됨

## 6. AndroidManifest 파일의 목적은 무엇인가요?

- AndroidManifest.xml 파일은 안드로이드 운영 체제에 애플리케이션에 대한 필수 정보를 정의하는 핵심 구성 파일

- 주요 기능
    - 애플리케이션 컴포넌트 선언: Activity, Service, BroadcastReceiver, ContentProvider 등록
    - 권한 (Permissions): 앱에 필요한 시스템/다른 앱 접근 권한 선언
    - 하드웨어 및 소프트웨어 요구 사항: 앱이 의존하는 기능 명시 (예: 카메라, 특정 화면 크기)
    - 앱 메타 정보: 패키지 이름, 버전, 최소/대상 API 레벨, 테마 등
    - 인텐트 필터 (Intent Filters): 컴포넌트가 응답할 수 있는 Intent 종류 정의

## 7. Activity 생명주기를 설명해주세요

- Activity 생명주기는 Activity가 생성(onCreate())부터 소멸(onDestroy())까지 거치는 다양한 상태를 나타냄

### 콜백 메서드

- `onCreate()`
    - 상태 전환: CREATED -> STARTED 준비
    - Activity 초기화, UI 컴포넌트 설정, 상태 복원 (전체 생명주기 중 단 한 번 호출)
- `onStart()`
    - 상태 전환: STARTED
    - Activity가 사용자에게 보이기 시작 (아직 상호 작용 불가)
- `onResume()`
    - 상태 전환: RESUMED (Running)
    - Activity가 포그라운드에 있으며 사용자와 상호 작용 가능 (UI 업데이트 재개)
- `onPause()`
    - 상태 전환: RESUMED -> STARTED
    - 다른 Activity가 부분적으로 가릴 때 호출 (여전히 보일 수 있지만 포커스 잃음), 애니메이션/센서 업데이트 일시 중지
- `onStop()`
    - 상태 전환: STARTED -> STOPPED
    - Activity가 더 이상 사용자에게 보이지 않을 때 호출, 리소스 해제
- `onRestart()`
    - 상태 전환: STOPPED -> STARTED
    - Activity가 중지되었다가 다시 시작될 때 `onStart()` 전에 호출
- `onDestroy()`
    - 상태 전환: STOPPED -> DESTROYED
    - Activity가 완전히 소멸되고 메모리에서 제거되기 전 최종 정리

## 8. Fragment 생명주기를 설명해주세요

- Fragment는 연결된 부모 Activity의 생명주기와 별도의 자체적인 생명주기를 가지며, Activity 생명주기의 영향을 받음

### 콜백 메서드

- `onAttach()`
	- Fragment가 부모 Activity와 연결될 때 호출
- `onCreate()`
	- Fragment 초기화 (UI 생성 전), 저장된 상태 복원 시도
- `onCreateView()`
	- Fragment의 UI가 처음 그려질 때 호출, 레이아웃의 루트 뷰 반환
- `onViewCreated()`
	- Fragment의 뷰가 생성된 후 호출, UI 컴포넌트 및 로직 설정에 사용
- `onViewStateRestored()`
	- Fragment 뷰의 저장된 상태가 복원된 후 호출
- `onStart()`
	- Fragment가 사용자에게 보이게 됨 (Activity의 onStart()와 유사)
- `onResume()`
	- Fragment가 완전히 활성 상태이며 상호 작용 가능 (Activity의 onResume()과 유사)
- `onPause()`
	- Fragment가 포그라운드에 있지 않지만 여전히 보일 때 호출 (Activity의 onPause()와 유사)
- `onStop()`
	- Fragment가 더 이상 보이지 않을 때 호출 (Activity의 onStop()과 유사)
- `onSaveInstanceState()`
	- Fragment가 소멸되기 전에 UI 관련 상태 데이터를 저장하도록 호출
- `onDestroyView()`
	- Fragment의 뷰 계층이 제거될 때 호출, 뷰 관련 리소스 정리
- `onDestroy()`
	- Fragment 자체가 소멸될 때 호출, 모든 리소스 정리
- `onDetach()`
	- Fragment가 부모 Activity에서 분리될 때 호출 (마지막 콜백)

## 9. Service란 무엇인가요?

- Service는 앱이 사용자 상호 작용과 독립적으로 장기적으로 작업을 수행할 수 있도록 하는 백그라운드 컴포넌트
- Activity와 달리 UI가 없으며 앱이 포그라운드에 있지 않을 때도 실행될 수 있음

### Started Service

- 시작 방식: `startService()`
- 자동 중지: `stopSelf()` 또는 `stopService()`로 명시적 중지 시까지 실행
- UI 알림 없음
- 목적: 백그라운드 음악 재생, 파일 업로드/다운로드

### Bound Service

- 시작 방식: `bindService()`
- 자동 중지: 모든 클라이언트의 연결이 끊어지면 자동 중지
- UI 알림 없음
- 목적: 컴포넌트 간 통신 (데이터 가져오기, 블루투스 연결 관리)

### Foreground Service

- 시작 방식: `startForeground()`
- 자동 중지: `stopSelf()` 또는 `stopService()`로 명시적 중지 시까지 실행
- UI 알림 필수 (지속적인 알림 표시)
- 사용자 인지 작업 (미디어 재생, 위치 추적)

## 10. BroadcastReceiver란 무엇인가요?

- BroadcastReceiver는 앱이 안드로이드 운영 체제 전체의 브로드캐스트 메시지나 앱 특정 브로드캐스트를 수신하고 응답할 수 있도록 하는 컴포넌트
- 시스템 또는 다른 앱에 의해 트리거될 수 있음

### 시스템 브로드캐스트

- 등록 방식: 정적(AndroidManifest.xml) 또는 동적(코드)
- 앱이 실행 중이 아니어도트리거될 수 있음
- ex. 배터리 잔량 변경, 네트워크 연결 업데이트

### 커스텀 브로드캐스트

- 등록 방식: 정적(AndroidManifest.xml) 또는 동적(코드)
- 앱 내부 또는 앱 간에 특정 정보 전달
- ex. 앱 내부의 특정 이벤트 알림

## 11. ContextProvider의 목적은 무엇이며, 애플리케이션 간의 안전한 데이터 공유를 어떻게 용이하게 하나요?

- ContentProvider는 구조화된 데이터에 대한 접근을 관리하고 애플리케이션 간 데이터 공유를 위한 표준화된 인터페이스를 제공하는 컴포넌트
- 목적
    - 데이터 접근 로직을 캡슐화하여 앱 간 데이터 공유를 더 쉽고 안전하게 만듦
    - 기본 데이터 소스 (SQLite, 파일 시스템 등)를 추상화
- 안전한 데이터 공유
    - URI (Uniform Resource Identifier) 기반으로 데이터 접근 주소를 사용 (Authority, Path, ID)
    - ContentResolver 클래스를 통해 다른 앱이 데이터에 접근
    - 권한 (Permissions)을 선언하여 액세스 권한을 세부적으로 제어할 수 있음 (읽기/쓰기 권한 등)