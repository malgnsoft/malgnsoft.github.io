# OpenAI 통합

맑은프레임워크의 OpenAI 클래스는 OpenAI API(ChatGPT)를 쉽게 통합하여 AI 기능을 웹 애플리케이션에 추가할 수 있게 합니다.

## 목차

- [기본 설정](#기본-설정)
- [텍스트 생성](#텍스트-생성)
- [대화형 챗봇](#대화형-챗봇)
- [시스템 프롬프트](#시스템-프롬프트)
- [고급 설정](#고급-설정)

---

## 기본 설정

### OpenAI 객체 생성

```jsp
OpenAI model = new OpenAI();

// API 키 설정 (필수)
model.apiKey("sk-your-api-key-here");

// 모델 선택 (기본값: gpt-3.5-turbo)
model.modelName("gpt-3.5-turbo");
// 또는 더 강력한 모델
model.modelName("gpt-4o-mini");
model.modelName("gpt-4");
```

### 디버그 모드

```jsp
OpenAI model = new OpenAI();
model.setDebug(out);  // HTTP 요청/응답 로그 출력
model.setDebug();     // 로그 파일로 출력
```

---

## 텍스트 생성

### 간단한 질문-응답

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

OpenAI model = new OpenAI();
model.apiKey("sk-your-api-key-here");
model.modelName("gpt-3.5-turbo");

// 질문하고 응답 받기
String answer = model.generate("대한민국의 수도는 어디인가?");
m.p(answer);
// "대한민국의 수도는 서울입니다."

%>
```

### 구조화된 응답

JSON 형식으로 응답을 받을 수 있습니다:

```jsp
OpenAI model = new OpenAI();
model.apiKey("sk-your-api-key-here");

String response = model.generate(
    "서울의 인구는 몇명인가?\n\n"
    + "output-format:{\"data\":\"xxx\"}"
);

m.p(response);
// {"data":"약 970만명"}

// JSON 파싱
Json j = new Json(response);
String population = j.getString("//data");
```

### 복잡한 데이터 생성

```jsp
OpenAI model = new OpenAI();
model.apiKey("sk-your-api-key-here");
model.modelName("gpt-4o-mini");

m.startTimer();

String response = model.generate(
    "AI와 관련된 문제 10개만 만들어줘. 보기는 4개로 하고, 정답과 해설도 작성해줘. 한글로 해줘.\n\n"
    + "output-format:[{\"question\":\"xxx\", \"choice1\":\"-\", \"choice2\":\"-\", \"choice3\":\"-\", \"choice4\":\"-\", \"answer\":\"1\", \"description\":\"-\"}]"
);

m.p(response);
m.p("소요 시간: " + m.stopTimer() + "ms");

// JSON 파싱하여 데이터베이스에 저장
Json j = new Json(response);
DataSet questions = j.getDataSet("//");
QuizDao dao = new QuizDao();

while(questions.next()) {
    DataMap data = new DataMap();
    data.put("question", questions.s("question"));
    data.put("choice1", questions.s("choice1"));
    data.put("choice2", questions.s("choice2"));
    data.put("choice3", questions.s("choice3"));
    data.put("choice4", questions.s("choice4"));
    data.put("answer", questions.s("answer"));
    data.put("description", questions.s("description"));
    dao.insert(data);
}
```

---

## 대화형 챗봇

### 대화 기록 유지

`chat()` 메소드를 사용하면 대화 컨텍스트가 자동으로 유지됩니다:

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 세션에서 OpenAI 객체 가져오기 (또는 새로 생성)
OpenAI model = (OpenAI)session.getAttribute("chatModel");
if(model == null) {
    model = new OpenAI();
    model.apiKey("sk-your-api-key-here");
    model.modelName("gpt-4o-mini");
    session.setAttribute("chatModel", model);
}

// 사용자 메시지
String userMessage = m.rs("message");
if(!userMessage.isEmpty()) {
    String response = model.chat(userMessage);

    j.put("message", response);
    j.print(out);
}

%>
```

### 대화 예제

```jsp
OpenAI model = new OpenAI();
model.apiKey("sk-your-api-key-here");
model.modelName("gpt-4o-mini");

// 첫 번째 메시지
String r1 = model.chat("내 이름은 임꺽정이야");
m.p(r1);
// "안녕하세요, 임꺽정님! 무엇을 도와드릴까요?"

// 두 번째 메시지 - 이전 대화 기억
String r2 = model.chat("내가 혹시 누군지 아니? 내 나이 37세");
m.p(r2);
// "네, 임꺽정님이시죠. 37세시라고 말씀하셨습니다."

// 세 번째 메시지 - 계속 대화 컨텍스트 유지
String r3 = model.chat("나이도 알아?");
m.p(r3);
// "네, 37세라고 하셨습니다."
```

### 수동으로 대화 기록 추가

```jsp
OpenAI model = new OpenAI();
model.apiKey("sk-your-api-key-here");

// 이전 대화 수동 추가
model.addChat("안녕하세요", "안녕하세요! 무엇을 도와드릴까요?");
model.addChat("날씨 좋네요", "네, 좋은 날씨입니다!");

// 새로운 메시지 (이전 컨텍스트 포함)
String response = model.chat("지금 몇 시야?");
```

---

## 시스템 프롬프트

### 시스템 메시지 설정

시스템 메시지로 AI의 역할과 행동을 정의할 수 있습니다:

```jsp
OpenAI model = new OpenAI();
model.apiKey("sk-your-api-key-here");
model.modelName("gpt-4o-mini");

// AI의 페르소나 설정
model.system("당신은 친절한 고객 서비스 담당자입니다. 항상 공손하고 도움이 되는 답변을 제공하세요.");

String response = model.generate("제품이 불량인 것 같아요");
// "죄송합니다. 불편을 드려 죄송합니다. 어떤 문제가 있으신지 자세히 말씀해주시겠어요? 최대한 빠르게 해결해드리겠습니다."
```

### 전문가 시스템

```jsp
// 프로그래밍 전문가
model.system("당신은 숙련된 Java 프로그래머입니다. 코드 리뷰와 최적화 제안을 제공하세요.");

String codeReview = model.generate(
    "다음 코드를 리뷰해주세요:\n\n"
    + "for(int i=0; i<list.size(); i++) {\n"
    + "    System.out.println(list.get(i));\n"
    + "}"
);

// 의료 정보 제공자
model.system("당신은 의료 정보를 제공하는 AI입니다. 정확하고 근거 있는 정보만 제공하며, 진단은 하지 마세요.");

// 언어 튜터
model.system("당신은 영어 선생님입니다. 학생의 영어 문장을 교정하고 더 나은 표현을 제안하세요.");

// 창의적 작가
model.system("당신은 창의적인 소설가입니다. 흥미진진하고 상상력 넘치는 이야기를 만들어주세요.");
```

### 응답 스타일 제어

```jsp
// 장황하게 답변
model.system("아주 장황하게 대답해줘. 가능한 한 많은 세부 정보를 포함하세요.");
String detailed = model.generate("서울의 인구는 몇명인가?");

// 간결하게 답변
model.system("아주 간결하게 핵심만 대답해줘. 한 문장으로만 답하세요.");
String brief = model.generate("서울의 인구는 몇명인가?");

// 특정 형식으로 답변
model.system("모든 답변을 JSON 형식으로만 제공하세요.");
String jsonResponse = model.generate("대한민국의 수도와 인구를 알려줘");
```

---

## 고급 설정

### Temperature 조절

Temperature는 응답의 무작위성을 제어합니다 (0.0 ~ 2.0):

```jsp
OpenAI model = new OpenAI();
model.apiKey("sk-your-api-key-here");

// 낮은 temperature (0.0 ~ 0.3): 일관되고 예측 가능한 응답
model.temperature(0.2);
String factual = model.generate("물의 끓는점은?");
// "100°C입니다" (매번 동일한 답변)

// 중간 temperature (0.7): 균형잡힌 응답 (기본값)
model.temperature(0.7);

// 높은 temperature (1.0 ~ 2.0): 창의적이고 다양한 응답
model.temperature(1.5);
String creative = model.generate("우주에 대한 시를 써줘");
// 매번 다른 창의적인 시
```

### 사용 예시

```jsp
// 팩트 기반 Q&A (낮은 temperature)
model.temperature(0.1);
String fact = model.generate("2 + 2 = ?");

// 창작 콘텐츠 (높은 temperature)
model.temperature(1.2);
String story = model.generate("우주 해적에 대한 짧은 이야기를 써줘");

// 일반 대화 (중간 temperature)
model.temperature(0.7);
String chat = model.generate("오늘 기분이 어때?");
```

---

## 실제 활용 예제

### 1. 자동 요약 기능

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="/init.jsp" %><%

String content = m.rs("content");

OpenAI model = new OpenAI();
model.apiKey("sk-your-api-key-here");
model.modelName("gpt-3.5-turbo");
model.system("당신은 전문 요약가입니다. 핵심 내용만 3-5문장으로 요약하세요.");

String summary = model.generate("다음 글을 요약해주세요:\n\n" + content);

j.success("요약 완료", new DataMap().put("summary", summary));

%>
```

### 2. 감정 분석

```jsp
OpenAI model = new OpenAI();
model.apiKey("sk-your-api-key-here");
model.system("사용자 리뷰의 감정을 분석하세요. positive, negative, neutral 중 하나로만 답하세요.");

String review = "이 제품 정말 최고예요! 강력 추천합니다.";
String sentiment = model.generate(review);
// "positive"
```

### 3. 콘텐츠 생성기

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

String topic = m.rs("topic");
String tone = m.rs("tone", "전문적");

OpenAI model = new OpenAI();
model.apiKey("sk-your-api-key-here");
model.modelName("gpt-4o-mini");
model.system("당신은 블로그 작가입니다. " + tone + "인 톤으로 글을 작성하세요.");

String article = model.generate(
    topic + "에 대한 블로그 글을 작성해주세요. 500자 내외로 작성하세요."
);

p.setBody("blog.article");
p.setVar("content", article);
p.display();

%>
```

### 4. 챗봇 서비스

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 세션에서 대화 모델 가져오기
OpenAI model = (OpenAI)session.getAttribute("chatbot");
if(model == null) {
    model = new OpenAI();
    model.apiKey("sk-your-api-key-here");
    model.modelName("gpt-4o-mini");
    model.system(
        "당신은 맑은프레임워크 기술 지원 챗봇입니다. " +
        "사용자의 기술적 질문에 정확하고 친절하게 답변하세요. " +
        "모르는 것은 솔직하게 모른다고 말하세요."
    );
    session.setAttribute("chatbot", model);
}

String userMsg = m.rs("message");
String botResponse = model.chat(userMsg);

j.success(new DataMap().put("response", botResponse));

%>
```

### 5. 번역 서비스

```jsp
OpenAI model = new OpenAI();
model.apiKey("sk-your-api-key-here");
model.system("당신은 전문 번역가입니다. 한국어를 영어로 정확하게 번역하세요.");

String korean = "안녕하세요. 만나서 반갑습니다.";
String english = model.generate(korean);
// "Hello. Nice to meet you."
```

---

## 에러 처리

```jsp
OpenAI model = new OpenAI();
model.apiKey("sk-your-api-key-here");

try {
    String response = model.generate("질문");

    if(response.isEmpty()) {
        // 응답이 비어있음
        m.jsError("AI 응답을 받지 못했습니다");
        return;
    }

    m.p(response);

} catch(Exception e) {
    // API 에러
    Malgn.errorLog("OpenAI API 에러", e);
    m.jsError("AI 서비스 오류가 발생했습니다");
}

// 에러 메시지 확인
if(model.errMsg != null) {
    m.p("Error: " + model.errMsg);
}
```

---

## 비용 최적화 팁

### 1. 적절한 모델 선택

```jsp
// 간단한 작업: gpt-3.5-turbo (저렴, 빠름)
model.modelName("gpt-3.5-turbo");

// 복잡한 작업: gpt-4o-mini (균형)
model.modelName("gpt-4o-mini");

// 고급 작업: gpt-4 (비쌈, 정확)
model.modelName("gpt-4");
```

### 2. 프롬프트 최적화

```jsp
// 비효율적: 장황한 프롬프트
String bad = model.generate(
    "안녕하세요. 저는 학생입니다. 제가 궁금한게 있는데요, " +
    "혹시 물의 끓는점이 몇 도인지 알려주실 수 있나요? " +
    "가능하면 자세히 설명해주세요."
);

// 효율적: 간결한 프롬프트
String good = model.generate("물의 끓는점은?");
```

### 3. 캐싱 활용

```jsp
// 자주 묻는 질문은 캐싱
Cache cache = new Cache();
String question = "맑은프레임워크란?";
String answer = cache.getString("faq_" + Malgn.md5(question));

if(answer == null) {
    // 캐시에 없으면 AI에게 질문
    answer = model.generate(question);
    cache.save("faq_" + Malgn.md5(question), answer);
}

m.p(answer);
```

---

## 주의사항

1. **API 키 보안**: API 키를 소스코드에 직접 넣지 말고 환경설정 파일 사용
   ```jsp
   model.apiKey(Config.get("openai.apiKey"));
   ```

2. **비용 관리**: OpenAI API는 사용량에 따라 과금됩니다
3. **응답 시간**: AI 응답은 수 초가 걸릴 수 있으므로 로딩 UI 필요
4. **콘텐츠 필터링**: 생성된 콘텐츠는 검토가 필요할 수 있음
5. **개인정보**: 민감한 개인정보를 AI에 전송하지 않도록 주의

---

## 관련 문서

- [HTTP 클라이언트](http-client.md) - OpenAI가 내부적으로 사용
- [JSON 처리](json.md) - AI 응답 파싱
- [환경설정 및 캐시](configuration.md) - API 키 관리 및 응답 캐싱
