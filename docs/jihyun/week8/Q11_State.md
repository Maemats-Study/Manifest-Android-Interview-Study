# 상태(state)란 무엇이며 이를 관리하는 데 사용되는 API는 무엇인가요?

Jetpack Compose에서 State는 앱 시나리오에서 흔히 변경될 수 있는 값이자, UI에서 동적으로 반영되는 데이터를 나타냄

흔히 사용되는 상태:

- 네트워크 오류에 대한 Snackbar 메시지 노출
- 사용자 입력 또는 상호작용으로 발생하는 애니메이션을 표시하기 위한 용도

상태는 UI 업데이트를 직접 트리거

상태가 변경되면 변경된 상태를 UI에 반영하기 위해 다시 렌더링

## State와 Composition

UI 업데이트: 컴포저블이 변경된 매개변수를 통해 호출될 때만 발생

이러한 동작은 composition 생명주기와 밀접하게 관련됨

- 초기 Composition: 컴포저블을 실행하여 UI 트리가 처음 생성되고 렌더링되는 프로세스
- Recomposition: 상태 변경 시 트리거되며, recomposition은 관련 컴포저블을 업데이트하여 새로운 상태를 반영

Compose Runtime: 상태 변경 사항을 자동으로 추적

→ 안드로이드 View 시스템에서 UI를 호출하기 위해 사용하는 View.invalidate() 메서드와 유사한 동작을 개발자 대신하여 UI를 업데이트

Recomposition: 업데이트된 상태를 반영해야 하는 컴포저블 함수에 대해서만 트리거

## Compose에서 상태 관리하기

Jetpack Compose는 상태를 효과적으로 관리하기 위해 여러 API들을 제공

- remember
    - 초기 컴포지션이 발생했을 때 메모리에 객체를 저장하고 recomposition이 발생하면 기존 메모리에 저장된 값을 꺼내옴
    - API의 이름 그대로 상태 값을 메모리에 ‘기억한다’고 이해하면 됨
- rememberSaveable
    - recomposition 뿐만 아니라, 화면 회전과 같은 구성 변경 시에도 상태를 유지
    - Bundle에 저장할 수 있는 유형 또는 그 외 유형에 대해서는 커스텀 saver 객체와 함께 작동할 수 있음
- mutableStateOf
    - 가변적인 상태를 의미
    - 상태 값이 변경될 때 recomposition을 트리거하는 관찰 가능한 상태 객체를 생성

### 왜 remember와 mutableStateOf가 함께 사용되나요?

- remember API는 객체를 메모리에 저장
- mutableStateOf는 상태값이 변할 때 recomposition을 트리거하기 위한 관찰 가능한 객체를 생성하기 위한 API

만약 mutableStateOf만 사용하면 상태값이 바뀔 때마다 recomposition을 트리거 → 문제는 해당 상태값 자체가 recomposition이 발생할 때마다 메모리에 저장되어있지 않기 때문에 초기화되고, 기대했던 동작과는 다른 결과가 나오게 됨

따라서 상태 또한 메모리에 저장을 해야 함