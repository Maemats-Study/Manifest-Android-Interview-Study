# Compose phase란

Jetpack Compose는 UI를 화면에 그릴 때 Composition, Layout, Drawing의 세 가지 주요 단계로 나누어진 렌더링 파이프라인을 따름

UI를 효율적으로 구축하고, 정렬하거나, 올바르게 렌더링하기 위해 해당 단계들을 순차적으로 수행

### 1. Composition (컴포지션)

Composition 단계는 UI 렌더링의 첫 번째 단계로, `@Composable` 함수를 실행하고 **UI 트리**를 구축

- 주요 역할
    - `@Composable` 함수를 실행하여 UI에 대한 설명을 생성
    - Compose는 초기 UI 구조를 구축하고 Slot Table이라는 데이터 구조에 컴포저블 간의 관계를 기록
    - Composition의 주요 작업
        - `@Composable` 함수 실행
        - UI 트리 생성 및 업데이트
        - `recomposition`을 위한 변경 사항 추적
- 상태 변경이 발생하면 Composition 단계는 영향을 받는 UI에 대해 다시 계산하고, 필요한 경우 recomposition을 트리거
- `@Composable` 함수 및 `@Composable` 람다 블록 내에서 상태를 읽음

### 2. Layout (레이아웃)

Layout 단계는 Composition 단계 바로 직후에 수행됨

- 주요 역할
    - 제공된 제약 조건에 따라 각 UI 컴포넌트의 크기와 위치를 결정
    - 각 컴포저블은 자식 요소를 측정(크기 및 위치 등)하고, 크기를 결정하며, 부모에 대한 상대적 위치를 배치
    - Layout의 주요 작업
        - UI 컴포넌트 측정
        - 너비, 높이 및 위치 정의
        - 부모 컨테이너 내 자식 배치
- `Layout()` 측정 람다, `Modifier.offset`, 등의 로직에서 상태를 읽음
- 콘텐츠가 변경된 후 크기 또는 위치가 변경된 경우에 Layout 단계가 트리거됨

### 3. Drawing (드로잉)

Drawing 단계는 앞선 Composition과 Layout 단계를 마친 UI 컴포넌트가 화면에 시각적으로 렌더링되는 절차

- 주요 역할
    - Compose는 안드로이드에서 해당 UI들을 렌더링하기 위해 Skia 그래픽 엔진을 사용하여 부드럽고 하드웨어 가속 기반의 렌더링을 제공
    - Drawing의 주요 작업
        - 시각적 요소 렌더링
        - 화면에 UI 컴포넌트 그리기
        - 커스텀 드로잉 작업 적용
- 커스텀 드로잉 로직은 Compose의 `Canvas` API를 사용하여 구현할 수 있음
- `Canvas()`, `Modifier.drawBehind`, `Modifier.drawWithContent`, 등의 로직에서 상태를 읽음
- 크기 또는 위치가 변경된 후에 Drawing 단계가 트리거됨