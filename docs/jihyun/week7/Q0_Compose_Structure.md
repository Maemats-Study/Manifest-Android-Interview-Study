# Jetpack Compose 동작 구조

- Jetpack Compose: 선언적 접근 방식 사용
- 네이티브 안드로이드 애플리케이션을 구축하기 위한 가장 최신의 UI 툴킷
- 3가지 계층으로 구성됨
    - Compose Compiler
    - Compose Runtime
    - Compose UI
- 각 계층은 UI 코드를 상호 작용 가능한 애플리케이션으로 변환하는 데 중요한 역할 함
    - Source → Compiler → Runtime ↔ UI

## Compose Compiler

Kotlin으로 작성된 선언적 UI 코드를 Jetpack Compose가 실행할 수 있는 최적화된 코드로 변환하는 역할

- 컴파일 타임에 `@Composable` 함수를 처리하고, 필요한 UI 업데이트 및 recomposition 로직을 생성
- 작동 방식
    - Kotlin 컴파일러로 구현되어 효율적인 코드 생성을 보장
    - 상태 관리, 코드 최적화 및 람다 리프팅(lambda lifting) 같은 기능을 지원
- 기존 도구와의 차이점
    - KAPT나 KSP와 같은 기존 어노테이션 처리 도구와 달리, Compose Compiler 플러그인은 FIR(Frontend Intermediate Representation) 에서 직접 작동
    - 이를 통해 컴파일러는 컴파일 타임에 정적 코드에 대해 자세히 접근할 수 있음
    - 개발자가 작성한 Kotlin 소스 코드를 동적으로 변환하고 최적화된 Java 바이트코드를 생성할 수 있음
- 내부 메커니즘
    - `@Composable`과 같은 Compose 라이브러리의 어노테이션은 코드 생성, `recomposition` 관리 및 성능 최적화와 같은 작업을 조율하는 Compose Compiler의 내부 메커니즘을 통해 작동

## Compose Runtime

Compose Runtime은 recomposition 및 상태 관리를 지원하는 데 필요한 핵심 기능을 제공

- 주요 역할
    - 변경 가능한 상태(mutable states)를 처리
    - 스냅샷(snapshots)을 관리
    - 애플리케이션 상태가 변경될 때마다 UI 업데이트를 트리거
    - Compose의 반응형 UI 시스템을 구동
- 자료 구조
    - 갭 버퍼(gap buffer) 자료 구조에서 영감을 받은 슬롯 테이블(slot table)을 사용
    - 컴포지션 상태를 메모이제이션(memoizing)하는 방식으로 작동
- 내부 동작
    - Side-effects 관리
    - `remember`를 사용한 상태 보존
    - 상태 변경 시 recomposition 트리거
    - `CompositionLocal`을 사용한 컨텍스트별 데이터 저장
    - UI 계층 구조를 효율적으로 생성하기 위한 Compose 레이아웃 노드 구축
- 전통적인 UI 시스템 위에서 동작하지만, 그 위에서 동작하는 Compose의 메커니즘이 기존 안드로이드 UI 시스템과 완전히 분리된 개념

## Compose UI

Compose UI 계층은 애플리케이션 구축을 위한 고수준 컴포넌트 및 UI 위젯을 제공

- 주요 역할
    - UI 렌더링에 즉시 사용할 수 있는 위젯 및 UI 컴포넌트를 제공
    - 텍스트, 버튼, 레이아웃 컨테이너와 같은 기본 요소와 커스텀 UI 컴포넌트 구축을 위한 상위 호환의 API를 포함
- 레이아웃: Compose Runtime에 의해 처리되는 Compose 레이아웃 트리 구축을 단순화하도록 설계된 광범위한 컴포넌트를 제공
- 크로스 플랫폼 지원
    - Kotlin Multiplatform 지원을 통해 JetBrains는 안드로이드, iOS, 데스크톱 및 WebAssembly를 포함한 여러 플랫폼에서 동일한 Compose UI 라이브러리를 사용하여 일관된 UI를 만들 수 있도록 개발 생태계를 만들고 있음
    - 이러한 크로스 플랫폼 호환성은 Flutter나 React Native와 유사하게 멀티 플랫폼 간의 개발을 간소화하고 일관성 있는 사용자 경험을 보장
- 계층화된 아키텍처는 전부 모듈식으로 구현되어 서로 독립적인 관계를 이루고 있으면서도, 효율적이며 유지 관리 가능하도록 함