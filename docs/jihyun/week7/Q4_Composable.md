# Composable 함수는 내부적으로 어떻게 작동하는가?

## 1. 선언적 UI 프레임워크의 원칙

Jetpack Compose는 개발자가 앱 상태(State)를 기반으로 UI가 무엇을 보여야 하는지를 표현할 수 있게 하는 선언적 UI 프레임워크

이는 뷰를 수동으로 업데이트하는 전통적인 명령형(Imperative) XML 개발 방식과 대비됨

### 선언적 UI의 주요 원칙

1. 함수로 UI 정의: `@Composable` 어노테이션이 달린 함수를 사용하여 UI 컴포넌트를 정의
2. 상태 관리/반응형 UI: Compose Runtime이 제공하는 `remember` 및 `State`와 같은 API를 사용하여 컴포저블의 상태와 생명주기를 관리. 상태 변경 시 UI가 자동으로 업데이트됨.
3. 직접 데이터 바인딩: 개발자는 모델 데이터를 컴포저블 함수의 매개변수를 통해 UI에 직접 바인딩하여 코드를 더 깔끔하게 만듦.
4. 컴포넌트 멱등성(Idempotence): `@Composable` 함수는 동일한 입력 매개변수에 대해 호출 횟수에 관계없이 항상 동일한 UI 출력을 생성하여 예측 가능하고 안정적인 UI 렌더링을 보장

## 2. Composable 함수의 내부 동작 및 변환

일반 Kotlin 함수에 `@Composable` 어노테이션을 붙이면, Compose Compiler Plugin에 의해 컴파일 타임에 Jetpack Compose의 내부 메커니즘에 맞게 상태 기반 UI 코드로 변환

### Compiler Transformation 및 Composer

- 컴파일러 역할: Compose 컴파일러는 컴포저블 함수에 `Composer`라는 암시적 매개변수를 추가하고 상태 추적 및 `recomposition`을 처리하는 기타 상태 로직을 주입
- Composer의 역할: `Composer`는 Compose 런타임의 저수준 API로, 중앙 상태 관리 역할을 함
    - 상태 관리: 상태를 추적하고 입력값이 변경될 때 컴포저블 함수가 올바르게 `recompose`되도록 보장
    - UI 계층 구조 구축: UI의 트리 구조를 유지하고 상태 변경 시 노드를 효율적으로 업데이트
    - Recomposition 제어: 특정 컴포저블에 대한 `recomposition` 시작, 건너뛰기, 종료를 포함하여 `recomposition`의 생명주기를 조정

### Composable 함수의 요구 사항: Side Effect 피하기

컴포저블 함수는 예측 가능하고 재사용 가능하도록 다음을 따라야 함

- 사이드 이펙트 피하기 (Avoid Side Effects)
    - 컴포저블 함수는 외부 상태나 내부 변수를 직접 수정하는 등 사이드 이펙트로부터 자유로워야 함
    - 멀티 스레드 환경에서도 안전하며, 개발자가 의도하지 않은 동작을 방지
- 순서에 의존하지 않는 실행
    - Compose는 UI 렌더링 프로세스를 최적화하기 위해 컴포저블 함수를 어떤 순서로든 실행할 수 있음
    - 따라서 컴포저블 함수는 이전에 실행된 다른 컴포저블의 결과에 의존해서는 안 됨

## 3. Recomposition (리컴포지션)과 성능

Recomposition은 상태 변경이 발생할 때마다 UI를 다시 그리는 메커니즘

이는 Compose Phases의 첫 번째 단계인 Composition부터 새로 시작됨

### Recomposition이 발생하는 조건

1. 매개변수 입력 변경(Input Changes) 시: 컴포저블 함수에 전달된 입력 매개변수가 변경되면 `equals()` 비교를 통해 변경 사항이 감지되어 `recomposition`이 트리거됨
2. 상태 변경 관찰(Observing State Changes) 시: `remember` 함수와 State API를 사용하여 모니터링되던 상태 객체가 변경될 때 `recomposition`이 트리거됨

### 앱 성능 및 최적화

`recomposition`은 반응형 UI의 핵심이지만, 과도하거나 불필요한 `recomposition`은 앱 성능을 저하시킬 수 있음

- 성능 최적화 도구
    - Layout Inspector를 사용하면 실제 앱에서 발생하는 `recomposition` 횟수를 추적하여 불필요한 `recomposition`을 식별할 수 있음
    - Composition Tracing도 성능 문제를 진단하는 데 유용한 도구