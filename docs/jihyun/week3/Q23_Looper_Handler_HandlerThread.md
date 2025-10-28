# Looper, Handler, HandlerThread의 역할은?

<aside>

Looper, Handler, HandlerThread

- 스레드를 관리하고 비동기 통신을 처리하기 위해 함께 작동하는 컴포넌트
- 백그라운드 스레드에서 작업을 수행하면서 UI 업데이트를 위해 메인 스레드와 상호작용하기 위한 필수적인 컴포넌트
</aside>

## Looper

- 스레드를 살아있게 유지하여 메시지 또는 작업 큐를 순차적으로 처리하는 안드로이드 스레딩 모델의 일부
- 안드로이드의 메인 스레드(UI 스레드) 및 다른 워커 스레드에서 중심적인 역할을 함
- 목적
    - 메시지 큐를 지속적으로 모니터링
    - 메시지나 작업을 적절한 핸들러에 가져와 디스패치
- 사용법
    - 메시지를 처리하는 모든 스레드에는 Looper가 필요
    - 메인 스레드에는 자동으로 Looper가 있지만, 워커 스레드의 경우 명시적으로 준비해야 함
- 초기화
    - Looper.prepare()를 사용하여 스레드에 Looper를 연결하고 Looper.loop()를 사용하여 루프를 시작

```kotlin
val thread = Thread {
		Looper.prepare() // 스레드에 Looper 연결
		val handler = Handler(Looper.myLooper()!!) // Looper를 사용하여 Handler 생성
		Looper.loop() // 메시지 루프 시작
}
thread.start()
```

## Handler

- 스레드의 메시지 큐 내에서 메시지나 작업을 보내고 처리
- Looper와 함께 작동
- 목적
    - 한 스레드에서 다른 스레드로 작업이나 메시지를 전달
    - ex. 백그라운드 스레드에서 UI 업데이트 하기
- 동작
    - Handler가 생성될 때, 생성된 스레드 및 해당 스레드의 Looper에 연결도미
    - Handler로 전송된 작업은 해당 스레드에서 처리됨

```kotlin
val handler = Handler(Looper.getMainLooper()) // 메인 스레드에서 실행됨

handler.post {
// UI 업데이트 코드
textView.text = "Updated from background thread"
}
```

## HandlerThread

- 내장된 Looper를 가진 특수한 Thread
- 작업 또는 메시지 큐를 처리할 수 있는 백그라운드 스레드를 생성하는 과정을 단순화
- 목적
    - 자체 Looper를 가진 워커 스레드를 생성하여 해당 스레드에서 작업을 순차적으로 처리할 수 있도록 함
- 생명주기
    - start()로 HandlerThread를 시작한 다음 getLooper()를 사용하여 Looper를 얻음
    - 리소스를 해제하려면 항상 quit() 또는 quitSafely()를 사용하여 Looper를 종료해야 함

```kotlin
val handlerThread = HandlerThread("WorkerThread")
handlerThread.start() // 스레드 시작

val workerHandler = Handler(handlerThread.looper) // 해당 Looper를 사용하여 작업 처리

workerHandler.post {
		// 백그라운드 작업 수행
		Thread.sleep(1000)
		Log.d("HandlerThread", "Task completed")
}

// 스레드 종료
handlerThread.quitSafely() // 처리 중인 메시지 완료 후 안전하게 종료
```

## 주요 차이점 및 관계

1. Looper: 메시지 처리의 중추. 스레드를 살아있게 유지하고 메시지 큐를 처리
2. Handler: Looper와 상호작용하여 메시지와 작업을 큐에 넣거나 처리
3. HandlerThread: 자동 Looper 설정으로 백그라운드 스레드 생성을 단순화

## 사용 사례

- Looper: 메인 스레드 또는 워커 스레드에서 연속적인 메시지 큐를 관리하는 데 사용
- Handler: 스레드 간 통신(ex. 백그라운드 스레드에서 UI 업데이트)에 이상적
- HandlerThread: 데이터 처리나 네트워크 요청과 같이 전용 스레드가 필요한 백그라운드 작업에 적합