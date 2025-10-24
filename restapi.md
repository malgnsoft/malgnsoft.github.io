# REST API 개발

맑은프레임워크의 **RestAPI 클래스**를 사용하면 HTTP 메소드(GET, POST, PUT, DELETE, PATCH)별로 분기 처리하여 RESTful API를 쉽게 개발할 수 있습니다.

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

라우팅 그룹을 등록하고 요청을 해당 JSP로 포워딩합니다.

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ page import="java.util.*, malgnsoft.util.*" %><%

RestAPI router = new RestAPI(request, response);

// CORS 설정 (모든 도메인 허용)
router.cors();

// OPTIONS 요청 (preflight) 처리
if(router.handlePreflight()) return;

// 라우팅 그룹 등록
router.use("/api/user", "/api/user.jsp");
router.use("/api/product", "/api/product.jsp");
router.use("/api/admin/user", "/api/admin/user.jsp");
router.use("/api/admin/stats", "/api/admin/stats.jsp");

// 등록된 라우트로 자동 포워딩
router.forward();

%>
```

또는 `dispatch()` 사용 (동일한 기능):
```jsp
router.dispatch();
```

**여러 도메인 허용:**
```jsp
// 여러 도메인 허용 - String 배열
router.cors(new String[]{
    "https://yourdomain.com",
    "https://app.yourdomain.com",
    "http://localhost:3000"
});
```

### 3. /api/init.jsp 공통 초기화 파일

API 폴더의 모든 JSP 파일에서 공통으로 사용할 객체와 인증을 초기화합니다.

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ page import="java.util.*, java.io.*, dao.*, malgnsoft.db.*, malgnsoft.util.*" %><%

RestAPI api = new RestAPI(request, response);

// CORS 설정 (외부 도메인 API 호출 허용)
api.cors();  // 모든 도메인 허용

// 또는 여러 도메인 허용 (화이트리스트)
// api.cors(new String[]{
//     "https://yourdomain.com",
//     "https://app.yourdomain.com",
//     "http://localhost:3000"
// });

// OPTIONS 요청 (preflight) 처리
if(api.handlePreflight()) return;

// 퍼블릭 라우팅 설정 (인증 불필요)
api.publicRoute(
    "/api/auth/login",
    "/api/auth/register",
    "/api/public/*"  // 와일드카드 사용 가능
);

Malgn m = new Malgn(request, response, out);

Form f = new Form();
f.setRequest(request);

Json j = new Json(out);

// JWT 토큰 인증 (퍼블릭 라우팅 자동 체크)
if(!api.auth()) return;

// 인증된 사용자 정보
int userId = api.getDataInt("user_id");
String userName = api.getData("user_name");
int userLevel = api.getDataInt("user_level");

%>
```

### 4. 퍼블릭 라우팅 설정

인증이 필요 없는 API 경로를 지정할 수 있습니다.

**퍼블릭 라우팅 지정:**
```jsp
// init.jsp에서 설정
api.publicRoute(
    "/api/auth/login",      // 로그인 API
    "/api/auth/register",   // 회원가입 API
    "/api/public/*"         // /api/public 하위 모든 경로
);
```

**와일드카드 패턴:**
- `/api/auth/*` - `/api/auth` 하위 모든 경로
- `/api/public/*` - `/api/public` 하위 모든 경로
- 정확히 일치하는 경로는 와일드카드 없이 지정

**인증 체크:**
```jsp
// init.jsp에서 자동 처리 - 매우 간단!
if(!api.auth()) return;

// 인증된 사용자 정보 자동 로드
int userId = api.getDataInt("user_id");
String userName = api.getData("user_name");
int userLevel = api.getDataInt("user_level");
```

**`api.auth()` 메소드 동작:**
1. 퍼블릭 라우팅이면 인증 불필요 (true 반환)
2. 퍼블릭이 아니면 JWT 토큰 검증
3. 토큰이 없거나 유효하지 않으면 에러 응답 자동 출력 (false 반환)
4. 토큰이 유효하면 사용자 정보 자동 로드 (true 반환)

**장점:**
- 퍼블릭 API는 인증 없이 접근 가능
- 나머지 API는 자동으로 인증 체크
- 각 API 파일에서 인증 코드 작성 불필요

### 5. 디렉토리 구조

```
webapp/
├── api/
│   ├── index.jsp          (라우터)
│   ├── init.jsp           (공통 초기화)
│   ├── user.jsp           → /api/user (인증 필요)
│   ├── product.jsp        → /api/product (인증 필요)
│   ├── auth/
│   │   ├── login.jsp      → /api/auth/login (퍼블릭)
│   │   └── register.jsp   → /api/auth/register (퍼블릭)
│   ├── public/
│   │   └── info.jsp       → /api/public/* (퍼블릭)
│   └── admin/
│       └── stats.jsp      → /api/admin/stats (인증 필요)
└── WEB-INF/
    └── web.xml
```

### 6. 동작 방식

- `/api/user` → `index.jsp` → `/api/user.jsp` → `api.get("/", ...)`
- `/api/user/123` → `index.jsp` → `/api/user.jsp` → `api.get("/:id", ...)` (path parameter)
- `/api/user?keyword=홍길동` → `index.jsp` → `/api/user.jsp` → `api.get("/", ...)` (query string)
- `/api/admin/stats` → `index.jsp` → `/api/admin/stats.jsp`

---

## web.xml 없이 직접 호출 방식

web.xml 설정 없이 JSP 파일을 직접 호출하여 REST API를 구현할 수도 있습니다.

### 언제 사용하나?

- **빠른 프로토타이핑**: web.xml 설정이 번거로울 때
- **간단한 API**: 복잡한 라우팅이 필요 없을 때
- **레거시 프로젝트**: web.xml을 수정할 수 없을 때

### 장단점

**장점:**
- ✅ web.xml 설정 불필요
- ✅ 간단한 구조
- ✅ 즉시 사용 가능

**단점:**
- ❌ URL에 `.jsp` 확장자 노출
- ❌ Path parameter 사용 불가 (경로는 `/`만 가능)
- ❌ RESTful 스타일 아님

### 사용 예시

#### /api/user.jsp (직접 호출)

```jsp
<%@ include file="/api/init.jsp" %><%

// GET /api/user.jsp - 목록 조회
api.get("/", () -> {
    UserDao user = new UserDao();
    DataSet list = user.find();
    j.success("조회되었습니다.", list);
});

// POST /api/user.jsp - 생성
api.post("/", () -> {
    UserDao user = new UserDao();
    user.item("name", f.get("name"));
    user.item("email", f.get("email"));

    if(user.insert()) {
        j.success("등록되었습니다.", user.id);
    } else {
        j.error("DATABASE_ERROR", user.getErrMsg());
    }
});

// PUT /api/user.jsp - 수정 (ID는 query string으로 전달)
api.put("/", () -> {
    int id = m.ri("id");  // query string에서 id 추출

    UserDao user = new UserDao();
    user.get(id);
    user.item("name", f.get("name"));
    user.item("email", f.get("email"));

    if(user.update()) {
        j.success("수정되었습니다.");
    } else {
        j.error("DATABASE_ERROR", user.getErrMsg());
    }
});

// DELETE /api/user.jsp - 삭제 (ID는 query string으로 전달)
api.delete("/", () -> {
    int id = m.ri("id");  // query string에서 id 추출

    UserDao user = new UserDao();
    if(user.delete(id)) {
        j.success("삭제되었습니다.");
    } else {
        j.error("DATABASE_ERROR", user.getErrMsg());
    }
});

%>
```

**주의:**
- `init.jsp`를 include하면 CORS, OPTIONS, 인증 처리가 자동으로 적용됩니다
- `api`, `m`, `f`, `j` 객체가 자동으로 생성됩니다
- JWT 인증이 필요하면 `init.jsp`에서 `api.publicRoute()`로 퍼블릭 경로를 지정하세요

### 클라이언트 호출

```javascript
// GET /api/user.jsp - 목록 조회
fetch('/api/user.jsp')
    .then(response => response.json())
    .then(data => console.log(data));

// POST /api/user.jsp - 생성
fetch('/api/user.jsp', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/x-www-form-urlencoded',
    },
    body: 'name=홍길동&email=hong@example.com'
})
    .then(response => response.json())
    .then(data => console.log(data));

// PUT /api/user.jsp?id=123 - 수정 (query string으로 ID 전달)
fetch('/api/user.jsp?id=123', {
    method: 'PUT',
    headers: {
        'Content-Type': 'application/x-www-form-urlencoded',
    },
    body: 'name=김철수&email=kim@example.com'
})
    .then(response => response.json())
    .then(data => console.log(data));

// DELETE /api/user.jsp?id=123 - 삭제 (query string으로 ID 전달)
fetch('/api/user.jsp?id=123', {
    method: 'DELETE'
})
    .then(response => response.json())
    .then(data => console.log(data));
```

### 주의사항

1. **경로는 `/`만 사용 가능**
   - ✅ `api.get("/", ...)` - 작동
   - ❌ `api.get("/:id", ...)` - 작동 안 함 (path parameter 불가)
   - ✅ Query string 사용: `/api/user.jsp?id=123`

2. **모든 HTTP 메소드 지원**
   - GET, POST, PUT, DELETE, PATCH 모두 사용 가능

3. **init.jsp include 권장**
   - `init.jsp`를 include하면 CORS, OPTIONS, JWT 인증이 자동으로 처리됩니다
   - `api`, `m`, `f`, `j` 객체가 자동으로 생성되어 편리합니다
   - 각 파일마다 객체 생성 코드를 작성할 필요가 없습니다

4. **인증이 필요 없는 경우**
   - 퍼블릭 API는 `init.jsp`에서 `api.publicRoute()`로 지정
   - 또는 `init.jsp`의 `api.auth()` 부분을 주석 처리

**권장사항:** 프로덕션 환경에서는 web.xml 라우팅 방식을 권장합니다.

---

## 기본 사용법

### /api/user.jsp 예시 (라우팅 그룹 방식)

```jsp
<%@ include file="/api/init.jsp" %><%

// 이 파일의 base path 설정 (선택사항 - 생략하면 자동으로 /api/user 사용)
api.setBasePath("/api/user");

// GET /api/user - 목록 조회
api.get("/", () -> {
    UserDao user = new UserDao();
    DataSet list = user.find();
    j.add("users", list);
    j.print();
});

// GET /api/user/list - 페이징 목록 (고정 경로)
api.get("/list", () -> {
    int page = m.ri("page", 1);
    int size = m.ri("size", 20);

    UserDao user = new UserDao();
    DataSet list = user.findWithPaging(page, size);
    int total = user.getTotal();

    j.add("users", list);
    j.add("total", total);
    j.add("page", page);
    j.add("size", size);
    j.print();
});

// GET /api/user/active - 활성 사용자만 (고정 경로)
api.get("/active", () -> {
    UserDao user = new UserDao();
    DataSet list = user.findByStatus("active");
    j.add("users", list);
    j.print();
});

// GET /api/user/:id - 단일 조회 (path parameter)
api.get("/:id", () -> {
    int id = api.getParamInt("id");  // path parameter에서 id 추출

    UserDao user = new UserDao();
    DataSet info = user.get(id);

    if(info.next()) {
        j.add("id", info.i("id"));
        j.add("name", info.s("name"));
        j.add("email", info.s("email"));
        j.print();
    } else {
        j.error("사용자를 찾을 수 없습니다.");
    }
});

// POST /api/user - 생성
api.post("/", () -> {
    UserDao user = new UserDao();
    user.item("name", f.get("name"));
    user.item("email", f.get("email"));

    if(user.insert()) {
        j.success("등록되었습니다.", user.id);
    } else {
        j.error(user.getErrMsg());
    }
});

// PUT /api/user/:id - 수정 (path parameter)
api.put("/:id", () -> {
    int id = api.getParamInt("id");

    UserDao user = new UserDao();
    user.get(id);
    user.item("name", f.get("name"));
    user.item("email", f.get("email"));

    if(user.update()) {
        j.success("수정되었습니다.");
    } else {
        j.error(user.getErrMsg());
    }
});

// PATCH /api/user/:id - 부분 수정 (path parameter)
api.patch("/:id", () -> {
    int id = api.getParamInt("id");

    UserDao user = new UserDao();
    user.get(id);

    // 전달된 필드만 업데이트
    String name = f.get("name");
    if(!"".equals(name)) user.item("name", name);

    String email = f.get("email");
    if(!"".equals(email)) user.item("email", email);

    if(user.update()) {
        j.success("수정되었습니다.");
    } else {
        j.error(user.getErrMsg());
    }
});

// DELETE /api/user/:id - 삭제 (path parameter)
api.delete("/:id", () -> {
    int id = api.getParamInt("id");

    UserDao user = new UserDao();
    if(user.delete(id)) {
        j.success("삭제되었습니다.");
    } else {
        j.error(user.getErrMsg());
    }
});

%>
```

**Base Path 자동 설정:**
- `RestAPI` 객체 생성 시 현재 JSP 파일 경로에서 자동 설정됩니다
- 예: `/api/user.jsp` → basePath는 `/api/user`
- 예: `/api/admin/stats.jsp` → basePath는 `/api/admin/stats`
- 다르게 사용하고 싶으면 `api.setBasePath("/api/user")` 호출

**장점:**
- **라우팅 그룹**: `/api/user` 관련 모든 경로를 user.jsp에서 처리
- **Path parameter 지원**: `/:id` 패턴으로 RESTful URL 구현 가능
- **Express.js 스타일**: 익숙한 라우팅 패턴
- **계층적 구조**: 리소스별로 파일 분리, 관리 용이
- **자동 base path**: 파일 경로에서 자동 설정되어 편리
- 각 API 파일은 비즈니스 로직만 작성

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
if(userLevel < 10) {
    api.error(403, "관리자만 접근할 수 있습니다.");
    return;
}

// GET /api/admin/user - 사용자 목록 조회
api.get("/", () -> {
    UserDao user = new UserDao();
    DataSet list = user.find();
    j.add("users", list);
    j.print();
});

// GET /api/admin/user/:id - 특정 사용자 조회
api.get("/:id", () -> {
    int id = api.getParamInt("id");

    UserDao user = new UserDao();
    DataSet info = user.get(id);

    if(info.next()) {
        j.add("user", info);
        j.print();
    } else {
        j.error("사용자를 찾을 수 없습니다.");
    }
});

// POST /api/admin/user - 사용자 생성
api.post("/", () -> {
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

// PUT /api/admin/user/:id - 사용자 수정
api.put("/:id", () -> {
    int id = api.getParamInt("id");

    UserDao user = new UserDao();
    user.get(id);
    user.item("name", f.get("name"));
    user.item("email", f.get("email"));

    if(user.update()) {
        j.success("수정되었습니다.");
    } else {
        j.error(user.getErrMsg());
    }
});

// DELETE /api/admin/user/:id - 사용자 삭제
api.delete("/:id", () -> {
    int id = api.getParamInt("id");

    UserDao user = new UserDao();
    if(user.delete(id)) {
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
api.post("/", () -> {
    String name = f.get("name");
    if("".equals(name)) {
        api.error(400, "이름은 필수입니다.");
        return;
    }

    // 정상 처리
    j.success("처리되었습니다.");
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

// GET /api/user/list - 페이징 목록 (고정 경로)
fetch('/api/user/list?page=1&size=20')
    .then(response => response.json())
    .then(data => console.log(data));

// GET /api/user/active - 활성 사용자 (고정 경로)
fetch('/api/user/active')
    .then(response => response.json())
    .then(data => console.log(data));

// GET /api/user/123 - 단일 조회 (path parameter)
fetch('/api/user/123')
    .then(response => response.json())
    .then(data => console.log(data));

// GET /api/user?keyword=홍길동 - 검색 (query string)
fetch('/api/user?keyword=홍길동')
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

// PUT /api/user/123 - 전체 수정 (path parameter)
fetch('/api/user/123', {
    method: 'PUT',
    headers: {
        'Content-Type': 'application/x-www-form-urlencoded',
    },
    body: 'name=김철수&email=kim@example.com'
})
    .then(response => response.json())
    .then(data => console.log(data));

// PATCH /api/user/123 - 부분 수정 (path parameter)
fetch('/api/user/123', {
    method: 'PATCH',
    headers: {
        'Content-Type': 'application/x-www-form-urlencoded',
    },
    body: 'name=김철수'  // 이름만 수정
})
    .then(response => response.json())
    .then(data => console.log(data));

// DELETE /api/user/123 - 삭제 (path parameter)
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

### 5. 파라미터 전달

**Path parameter (권장):**
```jsp
// /:id 패턴
api.get("/:id", () -> {
    int id = api.getParamInt("id");  // /api/user/123 → id=123
    // ...
});

// 복수 parameter
api.get("/:category/:id", () -> {
    String category = api.getParam("category");  // /api/product/food/123
    int id = api.getParamInt("id");
    // ...
});
```

**Query string (검색, 필터링):**
```jsp
api.get("/", () -> {
    String keyword = m.rs("keyword");  // /api/user?keyword=홍길동
    int page = m.ri("page");           // /api/user?page=2
    // ...
});
```

### 6. GET 파라미터 보안 (선택사항)

API에서도 GET 파라미터에 XSS 필터를 적용하려면 `m.rs()`/`m.ri()` 사용을 권장:

```jsp
api.get("/", () -> {
    String keyword = m.rs("keyword");  // XSS 필터 적용
    int page = m.ri("page");           // XSS 필터 적용

    UserDao user = new UserDao();
    DataSet list = user.search(keyword, page);
    j.add("users", list);
    j.print();
});
```

---

## 고급 라우팅

복잡한 Path Parameter 패턴과 RESTful 라우팅이 필요하다면 [고급 라우팅 가이드](restapi-advanced.md)를 참고하세요.

**주요 내용:**
- 고정 경로와 Path Parameter 혼합 사용
- 복수 Parameter 패턴 (`/:category/:id`)
- 매칭 우선순위
- Query String과 함께 사용
- 중첩 리소스 라우팅 (예: `/api/post/:id/comments`)

---

## 추가 기능

### JWT 인증

JWT(JSON Web Token) 기반 인증을 지원합니다. 자세한 내용은 [JWT 인증 가이드](restapi-jwt.md)를 참고하세요.

**주요 기능:**
- JWT 토큰 생성 및 검증
- 퍼블릭 라우팅 (인증 불필요 경로)
- 자동 사용자 정보 로드
- Bearer 토큰 자동 추출

### CORS 설정

외부 도메인에서 API 호출 시 필요한 CORS 헤더 설정을 지원합니다. 자세한 내용은 [CORS 설정 가이드](restapi-cors.md)를 참고하세요.

**주요 기능:**
- 모든 도메인 또는 특정 도메인만 허용
- 여러 도메인 화이트리스트
- OPTIONS Preflight 자동 처리
- Credentials 지원

### 응답 표준

일관된 JSON 응답 형식을 제공합니다. 자세한 내용은 [응답 표준 가이드](restapi-response.md)를 참고하세요.

**주요 기능:**
- 표준 에러 응답 (`success`, `error`, `message`, `details`)
- 표준 성공 응답 (`success`, `message`, `data`)
- 페이징 응답 표준
- HTTP 상태 코드 설정

---

## 관련 문서

- **[고급 라우팅](restapi-advanced.md)** - Path Parameter 상세, RESTful 패턴
- **[JWT 인증](restapi-jwt.md)** - JWT 토큰 생성, 검증, 퍼블릭 라우팅
- **[CORS 설정](restapi-cors.md)** - CORS 헤더 설정, 도메인 허용
- **[응답 표준](restapi-response.md)** - 에러/성공 응답, 페이징
- **[JSON 처리](json.md)** - Json 클래스 상세 가이드
- **[인증 처리](authentication.md)** - Auth 클래스 활용 (세션 기반)

---

[← 목차로 돌아가기](README.md)
