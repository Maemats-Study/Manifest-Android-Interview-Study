# 애플리케이션 용량을 어떻게 줄이나요?

## 사용하지 않는 리소스 제거하기

- 이미지, 레이아웃 또는 문자열과 같이 사용되지 않는 리소스는 불필요하게 APK 또는 AAB 크기를 증가시킴
- Android Studiod의 Lint와 같은 도구는 이러한 리소스를 식별하는 데 도움이 될 수 있음
- 사용하지 않는 리소스를 제거한 후, build.gradle 파일에서 shrinkResources를 활성화하여 빌드 프로세스 중에 사용되지 않는 리소스를 자동으로 제거하도록 함

```kotlin
android {
		buildTypes {
				release {
						minifyEnabled true
						shrinkResources true
				}
		}
}
```

## R8로 코드 축소 활성화하기

- ProGuard를 적절하게 사용하면 중요한 코드나 리플렉션 기반 라이브러리는 난독화를 생략하고 오동작하지 않도록 보장

```kotlin
android {
		buidlTypes {
				release {
						minifyEnalbed true
						proguardFiles getDefaultProguardFile('proguard-android-optimize.txt', 'proguard-rules.pro')
				}
		}
}
```

## 리소스 최적화 사용하기

이미지 및 XML 파일과 같은 리소스를 최적화 → 앱 용량 줄일 수 있음

- Vector Drawables
    - 확장 가능한 그래픽을 위해 래스터 이미지(PNG, JPEG) 대신에 용량을 덜 차지하는 vector drawables로 대체
- 이미지 압축
    - TinyPNG 또는 ImageMagick과 같은 도구를 사용하여 눈에 띄는 품질 손상 없이 래스터 이미지를 압축
- WebP 형식
    - 이미지를 PNG 또는 JPEG 보다 압축률이 좋은 WebP 형식으로 변환

```kotlin
android {
		defaultConfig {
				vectorDrawables.useSupportLibrary = true
		}
}
```

## Android App Bundles (AAB) 사용하기

- AAB 형식으로 전환하면 Google Play가 개별 기기에 맞는 최적화된 APK 제공
- 특정 구성에 필요한 리소스와 코드만 포함 → 앱 용량 줄임

```kotlin
android {
		bundle {
				density {
						enableSplit = true
				}
				abi {
						enableSplit = true
				}
				language {
						enableSplit = true
				}
		}
}
```

## 불필요한 의존성 제거하기

- 프로젝트의 의존성 검토 → 사용됮 않거나 중복되는 라이브러리 제거
- Android Studio의 Gradle Dependency Anlyzer를 사용 : 무거운 라이브러리 및 전이 의존성을 식별

## 네이티브 라이브러리 최적화하기

- 사용하지 않는 아키텍처 제외
    - build.gradle 파일의 abiFilters 옵션을 사용하여 필요한 ABI만 포함
- 디버깅 심볼 제거
    - stripDebugSymbols를 사용하여 네이티브 라이브러리에서 디버깅 심볼을 제거

```kotlin
android {
		defaultConfig {
				ndk {
						abiFilters "armeabi‑v7a", "arm64‑v8a" // 필요한 ABI만 포함
				}
		}
		packagingOptions {
				exclude "**/lib/**/*.so.debug"
		}
}
```

## Proguard 규칙을 구성하여 디버그 정보 줄이기

- 디버깅 메타데이터는 최종 APK 또는 AAB에 불필요한 무게를 더함
- proguard-ruels.pro 파일을 구성하여 이러한 정보를 제거

```
‑dontwarn com.example.unusedlibrary.**
‑keep class com.example.important.** { *; }
```

## 동적 기능(Dynamic Featrues) 사용학

- 동적 기능 모듈을 사용하면 자주 사용되지 않는 기능을 주문형 모듈(해당 기능이 필요한 경우 설치)로 분리하여 앱을 모듈화할 수 있음
- 초기 다운로드 용량을 줄일 수 있음

```kotlin
// app/build.gradle (Groovy DSL)
android {
		dynamicFeatures = [":feature1", ":feature2"]
}
```

## 앱 내 대용량 애셋 피하기

- 비디오나 고해상도 이미지와 같은 대용량 애셋은 콘텐츠 전송 네트워크(CDN)에 호스팅하고 런타임에 동적으로 로드
- 미디어 콘텐츠는 앱과 함께 번들링하는 대신 스트리밍을 사용