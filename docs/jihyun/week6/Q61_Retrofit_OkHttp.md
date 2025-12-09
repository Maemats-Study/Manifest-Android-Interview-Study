# 원격 데이터를 가져오기 위해 네트워크 요청을 어떻게 처리하며, 효율성과 신뢰성을 위해 어던 라이브러리나 기술을 사용하나요?

일반적으로 Retroift과 Okttp는 안드로이드 개발에서 백엔드로부터 네트워크 요청을 하는 데 흔히 사용되는 라이브러리

- Retrofit은 선언적 인터페이스를 제공하여 API 상호 작용을 단순화
- OkHttp는 기본적인 HTTP 클라이언트 역할을 하여 연결 풀링, 캐싱 및 효율적인 통신을 제공

## Retrofit을 사용한 네트워크 요청

- Retrofit은 HTTP 요청을 깔끔하고 타입 세이프한 API 인터페이스로 추상화
- JSON 응답을 Kotlin 또는 Java 객체로 변환하기 위해 Gson 또는 Moshi와 같은 직렬화 라이브러리와 원활하게 작동
- Retrofit을 사용하여 백엔드 데이터를 가져오기 위한 단계
    1. API 인터페이스 정의: 어노테이션을 사용하여 API 엔드포인트와 HTTP 메서드를 선언
        
        ```kotlin
        interface ApiService {
        		@GET("data")
        		suspend fun fetchData(): Response<DataModel>
        }
        ```
        
    2. Retrofit 인스턴스 설정: 베이스 URL과 JSON 직렬화를 위해 ConverterFactory를 통해 Retrofit 구성
        
        ```kotlin
        val retrofit = Retrofit.Builder()
        		.baseUrl("https://api.example.com/")
        		.addConverterFactory(Json{ ignoreUnknownKeys = true }
        		.asConverterFactory("application/json".toMediaType()))
        		.build()
        
        val apiService = retrofit.create(ApiService::class.java)
        ```
        
    3. 네트워크 호출하기: 코루틴을 사용하여 API를 비동기적으로 호출
        
        ```kotlin
        viewModelScope.launch {
        		try {
        				val response = apiService.fetchData()
        				if (response.isSuccessful) {
        						if (data != null) {
        								Log.d(TAG, "Data fetched: $it"
        						} else {
        								Log.e(TAG, "Response body is null")
        						}
        				} else {
        						Log.e(TAG, "Error: ${response.code()} - ${response.message()}")
        				}
        		} catch (e: Exception) {
        				Log.e(TAG, "Network request failed", e)
        		}
        }
        ```
        

## OkHttp를 사용한 커스텀 HTTP 요청

- OkHttp는 헤더, 캐싱 등을 세밀하기 제어할 수 있도록 HTTP 요청을 관리하기 위해 더 직접적인 접근 방식을 보여줌

## OkHttp와 Retrofit 통합하기

- Retrofit은 내부적으로 OkHttp를 HTTP 클라이언트로 사용
- 인터셉터를 추가하여 로깅, 인증 또는 캐싱과 같은 OkHttp 동작을 커스텀할 수 있음

## OkHttp Authenticator 및 Interceptor를 사용하여 OAuth 토큰 갱신하기

- OAuth로 보호되는 API로 작업할 때 토큰 만요 및 갱신 시나리오를 처리하는 것이 일반적
- OkHttp는 노큰을 가로채고 새로 고치는 두 가지 주요 매커니즘인 Authenticator와 Interceptor를 제공
- 둘 다 다른 목적을 가지고 있으며 애플리케이션의 특정 요구 사항에 따라 선택적으로 사용할 수 있음

### OkHttp Authenticator

- OkHttp의 Authenticator 인터페이스는 토큰 만료와 같은 인증 문제를 처리하도록 따로 설계됨
- 서버가 401 Unauthorized 상태 코드를 응답하면 Authenticator가 호출되어 업데이트된 인증 자격 증명이 포함된 새 요청을 제공
- 토큰을 새로 고치기 위해 Authenticator 구현하기
    
    ```kotlin
    class TokenAuthenticator(
    		private val tokenProvider
    ): Authenticator {
    		override fun authenticate(route: Route?, response: Response): Request? {
    				val previousToken = response.request.header("Authorization")
    				
    				Synchronized(this) {
    						val currentToken = tokenProvider.getToken()
    						if (previousToken != null && previousToken == "Bearer $currentToken") {
    								val newToken = tokenProvider.refreshToken()
    								if(newToken == null) {
    										tokenProvider.clearToekn()
    										return null
    								}
    						}
    						
    						val refreshedToken = tokenProvider.getToken()
    						if (refreshedToken != null) {
    								return response.request.newBuilder()
    										.header("Authorization", "Bearer ${refreshedToken")
    										.build()
    						}
    				}
    				
    				return null
    		}
    }
    
    interface TokenProvider {
    		fun getToken(): String?
    		fun refreshToken(): String?
    		fun clearToken()
    }
    ```
    
- TokenProvider는 일반적으로 새로 고침 엔드포인트에 동기 네트워크 호출을 하여 토큰을 새로 고치는 역할을 하는 커스텀 클래스
- Authenticator를 사용하려면 OkHttpClient를 생성할 때 아래와 같은 설정 필요
    
    ```kotlin
    val okHttpClient = OkHttpClient.Builder()
    		.authenticator(TokenAuthenticator(tokenProvider))
    		.build()
    ```
    

### OkHttp Interceptor 사용하기

- Interceptor는 토큰 추가 및 갱신 로직을 처리할 수 있는 더 유연한 접근 방식
- Authenticator와 달리 Interceptor는 요청 또는 응답이 처리되기 전에 가로채서 수정할 수 있음
- 일반적인 구현에는 401 상태 코드에 대한 응답을 확인하고 토큰을 인라인으로 갱신하는 방법이 있음
    
    ```kotlin
    class TokenInterceptor(
    		private val tokenProvider: TokenProvider
    ): Interceptor {
    		override fun intercept(chain: Interceptor.Chain): Response {
    				val token = tokenProvider.getToken()
    				val originalRequest = chain.request()
    				val requestWithToken = if (token != null) {
    						originalRequest.newBuilder()
    								.header("Authorization", "Bearer $token")
    								.build()
    				} else {
    						originalRequest
    				}
    				
    				var response = chain.proceed(requestWithToken)
    				if (response.code == 401) {
    						synchronized(this) {
    								val newToken = tokenProvider.refreshToken()
    								if(newToken != null) {
    										val newRequest = requestWithToken.newBuilder()
    												.header("Authorization", "Bearer $newToken")
    												.build()
    										response.close()
    										response = chain.proceed(newRequest)
    								} else {
    									tokenProvider.clearToken()
    								}
    						}
    				}
    				return response
    		}
    }
    ```
    
- OkHttpClient를 생성할 때 아래와 같이 인터셉터를 설정해줘야 함
    
    ```kotlin
    val OkHttpClient = OkHttpClient.Builder()
    		.addInterceptor(TokenInterceptor(tokenProvider))
    		.build()
    ```
    

### Authenticator와 Interceptor의 주요 차이점

- 목적
    - Authenticator는 일반적으로 401 응답에 의해 트리거되는 인증 문제를 처리하도록 설계됨
    - Interceptor는 요청 및 응답 처리 모두에 대해 더 세분화된 제어를 처리할 수 있음
- 자동적으로 트리거
    - Authenticator는 401 응답에 대해 자동으로 호출됨
    - Interceptor는 특정 시나리오를 감지하고 처리하기 위한 수동 로직이 필요
- 사례
    - 간단한 인증 문제: Authenticator
    - 복잡한 요청/응답 처리: Interceptor