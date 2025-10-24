# HTTP 클라이언트

[← 목차로 돌아가기](README.md)

---

## Http 클래스 개요

Http 클래스는 외부 API를 호출하거나 HTTP 요청을 보내기 위한 클라이언트 클래스입니다.

### 주요 기능

- GET, POST, PUT, DELETE 요청
- 헤더 설정
- 파라미터 전송
- JSON/XML 데이터 전송
- SSL 인증서 검증 비활성화
- 타임아웃 설정
- 비동기 요청 지원

---

## 기본 사용법

### GET 요청

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

Http http = new Http("https://api.example.com/users");
http.setParam("page", "1");
http.setParam("limit", "10");

String response = http.send("GET");
m.p(response);

%>
```

### POST 요청

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

Http http = new Http("https://api.example.com/users");
http.setParam("name", "홍길동");
http.setParam("email", "hong@example.com");
http.setParam("age", "30");

String response = http.send("POST");
m.p(response);

%>
```

---

## 헤더 설정

### 커스텀 헤더 추가

```jsp
Http http = new Http("https://api.example.com/data");

// Authorization 헤더
http.setHeader("Authorization", "Bearer " + apiToken);

// Content-Type 헤더
http.setHeader("Content-Type", "application/json");

// 커스텀 헤더
http.setHeader("X-API-Key", "your_api_key");
http.setHeader("X-Request-ID", Malgn.uuid());

String response = http.send("GET");
```

---

## JSON 데이터 전송

### JSON 요청

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

Http http = new Http("https://api.example.com/users");

// JSON 데이터 생성
JSONObject json = new JSONObject();
json.put("name", "홍길동");
json.put("email", "hong@example.com");
json.put("age", 30);

// JSON 데이터 설정 (자동으로 Content-Type: application/json 설정됨)
http.setData(json.toString());

String response = http.send("POST");
m.p(response);

%>
```

### JSON 응답 파싱

```jsp
Http http = new Http("https://api.example.com/users/123");
String response = http.send("GET");

// JSON 파싱
JSONObject json = new JSONObject(response);
String name = json.getString("name");
String email = json.getString("email");
int age = json.getInt("age");

m.p("이름: " + name);
m.p("이메일: " + email);
m.p("나이: " + age);
```

---

## XML 데이터 전송

```jsp
Http http = new Http("https://api.example.com/data");

String xmlData = "<?xml version=\"1.0\" encoding=\"UTF-8\"?>" +
                "<user>" +
                "  <name>홍길동</name>" +
                "  <email>hong@example.com</email>" +
                "</user>";

// XML 데이터 설정 (자동으로 Content-Type: application/xml 설정됨)
http.setData(xmlData);

String response = http.send("POST");
```

---

## 타임아웃 설정

```jsp
Http http = new Http("https://api.example.com/slow-endpoint");

// 타임아웃 설정 (초 단위)
http.setTimeout(30);  // 30초

try {
    String response = http.send("GET");
    m.p(response);
} catch(Exception e) {
    m.p("요청 시간 초과");
}
```

---

## SSL 인증서 검증

### SSL 검증 비활성화 (개발 환경)

```jsp
Http http = new Http("https://self-signed-cert.example.com");

// SSL 인증서 검증 비활성화
http.disableSSLVerification();

String response = http.send("GET");

// 검증 복원
http.restoreSSLVerification();
```

**주의**: 운영 환경에서는 SSL 검증을 비활성화하지 마세요!

---

## 응답 코드 확인

```jsp
Http http = new Http("https://api.example.com/users");
String response = http.send("GET");

int statusCode = http.getResponseCode();

if(statusCode == 200) {
    m.p("성공: " + response);
} else if(statusCode == 404) {
    m.p("리소스를 찾을 수 없습니다");
} else if(statusCode >= 500) {
    m.p("서버 오류");
} else {
    m.p("오류 코드: " + statusCode);
}
```

---

## 비동기 요청

```jsp
Http http = new Http("https://api.example.com/process");
http.setParam("task", "heavy_task");

// 비동기 요청 (백그라운드에서 실행)
http.send("POST", new HttpListener() {
    @Override
    public void execute(String result) {
        // 요청 완료 후 실행
        Malgn.logger("http", "완료: " + result);
    }
});

// 즉시 다음 코드 실행
m.p("요청이 백그라운드에서 실행 중입니다");
```

---

## 실전 예제

### 1. REST API 호출

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// GitHub API 호출
Http http = new Http("https://api.github.com/users/octocat");
http.setHeader("User-Agent", "MalgnFramework");

String response = http.send("GET");

JSONObject user = new JSONObject(response);
m.p("사용자명: " + user.getString("login"));
m.p("이름: " + user.getString("name"));
m.p("팔로워: " + user.getInt("followers"));

%>
```

### 2. 날씨 API 호출

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

String apiKey = Config.get("weatherApiKey");
String city = "Seoul";

Http http = new Http("https://api.openweathermap.org/data/2.5/weather");
http.setParam("q", city);
http.setParam("appid", apiKey);
http.setParam("units", "metric");

String response = http.send("GET");

JSONObject weather = new JSONObject(response);
double temp = weather.getJSONObject("main").getDouble("temp");
String description = weather.getJSONArray("weather").getJSONObject(0).getString("description");

m.p("서울 날씨: " + description);
m.p("온도: " + temp + "°C");

%>
```

### 3. 슬랙(Slack) 웹훅

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

String webhookUrl = Config.get("slackWebhookUrl");

JSONObject payload = new JSONObject();
payload.put("text", "새로운 주문이 접수되었습니다!");
payload.put("username", "알림봇");
payload.put("icon_emoji", ":bell:");

Http http = new Http(webhookUrl);
http.setData(payload.toString());

String response = http.send("POST");
m.p("슬랙 전송: " + response);

%>
```

### 4. 결제 API 연동

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

String apiUrl = "https://api.payment.com/v1/payments";
String apiKey = Config.get("paymentApiKey");

// 결제 요청 데이터
JSONObject payment = new JSONObject();
payment.put("amount", 50000);
payment.put("currency", "KRW");
payment.put("order_id", "ORD" + m.time());
payment.put("customer_name", "홍길동");
payment.put("customer_email", "hong@example.com");

Http http = new Http(apiUrl);
http.setHeader("Authorization", "Bearer " + apiKey);
http.setData(payment.toString());

String response = http.send("POST");

JSONObject result = new JSONObject(response);
if("success".equals(result.getString("status"))) {
    String paymentId = result.getString("payment_id");
    m.p("결제 성공: " + paymentId);
} else {
    m.p("결제 실패: " + result.getString("message"));
}

%>
```

### 5. 파일 다운로드

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

Http http = new Http("https://example.com/files/document.pdf");
String content = http.send("GET");

// 파일로 저장
String filePath = Config.getDataDir() + "/downloaded_document.pdf";
Malgn.writeFile(filePath, content);

m.p("파일 다운로드 완료: " + filePath);

%>
```

### 6. POST with Form Data

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

Http http = new Http("https://api.example.com/form");
http.setHeader("Content-Type", "application/x-www-form-urlencoded");
http.setParam("username", "hong");
http.setParam("password", "secret123");
http.setParam("remember", "true");

String response = http.send("POST");
m.p(response);

%>
```

### 7. PUT 요청 (업데이트)

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

int userId = 123;

JSONObject updateData = new JSONObject();
updateData.put("name", "김철수");
updateData.put("email", "kim@example.com");

Http http = new Http("https://api.example.com/users/" + userId);
http.setHeader("Authorization", "Bearer " + apiToken);
http.setData(updateData.toString());

String response = http.send("PUT");
m.p("업데이트 완료: " + response);

%>
```

### 8. DELETE 요청

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

int userId = 123;

Http http = new Http("https://api.example.com/users/" + userId);
http.setHeader("Authorization", "Bearer " + apiToken);

String response = http.send("DELETE");

if(http.getResponseCode() == 204) {
    m.p("삭제 성공");
} else {
    m.p("삭제 실패: " + response);
}

%>
```

---

## 디버깅

```jsp
Http http = new Http("https://api.example.com/data");
http.setDebug(out);  // 디버그 정보 출력

http.setHeader("X-API-Key", "test123");
http.setParam("query", "search_term");

String response = http.send("GET");
```

**디버그 출력 내용**:
- 요청 URL
- 요청 헤더
- 요청 파라미터
- 응답 코드
- 응답 내용

---

## 주요 메소드 정리

| 메소드 | 설명 |
|--------|------|
| Http(url) | HTTP 클라이언트 생성 |
| setUrl(url) | URL 설정 |
| setHeader(key, value) | 헤더 추가 |
| setParam(name, value) | 파라미터 추가 |
| setData(data) | Body 데이터 설정 (JSON/XML) |
| setTimeout(seconds) | 타임아웃 설정 (초) |
| send() | GET 요청 실행 |
| send(method) | 지정한 메소드로 요청 실행 |
| send(method, listener) | 비동기 요청 실행 |
| getResponseCode() | HTTP 응답 코드 |
| disableSSLVerification() | SSL 검증 비활성화 |
| restoreSSLVerification() | SSL 검증 복원 |
| setDebug(out) | 디버깅 모드 |

---

## 베스트 프랙티스

### 1. API 키는 설정 파일에

```jsp
// Bad
String apiKey = "sk-1234567890abcdef";

// Good
String apiKey = Config.get("apiKey");
```

### 2. 에러 처리

```jsp
try {
    Http http = new Http(apiUrl);
    http.setTimeout(10);
    String response = http.send("GET");

    int statusCode = http.getResponseCode();
    if(statusCode >= 400) {
        throw new Exception("API 오류: " + statusCode);
    }

    // 처리...

} catch(Exception e) {
    Malgn.errorLog("API 호출 실패", e);
    m.jsError("일시적인 오류가 발생했습니다");
}
```

### 3. 타임아웃 설정

```jsp
// 외부 API는 항상 타임아웃 설정
Http http = new Http(externalApi);
http.setTimeout(30);  // 30초
```

### 4. 응답 캐싱

```jsp
Cache cache = new Cache();
String cacheKey = "api_data_" + param;
String response = cache.getString(cacheKey);

if(response == null) {
    Http http = new Http(apiUrl);
    response = http.send("GET");
    cache.save(cacheKey, response);
}
```

---

[← 목차로 돌아가기](README.md)
