# Bitmap의 효율적인 처리

## Bitmap

: 메모리 내 이미지 표현

- 픽셀 데이터를 저장하며 리소스, 파일 또는 원격 소스에서 가져온 이미지를 화면에 렌더링하는데 자주 사용됨
- 고해상도 이미지의 경우 대량의 픽셀 데이터

⇒ 메모리 고갈 / OutOfMemoryError

### 큰 Bitmap 문제점

- 이미지 → 이를 표시하는 UI 컴포넌트에서 요구하는 크기보다 훨씬 큼
- 전체 해상도 이미지를 불필요하게 로드할 시
    - 과도한 메모리 소비
    - 성능 오버헤드
    - 메모리 압박으로 인해 크래시 발생 위험

### 메모리를 할당하지 않고 Bitmap 크기 읽기

- 비트맵을 로드하기 전에 크기를 검사
    - `BitmapFactory.Optiions` 클래스 : `inJustDecodeBounds = true` 설정 → 메모리 할당하지 않으면서 이미지 메타데이터를 디코딩

### 샘플링을 사용하여 축소된 Bitmap 로드하기

- 크기를 알게되면 `inSampleSize` 옵션 → 비트맵을 대상 크기에 맞게 축소
- 이미지를 2, 4 등의 배수로 서브샘플링
- `imSampleSize = 4`로 로드된 2048 x 1536 → 512 x 384

⇒ 이미지 품질 & 메모리 효율성 규현

### 서브샘플링을 사용한 전체 디코딩 프로세스

- `calculateInSampleSize` → 두 단계로 비트맵 디코딩
    1. 경계(bounds)만 디코딩
    2. 계산된 `inSampleSize`를 설정하고 축소된 비트맵을 디코딩

## 효율적 처리 방법 더 알아보기

### 캐싱 (Caching)

한번 로드한 비트맵은 재사용하여 네트워크 요청이나 디코딩 과정을 반복하지 않도록 함

- 메모리 캐시 (LruCache)
- 최근 사용된 비트맵을 RAM에 잠시 보관
    - `RecyclerView`를 스크롤할 때 이미지를 빠르게 다시 보여줄 수 있음
- 디스크 캐시 (DiskLruCache)
    - 처리된(다운샘플링된) 이미지를 앱의 내부 저장소에 파일로 저장
    - 앱을 껐다 켜도 이미지를 다시 다운로드할 필요가 없음

### 적절한 비트맵 설정(Config) 사용

모든 이미지가 픽셀당 4바이트(`ARGB_8888`)를 쓸 필요는 없음

- `ARGB_8888` (기본값, 4바이트): 고품질, 투명도(Alpha) 채널 포함
- `RGB_565` (2바이트): 투명도가 필요 없는 이미지(예: 배경화면)에 사용
    - 화질은 약간 저하되지만, 메모리 사용량을 절반으로 줄일 수 있음

### 비트맵 재사용 (inBitmap)

(API 11+) 비트맵을 디코딩할 때, 이전에 사용했던(지금은 필요 없는) 비트맵의 메모리 공간을 재사용하도록 지정할 수 있음

- `BitmapFactory.Options`의 `inBitmap` 속성에 재사용할 비트맵을 할당
- 이는 새로운 메모리를 할당하고 기존 메모리를 해제하는 과정을 줄여주어, 가비지 컬렉션(GC)으로 인한 앱 버벅임(Jank)을 줄이는 데 큰 도움이 됨

## Compose에서 Bitmap 효율적으로 사용하기

### 이미지 로딩 라이브러리 사용

- 예전 View 시스템: 개발자가 직접 명령해야 함
    1. `inJustDecodeBounds`로 크기만 읽어라
    2. `calculateInSampleSize`로 축소 비율을 계산해라
    3. 계산된 `inSampleSize`로 비트맵을 디코딩해라
    4. `LruCache`에 있는지 확인하고, 없으면 저장해라
    5. `ImageView`에 비트맵을 설정해라
- 현대 안드로이드 개발에서 대부분의 경우 Glide(View 시스템)나 Coil(Compose) 같은 라이브러리를 사용

### Coil (Coroutine Image Loader)

1. build.gradle에 의존성 추가: `implementation("io.coil-kt:coil-compose:2.5.0")`
2. Composable 함수 안에서 `AsyncImage`를 부름

### Coil이 자동으로 수행

- `inSampleSize` 계산
    - `AsyncImage`는 `Modifier.size(100.dp)`를 보고 이미지가 표시될 UI 크기를 자동으로 감지
    - 원본 이미지가 4000x4000 픽셀이라도, 딱 100.dp에 필요한 만큼만 다운샘플링하여 메모리에 로드(스크린샷의 `calculateInSampleSize`, `decodeSampledBitmapFromResource` 자동 수행)
- 디스크 캐시 (DiskLruCache)
    - Coil은 메모리 캐시뿐만 아니라 디스크 캐시도 자동으로 지원
    - 네트워크에서 받은 이미지를 디스크에 저장해 두어, 앱을 껐다 켜도 이미지를 다시 다운로드하지 않음
- 백그라운드 처리
    - 모든 디코딩 및 네트워크 작업은 코루틴을 통해 백그라운드 스레드에서 자동으로 일어남

### Coil 라이브러리가 어떻게 동작할까?

1. 다중 레벨 캐시 파이프라인 (Memory & Disk)
    - 1단계: 메모리 캐시 (RAM)
        - `LruCache`를 기반으로 함
        - 가장 빠름. 이미지가 RAM에 (디코딩된 `Bitmap` 객체로) 존재하므로 즉시 UI에 그릴 수 있음
        - 앱이 종료되면 사라짐
    - 2단계: 디스크 캐시 (Storage)
        - `DiskLruCache`를 기반으로 함
        - 이미지 데이터를 (디코딩되기 전 원본 또는 리사이징된 파일로) 디스크에 저장
        - 메모리 캐시보다는 느리지만(디스크 I/O 발생), 네트워크 통신보다는 훨씬 빠름
        - 앱을 껐다 켜도 유지
    
    `AsyncImage` 요청이 오면 Coil은 [메모리 캐시 확인] → [디스크 캐시 확인] → [네트워크/소스에서 로드] 순서로 이미지를 찾음
    
2. 비트맵 풀링 (Bitmap Pooling)
    
    ⇒ `OutOfMemoryError`와 앱 버벅임(Jank)을 막는 핵심 기술
    
    - `Bitmap` 객체는 네이티브 메모리(Heap 바깥쪽)를 많이 사용
        - 이미지를 다 쓰고 `null`로 만들면, 가비지 컬렉터(GC)가 이 큰 메모리를 해제하기 위해 자주 동작하고, 이때 앱이 잠시 멈칫(버벅)거림
    - 풀링으로 해결
        - 라이브러리는 `Bitmap`을 버리지 않고, '비트맵 재사용 풀(Pool)'에 반납
    - 동작
        1. 100x100 픽셀 비트맵이 필요하면, 풀(Pool)에 100x100 크기의 '놀고 있는' 비트맵이 있는지 확인
        2. 있으면? → 새 메모리를 할당하지 않고 기존 비트맵을 가져와 픽셀 데이터만 덮어씀 (매우 빠름)
        3. 없으면? → 새로 생성
        4. 이미지 로드가 끝나면(예: 다른 화면으로 넘어가서) 비트맵을 풀에 반납
    
    ⇒ `inBitmap` 옵션을 훨씬 더 체계적으로 관리하는 방식
    
3. 코루틴 기반 비동기 처리 (Dispatchers.IO)
    
    ⇒ Coil의 'Co'는 Coroutine의 약자
    
    - `AsyncImage`는 Composable 함수이므로 메인 스레드에서 실행됨
    - 하지만 이미지 로드 요청이 들어오는 즉시, Coil은 모든 무거운 작업을 `Dispatchers.IO` (백그라운드 I/O 스레드 풀)로 자동 전환
    - 무거운 작업
        - 네트워크 통신
        - 디스크 캐시 읽기/쓰기
        - 비트맵 디코딩 (스크린샷의 `BitmapFactory.decode...`)
        - 이미지 변환 (리사이징, 자르기 등)
    - 모든 작업이 끝나고 최종 `Bitmap`이 준비되면, 그 결과만 다시 메인 스레드로 가져와 화면에 그림
    - 개발자는 이 스레드 전환을 전혀 신경 쓸 필요가 없음
4. 인터셉터 파이프라인 (Fetchers & Decoders)
    
    ⇒ "Coil은 어떻게 URL, 로컬 파일, 리소스 ID를 모두 처리할 수 있을까?"에 대한 답
    
    - Coil(그리고 OkHttp)은 '인터셉터(Interceptor)' 체인을 사용합니다. 요청은 여러 단계의 인터셉터를 파이프라인처럼 통과
    
    > [요청] → (캐시 인터셉터) → (변환 인터셉터) → (Fetcher) → (Decoder) → [결과]
    > 
    - Fetcher: 입력 데이터(URL, Uri, File...)를 실제 이미지 바이트 스트림(Byte Stream)으로 가져오는 역할
        - `HttpUriFetcher`: URL을 받으면 OkHttp로 네트워크 통신을 함
        - `FileUriFetcher`: File 경로를 받으면 파일을 확인
    - Decoder: 바이트 스트림을 `Bitmap`이나 `Drawable` 객체로 디코딩(해석)하는 역할
        - `BitmapFactoryDecoder`: PNG, JPG, WEBP 파일을 디코딩
        - `GifDecoder`: GIF 파일을 애니메이션 `Drawable`로 디코딩
    
    ⇒ 이 구조 덕분에 개발자는 커스텀 Fetcher(암호화된 이미지를 로드) 등을 만들어 파이프라인에 쉽게 끼워 넣을 수 있음