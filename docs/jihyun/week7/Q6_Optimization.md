# 안정성 개선을 통해 Compose 성능 최적화

## 1. 안정성(Stability)의 개념과 성능 영향

Jetpack Compose의 안정성(Stability)은 클래스나 타입이 동일한 매개변수 입력값에 대해 일관된 결과를 생성하도록 보장하는 속성

Compose 컴파일러는 `@Composable` 함수의 매개변수를 `stable` 또는 `unstable`로 분류하며, 이는 런타임이 `recomposition`을 효율적으로 관리하고 앱 성능을 향상시키는 데 매우 중요

### Smart Recomposition 작동 방식

Smart Recomposition은 컴파일러가 제공하는 안정성 데이터에 의존하여 불필요한 업데이트를 선택적으로 건너뛰어 UI 성능과 응답성을 최적화하는 메커니즘

1. Stable 매개변수, 입력값 변화 없음: 매개변수가 `stable`하고 `equals()` 비교 결과 값이 변경되지 않은 경우, Compose는 해당 컴포넌트의 `recomposition`을 건너뜀
2. Stable 매개변수, 입력값 바뀜: 매개변수가 `stable`하고 값이 변경된 경우 (`equals()`가 `false` 반환), 런타임은 `recomposition`을 트리거
3. Unstable 매개변수: 매개변수가 `unstable`인 경우, Compose는 입력값 변경 여부에 관계없이 `recomposition`을 트리거

불필요한 `recomposition`을 피하는 것의 중요성: 건너뛰지 못한 중복된 `recomposition`은 함수를 다시 실행하고 UI 컴포넌트를 다시 그리는 불필요한 오버헤드를 발생시켜 성능을 저하시킬 수 있음

## 2. Stable vs. Unstable 타입 분류 기준

Compose 컴파일러는 다음 기준에 따라 매개변수 타입의 안정성을 분류:

| **분류** | **기준** | **예시** |
| --- | --- | --- |
| **Stable** | 읽기 전용 원시 타입: `val`로 정의된 `String`, `Int` 등. | `data class User(val id: Int, val name: String)` |
|  | `@Stable` 또는 `@Immutable` 어노테이션이 명시적으로 붙은 클래스. | `State<T>`, `@Immutable UserList` |
|  | 함수 타입: `(Int) -> String` 등 예측 가능한 동작을 하는 타입. | Non-Capturing Lambdas |
| **Unstable** | 가변적인 속성: 하나 이상의 가변적인(`var`) 또는 불안정하다고 간주되는 `public` 속성을 포함하는 클래스. | `data class MutableUser(val id: Int, var name: String)` |
|  | 인터페이스 및 추상 클래스: `List`, `Map`, `Any` 등 컴파일 시점에 구현체의 종류를 보장할 수 없는 타입. | `List<User>`, `MutableList<User>` |

### 3. 안정성 어노테이션: `@Immutable` vs. `@Stable`

개발자는 클래스에 안정성 어노테이션을 명시하여 Compose 컴파일러에게 해당 클래스의 안정성을 알려 `recomposition`을 최적화할 수 있음

### `@Immutable`

- 불변성 요구 사항: 클래스의 모든 public 속성이 완전히 불변(fully immutable)이어야 합니다. 객체가 생성된 후 상태가 변경될 수 없음을 보장
- Recomposition 동작: 동일한 매개변수를 가진 `Composable`에 대해 `recomposition`을 완전히 건너뛸 수 있음
- 유즈 케이스: I/O 작업(네트워크 응답, 데이터베이스 엔티티)의 응답 모델과 같이 변경이 완전히 불가능해야 하는 데이터 모델 클래스에 적합

### `@Stable`

- 불변성 요구 사항: 일부 제어된 가변성을 허용하지만, 객체가 동일한 입력에 대해 일관되고 예측 가능한 결과를 생성해야 함 (멱등성의 성질 준수)
- Recomposition 동작: 가변 속성이 변경되면 언제든지 `recomposition`이 트리거될 수 있지만, Smart Recomposition을 통해 효율적으로 관리됨
- 유즈 케이스: `State` 인터페이스와 같이 가변적이지만 런타임에 의해 변경이 잘 관리되는 클래스에 적합

## 4. 성능 최적화 전략 및 도구

Jetpack Compose는 `Smart Recomposition`을 통해 성능 최적화를 지원하지만, 개발자는 가능한 한 `stable` 클래스를 사용하여 불필요한 `recomposition`을 최소화해야 함

- 불변 컬렉션 사용: `List`나 `Map`과 같은 불안정한 컬렉션 대신, 안정성이 보장된 `kotlinx.collections.immutable` 또는 Google의 `Guava`와 같은 라이브러리의 불변 컬렉션을 사용
- Wrapper 클래스: 불안정한 서드파티 클래스나 타입을 `@Immutable` 어노테이션과 함께 래퍼 클래스로 만들어 안정성을 확보하고 `recomposition`을 최적화할 수 있음
- Stability Configuration File: Compose Compiler 1.5.0부터 공개된 안정성 구성 파일을 사용하여 서드파티 라이브러리 클래스 등 특정 패키지의 클래스를 전역적으로 `stable`로 지정할 수 있음
- Composition Tracing 및 Layout Inspector:
    - Layout Inspector는 실제 앱에서 발생하는 `recomposition` 횟수를 추적하여 과도하거나 불필요한 `recomposition`을 식별하는 데 유용
    - Composition Tracing은 시스템 추적을 통해 오버헤드가 낮은 측정을 제공하여 잠재적인 성능 문제를 진단하는 데 도움을 줌