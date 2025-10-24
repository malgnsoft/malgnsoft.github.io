# 코딩 원칙

맑은프레임워크의 핵심 설계 철학과 코딩 원칙을 설명합니다.

---

## 설계 철학

맑은프레임워크는 **"간결함과 안정성"**을 최우선 가치로 삼습니다.

### 핵심 가치

**1. JSP는 비즈니스 로직만**
- JSP 파일은 데이터 처리와 흐름 제어만 담당
- HTML 마크업은 템플릿 파일로 완전 분리
- 코드의 역할이 명확하여 유지보수 용이

**2. 예외는 내부에서 처리**
- 개발자가 try-catch를 작성할 필요 없음
- 모든 예외는 프레임워크 내부에서 처리
- 에러는 로그 파일에 자동 기록

**3. 외부 의존성 최소화**
- 외부 라이브러리는 DAO 패턴으로 캡슐화
- JSP에서는 맑은프레임워크 클래스만 사용
- 의존성 변경 시 영향 범위 최소화

---

## 코딩 원칙

### 1. JSP에서는 맑은프레임워크 클래스만 사용

JSP 파일에서는 맑은프레임워크가 제공하는 클래스만 사용해야 합니다.

**사용 가능한 클래스:**
```jsp
// 핵심 클래스
Malgn m = new Malgn(request, response);
Page p = new Page(request, response);
FormData f = new FormData(request);
FormValidator fv = new FormValidator(request);

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
/logs/error.log

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

list.first();
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

p.setLayout("layout.default");
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

---

### 5. Page 메소드 호출 순서 준수

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

### 6. 페이징이 필요 없으면 ListManager 사용 금지

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

### ❌ 5. 불필요한 ListManager 사용
```jsp
// 나쁜 예: Excel 다운로드인데 ListManager 사용
ListManager lm = new ListManager();
lm.setListNum(999999);
DataSet list = lm.getDataSet();  // 2번 쿼리 실행
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
p.setLayout("layout.default");
p.setBody("main.user_form");
p.setVar("form_script", f.getScript());
p.display();
%>
```

### ✅ 2. 조건부 검색

```jsp
<%
UserDao user = new UserDao();

// 검색어가 있을 때만 조건 추가
String keyword = f.get("keyword");
if(!"".equals(keyword)) {
    user.addSearch("name,email", keyword, "LIKE");
}

// 상태값이 있을 때만 조건 추가
int status = f.getInt("status", -1);
if(status >= 0) {
    user.addSearch("status", status);
}

user.setOrderBy("id DESC");
DataSet list = user.find();
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

---

## 체크리스트

새로운 JSP 파일을 작성할 때 다음 사항을 확인하세요:

- [ ] JSP에 HTML 코드가 없는가?
- [ ] try-catch를 사용하지 않았는가?
- [ ] 외부 라이브러리를 직접 사용하지 않았는가?
- [ ] if(m.isPost()) 블록에 return이 있는가?
- [ ] Page 메소드를 순서대로 호출했는가?
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
