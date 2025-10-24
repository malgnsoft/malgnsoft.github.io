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

라우팅 그룹을 등록하고 요청을 해당 JSP로 포워딩합니다.

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ page import="malgnsoft.util.*" %><%

RestAPI router = new RestAPI(request, response);

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

### 3. /api/init.jsp 공통 초기화 파일

API 폴더의 모든 JSP 파일에서 공통으로 사용할 객체와 인증을 초기화합니다.

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ page import="java.util.*, java.io.*, dao.*, malgnsoft.db.*, malgnsoft.util.*" %><%

Malgn m = new Malgn(request, response, out);

Form f = new Form();
f.setRequest(request);

Auth auth = new Auth(request, response);

Json j = new Json();

RestAPI api = new RestAPI(request, response);

// JWT 토큰 인증
int userId = 0;
String userName = null;
int userLevel = 0;

if(auth.isValidToken()) {
    userId = auth.getInt("user_id");
    userName = auth.getString("user_name");
    userLevel = auth.getInt("user_level");
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

- `/api/user` → `index.jsp` → `/api/user.jsp` → `api.get("/", ...)`
- `/api/user/123` → `index.jsp` → `/api/user.jsp` → `api.get("/:id", ...)` (path parameter)
- `/api/user?keyword=홍길동` → `index.jsp` → `/api/user.jsp` → `api.get("/", ...)` (query string)
- `/api/admin/stats` → `index.jsp` → `/api/admin/stats.jsp`

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

// PUT /api/user/123 - 수정 (path parameter)
fetch('/api/user/123', {
    method: 'PUT',
    headers: {
        'Content-Type': 'application/x-www-form-urlencoded',
    },
    body: 'name=김철수'
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

## Path Parameter 상세

### 지원 패턴

```jsp
// 1. 고정 경로 (fixed path)
api.get("/list", () -> {
    // /api/user/list
});

api.get("/active", () -> {
    // /api/user/active
});

api.get("/search", () -> {
    String keyword = m.rs("keyword");
    // /api/user/search?keyword=홍길동
});

// 2. 단일 parameter
api.get("/:id", () -> {
    int id = api.getParamInt("id");
    // /api/user/123 → id=123
});

// 3. 복수 parameter
api.get("/:category/:id", () -> {
    String category = api.getParam("category");
    int id = api.getParamInt("id");
    // /api/product/food/123 → category=food, id=123
});

// 4. 혼합 사용 (고정 경로 + parameter)
api.get("/:id/comments", () -> {
    int id = api.getParamInt("id");
    // /api/post/123/comments → id=123
});

api.get("/:id/orders", () -> {
    int id = api.getParamInt("id");
    // /api/user/123/orders → id=123
});
```

**매칭 우선순위:**
1. 고정 경로 먼저 매칭 (예: `/list`, `/active`)
2. Path parameter 패턴 매칭 (예: `/:id`)

따라서 `/api/user/list`는 `/:id`보다 `/list`에 먼저 매칭됩니다.

### Parameter 추출 메소드

| 메소드 | 반환 타입 | 설명 | 예시 |
|--------|----------|------|------|
| `api.getParam(name)` | String | 문자열로 반환 | `String name = api.getParam("category");` |
| `api.getParamInt(name)` | int | 정수로 변환 (실패시 0) | `int id = api.getParamInt("id");` |

### Query string과 함께 사용

Path parameter와 query string을 동시에 사용할 수 있습니다:

```jsp
// GET /api/user/123?includeOrders=true
api.get("/:id", () -> {
    int id = api.getParamInt("id");                    // path parameter
    boolean includeOrders = m.rb("includeOrders");     // query string

    UserDao user = new UserDao();
    DataSet info = user.get(id);

    if(info.next()) {
        j.add("user", info);

        if(includeOrders) {
            OrderDao order = new OrderDao();
            DataSet orders = order.getByUserId(id);
            j.add("orders", orders);
        }

        j.print();
    }
});
```

### RESTful 라우팅 예시

```jsp
<%@ include file="/api/init.jsp" %><%

api.setBasePath("/api/product");

// GET /api/product - 전체 목록
api.get("/", () -> {
    String keyword = m.rs("keyword");
    int page = m.ri("page", 1);

    ProductDao product = new ProductDao();
    DataSet list = product.find(keyword, page);
    j.add("products", list);
    j.print();
});

// GET /api/product/:category - 카테고리별 목록
api.get("/:category", () -> {
    String category = api.getParam("category");

    ProductDao product = new ProductDao();
    DataSet list = product.findByCategory(category);
    j.add("products", list);
    j.add("category", category);
    j.print();
});

// GET /api/product/:category/:id - 특정 상품 조회
api.get("/:category/:id", () -> {
    String category = api.getParam("category");
    int id = api.getParamInt("id");

    ProductDao product = new ProductDao();
    DataSet info = product.get(id);

    if(info.next() && category.equals(info.s("category"))) {
        j.add("product", info);
        j.print();
    } else {
        j.error("상품을 찾을 수 없습니다.");
    }
});

// POST /api/product - 상품 생성
api.post("/", () -> {
    ProductDao product = new ProductDao();
    product.item("name", f.get("name"));
    product.item("category", f.get("category"));
    product.item("price", f.get("price"));

    if(product.insert()) {
        j.success("등록되었습니다.", product.id);
    } else {
        j.error(product.getErrMsg());
    }
});

// PUT /api/product/:id - 상품 수정
api.put("/:id", () -> {
    int id = api.getParamInt("id");

    ProductDao product = new ProductDao();
    product.get(id);
    product.item("name", f.get("name"));
    product.item("price", f.get("price"));

    if(product.update()) {
        j.success("수정되었습니다.");
    } else {
        j.error(product.getErrMsg());
    }
});

// DELETE /api/product/:id - 상품 삭제
api.delete("/:id", () -> {
    int id = api.getParamInt("id");

    ProductDao product = new ProductDao();
    if(product.delete(id)) {
        j.success("삭제되었습니다.");
    } else {
        j.error(product.getErrMsg());
    }
});

%>
```

**클라이언트에서 호출:**
```javascript
// GET /api/product - 전체 목록
fetch('/api/product')

// GET /api/product?keyword=노트북 - 검색
fetch('/api/product?keyword=노트북')

// GET /api/product/electronics - 전자제품 카테고리
fetch('/api/product/electronics')

// GET /api/product/electronics/123 - 특정 상품
fetch('/api/product/electronics/123')

// POST /api/product - 상품 생성
fetch('/api/product', {
    method: 'POST',
    body: 'name=노트북&category=electronics&price=1000000'
})

// PUT /api/product/123 - 상품 수정
fetch('/api/product/123', {
    method: 'PUT',
    body: 'name=게이밍노트북&price=1500000'
})

// DELETE /api/product/123 - 상품 삭제
fetch('/api/product/123', { method: 'DELETE' })
```

---

## JWT 인증

맑은프레임워크는 `Auth` 클래스에서 JWT(JSON Web Token) 기반 인증을 지원합니다. 세션 기반 인증과 달리 stateless API에 적합합니다.

### 1. JWT 토큰 생성

로그인 성공 시 JWT 토큰을 생성하여 클라이언트에 전달합니다.

#### /api/auth/login.jsp

```jsp
<%@ include file="/api/init.jsp" %><%

api.post(() -> {
    String email = f.get("email");
    String password = f.get("password");

    // 사용자 인증 확인
    UserDao user = new UserDao();
    DataSet info = user.getByEmail(email);

    if(info.next() && user.checkPassword(password, info.s("password"))) {
        // Auth 객체에 사용자 정보 저장
        auth.put("user_id", info.i("id"));
        auth.put("user_name", info.s("name"));
        auth.put("user_level", info.i("level"));

        // JWT 토큰 생성 (60분 유효)
        String token = auth.generateToken(60);

        j.add("token", token);
        j.add("user_id", info.i("id"));
        j.add("user_name", info.s("name"));
        j.add("user_level", info.i("level"));
        j.print();
    } else {
        api.error(401, "이메일 또는 비밀번호가 일치하지 않습니다.");
    }
});

%>
```

### 2. JWT 토큰 검증

`/api/init.jsp`에서 자동으로 JWT 토큰을 검증합니다:

```jsp
// JWT 토큰 인증
if(auth.isValidToken()) {
    userId = auth.getInt("user_id");
    userName = auth.getString("user_name");
    userLevel = auth.getInt("user_level");
}
```

`auth.isValidToken()`은 다음을 자동으로 처리합니다:
1. `Authorization: Bearer <token>` 헤더에서 토큰 추출 (또는 `auth.setToken()`으로 설정된 토큰 사용)
2. 토큰 서명 검증
3. 토큰 만료 시간 확인
4. 토큰의 Claims를 Auth 객체에 저장

#### 파라미터로 토큰 전달 (선택사항)

Authorization 헤더를 사용하지 않고 파라미터로 토큰을 전달하는 경우:

```jsp
// /api/init.jsp 수정
String token = f.get("token");  // 파라미터에서 토큰 추출
if(token != null && !"".equals(token)) {
    auth.setToken(token);  // 토큰 설정
}

// JWT 토큰 인증
if(auth.isValidToken()) {
    userId = auth.getInt("user_id");
    userName = auth.getString("user_name");
    userLevel = auth.getInt("user_level");
}
```

클라이언트에서 사용:
```javascript
// URL 파라미터로 토큰 전달
fetch('/api/user?token=' + token)
    .then(response => response.json())
    .then(data => console.log(data));
```

### 3. 클라이언트 사용법

#### Authorization 헤더 방식 (권장)

JavaScript (fetch API):

```javascript
// 1. 로그인하여 토큰 받기
fetch('/api/auth/login', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/x-www-form-urlencoded',
    },
    body: 'email=user@example.com&password=1234'
})
.then(response => response.json())
.then(data => {
    // 토큰을 로컬스토리지에 저장
    localStorage.setItem('token', data.token);
    console.log('로그인 성공:', data);
});

// 2. 인증이 필요한 API 호출 시 토큰 전송
const token = localStorage.getItem('token');

fetch('/api/user', {
    method: 'GET',
    headers: {
        'Authorization': `Bearer ${token}`
    }
})
.then(response => response.json())
.then(data => console.log(data));

// 3. POST 요청에도 토큰 전송
fetch('/api/user', {
    method: 'POST',
    headers: {
        'Authorization': `Bearer ${token}`,
        'Content-Type': 'application/x-www-form-urlencoded',
    },
    body: 'name=홍길동&email=hong@example.com'
})
.then(response => response.json())
.then(data => console.log(data));
```

### 4. 권한 체크 예시

JWT 토큰으로 인증된 사용자의 권한을 체크합니다.

#### /api/admin/stats.jsp

```jsp
<%@ include file="/api/init.jsp" %><%

// 로그인 체크
if(userId == 0) {
    api.error(401, "로그인이 필요합니다.");
    return;
}

// 관리자 권한 체크 (userLevel은 init.jsp에서 자동 설정됨)
if(userLevel < 10) {
    api.error(403, "관리자만 접근할 수 있습니다.");
    return;
}

// GET - 통계 조회
api.get(() -> {
    StatsDao stats = new StatsDao();
    DataSet data = stats.getMonthlyStats();

    j.add("stats", data);
    j.add("requested_by", userName);  // JWT 토큰에서 추출된 userName 사용
    j.print();
});

%>
```

### 5. 보안 고려사항

1. **HTTPS 필수**: JWT 토큰은 반드시 HTTPS를 통해 전송해야 합니다.
2. **토큰 만료 시간**: 보안을 위해 짧은 만료 시간(30~60분)을 권장합니다.
3. **Refresh Token**: 장기 로그인이 필요한 경우 Refresh Token 패턴 구현을 고려하세요.
4. **비밀키 관리**: `Config.getSecretId()`로 반환되는 비밀키는 안전하게 보관하세요.
5. **토큰 저장**: XSS 공격 방지를 위해 `httpOnly` 쿠키 저장도 고려하세요.

---

## CORS 설정

외부 도메인에서 API를 호출할 때 필요한 CORS(Cross-Origin Resource Sharing) 헤더 설정입니다.

### 1. init.jsp에 CORS 헤더 추가

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ page import="java.util.*, java.io.*, dao.*, malgnsoft.db.*, malgnsoft.util.*" %><%

// CORS 헤더 설정
response.setHeader("Access-Control-Allow-Origin", "*");  // 모든 도메인 허용 (또는 특정 도메인)
response.setHeader("Access-Control-Allow-Methods", "GET, POST, PUT, DELETE, PATCH, OPTIONS");
response.setHeader("Access-Control-Allow-Headers", "Content-Type, Authorization");
response.setHeader("Access-Control-Max-Age", "3600");  // preflight 캐시 1시간

// OPTIONS 요청 (preflight) 처리
if("OPTIONS".equals(request.getMethod())) {
    response.setStatus(HttpServletResponse.SC_OK);
    return;
}

Malgn m = new Malgn(request, response, out);
// ... 나머지 초기화
%>
```

### 2. 특정 도메인만 허용

```jsp
// 특정 도메인만 허용
String origin = request.getHeader("Origin");
if("https://yourdomain.com".equals(origin) || "https://app.yourdomain.com".equals(origin)) {
    response.setHeader("Access-Control-Allow-Origin", origin);
    response.setHeader("Access-Control-Allow-Credentials", "true");  // 쿠키 전송 허용
}
```

### 3. 동적 도메인 허용

```jsp
// 화이트리스트 기반
String origin = request.getHeader("Origin");
List<String> allowedOrigins = Arrays.asList(
    "https://yourdomain.com",
    "https://app.yourdomain.com",
    "http://localhost:3000"  // 개발 환경
);

if(allowedOrigins.contains(origin)) {
    response.setHeader("Access-Control-Allow-Origin", origin);
    response.setHeader("Access-Control-Allow-Credentials", "true");
}
```

### 4. 클라이언트에서 CORS 요청

```javascript
// 쿠키를 포함한 요청 (credentials)
fetch('https://api.yourdomain.com/api/user', {
    method: 'GET',
    credentials: 'include',  // 쿠키 전송
    headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${token}`
    }
})
.then(response => response.json())
.then(data => console.log(data));
```

---

## 에러 응답 표준

일관된 에러 응답 형식을 사용하면 클라이언트에서 처리하기 쉽습니다.

### 1. Json 클래스 사용 (권장)

```jsp
api.get("/:id", () -> {
    int id = api.getParamInt("id");

    UserDao user = new UserDao();
    DataSet info = user.get(id);

    if(info.next()) {
        j.add("user", info);
        j.print();
    } else {
        // 에러 응답
        j.error("사용자를 찾을 수 없습니다.");
        // 또는 상세 에러
        j.add("error", "NOT_FOUND");
        j.add("message", "사용자를 찾을 수 없습니다.");
        j.add("id", id);
        j.print();
    }
});
```

### 2. HTTP 상태 코드와 함께

```jsp
api.post("/", () -> {
    String email = f.get("email");
    String name = f.get("name");

    // 유효성 검사
    if("".equals(email)) {
        response.setStatus(400);  // Bad Request
        j.add("error", "VALIDATION_ERROR");
        j.add("message", "이메일은 필수입니다.");
        j.add("field", "email");
        j.print();
        return;
    }

    UserDao user = new UserDao();
    user.item("email", email);
    user.item("name", name);

    if(user.insert()) {
        response.setStatus(201);  // Created
        j.success("등록되었습니다.", user.id);
    } else {
        response.setStatus(500);  // Internal Server Error
        j.add("error", "DATABASE_ERROR");
        j.add("message", user.getErrMsg());
        j.print();
    }
});
```

### 3. 표준 에러 응답 형식

```json
{
  "success": false,
  "error": "ERROR_CODE",
  "message": "사용자에게 보여줄 메시지",
  "details": {
    "field": "email",
    "value": "invalid@email"
  }
}
```

### 4. 주요 에러 코드 예시

| HTTP 상태 | 에러 코드 | 설명 |
|-----------|----------|------|
| 400 | `VALIDATION_ERROR` | 유효성 검사 실패 |
| 401 | `UNAUTHORIZED` | 인증 실패 |
| 403 | `FORBIDDEN` | 권한 없음 |
| 404 | `NOT_FOUND` | 리소스 없음 |
| 409 | `CONFLICT` | 중복 (이메일 등) |
| 500 | `SERVER_ERROR` | 서버 오류 |

---

## 페이징 응답 표준

페이징 정보를 포함한 일관된 응답 형식입니다.

### 1. 기본 페이징 응답

```jsp
api.get("/", () -> {
    int page = m.ri("page", 1);
    int size = m.ri("size", 20);

    UserDao user = new UserDao();
    DataSet list = user.findWithPaging(page, size);
    int total = user.getTotal();

    // 페이징 응답
    j.add("data", list);  // 실제 데이터
    j.add("pagination", new JSONObject()
        .put("page", page)
        .put("size", size)
        .put("total", total)
        .put("totalPages", (int) Math.ceil((double) total / size))
    );
    j.print();
});
```

**응답 예시:**
```json
{
  "success": true,
  "data": [
    {"id": 1, "name": "홍길동"},
    {"id": 2, "name": "김철수"}
  ],
  "pagination": {
    "page": 1,
    "size": 20,
    "total": 156,
    "totalPages": 8
  }
}
```

### 2. Json 클래스 활용

```jsp
api.get("/", () -> {
    int page = m.ri("page", 1);
    int size = m.ri("size", 20);
    String keyword = m.rs("keyword");

    UserDao user = new UserDao();
    DataSet list = user.search(keyword, page, size);
    int total = user.getTotal();

    j.add("users", list);
    j.add("total", total);
    j.add("page", page);
    j.add("size", size);
    j.add("totalPages", (int) Math.ceil((double) total / size));
    j.add("hasNext", page < Math.ceil((double) total / size));
    j.add("hasPrev", page > 1);
    j.print();
});
```

### 3. 커서 기반 페이징 (무한 스크롤)

```jsp
api.get("/", () -> {
    int cursor = m.ri("cursor", 0);  // 마지막 ID
    int size = m.ri("size", 20);

    UserDao user = new UserDao();
    DataSet list = user.findByCursor(cursor, size);

    int nextCursor = 0;
    if(list.last()) {
        nextCursor = list.i("id");
    }

    j.add("data", list);
    j.add("cursor", nextCursor);
    j.add("hasMore", list.size() == size);  // size만큼 가져왔으면 더 있을 가능성
    j.print();
});
```

**클라이언트 사용:**
```javascript
let cursor = 0;

function loadMore() {
    fetch(`/api/user?cursor=${cursor}&size=20`)
        .then(response => response.json())
        .then(data => {
            // 데이터 추가
            appendUsers(data.data);

            // 다음 커서 저장
            cursor = data.cursor;

            // 더 있으면 버튼 표시
            if(data.hasMore) {
                showLoadMoreButton();
            }
        });
}
```

### 4. 메타데이터 포함 응답

```jsp
j.add("data", list);
j.add("meta", new JSONObject()
    .put("total", total)
    .put("page", page)
    .put("size", size)
    .put("totalPages", totalPages)
    .put("timestamp", System.currentTimeMillis())
    .put("version", "v1")
);
j.print();
```

---

## 관련 문서

- [JSON 처리](json.md) - Json 클래스 상세 가이드
- [코딩 원칙 > AJAX 요청 처리](coding-principles.md#4-2-ajax-요청-처리) - AJAX 응답 방법
- [인증 처리](authentication.md) - Auth 클래스 활용

---

[← 목차로 돌아가기](README.md)
