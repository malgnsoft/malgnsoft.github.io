# REST API 개발

맑은프레임워크의 **RestAPI 클래스**를 사용하면 HTTP 메소드(GET, POST, PUT, DELETE)별로 분기 처리하여 RESTful API를 쉽게 개발할 수 있습니다.

---

## URL 매핑 설정 (web.xml)

REST API에서는 일반적으로 `/api/user`와 같이 확장자 없는 깔끔한 URL을 사용합니다. 이를 위해 web.xml에서 URL을 JSP 파일로 매핑해야 합니다.

### web.xml 설정 (방법 1: 간단한 매핑)

가장 간단한 방법은 JSP 파일을 직접 서블릿으로 매핑하는 것입니다:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
         http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">

    <!-- API 파일을 URL 패턴에 매핑 -->
    <servlet>
        <servlet-name>UserAPI</servlet-name>
        <jsp-file>/api/user.jsp</jsp-file>
    </servlet>
    <servlet-mapping>
        <servlet-name>UserAPI</servlet-name>
        <url-pattern>/api/user</url-pattern>
    </servlet-mapping>

    <servlet>
        <servlet-name>ProductAPI</servlet-name>
        <jsp-file>/api/product.jsp</jsp-file>
    </servlet>
    <servlet-mapping>
        <servlet-name>ProductAPI</servlet-name>
        <url-pattern>/api/product</url-pattern>
    </servlet-mapping>

</web-app>
```

**장점:**
- 필터 없이 web.xml만으로 처리
- 매우 간단하고 명확함

**단점:**
- API가 많아지면 web.xml이 길어짐
- 새로운 API 추가 시마다 설정 필요

---

### web.xml 설정 (방법 2: 필터를 통한 자동 매핑)

API가 많거나 자주 추가되는 경우, 필터를 사용하여 `/api/*` 경로를 자동 매핑할 수 있습니다:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
         http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">

    <!-- API URL 자동 매핑 라우터 (권장) -->
    <servlet>
        <servlet-name>ApiRouter</servlet-name>
        <servlet-class>malgnsoft.util.ApiRouter</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>ApiRouter</servlet-name>
        <url-pattern>/api/*</url-pattern>
    </servlet-mapping>

</web-app>
```

맑은프레임워크의 `ApiRouter` 서블릿이 자동으로 URL을 JSP 파일로 라우팅합니다.

**장점:**
- 한 번 설정으로 모든 `/api/*` 경로 처리
- 새로운 API 추가 시 설정 불필요
- 하위 폴더 자동 지원 (`/api/v1/user`, `/api/admin/user`)
- **Path parameter 지원** (`/api/user/123`)
- 서블릿 방식으로 필터보다 성능 우수
- 범용적으로 사용 가능 (`/rest/*`, `/v1/*` 등 어떤 경로에도 적용)

**동작 방식:**
1. 기본 경로:
   - `/api/user` → `/api/user.jsp`
   - `/api/v1/user` → `/api/v1/user.jsp`
   - `/rest/product` → `/rest/product.jsp`

2. **Path parameter 지원:**
   - `/api/user/123` → `/api/user.jsp` + `request.getAttribute("id") = "123"`
   - `/api/post/456/comment` → `/api/post.jsp` + `getAttribute("id") = "456"`, `getAttribute("subResource") = "comment"`
   - `/api/post/456/comment/789` → `/api/post.jsp` + `getAttribute("id") = "456"`, `getAttribute("subId") = "789"`

**Path parameter 사용 예시:**

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ page import="java.util.*, java.io.*, dao.*, malgnsoft.db.*, malgnsoft.util.*" %><%

Malgn m = new Malgn(request, response, out);
Json j = new Json();
Form f = new Form();
f.setRequest(request);

RestAPI api = new RestAPI(request, response);

// Path parameter 추출
String id = (String)request.getAttribute("id");

// GET /api/user/123 - 단일 조회
api.get(() -> {
    if(id != null && !id.isEmpty()) {
        UserDao user = new UserDao();
        DataSet info = user.get(Integer.parseInt(id));

        if(info.next()) {
            j.add("id", info.i("id"));
            j.add("name", info.s("name"));
            j.print();
        } else {
            j.error("사용자를 찾을 수 없습니다.");
        }
    } else {
        // GET /api/user - 목록 조회
        UserDao user = new UserDao();
        DataSet list = user.find();
        j.add("users", list);
        j.print();
    }
});

// PUT /api/user/123 - 수정
api.put(() -> {
    if(id == null || id.isEmpty()) {
        api.error(400, "ID는 필수입니다.");
        return;
    }

    UserDao user = new UserDao();
    user.get(Integer.parseInt(id));
    user.item("name", f.get("name"));

    if(user.update()) {
        j.success("수정되었습니다.");
    } else {
        j.error(user.getErrMsg());
    }
});

// DELETE /api/user/123 - 삭제
api.delete(() -> {
    if(id == null || id.isEmpty()) {
        api.error(400, "ID는 필수입니다.");
        return;
    }

    UserDao user = new UserDao();
    if(user.delete(Integer.parseInt(id))) {
        j.success("삭제되었습니다.");
    } else {
        j.error(user.getErrMsg());
    }
});

%>
```

---

### web.xml 설정 (방법 3: URLRewriteFilter 사용)

Tuckey URLRewriteFilter를 사용하면 정규식으로 간단하게 URL을 매핑할 수 있습니다:

**1. 의존성 추가 (pom.xml 또는 라이브러리 다운로드)**
```xml
<dependency>
    <groupId>org.tuckey</groupId>
    <artifactId>urlrewritefilter</artifactId>
    <version>4.0.4</version>
</dependency>
```

**2. web.xml 설정**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
         http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">

    <filter>
        <filter-name>UrlRewriteFilter</filter-name>
        <filter-class>org.tuckey.web.filters.urlrewrite.UrlRewriteFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>UrlRewriteFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

</web-app>
```

**3. WEB-INF/urlrewrite.xml 생성**
```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE urlrewrite PUBLIC "-//tuckey.org//DTD UrlRewrite 4.0//EN"
        "http://www.tuckey.org/res/dtds/urlrewrite4.0.dtd">
<urlrewrite>
    <!-- /api/* 경로를 /api/*.jsp로 매핑 -->
    <rule>
        <from>^/api/(.+)$</from>
        <to>/api/$1.jsp</to>
    </rule>
</urlrewrite>
```

**장점:**
- 정규식을 사용한 강력한 URL 매핑
- 설정 파일로 관리하여 유지보수 용이
- 외부 라이브러리 사용으로 검증된 솔루션

**단점:**
- 외부 라이브러리 의존성 추가
- 별도 설정 파일 관리 필요

---

### 권장 사항

- **소규모 프로젝트**: 방법 1 (직접 매핑) - 가장 간단
- **중대형 프로젝트**: 방법 2 (ApiRouter) - 맑은프레임워크 내장, 성능 우수 (권장)
- **복잡한 URL 패턴**: 방법 3 (URLRewriteFilter) - 정규식 지원

**디렉토리 구조:**
```
webapp/
├── api/
│   ├── user.jsp           → /api/user 로 호출
│   ├── product.jsp        → /api/product 로 호출
│   ├── v1/
│   │   ├── user.jsp       → /api/v1/user 로 호출 (필터 사용 시)
│   │   └── product.jsp    → /api/v1/product 로 호출 (필터 사용 시)
│   └── admin/
│       ├── user.jsp       → /api/admin/user 로 호출 (필터 사용 시)
│       └── stats.jsp      → /api/admin/stats 로 호출 (필터 사용 시)
└── WEB-INF/
    └── web.xml
```

**호출 예시:**
```javascript
// 기본 API (두 방법 모두 동일)
fetch('/api/user', { method: 'GET' });
fetch('/api/product', { method: 'POST' });

// 하위 폴더 API (필터 사용 시에만 가능)
fetch('/api/v1/user', { method: 'GET' });
fetch('/api/admin/user', { method: 'DELETE' });
```

---

## 기본 사용법

### 1. REST API 엔드포인트 생성

RestAPI 클래스는 HTTP 메소드를 자동으로 감지하여 해당하는 로직만 실행합니다.

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ page import="java.util.*, java.io.*, dao.*, malgnsoft.db.*, malgnsoft.util.*" %><%

Malgn m = new Malgn(request, response, out);

Form f = new Form();
f.setRequest(request);

RestAPI api = new RestAPI(request, response);

// GET 요청 처리
api.get(() -> {
    UserDao user = new UserDao();
    DataSet list = user.find();

    out.print("{\"users\": [");
    boolean first = true;
    while(list.next()) {
        if(!first) out.print(",");
        out.print("{");
        out.print("\"id\":" + list.i("id") + ",");
        out.print("\"name\":\"" + list.s("name") + "\"");
        out.print("}");
        first = false;
    }
    out.print("]}");
});

// POST 요청 처리
api.post(() -> {
    UserDao user = new UserDao();
    user.item("name", f.get("name"));
    user.item("email", f.get("email"));

    if(user.insert()) {
        out.print("{\"status\":\"success\",\"id\":" + user.id + "}");
    } else {
        out.print("{\"status\":\"error\",\"message\":\"" + user.getErrMsg() + "\"}");
    }
});

// PUT 요청 처리
api.put(() -> {
    int id = f.getInt("id");
    UserDao user = new UserDao();
    user.get(id);

    user.item("name", f.get("name"));
    user.item("email", f.get("email"));

    if(user.update()) {
        out.print("{\"status\":\"success\"}");
    } else {
        out.print("{\"status\":\"error\",\"message\":\"" + user.getErrMsg() + "\"}");
    }
});

// DELETE 요청 처리
api.delete(() -> {
    int id = f.getInt("id");
    UserDao user = new UserDao();

    if(user.delete(id)) {
        out.print("{\"status\":\"success\"}");
    } else {
        out.print("{\"status\":\"error\",\"message\":\"" + user.getErrMsg() + "\"}");
    }
});

%>
```

---

## 인증 처리

REST API에서 인증 실패 시 HTTP 상태 코드와 함께 에러를 반환할 수 있습니다.

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ page import="java.util.*, java.io.*, dao.*, malgnsoft.db.*, malgnsoft.util.*" %><%

Malgn m = new Malgn(request, response, out);

Form f = new Form();
f.setRequest(request);

Auth auth = new Auth(request, response);

RestAPI api = new RestAPI(request, response);

// 인증 체크
int userId = 0;
if(auth.isValid()) {
    userId = auth.getInt("user_id");
}

if(userId == 0) {
    api.error(401, "인증실패");
    return;
}

// 인증된 사용자만 접근 가능
api.get(() -> {
    UserDao user = new UserDao();
    DataSet info = user.get(userId);

    if(info.next()) {
        out.print("{");
        out.print("\"id\":" + info.i("id") + ",");
        out.print("\"name\":\"" + info.s("name") + "\",");
        out.print("\"email\":\"" + info.s("email") + "\"");
        out.print("}");
    }
});

api.post(() -> {
    // POST 처리 로직
    out.print("{\"status\":\"success\"}");
});

api.put(() -> {
    // PUT 처리 로직
    out.print("{\"status\":\"success\"}");
});

api.delete(() -> {
    // DELETE 처리 로직
    out.print("{\"status\":\"success\"}");
});

%>
```

---

## Json 클래스와 함께 사용

Json 클래스를 활용하면 더 간결하게 JSON 응답을 생성할 수 있습니다.

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ page import="java.util.*, java.io.*, dao.*, malgnsoft.db.*, malgnsoft.util.*" %><%

Malgn m = new Malgn(request, response, out);
Json j = new Json();

Form f = new Form();
f.setRequest(request);

RestAPI api = new RestAPI(request, response);

// GET - 목록 조회
api.get(() -> {
    UserDao user = new UserDao();
    DataSet list = user.find();

    j.add("users", list);
    j.print();
});

// POST - 생성
api.post(() -> {
    UserDao user = new UserDao();
    user.item("name", f.get("name"));
    user.item("email", f.get("email"));

    if(user.insert()) {
        j.success("등록되었습니다.", user.id);
    } else {
        j.error(user.getErrMsg());
    }
});

// PUT - 수정
api.put(() -> {
    int id = f.getInt("id");
    UserDao user = new UserDao();
    user.get(id);

    user.item("name", f.get("name"));

    if(user.update()) {
        j.success("수정되었습니다.");
    } else {
        j.error(user.getErrMsg());
    }
});

// DELETE - 삭제
api.delete(() -> {
    int id = f.getInt("id");
    UserDao user = new UserDao();

    if(user.delete(id)) {
        j.success("삭제되었습니다.");
    } else {
        j.error(user.getErrMsg());
    }
});

%>
```

---

## 에러 처리

### 1. 인증 실패 (401 Unauthorized)

```jsp
<%
if(userId == 0) {
    api.error(401, "인증실패");
    return;
}
%>
```

### 2. 잘못된 요청 (400 Bad Request)

```jsp
<%
api.post(() -> {
    String name = f.get("name");
    if("".equals(name)) {
        api.error(400, "이름은 필수입니다.");
        return;
    }

    // 정상 처리
    out.print("{\"status\":\"success\"}");
});
%>
```

### 3. 지원하는 에러 코드

- **401**: Unauthorized (인증 실패)
- **400**: Bad Request (잘못된 요청)

---

## 클라이언트 호출 예시

### JavaScript (fetch API)

```javascript
// GET 요청
fetch('/api/users.jsp')
    .then(response => response.json())
    .then(data => console.log(data));

// POST 요청
fetch('/api/users.jsp', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/x-www-form-urlencoded',
    },
    body: 'name=홍길동&email=hong@example.com'
})
    .then(response => response.json())
    .then(data => console.log(data));

// PUT 요청
fetch('/api/users.jsp', {
    method: 'PUT',
    headers: {
        'Content-Type': 'application/x-www-form-urlencoded',
    },
    body: 'id=123&name=김철수'
})
    .then(response => response.json())
    .then(data => console.log(data));

// DELETE 요청
fetch('/api/users.jsp?id=123', {
    method: 'DELETE'
})
    .then(response => response.json())
    .then(data => console.log(data));
```

### jQuery

```javascript
// GET 요청
$.get('/api/users.jsp', function(data) {
    console.log(data);
});

// POST 요청
$.post('/api/users.jsp', {
    name: '홍길동',
    email: 'hong@example.com'
}, function(data) {
    console.log(data);
});

// PUT 요청
$.ajax({
    url: '/api/users.jsp',
    method: 'PUT',
    data: {
        id: 123,
        name: '김철수'
    },
    success: function(data) {
        console.log(data);
    }
});

// DELETE 요청
$.ajax({
    url: '/api/users.jsp',
    method: 'DELETE',
    data: { id: 123 },
    success: function(data) {
        console.log(data);
    }
});
```

---

## 실전 예제

### 완전한 CRUD API

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ page import="java.util.*, java.io.*, dao.*, malgnsoft.db.*, malgnsoft.util.*" %><%

Malgn m = new Malgn(request, response, out);
Json j = new Json();

Form f = new Form();
f.setRequest(request);

Auth auth = new Auth(request, response);

RestAPI api = new RestAPI(request, response);

// 인증 체크
int userId = 0;
if(auth.isValid()) {
    userId = auth.getInt("user_id");
}

if(userId == 0) {
    api.error(401, "로그인이 필요합니다.");
    return;
}

// GET - 목록 조회 또는 단일 조회
api.get(() -> {
    int id = f.getInt("id");
    UserDao user = new UserDao();

    if(id > 0) {
        // 단일 조회
        DataSet info = user.get(id);
        if(info.next()) {
            j.add("id", info.i("id"));
            j.add("name", info.s("name"));
            j.add("email", info.s("email"));
            j.print();
        } else {
            api.error(400, "사용자를 찾을 수 없습니다.");
        }
    } else {
        // 목록 조회
        user.setOrderBy("id DESC");
        DataSet list = user.find();
        j.add("users", list);
        j.print();
    }
});

// POST - 생성
api.post(() -> {
    String name = f.get("name");
    String email = f.get("email");

    if("".equals(name)) {
        api.error(400, "이름은 필수입니다.");
        return;
    }

    UserDao user = new UserDao();
    user.item("name", name);
    user.item("email", email);
    user.item("created_by", userId);

    if(user.insert()) {
        j.add("status", "success");
        j.add("message", "등록되었습니다.");
        j.add("id", user.id);
        j.print();
    } else {
        j.error(user.getErrMsg());
    }
});

// PUT - 수정
api.put(() -> {
    int id = f.getInt("id");

    if(id == 0) {
        api.error(400, "ID는 필수입니다.");
        return;
    }

    UserDao user = new UserDao();
    user.get(id);

    String name = f.get("name");
    if(!"".equals(name)) {
        user.item("name", name);
    }

    String email = f.get("email");
    if(!"".equals(email)) {
        user.item("email", email);
    }

    if(user.update()) {
        j.success("수정되었습니다.");
    } else {
        j.error(user.getErrMsg());
    }
});

// DELETE - 삭제
api.delete(() -> {
    int id = f.getInt("id");

    if(id == 0) {
        api.error(400, "ID는 필수입니다.");
        return;
    }

    UserDao user = new UserDao();

    if(user.delete(id)) {
        j.success("삭제되었습니다.");
    } else {
        j.error(user.getErrMsg());
    }
});

%>
```

---

## 사용 시 주의사항

### 1. Content-Type 설정

REST API 파일은 반드시 `application/json`으로 설정하세요:

```jsp
<%@ page contentType="application/json; charset=utf-8" %>
```

### 2. return 키워드 사용

에러 처리 후에는 반드시 `return`을 사용하세요:

```jsp
if(userId == 0) {
    api.error(401, "인증실패");
    return;  // 필수!
}
```

### 3. Lambda 표현식

Java 8 이상에서만 사용 가능합니다. Lambda 표현식 `() -> { }` 사용.

### 4. 예외 처리

RestAPI 클래스는 내부적으로 예외를 처리하므로 try-catch가 필요 없습니다.

---

## RestAPI vs 기존 방식 비교

### 기존 방식 (m.isPost 사용)

```jsp
<%
String method = request.getMethod();

if("GET".equals(method)) {
    // GET 처리
} else if("POST".equals(method)) {
    // POST 처리
} else if("PUT".equals(method)) {
    // PUT 처리
} else if("DELETE".equals(method)) {
    // DELETE 처리
}
%>
```

### RestAPI 사용

```jsp
<%
RestAPI api = new RestAPI(request, response);

api.get(() -> {
    // GET 처리
});

api.post(() -> {
    // POST 처리
});

api.put(() -> {
    // PUT 처리
});

api.delete(() -> {
    // DELETE 처리
});
%>
```

**장점:**
- 코드가 간결하고 읽기 쉬움
- 각 메소드별 로직이 명확히 분리됨
- RESTful API 설계 원칙을 자연스럽게 따름

---

## 관련 문서

- [JSON 처리](json.md) - Json 클래스 상세 가이드
- [코딩 원칙 > AJAX 요청 처리](coding-principles.md#4-2-ajax-요청-처리) - AJAX 응답 방법
- [인증 처리](authentication.md) - Auth 클래스 활용

---

[← 목차로 돌아가기](README.md)
