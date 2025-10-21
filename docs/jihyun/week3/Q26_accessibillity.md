# 접근성(accessibility)을 어떻게 보장할까?

<aside>

### 접근성

- 시각, 청각 또는 신체 장애가 있는 사람들을 포함하여 모든 사람이 애플리케이션을 사용할 수 있도록 보장하는 것
- 접근성 기능 구현 → 사용자 경험 향상, WCAG(Web Content Accessibility Guidelines)와 같은 글로벌 접근성 표준 준수를 보장
</aside>

## 콘텐츠 설명(Content Descriptions) 활용하기

- 콘텐츠 설명은 UI 컴포넌트에 텍스트 레이블을 제공하여 TalkBack과 같은 스크린 리더가 시각 장애가 있는 사용자에게 해당 컴포넌트를 알릴 수 있도록 함
- 버튼, 이미지, 아이콘과 같이 상호 작용하거나 정보를 제공하는 컴포넌트 → `contentDescription` 매개변수 사용
    
    ```kotlin
    // 1. 아이콘 버튼
    IconButton(onClick = { /* ... */ }) {
        Icon(
            imageVector = Icons.Default.Menu,
            contentDescription = "메뉴 열기" // TalkBack이 "메뉴 열기, 버튼" 등으로 읽음
        )
    }
    
    // 2. 이미지
    Image(
        painter = painterResource(id = R.drawable.my_profile_image),
        contentDescription = "내 프로필 사진"
    )
    ```
    
- 더 복잡하거나 커스텀한 설정을 위해서는 `Modifier.semantics`를 사용
    - `contentDescription` 매개변수가 없거나, 여러 요소를 하나로 묶어서 설명해야 하는 등 더 세밀한 제어가 필요할 때 사용
    - `semantics` : 스크린 리더가 UI를 이해할 수 있도록 의미를 부여하는 수정자
    
    ```kotlin
    Box(
        modifier = Modifier
            .semantics {
                contentDescription = "새 메시지 알림" // 이 Box 전체에 대한 설명
                role = Role.Button // TalkBack이 "버튼"이라고 알려줌
            }
    ) {
        // ... 내부에 다른 UI 요소들
    }
    ```
    
- 단순히 장식용 컴포넌트 → `contentDescription = null`로 설정
    - `Modifier.clearAndSetSemantics { }` 사용
        - 해당 컴포저블과 그 자식 요소들의 모든 의미 정보(semantics)를 완전히 제거
        
        ```kotlin
        Box(
            // 이 Box와 Box 내부의 모든 요소를 스크린 리더가 완전히 무시함
            modifier = Modifier.clearAndSetSemantics { }
        ) {
            Text("이 텍스트도 무시됨")
            Icon(Icons.Default.Info, contentDescription = "이것도 무시됨")
        }
        ```
        

## 포커스 관리 및 탐색

### 기본 동작 (자동 탐색)

- Compose는 대부분의 경우 개발자가 특별히 설정하지 않아도 선언된 순서와 화면상 위치를 기반으로 논리적인 탐색 경로를 자동으로 결정
    - 키보드 Tab: 1차원 탐색. Composable이 선언된 순서대로 이동
    - D-Pad(방향키): 2차원 탐색. 상/하/좌/우 방향으로 가장 가까운 Composable을 찾아 이동

### 명시적 탐색 경로 지정

- `FocuseRequester`, `Modifier.focusProperties`, `FocusManager` 등을 사용하여 특정 방향의 다음 대상을 직접 제어할 수 있음
- `FocusRequester` 사용 (프로그래밍 방식)
    - 특정 Composable에 포커스를 **강제로 주거나** 요청할 때 사용
    - 용도:
        - 화면 진입 시 특정 `TextField`에 자동으로 포커스 주기
        - 다이얼로그가 떴을 때 '확인' 버튼에 바로 포커스 주기
    
    ```kotlin
    val focusRequester = remember { FocusRequester() }
    
    // 화면이 처음 뜰 때(LaunchedEffect) 포커스 요청
    LaunchedEffect(Unit) {
        focusRequester.requestFocus()
    }
    
    TextField(
        value = "...",
        onValueChange = { ... },
        modifier = Modifier.focusRequester(focusRequester) // 포커스 대상 지정
    )
    ```
    
- `FocusManager` 사용 (프로그래밍 방식)
    - 현재 포커스를 이동시키거나 해제할 때 사용
    - 특정 순간에 어디로 갈지 명령하는 프로그래밍 방식
    
    ```kotlin
    val focusManager = LocalFocusManager.current
    
    Column {
        TextField(
            value = "...",
            onValueChange = { ... },
            keyboardActions = KeyboardActions(
                onNext = {
                    // '다음' 버튼 누르면 '아래'로 포커스 이동
                    focusManager.moveFocus(FocusDirection.Down)
                }
            )
        )
        TextField(
            value = "...",
            onValueChange = { ... },
            keyboardActions = KeyboardActions(
                onDone = {
                    // '완료' 버튼 누르면 포커스 해제 (키보드 숨김)
                    focusManager.clearFocus()
                }
            )
        )
    }
    ```
    
- `Modifier.focusProperties` 사용 (선언적 방식)
    - "A 항목에서 '아래' 키를 누르면 무조건 C로 가라", "B에서 '왼쪽' 키를 누르면 A로 가라"처럼 각 방향의 경로를 선언적으로 지정
    - 레이아웃이 복잡해서 시스템의 자동 계산(위->아래, 왼->오른쪽)이 꼬일 때 사용
        - 특히 TV 리모컨(D-패드)처럼 상하좌우 탐색이 중요한 경우에 필수적
    - `FocuseRequester`로 각 요소에 “ID”를 부여하고 `focusProperties` 로 관계를 정의
    
    ```kotlin
    val (itemA, itemB, itemC, itemD) = remember { FocusRequester.createRefs() }
    
    Column {
        Row {
            Button(
                onClick = { },
                modifier = Modifier
                    .focusRequester(itemA)
                    .focusProperties {
                        // A에서 '오른쪽' 누르면 B로
                        right = itemB 
                        // A에서 '아래' 누르면 C로
                        down = itemC  
                    }
            ) { Text("A") }
            
            Button(
                onClick = { },
                modifier = Modifier.focusRequester(itemB)
            ) { Text("B") }
        }
        Row {
            Button(
                onClick = { },
                modifier = Modifier.focusRequester(itemC)
            ) { Text("C") }
            
            Button(
                onClick = { },
                modifier = Modifier.focusRequester(itemD)
            ) { Text("D") }
        }
    }
    ```
    

## 동적 글꼴 크기 지원하기

- 앱이 기기 설정에서 사용자가 설정한 글꼴 크기 환경 설정을 존중하도록 보장
- 접근성 설정에 따라 자동으로 크기가 조정되도록 텍스트 크기에는 sp 단위를 사용
- `sp` 와 `dp` 비교
    - 두 단위 보두 화면 밀도(DPI)에 따라 크기가 조절되어 다양한 기기에서 일관된 크기를 보여주는 것을 목표로 함
    - `sp` : (Scale-independent Pixels)
        - 확장 가능한 픽셀
        - 기준: 화면 밀도 + 사용자의 시스템 글꼴 크기 설정
        - 텍스트에만 사용
    - `dp`: (density-independent Pixels)
        - 밀도 독립적인 픽셀
        - 기준: 화면 밀도
        - 텍스트를 제외한 모든 레이아웃, 패딩, 간격, 컴포넌트 크기에 사용
    - 만약 앱이 텍스트 크기로 `sp`를 사용했다면, 앱 내의 모든 텍스트가 사용자가 설정한 크기로 자동으로 변경됨
    - 앱이 `dp`를 설정했다면, 사용자가 글씨 크기를 조절해도 `dp`는 이 설정을 무시함
    - 유연한 레이아웃을 만들어야 함
        - 텍스트가 들어가는 컴포넌트에 `Modifier.height(48.dp)` 처럼 고정된 높이를 주면, 글자가 커졌을 때 높이를 벗어나 잘리게 됨
        - `Modifier.wrapContentHeight()` 나 `Modifier.heightIn(min = 48.dp)` (최소 높이 지정) 등을 사용하여 텍스트 크기에 맞춰 레이아웃이 유연하게 늘어날 수 있도록 설계해야 함

## 색상 대비 및 시각적 접근성

- 저시력 또는 색맹 사용자의 가독성을 향상시키기 위해 텍스트와 배경색 간에 충분한 대비를 제공
- Android Studio의 Accessibility Scanner와 같은 도구 → 앱의 색상 대비를 평가하고 최적화하는데 도움

## 커스텀 뷰 및 접근성

- Compose에서는 `Modifier.semantics`를 사용하여 스크린 리더가 커스텀 UI 컴포넌트와 상호작용하는 방식을 정의

### Semantics

- Semantics(의미론)는 UI가 '어떻게 보이는지(Look)'가 아니라, '무엇을 의미하는지(Meaning)'를 설명하는 정보의 집합
- 이 정보는 사람의 눈이 아닌 도구(Tool)를 위한 것
- ex) 돋보기 모양의 `Icon`
    - 시각 정보 (UI Tree): "돋보기 모양의 벡터 그래픽"
    - 의미 정보 (Semantics Tree): "이것은 '검색'이라는 텍스트를 가졌고, '버튼' 역할을 한다."
- Compose는 이 의미 정보를 모아서 '시맨틱 트리(Semantics Tree)'라는 별도의 트리를 구축
- Semantics 정보가 필요한 이유
    1. 접근성 서비스 (예: TalkBack)
        - 시각 장애가 있는 사용자가 스크린 리더(TalkBack)를 켰을 때, 이 서비스는 UI를 "보는" 게 아니라 "시맨틱 트리"를 읽음
            - `Icon(..., contentDescription = "검색")`
            - `Modifier.semantics { role = Role.Button }`
            
            → TalkBack은 이 두 정보를 조합해서 "검색, 버튼, 두 번 탭하여 활성화"라고 사용자에게 음성으로 알려줌
            
            만약 시맨틱 정보가 없다면 TalkBack은 이 아이콘의 존재 자체를 무시하거나 "라벨 없음, 버튼"이라고만 말할 것
            
    2. 테스트 프레임워크 (UI Tests)
        - 자동화된 UI 테스트를 실행할 때, 테스트 코드는 화면의 픽셀을 분석하지 않음. 시맨틱 트리를 탐색하여 특정 노드(Node)를 찾고 클릭하거나 상태를 확인
            - `Modifier.semantics { testTag = "search_button" }`
            
            → 테스트 코드는 `onNodeWithTag("search_button")`라는 코드로 이 버튼을 정확하게 찾아내 `performClick()`을 실행할 수 있음
            

### Semantics 관리 방법

- Compose의 거의 모든 기본 컴포저블(`Text`, `Button`, `Image` 등)은 자동으로 적절한 시맨틱 정보를 트리에 추가함
    
    (예: `Text`는 `text` 속성을, `Button`은 `role = Role.Button`과 `onClick` 액션을 추가)
    
- 개발자가 직접 Semantics를 관리하는 경우
    1. `Box`, `Column` 등으로 커스텀 컴포넌트를 만들 때
    2. 기본 컴포저블의 시맨틱 정보를 덮어쓰거나 수정해야 할 때
    3. 여러 요소를 하나의 의미로 병합하고 싶을 때
- 핵심 도구: `Modifier.semantics`
    - `Modifier.semantics { ... }` 블록 내에서 UI 요소의 '의미'를 정의할 수 있음
    
    ```kotlin
    // Box와 Text를 묶어 하나의 '버튼'처럼 만듦
    Box(
        modifier = Modifier
            .size(100.dp)
            .background(Color.Blue)
            .clickable { /* 클릭 시 동작 */ }
            .semantics {
                // 1. 역할 정의
                role = Role.Button
                
                // 2. 내용 설명
                contentDescription = "새로운 글 작성하기"
                
                // 3. 클릭 액션 설명
                onClick(label = "새 글 작성") {
                    /* 클릭 시 실제 동작 */
                    true
                }
            }
    ) {
        Text("새 글", color = Color.White)
    }
    ```
    

### 주요 Semantics 속성

- `contentDescription` (String)
    - TalkBack이 읽어줄 핵심 텍스트
- `role` (Role)
    - 이 요소의 **역할**을 정의 (예: `Role.Button`, `Role.Checkbox`, `Role.Image`, `Role.Tab`)
    - TalkBack이 "버튼", "체크박스"라고 종류를 말해줌
- `stateDescription` (String)
    - 요소의 현재 상태를 설명 (예: `ToggleButton`이 "On"일 때 "켜짐", "Off"일 때 "꺼짐")
    - `role`이 `Role.Switch`일 때 "켜짐/꺼짐"을 알려주는 식
- `onClick(label = ...)` (Action)
    - `Modifier.clickable`이 있으면 자동으로 추가되지만, `label`을 설정하면 TalkBack이 "두 번 탭하여 [label]"이라고 더 구체적으로 안내 (예: `label = "삭제하기"`)
- `customActions = listOf(...)` (List<CustomAccessibilityAction>)
    - '클릭' 외의 커스텀 동작(예: "삭제", "공유", "수정")을 추가. TalkBack의 '작업' 메뉴에 노출됨
- `testTag` (String)
    - 오직 UI 테스트에서만 사용되는 식별자(ID)

### 접근성 탐색(탐색 순서 및 그룹화 속성)

- 스크린 리더(TalkBack) 사용자를 위한 탐색 순서
- 키보드 탐색과는 별개로 동작
- `isTraversalGroup` (Boolean)
    - 여러 Composable을 하나의 논리적 단위로 묶음
    - TalkBack이 이 그룹을 먼저 다 읽고 다음 요소로 넘어감
    
    ```kotlin
    // 이 Row 전체를 하나의 그룹으로 묶음
    Row(
        modifier = Modifier.semantics { isTraversalGroup = true }
    ) {
        Image(
            painter = ...,
            contentDescription = null // 그룹의 텍스트가 설명하므로 null
        )
        Column {
            Text("제목 텍스트")
            Text("부제목 텍스트")
        }
    }
    // TalkBack은 "제목 텍스트, 부제목 텍스트"라고 이 그룹을 한 번에 읽음
    ```
    
- `traversalIndex` (Float)
    - TalkBack이 탐색하는 순서를 수동으로 지정
    - 숫자가 낮을수록 먼저 우선순위 높음 ****(기본값 0)
    - 음수 값도 사용할 수 있음
    
    ```kotlin
    Column {
        // 3. 마지막으로 읽힘 (기본값 0f보다 큼)
        Text(
            text = "세 번째",
            modifier = Modifier.semantics { traversalIndex = 1f }
        )
        
        // 1. 가장 먼저 읽힘 (음수)
        Text(
            text = "첫 번째",
            modifier = Modifier.semantics { traversalIndex = -1f }
        )
    
        // 2. 중간에 읽힘 (기본값)
        Text(text = "두 번째") 
    }
    ```
    
- `mergeDescendants` (Boolean)
    - `true`로 설정하면, 이 노드의 모든 자식 노드들의 시맨틱 정보(예: `Text`)를 하나로 병합하여 부모의 단일 노드로 만듦
    - 예: `Image`와 `Text`가 있는 리스트 아이템을 하나의 클릭 가능한 요소로 묶을 때 사용
- `clearAndSetSemantics` (Lambda)
    - `mergeDescendants`보다 강력
    - 이 노드의 모든 자식 노드들의 시맨틱 정보를 완전히 무시(삭제)하고, 이 블록에서 새로 정의하는 속성만 사용

## 접근성 테스트하기

- Android Studio의 Accessibility Scanner 및 Layout Inspector와 같은 도구를 사용하여 접근성 문제를 식별하고 수정
- TalkBack
    - 설정에서 TalkBack을 직접 켜고, 눈을 감거나 화면을 보지 않은 채 스와이프와 두 번 탭하기만으로 앱의 모든 기능을 사용할 수 있는지 테스트해야 함
- Accessibility Scanner (접근성 스캐너)
    - Google Play 스토어에서 설치하는 앱
    - 현재 화면을 스캔하여 색상 대비, 터치 영역 크기, 콘텐츠 설명 누락 등 명백한 문제점들을 자동으로 찾아내고 수정 사항을 제안
- Android Studio 내장 도구
    - Compose Preview: 프리뷰 창 상단의 'UI 검사(UI Check)' 모드를 활성화하면 색상 대비, 터치 영역 등 기본적인 접근성 문제를 실시간으로 보여줌
    - Layout Inspector: 실행 중인 앱의 레이아웃을 분석할 때 접근성 속성(Semantics)이 어떻게 적용되었는지 확인할 수 있음
- 시스템 설정 변경
    - 글꼴 크기: '설정 > 접근성'에서 글꼴 크기를 '가장 크게'로 설정하고 앱이 깨지지 않는지 확인 (sp 단위 테스트)
    - 화면 표시 크기: '표시 크기'를 조절했을 때 레이아웃이 유연하게 반응하는지 확인
- UI 자동화 테스트 (Espresso / Compose Test)
    - 테스트 코드에서 `onNode(hasContentDescription("..."))`와 같은 Semantics Matcher를 사용해, 특정 요소에 올바른 콘텐츠 설명이나 역할이 부여되었는지 검증