# OpenAI 통합

[← 목차로 돌아가기](README.md)

---

## 개요

맑은프레임워크의 OpenAI 클래스는 OpenAI API(ChatGPT)를 쉽게 통합하여 AI 기능을 웹 애플리케이션에 추가할 수 있게 합니다.

### 주요 특징

- **JSON 문자열 기반 API**: 클라이언트에서 전체 대화 내역 전달
- **자동 히스토리 관리**: 서버 메모리 기반 대화 관리
- **스트림 지원**: 실시간 응답 출력
- **포맷 검증**: messages 배열 자동 검증
- **유연한 구조**: REST API와 세션 기반 모두 지원

---

## 목차

1. [기본 설정](#기본-설정)
2. [클라이언트 제어 방식](#클라이언트-제어-방식)
3. [서버 자동 관리 방식](#서버-자동-관리-방식)
4. [스트림 방식](#스트림-방식)
5. [히스토리 관리](#히스토리-관리)
6. [고급 설정](#고급-설정)
7. [실제 활용 예제](#실제-활용-예제)

---

## 기본 설정

### OpenAI 객체 생성

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

OpenAI ai = new OpenAI();

// API 키 설정 (필수)
ai.apiKey("sk-your-api-key-here");

// 모델 선택 (기본값: gpt-4o-mini)
ai.modelName("gpt-4o-mini");

// 또는 다른 모델
ai.modelName("gpt-4");
ai.modelName("gpt-3.5-turbo");

%>
```

### 환경설정 파일 사용

**config.xml**:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<config>
    <openaiApiKey>sk-your-api-key-here</openaiApiKey>
</config>
```

**사용:**
```jsp
OpenAI ai = new OpenAI();
ai.apiKey(Config.get("openaiApiKey"));
```

---

## 클라이언트 제어 방식

클라이언트에서 전체 messages 배열을 JSON 문자열로 전달하는 현대적인 방식입니다. REST API에 적합합니다.

### 1. 기본 사용법

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 클라이언트에서 messages JSON 받기
String messagesJson = m.rs("messages");

OpenAI ai = new OpenAI();
ai.apiKey(Config.get("openaiApiKey"));
ai.modelName("gpt-4o-mini");

// AI 호출
String response = ai.chat(messagesJson);

// 에러 체크
if(ai.errMsg != null) {
    j.error(ai.errMsg);
} else {
    j.success(response);
}

%>
```

### 2. 클라이언트 예제 (JavaScript)

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>AI Chat</title>
</head>
<body>
    <div id="chat"></div>
    <input type="text" id="message" placeholder="메시지를 입력하세요">
    <button onclick="sendMessage()">전송</button>

    <script>
    let messages = [];

    async function sendMessage() {
        const userMessage = document.getElementById('message').value;

        // 사용자 메시지 추가
        messages.push({
            role: "user",
            content: userMessage
        });

        // 서버로 전송
        const response = await fetch('/api/chat.jsp', {
            method: 'POST',
            headers: {'Content-Type': 'application/x-www-form-urlencoded'},
            body: 'messages=' + encodeURIComponent(JSON.stringify(messages))
        });

        const result = await response.json();

        if(result.success) {
            // AI 응답 추가
            messages.push({
                role: "assistant",
                content: result.data
            });

            // 화면에 표시
            displayMessage('user', userMessage);
            displayMessage('assistant', result.data);
        }

        document.getElementById('message').value = '';
    }

    function displayMessage(role, content) {
        const chatDiv = document.getElementById('chat');
        const msgDiv = document.createElement('div');
        msgDiv.className = role;
        msgDiv.textContent = content;
        chatDiv.appendChild(msgDiv);
    }
    </script>
</body>
</html>
```

### 3. 시스템 프롬프트 포함

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="/init.jsp" %><%

String messagesJson = m.rs("messages");

// messages에 system 메시지가 없으면 추가
JSONArray messages = new JSONArray(messagesJson);
if(messages.length() == 0 || !messages.getJSONObject(0).getString("role").equals("system")) {
    JSONArray newMessages = new JSONArray();
    newMessages.put(new JSONObject()
        .put("role", "system")
        .put("content", "당신은 친절한 AI 도우미입니다.")
    );
    for(int i = 0; i < messages.length(); i++) {
        newMessages.put(messages.getJSONObject(i));
    }
    messagesJson = newMessages.toString();
}

OpenAI ai = new OpenAI();
ai.apiKey(Config.get("openaiApiKey"));
String response = ai.chat(messagesJson);

j.success(response);

%>
```

---

## 서버 자동 관리 방식

세션에 대화 히스토리를 JSON 문자열로 저장하고 관리하는 간편한 방식입니다.

### 1. 세션 기반 챗봇 (권장)

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="/init.jsp" %><%

// OpenAI 객체 생성 (매번 새로 생성)
OpenAI ai = new OpenAI();
ai.apiKey(Config.get("openaiApiKey"));
ai.modelName("gpt-4o-mini");

// 세션에서 히스토리 로드
String historyJson = (String)session.getAttribute("chatHistory");
if(historyJson != null) {
    ai.setHistory(historyJson);
}

// 사용자 메시지
String message = m.rs("message");

// 자동으로 history에 추가되고 AI 호출
String response = ai.chatMemory(message);

// 세션에 히스토리 저장
session.setAttribute("chatHistory", ai.getHistory());

if(ai.errMsg != null) {
    j.error(ai.errMsg);
} else {
    j.success(response);
}

%>
```

### 2. 시스템 프롬프트와 함께 사용

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="/init.jsp" %><%

OpenAI ai = new OpenAI();
ai.apiKey(Config.get("openaiApiKey"));
ai.modelName("gpt-4o-mini");

// 세션에서 히스토리 로드
String historyJson = (String)session.getAttribute("chatHistory");
if(historyJson != null) {
    ai.setHistory(historyJson);
} else {
    // 첫 대화일 경우 시스템 메시지 설정
    ai.system("당신은 맑은프레임워크 전문가입니다. JSP와 Java에 대해 답변하세요.");
}

String message = m.rs("message");
String response = ai.chatMemory(message);

// 세션에 히스토리 저장
session.setAttribute("chatHistory", ai.getHistory());

j.success(response);

%>
```

### 3. 대화 히스토리 초기화

```jsp
// 세션에서 히스토리 제거
session.removeAttribute("chatHistory");

j.success("대화가 초기화되었습니다");
```

### 4. 히스토리 조회

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="/init.jsp" %><%

String historyJson = (String)session.getAttribute("chatHistory");

if(historyJson != null) {
    j.success(historyJson);
} else {
    j.success("[]");
}

%>
```

---

## 스트림 방식

실시간으로 AI 응답을 받아 출력하는 방식입니다. 긴 응답을 받을 때 유용합니다.

### 1. 기본 스트림

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

String messagesJson = m.rs("messages");

OpenAI ai = new OpenAI();
ai.apiKey(Config.get("openaiApiKey"));

// 실시간으로 out에 출력하면서 fullResponse에도 저장
String fullResponse = ai.streamChat(messagesJson, out);

%>
```

### 2. 서버 자동 관리 + 스트림

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

OpenAI ai = new OpenAI();
ai.apiKey(Config.get("openaiApiKey"));

// 세션에서 히스토리 로드
String historyJson = (String)session.getAttribute("chatHistory");
if(historyJson != null) {
    ai.setHistory(historyJson);
}

String message = m.rs("message");

// 실시간으로 출력하면서 history에도 자동 저장
String response = ai.streamMemory(message, out);

// 세션에 히스토리 저장
session.setAttribute("chatHistory", ai.getHistory());

%>
```

### 3. SSE (Server-Sent Events) 방식

```jsp
<%@ page contentType="text/event-stream; charset=utf-8" %><%
    response.setHeader("Cache-Control", "no-cache");
    response.setHeader("Connection", "keep-alive");
%><%@ include file="/init.jsp" %><%

String messagesJson = m.rs("messages");

OpenAI ai = new OpenAI();
ai.apiKey(Config.get("openaiApiKey"));

// 커스텀 Writer로 SSE 포맷 출력
Writer sseWriter = new Writer() {
    public void write(char[] cbuf, int off, int len) throws IOException {
        String chunk = new String(cbuf, off, len);
        out.write("data: " + chunk + "\n\n");
        out.flush();
    }
    public void flush() throws IOException { out.flush(); }
    public void close() throws IOException {}
};

String fullResponse = ai.streamChat(messagesJson, sseWriter);

out.write("data: [DONE]\n\n");
out.flush();

%>
```

---

## 히스토리 관리

대화 히스토리를 DB에 저장하고 불러오는 방법입니다.

### 1. DB에 히스토리 저장

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="/init.jsp" %><%

int userId = m.getInt("user_id");
String message = m.rs("message");

OpenAI ai = new OpenAI();
ai.apiKey(Config.get("openaiApiKey"));

// DB에서 히스토리 로드
ChatHistoryDao dao = new ChatHistoryDao();
DataSet ds = dao.find("user_id = ?", userId);
if(ds.next()) {
    String historyJson = ds.s("history");
    ai.setHistory(historyJson);
}

// 메시지 추가 및 AI 호출
String response = ai.chatMemory(message);

// DB에 히스토리 저장
String updatedHistory = ai.getHistory();
if(ds.getRow() > 0) {
    dao.update("history = ?", updatedHistory, "user_id = ?", userId);
} else {
    DataMap data = new DataMap();
    data.put("user_id", userId);
    data.put("history", updatedHistory);
    data.put("reg_date", m.time());
    dao.insert(data);
}

j.success(response);

%>
```

### 2. 히스토리 조회

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="/init.jsp" %><%

int userId = m.getInt("user_id");

ChatHistoryDao dao = new ChatHistoryDao();
DataSet ds = dao.find("user_id = ?", userId);

if(ds.next()) {
    String historyJson = ds.s("history");
    JSONArray history = new JSONArray(historyJson);

    // JSON으로 반환
    j.success(history.toString());
} else {
    j.success("[]");
}

%>
```

### 3. 히스토리 초기화

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="/init.jsp" %><%

int userId = m.getInt("user_id");

ChatHistoryDao dao = new ChatHistoryDao();
dao.delete("user_id = ?", userId);

j.success("대화 히스토리가 초기화되었습니다");

%>
```

---

## 고급 설정

### 1. Temperature 조절

Temperature는 응답의 무작위성을 제어합니다 (0.0 ~ 2.0):

```jsp
OpenAI ai = new OpenAI();
ai.apiKey(Config.get("openaiApiKey"));

// 낮은 temperature (0.0 ~ 0.3): 일관되고 예측 가능한 응답
ai.temperature(0.2);
String factual = ai.chat("[{\"role\":\"user\",\"content\":\"물의 끓는점은?\"}]");
// "100°C입니다" (매번 동일한 답변)

// 중간 temperature (0.7): 균형잡힌 응답 (기본값)
ai.temperature(0.7);

// 높은 temperature (1.0 ~ 2.0): 창의적이고 다양한 응답
ai.temperature(1.5);
String creative = ai.chat("[{\"role\":\"user\",\"content\":\"우주에 대한 시를 써줘\"}]");
// 매번 다른 창의적인 시
```

### 2. 디버그 모드

```jsp
OpenAI ai = new OpenAI();
ai.apiKey(Config.get("openaiApiKey"));

// HTTP 요청/응답을 out으로 출력
ai.setDebug(out);

// 또는 로그 파일로 출력
ai.setDebug();

String response = ai.chat(messagesJson);

// 에러 메시지 확인
if(ai.errMsg != null) {
    m.p("Error: " + ai.errMsg);
}
```

### 3. 시스템 메시지 설정

`system()` 메소드로 히스토리의 첫 번째에 시스템 메시지를 추가하거나 교체할 수 있습니다:

```jsp
OpenAI ai = new OpenAI();
ai.apiKey(Config.get("openaiApiKey"));

// 세션에서 히스토리 로드
String historyJson = (String)session.getAttribute("chatHistory");
if(historyJson != null) {
    ai.setHistory(historyJson);
}

// 시스템 메시지 설정 (이미 있으면 교체, 없으면 추가)
ai.system("당신은 친절한 AI 도우미입니다.");

String response = ai.chatMemory(message);
session.setAttribute("chatHistory", ai.getHistory());
```

**동적 시스템 메시지:**

```jsp
OpenAI ai = new OpenAI();
ai.apiKey(Config.get("openaiApiKey"));

String historyJson = (String)session.getAttribute("chatHistory");
if(historyJson != null) {
    ai.setHistory(historyJson);
}

// 모드에 따라 시스템 메시지 변경
String mode = m.rs("mode", "normal");
if(mode.equals("formal")) {
    ai.system("당신은 전문적이고 공손한 비즈니스 어시스턴트입니다.");
} else if(mode.equals("casual")) {
    ai.system("당신은 친근하고 편안한 친구같은 어시스턴트입니다.");
} else {
    ai.system("당신은 도움이 되는 AI 어시스턴트입니다.");
}

String response = ai.chatMemory(message);
session.setAttribute("chatHistory", ai.getHistory());
```

### 4. 최대 토큰 수 설정

응답의 최대 토큰 수를 제한할 수 있습니다 (기본값: 1000):

```jsp
OpenAI ai = new OpenAI();
ai.apiKey(Config.get("openaiApiKey"));

// 짧은 응답 (비용 절약)
ai.maxToken(100);

// 긴 응답
ai.maxToken(2000);

// 매우 긴 응답 (최대 4096)
ai.maxToken(4096);

String response = ai.chat(messagesJson);
```

### 5. 모델별 특징

```jsp
// gpt-3.5-turbo: 빠르고 저렴, 일반 작업에 적합
ai.modelName("gpt-3.5-turbo");

// gpt-4o-mini: 균형잡힌 성능, 대부분의 작업에 권장 (기본값)
ai.modelName("gpt-4o-mini");

// gpt-4: 가장 강력, 복잡한 추론 작업에 적합 (비쌈)
ai.modelName("gpt-4");

// gpt-4-turbo: gpt-4보다 빠르고 저렴
ai.modelName("gpt-4-turbo");
```

---

## 실제 활용 예제

### 1. 문서 요약 API

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="/init.jsp" %><%

String content = m.rs("content");

JSONArray messages = new JSONArray();
messages.put(new JSONObject()
    .put("role", "system")
    .put("content", "당신은 전문 요약가입니다. 핵심 내용만 3-5문장으로 요약하세요.")
);
messages.put(new JSONObject()
    .put("role", "user")
    .put("content", "다음 글을 요약해주세요:\n\n" + content)
);

OpenAI ai = new OpenAI();
ai.apiKey(Config.get("openaiApiKey"));
ai.temperature(0.3); // 일관된 요약

String summary = ai.chat(messages.toString());

j.success(new DataMap().put("summary", summary));

%>
```

### 2. 감정 분석

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="/init.jsp" %><%

String review = m.rs("review");

JSONArray messages = new JSONArray();
messages.put(new JSONObject()
    .put("role", "system")
    .put("content", "사용자 리뷰의 감정을 분석하세요. positive, negative, neutral 중 하나로만 답하세요.")
);
messages.put(new JSONObject()
    .put("role", "user")
    .put("content", review)
);

OpenAI ai = new OpenAI();
ai.apiKey(Config.get("openaiApiKey"));
ai.temperature(0.1); // 일관된 분류

String sentiment = ai.chat(messages.toString());

j.success(new DataMap()
    .put("review", review)
    .put("sentiment", sentiment.trim())
);

%>
```

### 3. JSON 데이터 생성

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="/init.jsp" %><%

JSONArray messages = new JSONArray();
messages.put(new JSONObject()
    .put("role", "system")
    .put("content", "모든 응답을 JSON 배열 형식으로만 제공하세요.")
);
messages.put(new JSONObject()
    .put("role", "user")
    .put("content",
        "AI 관련 객관식 문제 5개를 만들어주세요.\n\n" +
        "output-format: [{\"question\":\"질문\", \"choices\":[\"1\",\"2\",\"3\",\"4\"], \"answer\":1, \"explanation\":\"설명\"}]"
    )
);

OpenAI ai = new OpenAI();
ai.apiKey(Config.get("openaiApiKey"));
ai.temperature(0.8);

String response = ai.chat(messages.toString());

// JSON 파싱하여 DB에 저장
try {
    JSONArray questions = new JSONArray(response);
    QuizDao dao = new QuizDao();

    for(int i = 0; i < questions.length(); i++) {
        JSONObject q = questions.getJSONObject(i);
        DataMap data = new DataMap();
        data.put("question", q.getString("question"));
        data.put("choices", q.getJSONArray("choices").toString());
        data.put("answer", q.getInt("answer"));
        data.put("explanation", q.getString("explanation"));
        dao.insert(data);
    }

    j.success(questions.length() + "개의 문제가 생성되었습니다");

} catch(Exception e) {
    j.error("JSON 파싱 오류: " + e.getMessage());
}

%>
```

### 4. 번역 서비스

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="/init.jsp" %><%

String text = m.rs("text");
String sourceLang = m.rs("source", "한국어");
String targetLang = m.rs("target", "영어");

JSONArray messages = new JSONArray();
messages.put(new JSONObject()
    .put("role", "system")
    .put("content", "당신은 전문 번역가입니다. " + sourceLang + "를 " + targetLang + "로 정확하게 번역하세요.")
);
messages.put(new JSONObject()
    .put("role", "user")
    .put("content", text)
);

OpenAI ai = new OpenAI();
ai.apiKey(Config.get("openaiApiKey"));
ai.temperature(0.3);

String translation = ai.chat(messages.toString());

j.success(new DataMap()
    .put("original", text)
    .put("translation", translation)
);

%>
```

### 5. 코드 리뷰

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="/init.jsp" %><%

String code = m.rs("code");
String language = m.rs("language", "Java");

JSONArray messages = new JSONArray();
messages.put(new JSONObject()
    .put("role", "system")
    .put("content",
        "당신은 숙련된 " + language + " 프로그래머입니다. " +
        "코드를 분석하고 개선 사항을 제안하세요. " +
        "다음 형식으로 답변하세요:\n" +
        "1. 문제점\n2. 개선 제안\n3. 개선된 코드"
    )
);
messages.put(new JSONObject()
    .put("role", "user")
    .put("content", "다음 코드를 리뷰해주세요:\n\n```" + language + "\n" + code + "\n```")
);

OpenAI ai = new OpenAI();
ai.apiKey(Config.get("openaiApiKey"));
ai.modelName("gpt-4o-mini");
ai.temperature(0.5);

String review = ai.chat(messages.toString());

j.success(new DataMap().put("review", review));

%>
```

### 6. 콘텐츠 생성기

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

String topic = m.rs("topic");
String tone = m.rs("tone", "전문적");
String length = m.rs("length", "500자");

JSONArray messages = new JSONArray();
messages.put(new JSONObject()
    .put("role", "system")
    .put("content", "당신은 블로그 작가입니다. " + tone + "인 톤으로 글을 작성하세요.")
);
messages.put(new JSONObject()
    .put("role", "user")
    .put("content", topic + "에 대한 블로그 글을 " + length + " 내외로 작성해주세요.")
);

OpenAI ai = new OpenAI();
ai.apiKey(Config.get("openaiApiKey"));
ai.modelName("gpt-4o-mini");
ai.temperature(0.8);

String article = ai.chat(messages.toString());

p.setBody("blog.article");
p.setVar("topic", topic);
p.setVar("content", article);
p.display();

%>
```

---

## 에러 처리

### 1. 기본 에러 처리

```jsp
OpenAI ai = new OpenAI();
ai.apiKey(Config.get("openaiApiKey"));

String response = ai.chat(messagesJson);

// 에러 체크
if(ai.errMsg != null) {
    m.p("Error: " + ai.errMsg);
    Malgn.errorLog("OpenAI API 에러: " + ai.errMsg);
} else if(response.isEmpty()) {
    m.p("빈 응답이 반환되었습니다");
} else {
    m.p(response);
}
```

### 2. 검증 에러 처리

```jsp
String messagesJson = m.rs("messages");

OpenAI ai = new OpenAI();
ai.apiKey(Config.get("openaiApiKey"));

String response = ai.chat(messagesJson);

if(ai.errMsg != null) {
    if(ai.errMsg.contains("Invalid message format")) {
        j.error("메시지 형식이 올바르지 않습니다");
    } else if(ai.errMsg.contains("Invalid role")) {
        j.error("role 필드가 올바르지 않습니다");
    } else {
        j.error("API 오류: " + ai.errMsg);
    }
} else {
    j.success(response);
}
```

### 3. Try-Catch 패턴

```jsp
try {
    OpenAI ai = new OpenAI();
    ai.apiKey(Config.get("openaiApiKey"));

    String response = ai.chat(messagesJson);

    if(ai.errMsg != null) {
        throw new Exception(ai.errMsg);
    }

    j.success(response);

} catch(Exception e) {
    Malgn.errorLog("OpenAI API 에러", e);
    j.error("AI 서비스 오류: " + e.getMessage());
}
```

---

## 비용 최적화 팁

### 1. 적절한 모델 선택

```jsp
// 간단한 작업: gpt-3.5-turbo (저렴, 빠름)
ai.modelName("gpt-3.5-turbo");
// 예: 단순 질문 답변, 키워드 추출

// 일반 작업: gpt-4o-mini (균형) - 권장
ai.modelName("gpt-4o-mini");
// 예: 대부분의 챗봇, 요약, 번역

// 복잡한 작업: gpt-4 (비쌈, 정확)
ai.modelName("gpt-4");
// 예: 복잡한 추론, 코드 생성, 전문 지식
```

### 2. 프롬프트 최적화

```jsp
// 비효율적: 장황한 프롬프트
String bad = "[{\"role\":\"user\",\"content\":\"안녕하세요. 저는 학생입니다. 제가 궁금한게 있는데요, 혹시 물의 끓는점이 몇 도인지 알려주실 수 있나요?\"}]";

// 효율적: 간결한 프롬프트
String good = "[{\"role\":\"user\",\"content\":\"물의 끓는점은?\"}]";
```

### 3. 응답 길이 제한

```jsp
JSONArray messages = new JSONArray();
messages.put(new JSONObject()
    .put("role", "system")
    .put("content", "답변은 50자 이내로 간결하게 작성하세요.")
);
messages.put(new JSONObject()
    .put("role", "user")
    .put("content", "AI란 무엇인가요?")
);
```

### 4. 캐싱 활용

```jsp
Cache cache = new Cache();
String cacheKey = "ai_faq_" + Malgn.md5(question);
String answer = cache.getString(cacheKey);

if(answer == null) {
    // 캐시에 없으면 AI에게 질문
    OpenAI ai = new OpenAI();
    ai.apiKey(Config.get("openaiApiKey"));
    answer = ai.chat(messagesJson);

    // 캐시에 저장 (1시간)
    cache.save(cacheKey, answer, 3600);
}

j.success(answer);
```

---

## 주의사항

### 1. API 키 보안

```jsp
// ❌ 나쁜 예: 소스코드에 직접 입력
ai.apiKey("sk-your-api-key-here");

// ✅ 좋은 예: 환경설정 파일 사용
ai.apiKey(Config.get("openaiApiKey"));
```

### 2. 비용 관리

- OpenAI API는 사용량에 따라 과금됩니다
- 모델별 가격이 다릅니다 (gpt-4 > gpt-4o-mini > gpt-3.5-turbo)
- 토큰 수가 많을수록 비용이 높아집니다

### 3. 응답 시간

```jsp
// AI 응답은 수 초가 걸릴 수 있으므로 로딩 UI 필요
// 긴 응답은 스트림 방식 사용 권장
String response = ai.streamChat(messagesJson, out);
```

### 4. 콘텐츠 검증

```jsp
// 생성된 콘텐츠는 항상 검토 필요
String content = ai.chat(messagesJson);

// 부적절한 내용 필터링
if(content.contains("부적절한단어")) {
    content = "적절하지 않은 응답이 생성되었습니다";
}
```

### 5. 개인정보 보호

```jsp
// ❌ 나쁜 예: 민감한 정보 포함
String msg = "[{\"role\":\"user\",\"content\":\"내 주민번호는 123456-1234567이야\"}]";

// ✅ 좋은 예: 민감한 정보 제거 후 전송
String sanitized = message.replaceAll("\\d{6}-\\d{7}", "[주민번호]");
```

---

## 관련 문서

- [HTTP 클라이언트](http-client.md) - OpenAI가 내부적으로 사용
- [JSON 처리](json.md) - AI 응답 파싱
- [환경설정 및 캐시](configuration.md) - API 키 관리 및 응답 캐싱

---

[← 목차로 돌아가기](README.md)
