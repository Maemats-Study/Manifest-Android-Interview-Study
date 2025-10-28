# View 생명주기

안드로이드에서 View 생명주기(Lifecycle)는 View(예: TextView, Button)가 생성되고, Activity나 Fragment에 연결되며, 화면에 표시되고, 최종적으로 소멸되거나 분리되는 과정을 나타냄

이 생명주기를 이해하는 것은 View 초기화, 렌더링, 소멸 관리, 사용자 입력 처리, 커스텀 뷰 구현 및 리소스 정리에 중요

**주요 단계**

1. **View 생성 (onAttachedToWindow)**
    - View가 인스턴스화되거나 XML 레이아웃에서 인플레이션되는 단계
    - 초기 설정(리스너 설정, 데이터 바인딩 등)이 수행됨
    - `onAttachedToWindow()`는 View가 부모 뷰에 추가되어 렌더링 준비가 되었을 때 호출됨
2. **Layout 단계 (onMeasure, onLayout)**
    - View의 크기와 위치를 결정
    - `onMeasure()`: View의 너비와 높이를 계산
    - `onLayout()`: 부모 내에서 View의 최종 위치를 지정
3. **Drawing 단계 (onDraw)**
    - 크기와 위치가 결정된 후, `onDraw()` 메서드가 View의 콘텐츠(텍스트, 이미지 등)를 Canvas에 그림
    - 커스텀 View는 이 메서드를 재정의하여 커스텀 드로잉 로직을 구현할 수 있음
4. **이벤트 처리 (onTouchEvent, onClick)**
    - View가 터치 이벤트나 클릭과 같은 사용자 상호 작용을 처리하는 단계
5. **View 분리 (onDetachedFromWindow)**
    - View가 화면과 부모 ViewGroup에서 제거될 때(예: Activity 소멸 시) `onDetachedFromWindow()`가 호출됨
    - 리소스 정리나 리스너 분리에 이상적인 시점
6. **View 소멸**
    - View가 더 이상 사용되지 않으면 가비지 컬렉션됨
    - 메모리 누수를 방지하기 위해 모든 리소스가 적절히 해제되었는지 확인해야 함