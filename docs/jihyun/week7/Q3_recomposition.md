# Recomposition이란?

## 1. Recomposition (리컴포지션)의 정의와 발생 조건

**Recomposition**은 Jetpack Compose가 이미 렌더링된 UI 레이아웃을 업데이트하기 위해 세 가지 주요 단계(Composition, Layout, Drawing)를 통해 상태 변경이 발생할 때마다 UI를 다시 그리는 메커니즘

- Recomposition의 역할: UI가 상태 변화를 자동으로 감지하고 동기화하여 반응형 특성을 담당하는 핵심 기능
- Recomposition 발생 시: Composition 단계부터 새롭게 시작되며, 여기서 컴포저블 노드는 UI 변경 사항을 Compose Runtime에 알리고 업데이트된 UI가 최신 상태를 반영하도록 보장
    - Frame 1: Composition → Layout → Drawing
    - (상태 변경)
    - Frame 2: Composition (Reinvoke the reader) → Layout → Drawing

### Recomposition이 발생하는 두 가지 주요 조건

1. 매개변수 입력 변경(Input Changes) 시
    - 컴포저블 함수에 입력 매개변수가 변경되면 `recomposition`을 트리거
    - Compose 런타임은 `equals()` 함수를 사용하여 새 매개변수 값을 이전 매개변수 값과 비교하며, 변경 사항을 인지하면 `recomposition`을 트리거하여 필요한 부분만 UI를 업데이트
2. 상태 변경 관찰(Observing State Changes) 시
    - Jetpack Compose는 일반적으로 `remember` 함수와 State API를 사용하여 상태 변경을 모니터링
    - 이러한 접근 방식은 상태 객체를 메모리에 보존하고, `recomposition`이 발생했을 때도 저장된 값을 복원하여 UI에 최신 상태를 일관되게 반영하도록 보장

## 2. Compose Phases (렌더링 단계)

Jetpack Compose는 UI를 화면에 그릴 때 **Composition, Layout, Drawing**의 세 가지 주요 단계로 나누어진 렌더링 파이프라인을 순차적으로 수행

| **단계** | **주요 역할** | **상태 읽기 (State reads)** | **트리거 조건** |
| --- | --- | --- | --- |
| **Composition (컴포지션)** | `@Composable` 함수를 실행하여 UI 트리(Slot Table)를 구축하고, `recomposition`을 위한 변경 사항을 추적합니다. | `@Composable` 함수 및 `@Composable` 람다 블록 | **콘텐츠**가 변경된 경우 |
| **Layout (레이아웃)** | 각 UI 컴포넌트의 **크기**와 **위치**를 결정합니다 (자식 요소 측정 및 배치). | `Layout()` 측정 람다, `Modifier.offset`, 등 | **크기** 또는 **위치**가 변경된 경우 |
| **Drawing (드로잉)** | Layout 단계를 마친 UI 컴포넌트가 **시각적으로 렌더링**됩니다 (Skia 엔진 사용). | `Canvas()`, `Modifier.drawBehind`, `Modifier.drawWithContent`, 등 | **크기** 또는 **위치**가 변경된 경우 |

## 3. Recomposition과 앱 성능

`Recomposition`은 반응형 UI의 핵심이지만, 과도하거나 불필요한 `recomposition`은 앱 성능을 저하시킬 수 있음

`Recomposition`을 최적화하고 앱 성능을 향상시키는 것이 중요

- 성능 최적화 도구
    - Layout Inspector: 실제 앱에서 발생하는 `recomposition` 횟수를 추적하여 불필요한 `recomposition`을 식별하는 데 효과적
    - Composition Tracing: 잠재적인 성능 문제를 진단하는 데 유용한 도구로, 오버헤드가 낮은 측정을 제공