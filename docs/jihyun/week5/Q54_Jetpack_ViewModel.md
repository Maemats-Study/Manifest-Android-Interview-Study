# Jetpack ViewModel이란

- 생명주기를 인식하는 방식으로 UI 관련 데이터를 저장하고 관리하도록 설계된 안드로이드 아키텍처 컴포넌트의 핵심 구성 요소
- 회면 회전과 같은 구성 변경 시에도 데이터가 유지되도록 보장
- UI 로직과 비즈니스 로직은 분리하여 개발자가 견고하고 유지 관리 가능한 앱을 만드는데 도움을 줌

## ViewModel의 특징

1. 생명주기 인식(Lifecycle Awareness)
    - ViewModel은 Activity 또는 Fragment의 생명주기에 범위가 지정됨
    - 사용자가 화면에서 벗어나는 등 연관된 UI 컴포넌트가 더 이상 사용되지 않을 때 자동으로 소멸됨
2. 구성 변경 간 지속성(Persistence Across Configuration Changes)
    - 구성 변경 중에 소멸되고 다시 생성되는 Activity 또는 Fragment와 달리 ViewModel은 상태를 유지하여 데이터 손실을 방지 & 데이터 반복적인 재로드를 피함
3. 관심사 분리(Sepration of Concerns)
    - ViewModel은 UI 관련 로직과 비즈니스 로직을 분리하여 더 깔끔하고 유지 관리하기 쉬운 코드를 설계할 수 있음
    - UI 레이어는 ViewModel에서 업데이트를 관찰 → 반응형 프로그래밍 원칙을 구현하기가 더 쉬움

## ViewModel 생성 및 사용

- ViewModel 인스턴스는 생명주기를 관리하기 위한 메커니즘 역할을 하는 ViewModelStoreOwner에 스코프가 지정딤
- ViewModelStoreOwner는 Activity, Fragment, Navigation 그래프, Navigation 그래프 내 대상 또는 개발자가 정의한 커스텀 ViewModelStoreOwner가 될 수도 있음

## ViewModel의 생명주기

- ViewModel의 생명주기는 ViewModelStoreOwner에 연결됨
- ViewModel은 ViewModelStoreOwner의 범위 내에서 존재
- 화면 회전과 같은 구성 변경 시에도 데이터와 상태가 유지되도록 보장
- 생명주기에 기반한 설계 → ViewModel을 UI 관련 데이터를 효율적으로 관리하고, 변경 사항 전반에 걸쳐 데이터를 보존하는 데 중요한 역할
- ex. Activity의 경우, ViewModel은 Activity가 완전히 파괴되고 메모리에서 제거될 때까지 유지

## 구성 변경에도 ViewModel이 유지되는 이유

- Activity와 Fragment과 같이 ViewModel 생명주기를 관리하도록 설계된 안드로이드 컴포넌트는 자체적인 ViewModelStore을 가짐
- 이는 ViewModel 인스턴스가 구성 변경 시에도 유지되도록 보장

## Jetpack ViewModel과 Microsoft에서 제시한 MVVM 아키텍처 ViewModel의 차이점

### Jetpack ViewModel

- Jetpack ViewModel은 비즈니스 로직 또는 UI 상태 홀더 역할을 하도록 설계된 생명주기를 인식하는 컴포넌트
- 관련 비즈니스 로직을 캡슐화하면서 상태를 관리하고 UI에 제공
- 장점: 상태를 보유 & 구성 변경 시에도 데이터 유지
    - 불필요하게 데이터를 다시 요청할 필요 X
    - 효율성, 응답성 향상
- 안드로이드 애플리케이션 내에서 생명주기를 인식하고, UI 상태 관리에 최적화

### MVVM ViewModel

- ViewModel이 View와 Model 간의 다리 역할을 수행
- MVVM ViewModel은 View가 데이터 바인딩할 수 있는 속성과 명령을 구현 → UI에 필요한 기능을 제공
- 변경 사항에 대한 알림 이벤트를 통해 View에 상태 변경을 알림
- View와 Model 간의 상호 작용을 조정하고 UI에 필요한 비즈니스 로직을 추상화하거나 호출을 중개하는 역할
- 상태 관리 및 안드로이드 컴포넌트 생명주기를 인지하는 것에 중점을 둔 Jetpack ViewModel과 달리, MVVM VeiwModel은 View가 상태 변경에 반응할 수 있도록 하는 바인딩 메커니즘을 강조 → 더 선언적이고 모듈화된 설계