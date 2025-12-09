# Jetpack Compose의 안정성(stability)이란 무엇이며, 성능과 어떤 관련이 있는가?

## 1. 선언적 UI와 Composable 함수

Jetpack Compose는 개발자가 앱 상태를 기반으로 UI가 무엇을 표시해야 하는지를 표현하는 선언적 UI 프레임워크

이는 개발자가 뷰를 수동으로 조작하는 전통적인 명령형(Imperative) XML 개발 방식과 대조됨

### `@Composable` 함수의 특징

- UI 정의: `@Composable` 어노테이션이 달린 함수를 사용하여 UI 컴포넌트를 정의
- 컴파일러 변환: Compose 컴파일러 플러그인은 컴파일 타임에 `@Composable` 함수를 `Composer`라는 암시적 매개변수를 포함하는 함수로 변환하고, 상태 추적 및 `recomposition` 로직을 주입
- Composer의 역할: `Composer`는 Compose Runtime의 저수준 API로, 중앙 상태 관리 역할을 하며, 상태를 추적하고 UI 계층 구조를 유지하며 `recomposition`의 생명주기를 제어
- 멱등성(Idempotence): `@Composable` 함수는 동일한 입력에 대해 항상 동일한 UI 결과를 생성하는 멱등성을 가지며, 예측 가능한 안정적인 렌더링을 보장
- 사이드 이펙트 피하기: 컴포저블 함수는 외부 상태를 직접 수정하거나 `var`와 같은 가변 로컬 변수를 사용하는 사이드 이펙트로부터 자유로워야 함. 멀티스레드 환경에서도 안전하며, 상태 관리는 `remember`와 `State`와 같은 전용 API를 통해 이루어져야 함.
- 순서에 의존하지 않는 실행: Compose는 UI 렌더링 프로세스 최적화를 위해 컴포저블 함수를 어떤 순서로든 실행할 수 있음

## 2. Recomposition (리컴포지션)과 렌더링 파이프라인

Recomposition은 상태 변경이 발생할 때마다 UI를 다시 그리는 메커니즘으로, Compose의 반응형(Reactive) 특성을 담당하는 핵심 기능

### Recomposition 발생 조건

1. 매개변수 입력 변경(Input Changes): 컴포저블 함수에 전달된 매개변수가 변경되었을 때, Compose 런타임이 `equals()` 함수를 사용하여 이전 값과 비교하고 변경 사항이 감지되면 `recomposition`을 트리거
2. 상태 변경 관찰(Observing State Changes): `remember` 및 State API를 통해 모니터링되던 상태 객체가 변경될 때 `recomposition`을 트리거

### Compose Phases (렌더링 단계)

`Recomposition`이 발생하면 Compose는 UI 렌더링을 위해 다음 세 단계를 순차적으로 다시 시작

1. Composition (컴포지션): `@Composable` 함수를 실행하고 UI 트리(Slot Table)를 구축하여 UI에 대한 설명을 생성
    - 상태 읽기: `@Composable` 함수 및 람다 블록
2. Layout (레이아웃): 각 UI 컴포넌트의 크기와 위치를 결정하고, 자식 요소를 측정하며 배치
    - 상태 읽기: `Layout()` 측정 람다, `Modifier.offset` 등
3. Drawing (드로잉): 최종적으로 UI 컴포넌트가 시각적으로 렌더링됨 (Skia 그래픽 엔진 사용)
    - 상태 읽기: `Canvas()`, `Modifier.drawBehind` 등

## 3. 안정성(Stability)과 성능 최적화

- 안정성(Stability)은 클래스나 타입이 동일한 매개변수 입력에 대해 일관된 결과를 생성하도록 보장하는 속성
- Compose 컴파일러는 `@Composable` 함수의 매개변수를 `stable` 또는 `unstable`로 분류하여 `recomposition`을 효율적으로 관리하고 앱 성능을 향상시키는 데 사용

### Stable/Unstable 분류 기준

- Stable로 간주되는 경우
    - 읽기 전용(val) 원시 타입: `String`, `Int` 등
    - 읽기 전용(val) public 속성을 가진 데이터 클래스 (Immutable)
    - `@Stable` 또는 `@Immutable` 어노테이션이 명시적으로 붙은 클래스
    - 함수 타입 (`(Int) -> String` 등)
- Unstable로 간주되는 경우:
    - 가변적인(var) 속성을 하나라도 포함하는 클래스
    - `List`, `Map`, `Any` 등 컴파일 시점에 구현체의 종류를 보장할 수 없는 인터페이스나 추상 클래스

### 안정성 어노테이션의 종류

| **어노테이션** | **불변성 요구 사항** | **유즈 케이스** | **특징** |
| --- | --- | --- | --- |
| `@Immutable` | 모든 public 속성이 완전히 불변(fully immutable)이어야 함. | 변경이 완전히 불가능해야 하는 데이터 모델 클래스 (ex. I/O 작업 결과). | 클래스 내부 상태 변경이 없음을 보장하여 컴파일러가 해당 클래스를 매개변수로 받는 컴포저블에 대해 `recomposition`을 안전하게 건너뛸 수 있음. |
| `@Stable` | 일부 가변 속성을 허용하지만, 제어되고 예측 가능한 방식으로 작동하여 일관된 결과를 생성해야 함 (멱등성의 성질). | `State` 인터페이스와 같이 가변적이지만 런타임에서 변경이 잘 관리되는 클래스. | 컴포저블의 매개변수가 `@Stable`이면 입력 값 변경 여부를 확인하여 `Smart Recomposition`을 통해 불필요한 `recomposition`을 건너뛰거나 트리거할 수 있음. |

### 성능 최적화

- `Smart Recomposition`은 `stable` 매개변수의 변경 사항만 비교하여 불필요한 `recomposition`을 건너뛰고 UI 업데이트를 선택적으로 수행하여 성능을 최적화하는 메커니즘
- 성능 추적 도구: Android Studio의 Layout Inspector를 사용하면 실제 앱에서 발생하는 `recomposition` 횟수를 추적하여 불필요한 `recomposition`을 식별할 수 있음