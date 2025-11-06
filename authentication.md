# 인증 처리

[← 목차로 돌아가기](README.md)

---

## 인증 시스템 개요

맑은프레임워크는 **세션 및 쿠키 기반**의 강력한 인증 시스템을 제공합니다.

### 핵심 개념

**Auth 클래스는 init.jsp에서 주로 사용됩니다:**

1. **객체 생성 후 `isValid()` 호출**: 인증 데이터 검증
2. **검증 성공 시**: 전역 변수(userId, userName 등) 설정
3. **세부 페이지에서**: `userId == 0` 또는 `userId == null`로 로그인 상태 확인

### 주요 클래스

- **Auth 클래스**: 인증 정보 저장/조회/검증
- **Malgn 클래스 (m)**: 세션/쿠키 관리 메소드

---

## init.jsp에서 인증 처리 패턴

**init.jsp에서 Auth 객체를 생성하고 전역 변수를 설정하는 것이 핵심입니다:**

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ page import="java.util.*, java.io.*, dao.*, malgnsoft.db.*, malgnsoft.util.*" %><%

Malgn m = new Malgn(request, response, out);

Form f = new Form();
f.setRequest(request);

Page p = new Page();
p.setRequest(request);
p.setWriter(out);
p.setPageContext(pageContext);

// Auth 객체 생성
Auth auth = new Auth(request, response);

// 전역 변수 초기화
int userId = 0;
String userName = null;
int userLevel = 0;

// 인증 데이터 검증
if(auth.isValid()) {
    // 인증 성공: 사이트 전체에서 사용할 변수 설정
    userId = auth.getInt("user_id");
    userName = auth.getString("user_name");
    userLevel = auth.getInt("user_level");
}

// 템플릿에서 사용할 수 있도록 설정
p.setVar("userId", userId);
p.setVar("userName", userName);
p.setVar("userLevel", userLevel);

%>
```

**세부 페이지에서 로그인 체크:**

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// userId가 0이면 로그인 안된 상태
if(userId == 0) {
    m.jsAlert("로그인이 필요합니다.");
    m.jsReplace("/main/login.jsp");
    return;
}

// 로그인된 사용자 처리
m.p("환영합니다, " + userName + "님!");

%>
```

---

## Auth 클래스

### 기본 개념

Auth 클래스는 로그인 사용자 정보를 **암호화하여** 세션 또는 쿠키에 저장합니다.

```jsp
Auth auth = new Auth(request, response);
```

### 주요 메소드

| 메소드 | 설명 |
|--------|------|
| isValid() | 인증 데이터 검증 (init.jsp에서 사용) |
| put(key, value) | 인증 정보 추가 |
| save() | 세션/쿠키에 저장 |
| getInt(key) | 정수 값 가져오기 |
| getString(key) | 문자열 값 가져오기 |
| delete() | 인증 정보 삭제 (로그아웃) |
| loginForm() | 로그인 페이지로 리다이렉트 |

### Auth 설정 옵션

Auth 클래스는 다양한 설정을 지원합니다:

| 설정 메소드 | 기본값 | 설명 |
|------------|--------|------|
| setKeyName(String) | "AUTHID" | 세션/쿠키 키 이름 |
| setLoginURL(String) | "../member/login.jsp" | 로그인 페이지 URL |
| setPath(String) | "/" | 쿠키 경로 |
| setDomain(String) | null | 쿠키 도메인 (.example.com) |
| setSecure(boolean) | false | HTTPS에서만 쿠키 전송 |
| setHttpOnly(boolean) | false | JavaScript에서 쿠키 접근 차단 |
| setSameSite(String) | null | SameSite 속성 (Strict/Lax/None) |
| setValidTime(int) | -1 | 인증 유효 시간 (초), -1이면 무제한 |
| setMaxAge(int) | -1 | 쿠키 유효 기간 (초), -1이면 브라우저 종료 시 삭제 |

**설정 예시:**

```jsp
Auth auth = new Auth(request, response);
auth.setKeyName("MY_AUTH");
auth.setLoginURL("/member/login.jsp");
auth.setSecure(true);  // HTTPS 전용
auth.setHttpOnly(true);  // XSS 방지
auth.setSameSite("Strict");  // CSRF 방지
auth.setValidTime(3600);  // 1시간 후 재인증 필요
auth.setMaxAge(86400);  // 쿠키 24시간 유지
```

---

## 로그인 구현

### 기본 로그인 예제

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

f.addElement("id", null, "title:'아이디', required:'Y'");
f.addElement("passwd", null, "title:'비밀번호', required:'Y'");

if(m.isPost() && f.validate()) {

    String id = f.get("id");
    String passwd = f.get("passwd");

    // 데이터베이스에서 사용자 확인
    UserDao dao = new UserDao();
    DataSet user = dao.query("WHERE user_id = ? AND passwd = ?", new Object[]{id, Malgn.sha256(passwd)});

    if(user.next()) {
        // 로그인 성공 - 인증 정보 저장
        auth.put("user_id", user.getInt("id"));
        auth.put("user_name", user.getString("name"));
        auth.put("user_level", user.getInt("level"));
        auth.save();

        m.jsAlert("로그인 성공");
        m.jsReplace("/main/index.jsp");
    } else {
        // 로그인 실패
        m.jsError("아이디 또는 비밀번호가 올바르지 않습니다.");
    }
    return;
}

p.setBody("main.login");
p.setVar("form_script", f.getScript());
p.display();

%>
```

### HTML 템플릿 (/html/main/login.html)

```html
<h2>로그인</h2>

<form name="form1" method="post">
    <div>
        <label>아이디</label>
        <input type="text" name="id" placeholder="아이디를 입력하세요">
    </div>
    <div>
        <label>비밀번호</label>
        <input type="password" name="passwd" placeholder="비밀번호를 입력하세요">
    </div>
    <button type="submit">로그인</button>
</form>

{{form_script}}
```

---

## 로그아웃 구현

### 기본 로그아웃

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 인증 정보 삭제
auth.delete();

m.jsAlert("로그아웃되었습니다.");
m.jsReplace("/main/login.jsp");

%>
```

---

## 로그인 체크

### 페이지별 로그인 체크

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 로그인 체크 (userId는 init.jsp에서 설정됨)
if(userId == 0) {
    m.jsAlert("로그인이 필요합니다.");
    m.jsReplace("/main/login.jsp");
    return;
}

// 로그인한 사용자 정보 사용
m.p("로그인 사용자: " + userName + " (ID: " + userId + ")");

%>
```

### 권한 레벨 체크

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 로그인 체크
if(userId == 0) {
    m.jsAlert("로그인이 필요합니다.");
    m.jsReplace("/main/login.jsp");
    return;
}

// 관리자 권한 체크 (userLevel은 init.jsp에서 설정됨)
if(userLevel < 10) {
    m.jsError("관리자만 접근할 수 있습니다.");
    return;
}

// 관리자 페이지 처리
p.setBody("admin.dashboard");
p.display();

%>
```

---

## 세션 관리

### 세션 값 설정

```jsp
// 세션에 값 저장
m.setSession("user_name", "홍길동");
m.setSession("user_level", 5);
```

### 세션 값 가져오기

```jsp
// 문자열로 가져오기
String userName = m.getSession("user_name");

// 정수로 가져오기
int userLevel = m.getSessionInt("user_level");

// null 체크
String value = m.getSession("key");
if(value != null) {
    // 처리
}
```

### 세션 값 삭제

```jsp
// 특정 세션 값 삭제
m.removeSession("user_name");

// 모든 세션 삭제
session.invalidate();
```

---

## 쿠키 관리

### 쿠키 설정

```jsp
// 기본 쿠키 (브라우저 종료 시 삭제)
m.setCookie("remember_id", "user001");

// 유효기간이 있는 쿠키 (7일)
m.setCookie("remember_id", "user001", 60 * 60 * 24 * 7);
```

### 쿠키 읽기

```jsp
String rememberId = m.getCookie("remember_id");
if(rememberId != null) {
    m.p("저장된 아이디: " + rememberId);
}
```

### 쿠키 삭제

```jsp
// 유효기간을 0으로 설정하여 삭제
m.setCookie("remember_id", "", 0);
```

---

## 실전 예제

### 1. 아이디 저장 기능이 있는 로그인

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

f.addElement("id", null, "required:Y");
f.addElement("passwd", null, "required:Y");
f.addElement("remember_id", null, null);

if(m.isPost() && f.validate()) {

    String id = f.get("id");
    String passwd = f.get("passwd");
    String rememberId = f.get("remember_id");

    // 사용자 확인
    UserDao dao = new UserDao();
    DataSet user = dao.query("WHERE user_id = ? AND passwd = ?", new Object[]{id, Malgn.sha256(passwd)});

    if(user.next()) {
        // 로그인 성공
        auth.put("user_id", user.getInt("id"));
        auth.put("user_name", user.getString("name"));
        auth.save();

        // 아이디 저장 체크박스 처리
        if("Y".equals(rememberId)) {
            m.setCookie("remember_id", id, 60 * 60 * 24 * 30);  // 30일
        } else {
            m.setCookie("remember_id", "", 0);  // 삭제
        }

        m.jsReplace("/main/index.jsp");
    } else {
        m.jsError("로그인 실패");
    }
    return;
}

// 저장된 아이디 불러오기
String savedId = m.getCookie("remember_id");

p.setBody("main.login");
p.setVar("saved_id", savedId != null ? savedId : "");
p.setVar("form_script", f.getScript());
p.display();

%>
```

**HTML**:
```html
<form name="form1" method="post">
    <input type="text" name="id" value="{{saved_id}}">
    <input type="password" name="passwd">
    <label>
        <input type="checkbox" name="remember_id" value="Y"> 아이디 저장
    </label>
    <button type="submit">로그인</button>
</form>
```

### 2. 자동 로그인 기능 (init.jsp에서 처리)

**init.jsp에 자동 로그인 로직 추가:**

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ page import="java.util.*, java.io.*, dao.*, malgnsoft.db.*, malgnsoft.util.*" %><%

Malgn m = new Malgn(request, response, out);

Form f = new Form();
f.setRequest(request);

Page p = new Page();
p.setRequest(request);
p.setWriter(out);
p.setPageContext(pageContext);

Auth auth = new Auth(request, response);

int userId = 0;
String userName = null;

// 인증 검증
if(auth.isValid()) {
    userId = auth.getInt("user_id");
    userName = auth.getString("user_name");
} else {
    // 자동 로그인 체크
    String autoLoginToken = m.getCookie("auto_login_token");
    if(autoLoginToken != null) {
        UserDao dao = new UserDao();
        DataSet user = dao.query("WHERE auto_login_token = ?", new Object[]{autoLoginToken});

        if(user.next()) {
            // 자동 로그인 처리
            auth.put("user_id", user.getInt("id"));
            auth.put("user_name", user.getString("name"));
            auth.save();

            userId = user.getInt("id");
            userName = user.getString("name");
        }
    }
}

p.setVar("userId", userId);
p.setVar("userName", userName);

%>
```

**로그인 페이지에서 자동 로그인 토큰 저장:

// 로그인 폼 처리
if(m.isPost() && f.validate()) {

    String id = f.get("id");
    String passwd = f.get("passwd");
    String autoLogin = f.get("auto_login");

    UserDao dao = new UserDao();
    DataSet user = dao.query("WHERE user_id = ? AND passwd = ?", new Object[]{id, Malgn.sha256(passwd)});

    if(user.next()) {
        auth.put("user_id", user.getInt("id"));
        auth.put("user_name", user.getString("name"));
        auth.save();

        // 자동 로그인 체크박스 처리
        if("Y".equals(autoLogin)) {
            // 고유 토큰 생성
            String token = Malgn.uuid();

            // DB에 토큰 저장
            dao.item("auto_login_token", token);
            dao.update("id = ?", new Object[] { user.getInt("id") });

            // 쿠키에 저장 (30일)
            m.setCookie("auto_login_token", token, 60 * 60 * 24 * 30);
        }

        m.jsReplace("/main/index.jsp");
    }
}

%>
```

### 3. 마지막 로그인 시간 기록

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

if(m.isPost() && f.validate()) {

    String id = f.get("id");
    String passwd = f.get("passwd");

    UserDao dao = new UserDao();
    DataSet user = dao.query("WHERE user_id = ? AND passwd = ?", new Object[]{id, Malgn.sha256(passwd)});

    if(user.next()) {
        int userId = user.getInt("id");

        // 마지막 로그인 시간 업데이트
        dao.item("last_login", m.time());
        dao.item("login_count", user.getInt("login_count") + 1);
        dao.update("id = ?", new Object[] { userId });

        // 인증 정보 저장
        auth.put("user_id", userId);
        auth.put("user_name", user.getString("name"));
        auth.put("last_login", m.time());
        auth.save();

        m.jsReplace("/main/index.jsp");
    } else {
        m.jsError("로그인 실패");
    }
}

%>
```

### 4. init.jsp에 전역 인증 변수 설정

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ page import="java.util.*, java.io.*, dao.*, malgnsoft.db.*, malgnsoft.util.*" %><%

Malgn m = new Malgn(request, response, out);

Form f = new Form();
f.setRequest(request);

Page p = new Page();
p.setRequest(request);
p.setWriter(out);
p.setPageContext(pageContext);

Auth auth = new Auth(request, response);

// 전역 변수로 사용자 ID 설정
int userId = 0;
String userName = null;

if(auth.isValid()) {
    userId = auth.getInt("user_id");
    userName = auth.getString("user_name");
}

// 템플릿에서 사용할 수 있도록 설정
p.setVar("isLogin", userId != 0);
p.setVar("userId", userId);
p.setVar("userName", userName);

%>
```

**HTML 템플릿에서 사용**:
```html
<!--@if(isLogin)-->
<div class="user-info">
    <span>{{userName}}님 환영합니다</span>
    <a href="/main/logout.jsp">로그아웃</a>
</div>
<!--@else-->
<div class="login-buttons">
    <a href="/main/login.jsp">로그인</a>
    <a href="/main/join.jsp">회원가입</a>
</div>
<!--/if(isLogin)-->
```

### 5. API 인증

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="/init.jsp" %><%

// API 토큰 인증
String apiToken = m.rq("token");

if(apiToken == null || apiToken.isEmpty()) {
    j.error(401, "인증 토큰이 필요합니다");
    return;
}

// 토큰 검증
UserDao dao = new UserDao();
DataSet user = dao.query("WHERE api_token = ? AND status = 1", new Object[]{apiToken});

if(!user.next()) {
    j.error(401, "유효하지 않은 토큰입니다");
    return;
}

// API 처리
DataMap result = new DataMap();
result.put("user_id", user.getInt("id"));
result.put("user_name", user.getString("name"));

j.success("인증 성공", result);

%>
```

---

## 비밀번호 암호화

### SHA-256 해시

```jsp
// 회원가입 시 비밀번호 암호화
String plainPassword = f.get("passwd");
String hashedPassword = Malgn.sha256(plainPassword);

UserDao dao = new UserDao();
dao.item("user_id", f.get("id"));
dao.item("passwd", hashedPassword);
dao.item("name", f.get("name"));
dao.insert();
```

### 다양한 해시 알고리즘

```jsp
// MD5 (권장하지 않음)
String md5 = Malgn.md5("password");

// SHA-1
String sha1 = Malgn.sha1("password");

// SHA-256 (권장)
String sha256 = Malgn.sha256("password");

// SHA-512
String sha512 = Malgn.sha512("password");
```

---

## 세션 타임아웃

### web.xml에서 설정

```xml
<session-config>
    <session-timeout>30</session-timeout>  <!-- 30분 -->
</session-config>
```

### JSP에서 동적 설정

```jsp
// 세션 유지 시간 설정 (초 단위)
session.setMaxInactiveInterval(60 * 30);  // 30분
```

---

## 보안 고려사항

### 1. HTTPS 사용

프로덕션 환경에서는 반드시 HTTPS를 사용하세요.

### 2. 비밀번호 정책

```jsp
// 비밀번호 강도 체크
f.addElement("passwd", null, "required:Y, minlength:8, maxlength:20");

// 비밀번호 확인
f.addElement("passwd_confirm", null, "required:Y, equalTo:'passwd'");
```

### 3. SQL Injection 방지

```jsp
// Bad - SQL Injection 위험
String sql = "SELECT * FROM tb_user WHERE user_id = '" + id + "'";

// Good - Prepared Statement 사용
DataSet user = dao.query("WHERE user_id = ?", new Object[]{id});
```

### 4. XSS 방지

```jsp
// 출력 시 HTML 이스케이프
String safeName = Malgn.htt(userName);
m.p(safeName);
```

---

## 주요 메소드 정리

### Auth 클래스

| 메소드 | 설명 |
|--------|------|
| put(key, value) | 인증 정보 추가 |
| save() | 세션에 저장 |
| get(key) | 값 가져오기 |
| getInt(key) | 정수로 가져오기 |
| getString(key) | 문자열로 가져오기 |
| clear() | 인증 정보 삭제 |
| isLogin() | 로그인 여부 |

### 세션/쿠키 메소드 (Malgn)

| 메소드 | 설명 |
|--------|------|
| setSession(key, value) | 세션 값 설정 |
| getSession(key) | 세션 값 가져오기 |
| getSessionInt(key) | 세션 정수 가져오기 |
| removeSession(key) | 세션 값 삭제 |
| setCookie(name, value) | 쿠키 설정 |
| setCookie(name, value, age) | 쿠키 설정 (유효기간) |
| getCookie(name) | 쿠키 값 가져오기 |

---

## 다음 단계

- [환경 설정](configuration.md)로 이동하여 Config 클래스 사용법을 배워보세요.

---

[← 이전: Excel 처리](excel.md) | [목차로 돌아가기](README.md) | [다음: 환경 설정 →](configuration.md)
