# View와 ViewGroup의 차이점

**View**와 **ViewGroup**은 안드로이드 UI를 구성하는 기본 요소이지만 역할이 다름

- **View**
    - 화면에 그려지는 **가장 작은 단위**의 UI 컴포넌트(예: Button, TextView, ImageView)
    - 사용자와의 **상호작용**(터치, 키 이벤트 등)을 처리
    - UI 계층 구조에서 **가장 끝 노드**(leaf node)에 해당하며, 다른 뷰를 포함할 수 없음
    - 자신의 크기와 위치를 가짐
- **ViewGroup**
    - 다른 **View나 ViewGroup을 담는 컨테이너** (예: LinearLayout, ConstraintLayout)
    - 자식 뷰들의 **레이아웃**(크기, 위치)을 관리하고 결정
    - UI 계층 구조에서 **중간 노드**(branch node) 역할을 하며, 여러 자식을 포함할 수 있음
    - 자식 뷰의 이벤트를 가로채거나 관리할 수 있음
    - View를 상속하며, 자식 뷰를 추가/제거하는 기능(`ViewManager`)과 자식 뷰의 레이아웃 및 이벤트 처리를 관리하는 기능(`ViewParent`)을 가짐

**핵심 차이점**

- **View**는 개별 UI 요소이고, **ViewGroup**은 이러한 요소들을 담고 배치하는 컨테이너
- ViewGroup은 계층 구조로 인해 복잡성을 더하며, 과도하게 중첩하면 성능에 영향을 줄 수 있음