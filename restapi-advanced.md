# REST API - 고급 라우팅

Path Parameter를 활용한 고급 라우팅 패턴과 RESTful API 설계입니다.

---

## Path Parameter 기본

Path Parameter를 사용하면 URL 경로에서 값을 추출할 수 있습니다.

### 기본 사용법

```jsp
<%@ include file="/api/init.jsp" %><%

// GET /api/user/:id - 단일 조회
api.get("/:id", () -> {
    int id = api.paramInt("id");  // /api/user/123 → id=123

    UserDao user = new UserDao();
    DataSet info = user.get(id);

    if(info.next()) {
        j.success(info);
    } else {
        j.error("NOT_FOUND", "사용자를 찾을 수 없습니다.");
    }
});

%>
```

---

## 지원 패턴

### 1. 고정 경로 (Fixed Path)

```jsp
// GET /api/user/list
api.get("/list", () -> {
    int page = m.ri("page", 1);
    int size = m.ri("size", 20);

    UserDao user = new UserDao();
    DataSet list = user.findWithPaging(page, size);
    j.success("조회되었습니다.", list);
});

// GET /api/user/active
api.get("/active", () -> {
    UserDao user = new UserDao();
    DataSet list = user.findByStatus("active");
    j.success(list);
});

// GET /api/user/search
api.get("/search", () -> {
    String keyword = m.rs("keyword");  // query string
    UserDao user = new UserDao();
    DataSet list = user.search(keyword);
    j.success(list);
});
```

---

### 2. 단일 Parameter

```jsp
// GET /api/user/:id
api.get("/:id", () -> {
    int id = api.paramInt("id");  // /api/user/123 → id=123

    UserDao user = new UserDao();
    DataSet info = user.get(id);

    if(info.next()) {
        j.success(info);
    } else {
        j.error("NOT_FOUND", "사용자를 찾을 수 없습니다.");
    }
});
```

---

### 3. 복수 Parameter

```jsp
// GET /api/product/:category/:id
api.get("/:category/:id", () -> {
    String category = api.param("category");
    int id = api.paramInt("id");

    // /api/product/food/123 → category=food, id=123

    ProductDao product = new ProductDao();
    DataSet info = product.get(id);

    if(info.next() && category.equals(info.s("category"))) {
        j.success(info);
    } else {
        j.error("NOT_FOUND", "상품을 찾을 수 없습니다.");
    }
});
```

---

### 4. 혼합 사용 (고정 경로 + Parameter)

```jsp
// GET /api/post/:id/comments
api.get("/:id/comments", () -> {
    int postId = api.paramInt("id");

    CommentDao comment = new CommentDao();
    DataSet list = comment.findByPostId(postId);
    j.success(list);
});

// GET /api/user/:id/orders
api.get("/:id/orders", () -> {
    int userId = api.paramInt("id");

    OrderDao order = new OrderDao();
    DataSet list = order.findByUserId(userId);
    j.success(list);
});

// GET /api/user/:id/orders/:orderId
api.get("/:id/orders/:orderId", () -> {
    int userId = api.paramInt("id");
    int orderId = api.paramInt("orderId");

    OrderDao order = new OrderDao();
    DataSet info = order.getByUserAndOrder(userId, orderId);

    if(info.next()) {
        j.success(info);
    } else {
        j.error("NOT_FOUND", "주문을 찾을 수 없습니다.");
    }
});
```

---

## 매칭 우선순위

RestAPI 클래스는 다음 순서로 경로를 매칭합니다:

1. **고정 경로** 먼저 매칭 (예: `/list`, `/active`)
2. **Path parameter** 패턴 매칭 (예: `/:id`)

따라서 `/api/user/list`는 `/:id`보다 `/list`에 먼저 매칭됩니다.

### 예시

```jsp
// 1. 고정 경로
api.get("/list", () -> {
    // /api/user/list → 여기에 매칭됨
});

api.get("/active", () -> {
    // /api/user/active → 여기에 매칭됨
});

// 2. Path parameter
api.get("/:id", () -> {
    // /api/user/123 → 여기에 매칭됨
    // /api/user/list → 매칭 안됨 (위의 /list가 먼저 매칭됨)
});
```

---

## Parameter 추출 메소드

| 메소드 | 반환 타입 | 설명 | 예시 |
|--------|----------|------|------|
| `api.param(name)` | String | 문자열로 반환 | `String category = api.param("category");` |
| `api.paramInt(name)` | int | 정수로 변환 (실패시 0) | `int id = api.paramInt("id");` |

### 사용 예시

```jsp
api.get("/:category/:id", () -> {
    // 문자열 parameter
    String category = api.param("category");

    // 정수 parameter
    int id = api.paramInt("id");

    // /api/product/electronics/123
    // category = "electronics"
    // id = 123
});
```

---

## Query String과 함께 사용

Path parameter와 query string을 동시에 사용할 수 있습니다:

```jsp
// GET /api/user/123?includeOrders=true
api.get("/:id", () -> {
    // Path parameter
    int id = api.paramInt("id");

    // Query string
    boolean includeOrders = m.rb("includeOrders");

    UserDao user = new UserDao();
    DataSet info = user.get(id);

    if(info.next()) {
        j.put("user", info);

        // includeOrders가 true면 주문 목록도 포함
        if(includeOrders) {
            OrderDao order = new OrderDao();
            DataSet orders = order.findByUserId(id);
            j.put("orders", orders);
        }

        j.success();
    } else {
        j.error("NOT_FOUND", "사용자를 찾을 수 없습니다.");
    }
});
```

**클라이언트 호출:**
```javascript
// 기본 조회
fetch('/api/user/123')

// 주문 목록 포함
fetch('/api/user/123?includeOrders=true')
```

---

## RESTful 라우팅 예시

### /api/product.jsp

```jsp
<%@ include file="/api/init.jsp" %><%

api.setBasePath("/api/product");

// GET /api/product - 전체 목록
api.get("/", () -> {
    String keyword = m.rs("keyword");
    int page = m.ri("page", 1);
    int size = m.ri("size", 20);

    ProductDao product = new ProductDao();
    DataSet list = product.find(keyword, page, size);
    int total = product.getTotal();

    j.put("products", list);
    j.put("total", total);
    j.put("page", page);
    j.put("size", size);
    j.success();
});

// GET /api/product/:category - 카테고리별 목록
api.get("/:category", () -> {
    String category = api.param("category");

    ProductDao product = new ProductDao();
    DataSet list = product.findByCategory(category);

    j.put("products", list);
    j.put("category", category);
    j.success();
});

// GET /api/product/:category/:id - 특정 상품 조회
api.get("/:category/:id", () -> {
    String category = api.param("category");
    int id = api.paramInt("id");

    ProductDao product = new ProductDao();
    DataSet info = product.get(id);

    if(info.next() && category.equals(info.s("category"))) {
        j.success(info);
    } else {
        j.error("NOT_FOUND", "상품을 찾을 수 없습니다.");
    }
});

// POST /api/product - 상품 생성
api.post("/", () -> {
    ProductDao product = new ProductDao();
    product.item("name", f.get("name"));
    product.item("category", f.get("category"));
    product.item("price", f.getInt("price"));
    product.item("description", f.get("description"));

    if(product.insert()) {
        j.success("등록되었습니다.", product.id);
    } else {
        j.error("DATABASE_ERROR", product.getErrMsg());
    }
});

// PUT /api/product/:id - 상품 수정
api.put("/:id", () -> {
    int id = api.paramInt("id");

    ProductDao product = new ProductDao();
    product.get(id);
    product.item("name", f.get("name"));
    product.item("category", f.get("category"));
    product.item("price", f.getInt("price"));
    product.item("description", f.get("description"));

    if(product.update()) {
        j.success("수정되었습니다.");
    } else {
        j.error("DATABASE_ERROR", product.getErrMsg());
    }
});

// PATCH /api/product/:id - 부분 수정
api.patch("/:id", () -> {
    int id = api.paramInt("id");

    ProductDao product = new ProductDao();
    product.get(id);

    // 전달된 필드만 업데이트
    String name = f.get("name");
    if(!"".equals(name)) product.item("name", name);

    String category = f.get("category");
    if(!"".equals(category)) product.item("category", category);

    int price = f.getInt("price");
    if(price > 0) product.item("price", price);

    if(product.update()) {
        j.success("수정되었습니다.");
    } else {
        j.error("DATABASE_ERROR", product.getErrMsg());
    }
});

// DELETE /api/product/:id - 상품 삭제
api.delete("/:id", () -> {
    int id = api.paramInt("id");

    ProductDao product = new ProductDao();
    if(product.delete(id)) {
        j.success("삭제되었습니다.");
    } else {
        j.error("DATABASE_ERROR", product.getErrMsg());
    }
});

%>
```

---

## 클라이언트 호출 예시

```javascript
// GET /api/product - 전체 목록
fetch('/api/product')
    .then(response => response.json())
    .then(data => console.log(data));

// GET /api/product?keyword=노트북&page=1&size=20 - 검색
fetch('/api/product?keyword=노트북&page=1&size=20')
    .then(response => response.json())
    .then(data => console.log(data));

// GET /api/product/electronics - 전자제품 카테고리
fetch('/api/product/electronics')
    .then(response => response.json())
    .then(data => console.log(data));

// GET /api/product/electronics/123 - 특정 상품
fetch('/api/product/electronics/123')
    .then(response => response.json())
    .then(data => console.log(data));

// POST /api/product - 상품 생성
fetch('/api/product', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/x-www-form-urlencoded',
    },
    body: 'name=노트북&category=electronics&price=1000000&description=고성능 노트북'
})
    .then(response => response.json())
    .then(data => console.log(data));

// PUT /api/product/123 - 전체 수정
fetch('/api/product/123', {
    method: 'PUT',
    headers: {
        'Content-Type': 'application/x-www-form-urlencoded',
    },
    body: 'name=게이밍노트북&category=electronics&price=1500000&description=고성능 게이밍 노트북'
})
    .then(response => response.json())
    .then(data => console.log(data));

// PATCH /api/product/123 - 부분 수정 (가격만)
fetch('/api/product/123', {
    method: 'PATCH',
    headers: {
        'Content-Type': 'application/x-www-form-urlencoded',
    },
    body: 'price=1200000'
})
    .then(response => response.json())
    .then(data => console.log(data));

// DELETE /api/product/123 - 삭제
fetch('/api/product/123', {
    method: 'DELETE'
})
    .then(response => response.json())
    .then(data => console.log(data));
```

---

## 중첩 리소스 라우팅

### /api/post.jsp

```jsp
<%@ include file="/api/init.jsp" %><%

api.setBasePath("/api/post");

// GET /api/post/:id/comments - 게시글의 댓글 목록
api.get("/:id/comments", () -> {
    int postId = api.paramInt("id");

    CommentDao comment = new CommentDao();
    DataSet list = comment.findByPostId(postId);

    j.put("comments", list);
    j.put("post_id", postId);
    j.success();
});

// POST /api/post/:id/comments - 댓글 작성
api.post("/:id/comments", () -> {
    int postId = api.paramInt("id");
    String content = f.get("content");

    CommentDao comment = new CommentDao();
    comment.item("post_id", postId);
    comment.item("content", content);
    comment.item("user_id", userId);  // init.jsp에서 설정

    if(comment.insert()) {
        j.success("댓글이 등록되었습니다.", comment.id);
    } else {
        j.error("DATABASE_ERROR", comment.getErrMsg());
    }
});

// GET /api/post/:id/comments/:commentId - 특정 댓글 조회
api.get("/:id/comments/:commentId", () -> {
    int postId = api.paramInt("id");
    int commentId = api.paramInt("commentId");

    CommentDao comment = new CommentDao();
    DataSet info = comment.get(commentId);

    if(info.next() && postId == info.i("post_id")) {
        j.success(info);
    } else {
        j.error("NOT_FOUND", "댓글을 찾을 수 없습니다.");
    }
});

// DELETE /api/post/:id/comments/:commentId - 댓글 삭제
api.delete("/:id/comments/:commentId", () -> {
    int postId = api.paramInt("id");
    int commentId = api.paramInt("commentId");

    CommentDao comment = new CommentDao();
    DataSet info = comment.get(commentId);

    // 권한 체크: 작성자만 삭제 가능
    if(info.next() && info.i("user_id") == userId) {
        if(comment.delete(commentId)) {
            j.success("삭제되었습니다.");
        } else {
            j.error("DATABASE_ERROR", comment.getErrMsg());
        }
    } else {
        j.error("FORBIDDEN", "권한이 없습니다.");
    }
});

%>
```

---

## 관련 문서

- [REST API 개발](restapi.md) - REST API 기본 가이드
- [JWT 인증](restapi-jwt.md) - JWT 인증 가이드
- [CORS 설정](restapi-cors.md) - CORS 설정 가이드
- [응답 표준](restapi-response.md) - 에러/성공 응답 표준

---

[← 목차로 돌아가기](README.md)
