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

단순히 확장자만 추가하여 해당 JSP 파일로 포워딩합니다.

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%

// 요청 경로에 .jsp 확장자 추가하여 포워딩
String requestURI = request.getRequestURI();
String contextPath = request.getContextPath();
String path = requestURI.substring(contextPath.length());

// /api/user → /api/user.jsp
String jspPath = path + ".jsp";

try {
    request.getRequestDispatcher(jspPath).forward(request, response);
} catch(Exception e) {
    response.sendError(404, "API endpoint not found: " + path);
}

%>
```

### 3. /api/init.jsp 공통 초기화 파일

API 폴더의 모든 JSP 파일에서 공통으로 사용할 객체와 인증을 초기화합니다.

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ page import="java.util.*, java.io.*, dao.*, malgnsoft.db.*, malgnsoft.util.*" %><%

Malgn m = new Malgn(request, response, out);

Form f = new Form();
f.setRequest(request);

Page p = new Page();
p.setRequest(request);
p.setWriter(out);
p.setPageContext(pageContext);

Auth auth = new Auth(request, response);

Json j = new Json();

RestAPI api = new RestAPI(request, response);

// 인증 체크
int userId = 0;
String userName = null;

if(auth.isValid()) {
    userId = auth.getInt("user_id");
    userName = auth.getString("user_name");
}

%>
```

### 4. 디렉토리 구조

```
webapp/
├── api/
│   ├── index.jsp          (라우터)
│   ├── init.jsp           (공통 초기화)
│   ├── user.jsp           → /api/user
│   ├── product.jsp        → /api/product
│   └── admin/
│       └── stats.jsp      → /api/admin/stats
└── WEB-INF/
    └── web.xml
```

### 5. 동작 방식

- `/api/user` → `index.jsp` → `/api/user.jsp`
- `/api/user?id=123` → `index.jsp` → `/api/user.jsp` (f.get("id")로 처리)
- `/api/product` → `index.jsp` → `/api/product.jsp`
- `/api/admin/stats` → `index.jsp` → `/api/admin/stats.jsp`

---

## 기본 사용법

### /api/user.jsp 예시 (init.jsp 사용)

```jsp
<%@ include file="/api/init.jsp" %><%

// GET /api/user - 목록 조회
// GET /api/user?id=123 - 단일 조회
api.get(() -> {
    UserDao user = new UserDao();
    String id = f.get("id");

    if(!"".equals(id)) {
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

// PUT /api/user?id=123 - 수정
api.put(() -> {
    String id = f.get("id");
    if("".equals(id)) {
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

// DELETE /api/user?id=123 - 삭제
api.delete(() -> {
    String id = f.get("id");
    if("".equals(id)) {
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

**장점:**
- `/api/init.jsp`에서 공통 객체 초기화 (m, f, p, auth, j, api)
- 각 API 파일은 비즈니스 로직만 작성
- 표준 query string 방식으로 파라미터 전달 (`f.get("id")`)
- 코드가 매우 간결하고 명확함

---

## 인증 처리

`/api/init.jsp`에서 기본적인 인증 체크를 수행하지만, 각 API에서 추가 인증이 필요한 경우 처리할 수 있습니다.

### /api/admin/user.jsp 예시 (관리자 전용 API)

```jsp
<%@ include file="/api/init.jsp" %><%

// 로그인 체크
if(userId == 0) {
    api.error(401, "로그인이 필요합니다.");
    return;
}

// 관리자 권한 체크
int userLevel = auth.getInt("user_level");
if(userLevel < 10) {
    api.error(403, "관리자만 접근할 수 있습니다.");
    return;
}

// GET - 사용자 목록 조회
api.get(() -> {
    UserDao user = new UserDao();
    DataSet list = user.find();
    j.add("users", list);
    j.print();
});

// POST - 사용자 생성
api.post(() -> {
    UserDao user = new UserDao();
    user.item("name", f.get("name"));
    user.item("email", f.get("email"));
    user.item("created_by", userId);  // init.jsp에서 설정된 userId 사용

    if(user.insert()) {
        j.success("등록되었습니다.", user.id);
    } else {
        j.error(user.getErrMsg());
    }
});

// DELETE - 사용자 삭제
api.delete(() -> {
    String id = f.get("id");
    if("".equals(id)) {
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

**장점:**
- `init.jsp`에서 기본 인증 처리 (userId, userName 설정)
- 각 API에서 필요한 추가 권한 체크만 수행
- 로그인 사용자 정보 자동 설정

---

## 에러 처리

### 1. 인증 실패 (401 Unauthorized)

```jsp
if(userId == 0) {
    api.error(401, "인증실패");
    return;  // 필수!
}
```

### 2. 권한 없음 (403 Forbidden)

```jsp
if(userLevel < 10) {
    api.error(403, "관리자만 접근할 수 있습니다.");
    return;
}
```

### 3. 잘못된 요청 (400 Bad Request)

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

### 4. 지원하는 에러 코드

- **401**: Unauthorized (인증 실패)
- **403**: Forbidden (권한 없음)
- **400**: Bad Request (잘못된 요청)

---

## 클라이언트 호출 예시

### JavaScript (fetch API)

```javascript
// GET /api/user - 목록 조회
fetch('/api/user')
    .then(response => response.json())
    .then(data => console.log(data));

// GET /api/user?id=123 - 단일 조회
fetch('/api/user?id=123')
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

// PUT /api/user?id=123 - 수정
fetch('/api/user?id=123', {
    method: 'PUT',
    headers: {
        'Content-Type': 'application/x-www-form-urlencoded',
    },
    body: 'name=김철수'
})
    .then(response => response.json())
    .then(data => console.log(data));

// DELETE /api/user?id=123 - 삭제
fetch('/api/user?id=123', {
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

### 5. 파라미터 전달

Path parameter 대신 표준 query string 사용:
- ✅ `/api/user?id=123` (권장)
- ❌ `/api/user/123` (미지원)

### 6. GET 파라미터 보안 (선택사항)

API에서도 GET 파라미터에 XSS 필터를 적용하려면 `m.rs()`/`m.ri()` 사용을 권장:

```jsp
api.get(() -> {
    int id = m.ri("id");  // XSS 필터 적용
    String keyword = m.rs("keyword");  // XSS 필터 적용

    UserDao user = new UserDao();
    if(id > 0) {
        DataSet info = user.get(id);
        // ...
    }
});
```

---

## 관련 문서

- [JSON 처리](json.md) - Json 클래스 상세 가이드
- [코딩 원칙 > AJAX 요청 처리](coding-principles.md#4-2-ajax-요청-처리) - AJAX 응답 방법
- [인증 처리](authentication.md) - Auth 클래스 활용

---

[← 목차로 돌아가기](README.md)
