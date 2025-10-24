# REST API 개발

맑은프레임워크의 **RestAPI 클래스**를 사용하면 HTTP 메소드(GET, POST, PUT, DELETE)별로 분기 처리하여 RESTful API를 쉽게 개발할 수 있습니다.

---

## URL 라우팅 설정

REST API에서는 `/api/user`와 같이 확장자 없는 깔끔한 URL을 사용합니다. `/api/*` 경로를 `/api/index.jsp`로 매핑하고, index.jsp에서 라우팅을 처리합니다.

### 1. web.xml 설정

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
         http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">

    <!-- /api/* 요청을 /api/index.jsp로 매핑 -->
    <servlet>
        <servlet-name>APIRouter</servlet-name>
        <jsp-file>/api/index.jsp</jsp-file>
    </servlet>
    <servlet-mapping>
        <servlet-name>APIRouter</servlet-name>
        <url-pattern>/api/*</url-pattern>
    </servlet-mapping>

</web-app>
```

### 2. /api/index.jsp 라우터 구현

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ page import="java.util.*, java.io.*, malgnsoft.util.*" %><%

// 요청 경로 분석
String requestURI = request.getRequestURI();
String contextPath = request.getContextPath();
String path = requestURI.substring(contextPath.length() + 5); // "/api/" 제거

// 빈 경로 처리
if(path.isEmpty() || "/".equals(path)) {
    response.sendError(404, "API endpoint not found");
    return;
}

// Path 파라미터 분리
String[] segments = path.split("/");
String resource = segments[0];
String id = segments.length > 1 ? segments[1] : null;

// request attribute에 설정 (하위 JSP에서 사용)
request.setAttribute("resource", resource);
request.setAttribute("id", id);

// 리소스별 라우팅
if("user".equals(resource)) {
    request.getRequestDispatcher("/api/user.jsp").forward(request, response);
} else if("product".equals(resource)) {
    request.getRequestDispatcher("/api/product.jsp").forward(request, response);
} else if("order".equals(resource)) {
    request.getRequestDispatcher("/api/order.jsp").forward(request, response);
} else {
    response.sendError(404, "Unknown resource: " + resource);
}

%>
```

### 3. 디렉토리 구조

```
webapp/
├── api/
│   ├── index.jsp          (라우터)
│   ├── user.jsp           → /api/user, /api/user/123
│   ├── product.jsp        → /api/product, /api/product/456
│   └── order.jsp          → /api/order, /api/order/789
└── WEB-INF/
    └── web.xml
```

### 4. 동작 방식

- `/api/user` → `index.jsp` → `/api/user.jsp`
- `/api/user/123` → `index.jsp` → `/api/user.jsp` (id="123")
- `/api/product` → `index.jsp` → `/api/product.jsp`
- `/api/product/456` → `index.jsp` → `/api/product.jsp` (id="456")

---

## 기본 사용법

### /api/user.jsp 예시 (Path parameter 활용)

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ page import="java.util.*, java.io.*, dao.*, malgnsoft.db.*, malgnsoft.util.*" %><%

Malgn m = new Malgn(request, response, out);
Json j = new Json();
Form f = new Form();
f.setRequest(request);

RestAPI api = new RestAPI(request, response);

// index.jsp에서 설정한 path parameter 가져오기
String id = (String)request.getAttribute("id");

// GET /api/user - 목록 조회
// GET /api/user/123 - 단일 조회
api.get(() -> {
    UserDao user = new UserDao();

    if(id != null && !id.isEmpty()) {
        // 단일 조회
        DataSet info = user.get(Integer.parseInt(id));
        if(info.next()) {
            j.add("id", info.i("id"));
            j.add("name", info.s("name"));
            j.add("email", info.s("email"));
            j.print();
        } else {
            j.error("사용자를 찾을 수 없습니다.");
        }
    } else {
        // 목록 조회
        DataSet list = user.find();
        j.add("users", list);
        j.print();
    }
});

// POST /api/user - 생성
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

## 인증 처리

REST API에서 인증 실패 시 HTTP 상태 코드와 함께 에러를 반환할 수 있습니다.

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

// Path parameter
String id = (String)request.getAttribute("id");

// GET - 목록 조회 또는 단일 조회
api.get(() -> {
    UserDao user = new UserDao();

    if(id != null && !id.isEmpty()) {
        // 단일 조회
        DataSet info = user.get(Integer.parseInt(id));
        if(info.next()) {
            j.add("id", info.i("id"));
            j.add("name", info.s("name"));
            j.print();
        } else {
            j.error("사용자를 찾을 수 없습니다.");
        }
    } else {
        // 목록 조회
        DataSet list = user.find();
        j.add("users", list);
        j.print();
    }
});

// POST - 생성
api.post(() -> {
    UserDao user = new UserDao();
    user.item("name", f.get("name"));
    user.item("created_by", userId);

    if(user.insert()) {
        j.success("등록되었습니다.", user.id);
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
if(userId == 0) {
    api.error(401, "인증실패");
    return;  // 필수!
}
```

### 2. 잘못된 요청 (400 Bad Request)

```jsp
api.post(() -> {
    String name = f.get("name");
    if("".equals(name)) {
        api.error(400, "이름은 필수입니다.");
        return;
    }

    // 정상 처리
    out.print("{\"status\":\"success\"}");
});
```

### 3. 지원하는 에러 코드

- **401**: Unauthorized (인증 실패)
- **400**: Bad Request (잘못된 요청)

---

## 클라이언트 호출 예시

### JavaScript (fetch API)

```javascript
// GET /api/user - 목록 조회
fetch('/api/user')
    .then(response => response.json())
    .then(data => console.log(data));

// GET /api/user/123 - 단일 조회
fetch('/api/user/123')
    .then(response => response.json())
    .then(data => console.log(data));

// POST /api/user - 생성
fetch('/api/user', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/x-www-form-urlencoded',
    },
    body: 'name=홍길동&email=hong@example.com'
})
    .then(response => response.json())
    .then(data => console.log(data));

// PUT /api/user/123 - 수정
fetch('/api/user/123', {
    method: 'PUT',
    headers: {
        'Content-Type': 'application/x-www-form-urlencoded',
    },
    body: 'name=김철수'
})
    .then(response => response.json())
    .then(data => console.log(data));

// DELETE /api/user/123 - 삭제
fetch('/api/user/123', {
    method: 'DELETE'
})
    .then(response => response.json())
    .then(data => console.log(data));
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

## 관련 문서

- [JSON 처리](json.md) - Json 클래스 상세 가이드
- [코딩 원칙 > AJAX 요청 처리](coding-principles.md#4-2-ajax-요청-처리) - AJAX 응답 방법
- [인증 처리](authentication.md) - Auth 클래스 활용

---

[← 목차로 돌아가기](README.md)
