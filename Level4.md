# 안드로이드 로깅 전략

안드로이드 개발을 하다 보면 “어디서, 왜 에러가 났는지” "어떤 에러가 많이 발생하였는지"를 빠르게 파악하는 것이 중요하다.  
이를 돕는 도구가 바로 **로깅(logging)**이다.  
로깅은 단순히 콘솔에 출력하는 수준을 넘어, 실제 운영 단계에서 앱 품질을 유지하는 핵심 도구다.  

이번 글에서는 안드로이드에서 사용할 수 있는 다양한 로깅 전략을 살펴본다.  
기본 Log 클래스부터 Timber, Crashlytics, 그리고 Webhook을 통한 실시간 알림까지 단계별로 정리해보자.  
  
  
## 안드로이드 여러 로깅 전략

### 기본 Log 클래스

안드로이드 SDK가 제공하는 가장 기본적인 로깅 도구는 android.util.Log다.
```kotlin
Log.d("MainActivity", "onCreate called")
Log.e("MainActivity", "Network error", exception)
```
이 방식은 간단하지만, 단점도 분명하다.
- 로그 태그(TAG)를 직접 지정해야 함
- 릴리즈 빌드에서 로그를 필터링하기 어려움
- 구조화된 로깅 불가능
  
따라서 프로젝트가 커질수록 유지보수가 어렵다.
그럴 때 등장하는 게 바로 Logger나 Timber 같은 고수준 로깅 라이브러리다.

### Logger

<img width="467" height="30" alt="스크린샷 2025-10-14 오후 4 15 39" src="https://github.com/user-attachments/assets/f814f235-2ede-46ab-98a6-510703796a44" />

Logger는 Log를 개선한 라이브러리로,
다양한 포맷과 JSON, XML 포매팅을 지원한다.

```kotlin
Logger.d("This is a debug log")
Logger.json("{\"user\":\"hyunseok\", \"age\":26}")
```
특징
- JSON, XML 자동 포매팅
- 로그 출력 포맷 커스터마이징
- 릴리즈 빌드에서 자동 로그 차단 가능
  
개발 중에는 보기 좋고, 가독성이 좋아 디버깅 효율을 높인다.

### Timber 

<img width="466" height="30" alt="스크린샷 2025-10-14 오후 4 12 30" src="https://github.com/user-attachments/assets/b212e08d-9ec5-4480-b6eb-490ebcbd8456" />

Timber는 Square에서 만든 경량 로깅 라이브러리이다.
```kotlin
class TuripDebugTree : Timber.DebugTree()
class TuripReleaseTree : Timber.Tree() 
```
```kotlin
Timber.d("User clicked button: %s", buttonId)
Timber.e(exception, "Network request failed")
```
특징
- Log 대비 보일러플레이트 코드 최소화
- 릴리즈 빌드에서 로그 자동 차단 가능
- 커스텀 트리(Tree)로 외부 서비스 연동 가능

```kotlin
if (BuildConfig.DEBUG) {
    Timber.plant(Timber.DebugTree())
} else {
    Timber.plant(TuripReleaseTree())
}
```
Timber는 Application 단에서 위와 같은 간단한 코드로 빌드를 구분하여 로그를 찍을 수 있다.
  
즉, 개발 단계에서는 Timber 하나로 대부분의 로깅 니즈를 해결할 수 있다.

### 로깅 레벨
로깅에는 여러 로깅레벨이 있다.
보통 다음과 같이 구분한다.

| **레벨(Level)** | **설명(Description)** | **예시(Example)** |
|------------------|------------------------|--------------------|
| **VERBOSE** | 모든 세부 로그 (내부 상태 확인용) | 내부 상태, 상세 디버깅 로그 |
| **DEBUG** | 디버깅용 로그 | 함수 호출, 변수 값 출력 |
| **INFO** | 일반적인 정보 | 사용자 액션, 앱 상태 변화 |
| **WARN** | 주의가 필요한 상황 | 일시적 네트워크 오류, 성능 저하 경고 |
| **ERROR** | 심각한 문제 | 예외 발생, 실패 이벤트 |
| **ASSERT** | 절대 발생해서는 안 되는 상황 | 치명적 버그 감지, 비논리적 오류 |

Timber나 Logger는 이 레벨들을 자동 지원하며,  
Firebase나 Webhook 연동 시에도 이 레벨을 활용해 필터링할 수 있다.

## Firebase를 통한 로깅 방법


### Firebase Analytics — 사용자 행동 기반 로깅

Firebase Analytics는 사용자의 행동을 중심으로 로그를 수집한다.
예를 들어, 어떤 버튼을 눌렀는지, 어떤 화면을 자주 방문하는지 등을 분석할 수 있다.
```kotlin
val bundle = Bundle().apply {
    putString("button_name", "purchase")
}
Firebase.analytics.logEvent("button_click", bundle)
```
활용 예시
- 사용자 전환율 분석
- 앱 내 주요 이벤트 추적
- UI/UX 개선 근거 데이터 확보
  
즉, Analytics는 “앱이 어떻게 사용되는가”를 로깅하는 데 초점을 둔다.

### Firebase Crashlytics — 크래시 및 ANR 추적

Crashlytics는 앱에서 발생한 **크래시, 비정상 종료(ANR)**를 자동으로 수집한다.
- 스택 트레이스와 기기 정보 자동 수집
- 발생 빈도, 사용자 영향도 분석
- ANR 및 자주 발생하는 오류 자동 추적
```kotlin
try {
    riskyFunction()
} catch (e: Exception) {
    Firebase.crashlytics.recordException(e)
}
```
ANR(Application Not Responding)이나 빈도가 높은 에러는 Crashlytics 대시보드에서 한눈에 확인할 수 있다.

### Timber + Crashlytics 연동 예시
```kotlin
class TuripReleaseTree : Timber.Tree() {
    override fun log(
        priority: Int,
        tag: String?,
        message: String,
        t: Throwable?,
    ) {
        if (priority >= Log.ERROR) {
            FirebaseCrashlytics.getInstance().apply {
                log(message)
                recordException(t ?: Exception(message))
                setCustomKey("log_priority", priority)
            }
        }
    }
}
```
위와 같은 방식으로 ERROR 이상 로그만 Crashlytics로 전송하도록 설정하면,  
운영 환경에서도 불필요한 로그 없이 핵심 오류만 관리할 수 있다.

| ![Crashlytics Dashboard 1](https://github.com/user-attachments/assets/80a39ff8-3e2f-4674-a1f9-165b2b6e1940) | ![Crashlytics Dashboard 2](https://github.com/user-attachments/assets/9463fce8-0cc9-4372-b090-86bab77c5b99) |
|:--:|:--:|
|  **Crash 로그 상세 화면** |  **로그 필터링 및 통계 화면** |

---
## Webhook을 통한 실시간 로깅 알림

Firebase가 수집한 로그는 대시보드에서만 확인할 수 있다.  
하지만 팀이 빠르게 대응하려면, Slack이나 Discord로 바로 알림을 보내는 Webhook을 활용하는 것이 좋다.

### Retrofit을 통한 Webhook 연동
```kotlin
interface WebhookService {
    @POST("") // 사용할 웹훅 주소
    suspend fun sendLog(@Body payload: Map<String, Any>)
}
```
에러 발생 시 JSON 형태로 로그를 전송하여 팀 채널에 즉시 표시할 수 있다.

<div style="border-left: 4px solid #facc15; padding: 12px; background: #fffbe6;">
  ⚠️ <strong>주의</strong><br>
  사용자 수가 많을 경우 로그 폭주로 인해 알림이 과도하게 전송될 수 있으므로,<br>
  Google Apps Script 기반의 중간 처리 방식을 권장한다.
</div>


### Google Apps Script를 통한 Webhook 연동
```kotlin
var webhookUrl = "" // 사용할 웹훅 주소
  var params = {
    method: "post",
    contentType: "application/json",
    payload: JSON.stringify(payload)
  };
  UrlFetchApp.fetch(webhookUrl, params);
```
> 구조
> 1. Gmail로 수신된 오류 메일을 Apps Script로 파싱
> 2. Slack/Discord Webhook으로 전송
> 3. 팀은 별도 앱 실행 없이 실시간 에러 확인 가능

<img width="400" height="400" alt="스크린샷 2025-10-14 오후 4 31 05" src="https://github.com/user-attachments/assets/11819e2b-9c58-49c7-9d6d-8e078d427a72" />

또한, Apps Script는 위와 같은 트리거를 설정하여 일정 시간동안 받은 데이터를 한꺼번에 취합해서 메시지를 전송받도록 할 수 있다.


## 전체 흐름

| Crashlytics 감지 | 이메일 전송 | Discord Webhook 알림 |
|:--:|:--:|:--:|
| <img src="https://github.com/user-attachments/assets/f46aed86-e564-4880-99ab-3f2a36c9b18c" width="300" /> | <img src="https://github.com/user-attachments/assets/07d0ecfc-126e-49a5-a61b-aa089e7fccb0" width="200" /> | <img src="https://github.com/user-attachments/assets/1b0f1328-a154-4e78-86a8-08b7eb9cb8a9" width="180" /> |

>  **설명**  
> 1. **Crashlytics**가 앱에서 발생한 예외를 감지한다.  
> 2. **Google Apps Script**를 통해 이메일로 에러 리포트를 수신한다.  
> 3. 이메일 내용을 분석하여 **Discord Webhook**으로 실시간 전송한다.  

## 마치며

로깅은 단순한 **디버깅 도구**가 아니다.  
앱의 생명주기 전반에 걸쳐 **운영**과 **유지보수**의 품질을 좌우하는 핵심 요소다.  
효율적인 로깅 전략을 세워두면, 문제를 **미리 감지하고**, **즉시 대응하며**, **지속적으로 개선**할 수 있다.

> 개발 단계에서는 Timber,
운영 단계에서는 Crashlytics + Webhook 조합으로
안정적이고 효율적인 로깅 체계를 구축하자.
