# build variants와 flavors란?

빌드 변형과 플레이브 → 단일 코드베이스에서 애플리케이션의 다양한 버전을 생성하는 유연한 방법을 제공

- 이 시스템을 통해 동일한 프로젝트 내에서 개발 및 프로덕션 빌드 또는 무료 및 유료 버전과 같은 여러 구성을 효율적으로 관리할 수 있음

## 빌드 변형 (Build Variants)

<aside>

- 빌드 타입(build type)과 제품 플레이버(product flavor)(플레이버가 정의된 경우)를 결합한 결과
- 안드로이드 Gradle 플러그인은 각 조합에 대해 빌드 변형을 생성하여 다양한 사용 사례에 맞는 APK 또는 번들을 생성할 수 있도록 함

</aside>

- 빌드 타입
    - 디버그(Debug)
        - 개발 중에 사용되는 빌드 구성
        - 디버그 도구, 로그 및 테스트용 디버그 툴 → 향상성 높임
    - 릴리스(Release)
        - 배포에 최적화된 구성
        - 리소스 최적화 및 최소화, 난독화
        - 스토어 게시를 위해 별도의 릴리스 키로 서명
- 기본적으로 모든 안드로이드 프로젝트에는 debug 및 release 빌드 타입이 포함됨
- 개발자는 요구 사항에 맞게 커스텀 빌드 타입을 추가할 수 있음

## Product Flavors

<aside>

- 개발자는 무료 및 유료 버전이나 us 및 eu와 같은 지역별 버전과 같이 앱의 다양한 변형을 정의할 수 있음
</aside>

- 각 플레이버는 애플리케이션 ID, 버전 이름 또는 리소스와 같은 고유한 구성을 가질 수 있음
- 이를 통해 코드를 복제하지 않고 맞춤형 빌드를 쉽게 만들 수 있음

```kotlin
// Kotlin DSL (build.gradle.kts)
android {
		...
		flavorDimensions += "version" // 플레이버 차원(dimension) 정의
		
		productFlavors {
				create("free) {
						dimension = "verison"
						applicationIdSuffix = ".free"
						versionNameSuffix = "-free"
				}
				
				create("paid") {
						dimension = "version"
						applicationIdSuffix = ".paid"
						versionNameSuffix = "-paid"
				}
		}
}
```

→ 안드로이드 Gradle 플러그인은 freeDebug, freeRelease, paidDebug, paidRelease 총 4가지 조합으로 제품 변형에 따른 빌드를 수행할 수 있음

## 빌드 타입과 플레이버 결합하기

빌드 변형 시스템은 빌드 타입과 제품 플레이버를 결합하여 가능한 빌드의 매트릭스를 만듦

- freeDebug: 디버깅용 무료 버전
- pairRelease: 릴리스에 최적화된 유료 버전

각 조합은 변형 조건에 따른 설정, 리소스를 가지거나 코드를 다르게 동작

(ex. 무료 버전에서는 광고 표시 / 유료 버전에서는 비활성화)

빌드 타입 및 플레이버에 따라 서로 다른 리소스 디렉ㅌ뢰 및 Java/Kotlin 코드로 빌드함으로써 다르게 동작하는 앱을 빌드할 수 있음

## 빌드 변형 및 플레이버 사용의 이점

1. 효율적인 구성
    - 프로젝트를 통째로 복제할 필요가 없음 → 중복 코드 줄이고 단일 코드베이스에서 여러 빌드 처리 가능
2. 커스텀 동작
    - 유료 버전에서 프리미엄 기능을 활성화
    - 디버그와 릴리스 빌드에서 각각 다른 API를 사용
    - 앱 동작을 맞춤 설정 가능
3. 자동화
    - Gradle은 빌드 변형에 따라 APK 서명, 최적화 및 난독화와 같은 작업을 자동화