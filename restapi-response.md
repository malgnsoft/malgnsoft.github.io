# REST API - 응답 표준

일관된 JSON 응답 형식을 사용하면 클라이언트에서 처리하기 쉽습니다.

---

## 에러 응답 표준

### 1. 기본 에러 응답

**간단한 에러 메시지:**
```jsp
api.get("/:id", () -> {
    int id = api.getParamInt("id");

    UserDao user = new UserDao();
    DataSet info = user.get(id);

    if(info.next()) {
        j.success(info);
    } else {
        // 에러 응답
        j.error("사용자를 찾을 수 없습니다.");
    }
});
```

**응답:**
```json
{
  "success": false,
  "error": "ERROR",
  "message": "사용자를 찾을 수 없습니다."
}
```

---

### 2. 에러 코드 포함

**에러 코드와 메시지:**
```jsp
api.get("/:id", () -> {
    int id = api.getParamInt("id");

    UserDao user = new UserDao();
    DataSet info = user.get(id);

    if(info.next()) {
        j.success(info);
    } else {
        j.error("NOT_FOUND", "사용자를 찾을 수 없습니다.");
    }
});
```

**응답:**
```json
{
  "success": false,
  "error": "NOT_FOUND",
  "message": "사용자를 찾을 수 없습니다."
}
```

---

### 3. 상세 정보 포함

#### 방법 1: put()으로 details 설정 (간편함)

```jsp
api.post("/", () -> {
    String email = f.get("email");

    // 유효성 검사
    if("".equals(email)) {
        // put()으로 details 데이터 설정
        j.put("field", "email");
        j.put("reason", "required");

        // error() 호출 시 자동으로 details에 포함됨
        j.error("VALIDATION_ERROR", "이메일은 필수입니다.");
        return;
    }

    // 이메일 형식 체크
    if(!email.contains("@")) {
        j.put("field", "email");
        j.put("value", email);
        j.put("reason", "invalid_format");

        j.error("VALIDATION_ERROR", "이메일 형식이 올바르지 않습니다.");
        return;
    }

    // 정상 처리
    UserDao user = new UserDao();
    user.item("email", email);

    if(user.insert()) {
        j.success("등록되었습니다.", user.id);
    } else {
        j.error("DATABASE_ERROR", user.getErrMsg());
    }
});
```

#### 방법 2: Map으로 details 직접 전달

```jsp
api.post("/", () -> {
    String email = f.get("email");

    if("".equals(email)) {
        Map<String, Object> details = new HashMap<>();
        details.put("field", "email");
        details.put("reason", "required");

        j.error("VALIDATION_ERROR", "이메일은 필수입니다.", details);
        return;
    }

    // ...
});
```

**응답:**
```json
{
  "success": false,
  "error": "VALIDATION_ERROR",
  "message": "이메일은 필수입니다.",
  "details": {
    "field": "email",
    "reason": "required"
  }
}
```

---

### 4. HTTP 상태 코드와 함께

```jsp
api.post("/", () -> {
    String email = f.get("email");

    if("".equals(email)) {
        response.setStatus(400);  // Bad Request
        j.error("VALIDATION_ERROR", "이메일은 필수입니다.");
        return;
    }

    UserDao user = new UserDao();
    user.item("email", email);

    if(user.insert()) {
        response.setStatus(201);  // Created
        j.success("등록되었습니다.", user.id);
    } else {
        response.setStatus(500);  // Internal Server Error
        j.error("DATABASE_ERROR", user.getErrMsg());
    }
});
```

---

### 5. 주요 에러 코드 예시

| HTTP 상태 | 에러 코드 | 설명 | 사용 예시 |
|-----------|----------|------|----------|
| 400 | `VALIDATION_ERROR` | 유효성 검사 실패 | `j.error("VALIDATION_ERROR", "이메일은 필수입니다.")` |
| 401 | `UNAUTHORIZED` | 인증 실패 | `j.error("UNAUTHORIZED", "로그인이 필요합니다.")` |
| 403 | `FORBIDDEN` | 권한 없음 | `j.error("FORBIDDEN", "관리자만 접근할 수 있습니다.")` |
| 404 | `NOT_FOUND` | 리소스 없음 | `j.error("NOT_FOUND", "사용자를 찾을 수 없습니다.")` |
| 409 | `CONFLICT` | 중복 (이메일 등) | `j.error("CONFLICT", "이미 등록된 이메일입니다.")` |
| 500 | `DATABASE_ERROR` | 데이터베이스 오류 | `j.error("DATABASE_ERROR", user.getErrMsg())` |
| 500 | `SERVER_ERROR` | 서버 오류 | `j.error("SERVER_ERROR", "서버 오류가 발생했습니다.")` |

---

## 성공 응답 표준

### 1. 기본 성공 응답

```jsp
j.success();
```

**응답:**
```json
{
  "success": true,
  "message": "success"
}
```

---

### 2. 메시지 포함

```jsp
j.success("등록되었습니다.");
```

**응답:**
```json
{
  "success": true,
  "message": "등록되었습니다."
}
```

---

### 3. 메시지와 데이터

```jsp
UserDao user = new UserDao();
DataSet list = user.find();
j.success("조회되었습니다.", list);
```

**응답:**
```json
{
  "success": true,
  "message": "조회되었습니다.",
  "data": [
    {"id": 1, "name": "홍길동", "email": "hong@example.com"},
    {"id": 2, "name": "김철수", "email": "kim@example.com"}
  ]
}
```

---

### 4. put()으로 데이터 설정 (자동 포함)

```jsp
api.post("/", () -> {
    UserDao user = new UserDao();
    user.item("name", f.get("name"));
    user.item("email", f.get("email"));

    if(user.insert()) {
        j.put("user_id", user.id);
        j.put("created_at", System.currentTimeMillis());
        j.success("등록되었습니다.");  // put()으로 설정한 데이터가 자동 포함
    } else {
        j.error("DATABASE_ERROR", user.getErrMsg());
    }
});
```

**응답:**
```json
{
  "success": true,
  "message": "등록되었습니다.",
  "data": {
    "user_id": 123,
    "created_at": 1704067200000
  }
}
```

---

## 페이징 응답 표준

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

**응답:**
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

---

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

**응답:**
```json
{
  "users": [...],
  "total": 156,
  "page": 1,
  "size": 20,
  "totalPages": 8,
  "hasNext": true,
  "hasPrev": false
}
```

---

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

**응답:**
```json
{
  "data": [...],
  "cursor": 20,
  "hasMore": true
}
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

---

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

**응답:**
```json
{
  "data": [...],
  "meta": {
    "total": 156,
    "page": 1,
    "size": 20,
    "totalPages": 8,
    "timestamp": 1704067200000,
    "version": "v1"
  }
}
```

---

## 관련 문서

- [REST API 개발](restapi.md) - REST API 기본 가이드
- [JWT 인증](restapi-jwt.md) - JWT 인증 가이드
- [CORS 설정](restapi-cors.md) - CORS 설정 가이드
- [JSON 처리](json.md) - Json 클래스 상세 가이드

---

[← 목차로 돌아가기](README.md)
