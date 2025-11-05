# 코딩 원칙

맑은프레임워크의 핵심 설계 철학과 코딩 원칙을 설명합니다.

---

## 설계 철학

맑은프레임워크는 **"단순함(Simplicity)"과 "명확함(Clarity)"**을 최우선 가치로 삼습니다.

> **"복잡함은 버그의 온상이다. 단순함은 신뢰성의 기반이다."**

### 핵심 원칙

#### 1. 관심사의 명확한 분리 (Clear Separation of Concerns)

**JSP는 비즈니스 로직만, 템플릿은 출력만**

```
[JSP]                    [Template]
데이터 처리       →      변수 출력
조건 분기         →      조건 블록 표시/숨김
반복 준비         →      반복 렌더링
```

- JSP 파일: 데이터 가져오기, 가공하기, 조건 판단하기
- 템플릿 파일: 변수 치환, 루프, IF/ELSE 블록만 (로직 없음)
- **템플릿에는 연산, 함수 호출, 삼항연산자 등 일체의 로직을 넣지 않음**

**왜 이렇게 하는가?**
- 디자이너는 HTML만, 개발자는 Java만 집중
- 로직 변경 시 HTML에 영향 없음
- HTML 변경 시 로직에 영향 없음
- 각자의 역할이 명확하여 협업 효율 극대화

**잘못된 접근 (피해야 할 것):**
```html
<!-- ❌ 템플릿에 로직 넣기 (다른 프레임워크 방식) -->
<option {{status == 'active' ? 'selected' : ''}}>활성</option>
<div>{{ price * quantity }}</div>
<span>{{ user.getName().toUpperCase() }}</span>
```

**올바른 접근 (맑은프레임워크 방식):**
```jsp
// ✅ JSP에서 로직 처리
p.setVar("selected", status.equals("active") ? "selected" : "");
p.setVar("total", price * quantity);
p.setVar("userName", user.getName().toUpperCase());
```
```html
<!-- ✅ 템플릿은 출력만 -->
<option {{selected}}>활성</option>
<div>{{total}}</div>
<span>{{userName}}</span>
```

---

#### 2. 예외는 내부에서 처리 (Exception Handling Inside)

**개발자는 비즈니스 로직에만 집중**

- 모든 프레임워크 메소드는 예외를 내부에서 처리
- try-catch 작성 불필요
- 성공/실패는 boolean 리턴값으로 판단
- 에러 메시지는 `getErrMsg()`로 확인
- 모든 예외는 자동으로 로그 파일에 기록

**왜 이렇게 하는가?**
- 코드가 간결해지고 가독성 향상
- 비즈니스 로직과 예외 처리가 섞이지 않음
- 프레임워크가 에러를 일관되게 처리
- 개발자는 핵심 로직에만 집중

```jsp
// ✅ 맑은프레임워크 방식 (간결함)
if(dao.insert()) {
    m.jsAlert("성공");
} else {
    m.jsAlert("실패: " + dao.getErrMsg());
}

// ❌ 전통적인 방식 (복잡함)
try {
    dao.insert();
    m.jsAlert("성공");
} catch(SQLException e) {
    m.jsAlert("실패: " + e.getMessage());
    log.error("Insert error", e);
} finally {
    // cleanup...
}
```

---

#### 3. 투명한 동작 (Transparent Behavior)

**프레임워크가 알아서 하되, 숨기지 않는다**

맑은프레임워크는 반복 작업을 자동화하지만, 개발자가 내부 동작을 이해하고 제어할 수 있도록 설계되었습니다.

**자동화 예시 1: Read/Write Splitting (rojndi)**
```xml
<!-- config.xml 설정만으로 자동 분리 -->
<jndi>jdbc/master</jndi>
<rojndi>jdbc/slave</rojndi>
```
```jsp
// 개발자는 코드 변경 없음
dao.query(...);   // 자동으로 Slave DB 사용
dao.insert(...);  // 자동으로 Master DB 사용
```

**자동화 예시 2: XSS 필터링**
```jsp
String keyword = m.rs("keyword");  // GET 파라미터는 자동 필터링
String content = f.get("content"); // POST 데이터는 원본 유지
```

**자동화 예시 3: DataSet 직렬화**
```jsp
p.setLoop("users", userList);  // 자동으로 first() 호출
p.display();                   // JSON/HTML 자동 선택
```

**왜 투명한가?**
- 설정 파일(config.xml)로 동작 방식 명시
- 메소드 이름이 동작을 명확히 표현 (`rs` = request string with sanitize)
- 필요시 기본 동작 재정의 가능 (`dao.setJndi()`)
- 디버그 모드로 내부 동작 확인 가능 (`dao.setDebug()`)

---

#### 4. 외부 의존성 격리 (Dependency Isolation)

**외부 라이브러리는 DAO 레이어로 캡슐화**

- JSP에서는 맑은프레임워크 클래스만 사용
- 외부 라이브러리는 DAO나 유틸리티 클래스로 감싸기
- 라이브러리 변경 시 JSP 코드는 영향 없음

**왜 격리하는가?**
- 외부 라이브러리 버전 업그레이드 시 영향 범위 최소화
- JSP 코드가 프레임워크 API로 통일되어 일관성 유지
- 테스트와 목업(Mock) 작성 용이

```jsp
// ❌ JSP에서 외부 라이브러리 직접 사용
import org.apache.http.client.HttpClient;
HttpClient client = HttpClients.createDefault();
HttpResponse response = client.execute(request);

// ✅ DAO로 캡슐화
ApiDao api = new ApiDao();
String result = api.fetchData(url);
```

---

#### 5. 단순함이 정답 (Simplicity over Complexity)

**간단한 해결책을 먼저 고려**

- 80%의 요구사항을 해결하는 단순한 방법 제공
- 나머지 20%는 개발자가 확장 가능하도록 설계
- "할 수 있다"고 모든 기능을 넣지 않음

**예시: 템플릿 엔진**
- 변수 치환: `{{var}}`
- 반복: `<!--@loop(list)-->...<!--/loop(list)-->`
- 조건: `<!--@if(var)-->...<!--/if(var)-->`
- **삼항연산자, 함수 호출, 연산은 제공하지 않음** → JSP에서 처리

**왜 단순함인가?**
- 학습 곡선이 낮아 빠르게 적용 가능
- 버그 발생 가능성 감소
- 유지보수 비용 감소
- 중소형 프로젝트에 최적화

---

### AI 에이전트를 위한 핵심 메시지

**맑은프레임워크 코드를 작성하거나 수정할 때 다음을 기억하세요:**

1. **템플릿은 출력 전용**: 템플릿에 로직(연산, 삼항연산자, 함수 호출 등)을 절대 넣지 마세요. 모든 로직은 JSP에서 처리합니다.

2. **try-catch 금지**: JSP에서 try-catch를 작성하지 마세요. 프레임워크가 예외를 처리합니다.

3. **단순함 유지**: 복잡한 기능을 추가하기보다, 단순하고 명확한 코드를 작성하세요.

4. **관심사 분리**: JSP는 데이터 처리, 템플릿은 출력, DAO는 데이터베이스 접근. 각자의 역할을 명확히 구분하세요.

5. **투명성 유지**: 자동화를 추가할 때는 개발자가 동작을 이해하고 제어할 수 있도록 설계하세요.

---

## 코딩 원칙

### 1. JSP에서는 맑은프레임워크 클래스만 사용

JSP 파일에서는 맑은프레임워크가 제공하는 클래스만 사용해야 합니다.

**사용 가능한 클래스:**
```jsp
// 핵심 클래스
Malgn m = new Malgn(request, response);
Page p = new Page(request, response);
Form f = new Form(request);

// 데이터 처리
UserDao user = new UserDao();
DataSet list = user.find();

// 유틸리티
ExcelX excel = new ExcelX();
Json json = new Json();
Xml xml = new Xml();
```

**외부 라이브러리는 DAO로 감싸기:**

```jsp
// ❌ 잘못된 예: JSP에서 외부 라이브러리 직접 사용
<%
import org.apache.http.client.HttpClient;
import org.apache.http.impl.client.HttpClients;

HttpClient client = HttpClients.createDefault();
HttpGet request = new HttpGet("https://api.example.com/data");
HttpResponse response = client.execute(request);
%>

// ✅ 올바른 예: DAO로 캡슐화
<%
ApiDao api = new ApiDao();
String result = api.fetchData("https://api.example.com/data");
%>
```

**이유:**
- 외부 라이브러리 버전 변경 시 DAO만 수정
- JSP 코드가 간결하고 읽기 쉬움
- 테스트와 유지보수가 용이

---

### 2. try-catch 사용 금지

JSP에서는 try-catch를 사용하지 않습니다. 모든 예외 처리는 프레임워크 내부에서 자동으로 처리됩니다.

**❌ 잘못된 예:**
```jsp
<%
UserDao user = new UserDao();

try {
    user.item("name", "홍길동");
    user.item("email", "hong@example.com");
    user.insert();
    m.p("성공");
} catch(Exception e) {
    m.p("오류: " + e.getMessage());
}
%>
```

**✅ 올바른 예:**
```jsp
<%
UserDao user = new UserDao();
user.item("name", "홍길동");
user.item("email", "hong@example.com");

if(user.insert()) {
    m.p("성공");
} else {
    m.p("오류: " + user.getErrMsg());
}
%>
```

**동작 방식:**
- `insert()`, `update()`, `delete()` 등은 boolean 리턴
- 성공 시 `true`, 실패 시 `false` 반환
- 에러 메시지는 `getErrMsg()`로 확인
- 모든 예외는 내부에서 처리되고 로그 파일에 자동 기록

**로그 확인:**
```bash
# 에러 로그 위치
/data/logs/error_20250101.log

# 예외 발생 시 자동으로 기록됨
[2025-01-24 10:30:15] SQLException: Duplicate entry 'hong@example.com' for key 'email'
```

---

### 3. JSP와 HTML 완전 분리

JSP 파일에는 HTML 마크업을 작성하지 않습니다.

**❌ 잘못된 예:**
```jsp
<%
UserDao user = new UserDao();
DataSet list = user.find();

while(list.next()) {
%>
    <div class="user">
        <h3><%= list.s("name") %></h3>
        <p><%= list.s("email") %></p>
    </div>
<%
}
%>
```

**✅ 올바른 예:**

JSP 파일 (`/main/user_list.jsp`):
```jsp
<%
UserDao user = new UserDao();
DataSet list = user.find();

p.setLayout("default");
p.setBody("main.user_list");
p.setLoop("user", list);
p.display();
%>
```

HTML 템플릿 파일 (`/html/main/user_list.html`):
```html
<div class="user-list">
    <!--@loop(user)-->
    <div class="user">
        <h3>{{user.name}}</h3>
        <p>{{user.email}}</p>
    </div>
    <!--/loop(user)-->
</div>
```

**이유:**
- 디자이너와 개발자의 협업 용이
- 로직 변경 시 HTML에 영향 없음
- HTML 변경 시 로직에 영향 없음

---

### 4. POST 처리 후 반드시 return

`if(m.isPost())` 블록에서는 처리 완료 후 **반드시 return**을 해야 합니다.

**❌ 잘못된 예:**
```jsp
<%
if(m.isPost()) {
    UserDao user = new UserDao();
    user.item("name", f.get("name"));

    if(user.insert()) {
        m.jsAlert("등록되었습니다.");
        m.jsReplace("list.jsp");
    }
    // return이 없음!
}

// POST 처리 후에도 아래 코드가 실행됨
p.setBody("main.user_form");
p.display();
%>
```

**✅ 올바른 예:**
```jsp
<%
if(m.isPost()) {
    UserDao user = new UserDao();
    user.item("name", f.get("name"));

    if(user.insert()) {
        m.jsAlert("등록되었습니다.");
        m.jsReplace("list.jsp");
    }
    return;  // 필수!
}

p.setBody("main.user_form");
p.display();
%>
```

**이유:**
- POST 처리 후 화면 렌더링 방지
- 이중 처리 방지
- 명확한 흐름 제어

### 4-1. 페이지 이동: jsReplace vs redirect

페이지 이동 시 메시지 표시 여부에 따라 적절한 메소드를 선택하세요.

**jsReplace() - 메시지와 함께 이동:**
```jsp
<%
if(m.isPost()) {
    UserDao user = new UserDao();
    user.item("name", f.get("name"));

    if(user.insert()) {
        m.jsAlert("등록되었습니다.");
        m.jsReplace("list.jsp");  // 메시지 표시 후 이동
    }
    return;
}
%>
```

**redirect() - 즉시 이동:**
```jsp
<%
if(m.isPost()) {
    UserDao user = new UserDao();
    user.item("name", f.get("name"));

    if(user.insert()) {
        m.redirect("list.jsp");  // 즉시 이동 (메시지 없음)
    }
    return;
}
%>
```

**사용 기준:**
- **jsReplace()**: 사용자에게 메시지를 보여줄 필요가 있을 때
  - 성공/실패 알림과 함께 이동
  - 예: 로그인 성공, 등록 완료, 수정 완료
- **redirect()**: 메시지 없이 바로 이동할 때
  - 로그인 후 메인 페이지로 이동
  - 권한 체크 후 리다이렉트
  - 예: 로그인 안된 사용자를 로그인 페이지로 이동

**차이점:**
- `jsReplace()`: JavaScript alert/confirm 등과 함께 사용, 사용자 확인 후 이동
- `redirect()`: HTTP 302 리다이렉트, 즉시 이동

### 4-2. AJAX 요청 처리

AJAX를 통해 form을 submit하는 경우 **jsReplace()나 redirect()를 사용하면 안 됩니다**. 반드시 JSON이나 텍스트로 응답해야 합니다.

**❌ 잘못된 예 (AJAX에서 jsReplace 사용):**
```jsp
<%
if(m.isPost()) {
    UserDao user = new UserDao();
    user.item("name", f.get("name"));

    if(user.insert()) {
        m.jsAlert("등록되었습니다.");
        m.jsReplace("list.jsp");  // ❌ AJAX에서는 작동하지 않음
    }
    return;
}
%>
```

**✅ 올바른 예 1: out.print() 사용**
```jsp
<%
if(m.isPost()) {
    UserDao user = new UserDao();
    user.item("name", f.get("name"));

    if(user.insert()) {
        out.print("success");  // 단순 문자열 응답
    } else {
        out.print("error");
    }
    return;
}
%>
```

**✅ 올바른 예 2: Json.success() / Json.error() 사용**
```jsp
<%
Json j = new Json();

if(m.isPost()) {
    UserDao user = new UserDao();
    user.item("name", f.get("name"));

    if(user.insert()) {
        j.success("등록되었습니다.");  // {"status": "success", "message": "등록되었습니다."}
    } else {
        j.error("등록 실패: " + user.getErrMsg());  // {"status": "error", "message": "..."}
    }
    return;
}
%>
```

**✅ 올바른 예 3: displayJSON() 사용**
```jsp
<%
if(m.isPost()) {
    UserDao user = new UserDao();
    user.item("name", f.get("name"));

    if(user.insert()) {
        p.setVar("status", "success");
        p.setVar("message", "등록되었습니다.");
        p.setVar("id", user.id);
    } else {
        p.setVar("status", "error");
        p.setVar("message", user.getErrMsg());
    }
    p.displayJSON();  // JSON 형식으로 출력
    return;
}
%>
```

**✅ 올바른 예 4: API 응답 (p.setType(2))**
```jsp
<%
// REST API 엔드포인트
UserDao user = new UserDao();
user.addSearch("status", 1);
DataSet list = user.find();

p.setType(2);  // JSON 응답 타입 설정
p.setLoop("users", list);
p.display();
%>
```

**✅ 올바른 예 5: RestAPI 클래스 사용 (권장)**
```jsp
<%
Json j = new Json();
RestAPI api = new RestAPI(request, response);

api.get(() -> {
    UserDao user = new UserDao();
    DataSet list = user.find();
    j.add("users", list);
    j.print();
});

api.post(() -> {
    UserDao user = new UserDao();
    user.item("name", f.get("name"));

    if(user.insert()) {
        j.success("등록되었습니다.", user.id);
    } else {
        j.error(user.getErrMsg());
    }
});
%>
```

**사용 기준:**
- **out.print()**: 간단한 성공/실패 여부만 전달할 때
- **j.success() / j.error()**: 메시지와 함께 표준 JSON 응답
- **p.displayJSON()**: 여러 데이터를 포함한 JSON 응답
- **p.setType(2)**: REST API 엔드포인트, 리스트 데이터 응답
- **RestAPI 클래스**: RESTful API 개발 시 권장 (GET/POST/PUT/DELETE 분기)

**클라이언트 JavaScript 예시:**
```javascript
// fetch API 사용
fetch('/user_register.jsp', {
    method: 'POST',
    body: formData
})
.then(response => response.json())
.then(data => {
    if(data.status === 'success') {
        alert(data.message);
        location.href = 'list.jsp';
    } else {
        alert(data.message);
    }
});
```

---

### 5. GET/POST 파라미터 처리 구분 (보안)

**보안상의 이유로 GET과 POST 파라미터는 다른 메소드를 사용해야 합니다.**

**GET 파라미터: m.rs(), m.ri() 사용 (XSS 필터 자동 적용)**
```jsp
<%
// GET으로 전달된 파라미터 (쿼리스트링)
String keyword = m.rs("keyword");  // XSS 필터 적용됨
int page = m.ri("page");           // 정수 변환
int id = m.ri("id");

// 검색 조건에 사용
UserDao user = new UserDao();
user.addSearch("name", keyword, "LIKE");
DataSet list = user.find();
%>
```

**POST 파라미터: f.get() 사용 (원본 데이터)**
```jsp
<%
if(m.isPost()) {
    UserDao user = new UserDao();

    // POST로 전달된 데이터 (Form 데이터)
    user.item("name", f.get("name"));      // 원본 데이터 그대로 저장
    user.item("content", f.get("content")); // HTML 에디터 내용 등

    if(user.insert()) {
        m.redirect("list.jsp");
    }
    return;
}
%>
```

**이유:**
- **m.rs()**: GET 파라미터는 URL에 노출되므로 XSS 공격 위험이 있음. 자동 필터링 필요
- **f.get()**: POST 데이터는 DB에 저장할 원본 데이터이므로 필터링하지 않음 (HTML 에디터 내용 등)
- 출력 시에는 별도로 필터링하여 XSS 방어

**출력 시 XSS 방어:**
```jsp
// 템플릿에서 출력 시 자동 escape
{{user.content}}  // 자동으로 HTML escape됨

// JSP에서 직접 출력 시
<%= m.escape(content) %>  // HTML escape
```

**사용 기준:**
- **GET (m.rs, m.ri)**: 검색 키워드, 페이지 번호, ID 조회, 필터 조건
- **POST (f.get)**: 게시글 내용, 사용자 입력 데이터, 파일 업로드

---

### 6. Page 메소드 호출 순서 준수

Page 객체의 메소드는 정해진 순서대로 호출해야 합니다.

**올바른 순서:**
```jsp
p.setLayout("layout.default");  // 1. 레이아웃 설정
p.setBody("main.content");      // 2. 본문 템플릿 설정
p.setVar("title", "제목");       // 3. 변수 설정
p.setLoop("list", data);        // 4. 반복 데이터 설정
p.display();                    // 5. 출력
```

**❌ 잘못된 순서:**
```jsp
p.setVar("title", "제목");       // 변수를 먼저 설정
p.setBody("main.content");      // 본문을 나중에 설정
p.display();                    // Layout이 설정되지 않음
```

---

### 7. DataSet 사용 전 반드시 next() 호출

`find()` 또는 `query()`로 리턴된 DataSet을 사용하려면 **반드시 `next()`를 호출**해야 합니다.

**내부 동작 원리:**
- DataSet은 내부적으로 커서(cursor)를 가지고 있음
- 초기 커서 위치는 `-1` (데이터 이전)
- `next()`를 호출하면 커서가 다음 레코드로 이동
- 레코드가 하나인 경우, `next()`는 데이터 존재 여부를 체크하는 역할

**❌ 잘못된 예:**
```jsp
<%
UserDao user = new UserDao();
DataSet info = user.get(123);

// next()를 호출하지 않음!
String name = info.s("name");  // 에러 또는 빈 값
%>
```

**✅ 올바른 예 (단일 레코드):**
```jsp
<%
UserDao user = new UserDao();
DataSet info = user.get(123);

if(info.next()) {
    // 레코드가 존재함
    String name = info.s("name");
    String email = info.s("email");
    m.p("이름: " + name);
} else {
    // 레코드가 없음
    m.p("사용자를 찾을 수 없습니다.");
}
%>
```

**✅ 올바른 예 (여러 레코드):**
```jsp
<%
UserDao user = new UserDao();
DataSet list = user.find();

// first() 없이 바로 while(next()) 사용
while(list.next()) {
    String name = list.s("name");
    m.p(name);
}
%>
```

**✅ first()가 필요한 경우:**
```jsp
<%
UserDao user = new UserDao();
DataSet list = user.find();

// 첫 번째 순회
while(list.next()) {
    m.p(list.s("name"));
}
// 커서가 마지막에 위치

// 다시 처음부터 순회하려면 first() 필요
list.first();
while(list.next()) {
    m.p(list.s("email"));
}
%>
```

**주의사항:**
- `p.setLoop()`는 내부에서 자동으로 `first()` 수행
- 템플릿에 전달할 DataSet은 first() 불필요
```jsp
DataSet list = user.find();
while(list.next()) {
    // 로직 처리로 커서가 끝까지 이동
}
p.setLoop("list", list);  // 자동으로 first() 수행됨
p.display();
```

**이유:**
- 커서가 -1에서 시작하므로 next()로 첫 레코드로 이동 필요
- 단일 레코드: next()가 데이터 존재 여부 체크
- 여러 레코드: while(next())로 순회 (first() 불필요)
- first()는 커서를 재사용할 때만 필요

---

### 8. 페이징이 필요 없으면 ListManager 사용 금지

ListManager는 **페이징 처리를 위해 쿼리를 2번 실행**합니다.

1. COUNT 쿼리 (전체 개수 조회)
2. SELECT 쿼리 (데이터 조회)

페이징이 필요 없는 경우 DataObject를 직접 사용하여 성능을 개선하세요.

**❌ 비효율적인 예 (Excel 다운로드):**
```jsp
// 페이징이 필요 없는데 ListManager 사용
ListManager lm = new ListManager();
lm.setTable("tb_user");
lm.setListNum(100000);  // 큰 숫자로 페이징 우회
DataSet list = lm.getDataSet();  // COUNT + SELECT 쿼리 2번 실행

ExcelX excel = new ExcelX();
excel.setData(list, columns);
excel.write(response, "users.xlsx");
```

**✅ 효율적인 예:**
```jsp
// 페이징이 필요 없으므로 DataObject 사용
UserDao user = new UserDao();
user.setOrderBy("id DESC");
DataSet list = user.find();  // SELECT 쿼리 1번만 실행

ExcelX excel = new ExcelX();
excel.setData(list, columns);
excel.write(response, "users.xlsx");
```

**사용 기준:**
- **ListManager 사용**: 화면에 페이징이 필요한 경우
- **DataObject 사용**: 전체 데이터 조회, Excel 다운로드, API 응답 등

---

## 안티패턴 (하지 말아야 할 것)

### ❌ 1. JSP에 HTML 혼재
```jsp
// 나쁜 예
<div class="container">
<%
UserDao user = new UserDao();
DataSet list = user.find();
while(list.next()) {
%>
    <p><%= list.s("name") %></p>
<% } %>
</div>
```

### ❌ 2. 예외를 직접 처리
```jsp
// 나쁜 예
try {
    dao.insert();
} catch(SQLException e) {
    e.printStackTrace();
}
```

### ❌ 3. 외부 라이브러리 직접 사용
```jsp
// 나쁜 예
import com.google.gson.Gson;
Gson gson = new Gson();
String json = gson.toJson(data);
```

### ❌ 4. isPost() 후 return 누락
```jsp
// 나쁜 예
if(m.isPost()) {
    dao.insert();
    m.jsAlert("완료");
    // return이 없음!
}
p.display();  // POST 처리 후에도 실행됨
```

### ❌ 5. DataSet에서 next() 호출 누락
```jsp
// 나쁜 예: next()를 호출하지 않음
UserDao user = new UserDao();
DataSet info = user.get(123);
String name = info.s("name");  // 에러 또는 빈 값
```

### ❌ 6. 불필요한 ListManager 사용
```jsp
// 나쁜 예: Excel 다운로드인데 ListManager 사용
ListManager lm = new ListManager();
lm.setListNum(999999);
DataSet list = lm.getDataSet();  // 2번 쿼리 실행
```

### ❌ 7. AJAX 요청에서 jsReplace/redirect 사용
```jsp
// 나쁜 예: AJAX에서 페이지 이동 메소드 사용
if(m.isPost()) {
    dao.insert();
    m.jsAlert("완료");
    m.jsReplace("list.jsp");  // AJAX에서는 작동하지 않음
}
```

### ❌ 8. GET/POST 파라미터 처리 혼용
```jsp
// 나쁜 예: GET 파라미터를 f.get()으로 받음 (XSS 필터 없음)
String keyword = f.get("keyword");  // XSS 공격 위험!
user.addSearch("name", keyword, "LIKE");

// 나쁜 예: POST 데이터를 m.rs()로 받음 (필터링되어 원본 손실)
if(m.isPost()) {
    String content = m.rs("content");  // HTML 에디터 내용이 손상됨
    user.item("content", content);
}
```

---

## 베스트 프랙티스

### ✅ 1. 명확한 흐름 제어

```jsp
<%
// POST 처리
if(m.isPost() && f.validate()) {
    UserDao user = new UserDao();
    user.item("name", f.get("name"));

    if(user.insert()) {
        m.jsAlert("등록되었습니다.");
        m.jsReplace("list.jsp");
    } else {
        m.jsError("등록 실패: " + user.getErrMsg());
    }
    return;
}

// GET 처리 (폼 표시)
p.setLayout("default");
p.setBody("main.user_form");
p.setVar("form_script", f.getScript());
p.display();
%>
```

### ✅ 2. GET/POST 파라미터 올바른 처리

```jsp
<%
UserDao user = new UserDao();

// GET 파라미터: m.rs(), m.ri() 사용 (XSS 필터 적용)
String keyword = m.rs("keyword");  // 검색어
int page = m.ri("page");           // 페이지 번호
int id = m.ri("id");               // 조회 ID

// 검색어가 있을 때만 조건 추가
if(!"".equals(keyword)) {
    user.addSearch("name,email", keyword, "LIKE");
}

user.setOrderBy("id DESC");
DataSet list = user.find();

// POST 처리: f.get() 사용 (원본 데이터)
if(m.isPost()) {
    user.item("name", f.get("name"));
    user.item("content", f.get("content"));  // HTML 에디터 내용

    if(user.insert()) {
        m.jsReplace("list.jsp");
    }
    return;
}
%>
```

### ✅ 3. 반복 작업에서 clear() 사용

```jsp
<%
UserDao user = new UserDao();

for(int i = 0; i < dataList.size(); i++) {
    user.item("name", dataList.get(i));
    user.item("email", emailList.get(i));

    if(user.insert()) {
        successCount++;
    }

    user.clear();  // 다음 반복을 위해 초기화
}
%>
```

### ✅ 4. 적절한 도구 선택

```jsp
// 페이징이 필요한 경우
ListManager lm = new ListManager();
lm.setRequest(request);
lm.setTable("tb_user");
lm.setListNum(20);
DataSet list = lm.getDataSet();
String pager = lm.getPaging();

// 페이징이 필요 없는 경우
UserDao user = new UserDao();
DataSet list = user.find();
```

### ✅ 5. 디버깅 활용

맑은프레임워크의 대부분의 클래스는 `setDebug()` 메소드를 지원합니다.

**개발 중 화면 디버깅:**
```jsp
<%
UserDao user = new UserDao();
user.setDebug(out);  // 디버그 정보를 화면에 출력

user.addSearch("name", "홍길동", "LIKE");
user.setOrderBy("id DESC");
DataSet list = user.find();
// 실행된 SQL 쿼리, 파라미터, 실행 시간이 화면에 출력됨
%>
```

**운영 중 로그파일 디버깅:**
```jsp
<%
UserDao user = new UserDao();
user.setDebug();  // 디버그 정보를 로그파일에 기록

user.addSearch("status", 1);
DataSet list = user.find();
// /logs/error.log에 SQL 쿼리와 실행 정보가 기록됨
%>
```

**지원하는 클래스:**
- `DataObject` (모든 DAO)
- `ListManager`
- `ExcelX`
- `Http`
- `Xml`
- 기타 대부분의 유틸리티 클래스

**디버그 출력 예시:**
```
[DEBUG] SQL: SELECT * FROM tb_user WHERE name LIKE ? ORDER BY id DESC
[DEBUG] Params: [%홍길동%]
[DEBUG] Execution Time: 15ms
```

**사용 기준:**
- **setDebug(out)**: 개발 환경에서 즉시 확인, 로그 파일 확인 불필요
- **setDebug()**: 운영 환경에서 실시간 디버깅, 화면에 정보 노출 방지

---

## 체크리스트

새로운 JSP 파일을 작성할 때 다음 사항을 확인하세요:

- [ ] JSP에 HTML 코드가 없는가?
- [ ] try-catch를 사용하지 않았는가?
- [ ] 외부 라이브러리를 직접 사용하지 않았는가?
- [ ] if(m.isPost()) 블록에 return이 있는가?
- [ ] GET 파라미터는 m.rs()/m.ri()를, POST 데이터는 f.get()을 사용했는가?
- [ ] AJAX 요청에서 jsReplace/redirect 대신 JSON을 사용했는가?
- [ ] Page 메소드를 순서대로 호출했는가?
- [ ] DataSet 사용 전에 next()를 호출했는가?
- [ ] 페이징이 필요 없는데 ListManager를 사용하지 않았는가?
- [ ] 반복문에서 dao.clear()를 호출했는가?

---

## 더 알아보기

- [빠른 시작](getting-started.md) - 프레임워크 기본 사용법
- [템플릿 시스템](template.md) - JSP와 HTML 분리 방법
- [데이터베이스 연동](database.md) - DAO 패턴 활용
- [데이터 입력 및 유효성 체크](form-validation.md) - POST 처리 방법

---

[← 목차로 돌아가기](README.md)
