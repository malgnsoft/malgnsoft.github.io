# REST API - CORS 설정

외부 도메인에서 API를 호출할 때 필요한 CORS(Cross-Origin Resource Sharing) 헤더 설정입니다.

---

## 1. 기본 사용법

### 모든 도메인 허용

```jsp
<%@ include file="/api/init.jsp" %><%

// init.jsp에 이미 api.cors() 설정되어 있음
// 추가 설정 불필요

api.get("/", () -> {
    // 정상 처리
    j.success("처리되었습니다.");
});

%>
```

---

## 2. 특정 도메인만 허용

### init.jsp 수정

```jsp
// 특정 도메인만 허용
api.cors("https://yourdomain.com");

// OPTIONS 요청 처리
if(api.handlePreflight()) return;
```

---

## 3. 여러 도메인 허용 (화이트리스트)

### init.jsp 또는 index.jsp 수정

```jsp
// 여러 도메인 허용 - String 배열
api.cors(new String[]{
    "https://yourdomain.com",
    "https://app.yourdomain.com",
    "http://localhost:3000"  // 개발 환경
});

// OPTIONS 요청 처리
if(api.handlePreflight()) return;
```

---

## 4. 상세 CORS 설정

RestAPI 클래스의 cors() 메소드는 다음과 같이 사용할 수 있습니다:

```jsp
// 1. 모든 도메인 허용 (기본값)
api.cors();

// 2. 특정 도메인 허용
api.cors("https://yourdomain.com");

// 3. 여러 도메인 허용 (배열)
api.cors(new String[]{
    "https://yourdomain.com",
    "https://app.yourdomain.com"
});

// 4. 도메인 및 메소드 지정
api.cors("https://yourdomain.com", "GET, POST, PUT, DELETE");

// 5. 도메인, 메소드, 헤더 지정
api.cors("https://yourdomain.com", "GET, POST", "Content-Type, Authorization");

// 6. 전체 옵션 (preflight 캐시 시간 포함)
api.cors("https://yourdomain.com", "GET, POST", "Content-Type, Authorization", 7200);  // 2시간
```

### 여러 도메인 허용 시 동작

- 요청의 `Origin` 헤더를 확인합니다
- 화이트리스트에 포함되어 있으면 해당 도메인을 허용합니다
- 화이트리스트에 없으면 CORS 헤더를 설정하지 않습니다

### 기본값

- **origin**: `*` (모든 도메인)
- **methods**: `GET, POST, PUT, DELETE, PATCH, OPTIONS`
- **headers**: `Content-Type, Authorization`
- **maxAge**: `3600` (1시간)
- **credentials**: origin이 `*`가 아닐 때 자동으로 `true` 설정

---

## 5. OPTIONS Preflight 처리

CORS preflight 요청을 자동으로 처리합니다:

```jsp
// init.jsp 또는 index.jsp
api.cors();

// OPTIONS 요청 처리
if(api.handlePreflight()) return;
```

### Preflight란?

브라우저가 실제 요청을 보내기 전에 OPTIONS 메소드로 서버에 허용 여부를 확인하는 요청입니다.

**Preflight가 발생하는 경우:**
- `PUT`, `DELETE`, `PATCH` 메소드 사용
- `Content-Type`이 `application/json`인 경우
- 커스텀 헤더 사용 (예: `Authorization`)

---

## 6. 클라이언트에서 CORS 요청

### 일반 요청

```javascript
// 기본 GET 요청
fetch('https://api.yourdomain.com/api/user', {
    method: 'GET',
    headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${token}`
    }
})
.then(response => response.json())
.then(data => console.log(data));
```

### 쿠키를 포함한 요청 (credentials)

```javascript
// 쿠키를 포함한 요청
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

**주의:** `credentials: 'include'` 사용 시 서버의 `Access-Control-Allow-Origin`은 `*`가 아닌 특정 도메인이어야 합니다.

---

## 7. CORS 오류 해결

### "No 'Access-Control-Allow-Origin' header is present"

**원인:** CORS 헤더가 설정되지 않았습니다.

**해결:**
```jsp
// init.jsp 또는 index.jsp에 추가
api.cors();
```

### "The 'Access-Control-Allow-Origin' header contains multiple values"

**원인:** CORS 헤더가 중복 설정되었습니다.

**해결:**
- `init.jsp`와 각 API 파일에서 중복 호출 제거
- `init.jsp`에서 한 번만 설정

### "Response to preflight request doesn't pass access control check"

**원인:** OPTIONS preflight 요청이 처리되지 않았습니다.

**해결:**
```jsp
api.cors();
if(api.handlePreflight()) return;
```

---

## 8. 보안 고려사항

### 1. 프로덕션 환경에서는 특정 도메인만 허용

```jsp
// 좋은 예 - 특정 도메인만
api.cors(new String[]{
    "https://yourdomain.com",
    "https://app.yourdomain.com"
});

// 나쁜 예 - 모든 도메인 허용 (보안 취약)
api.cors();  // origin: *
```

### 2. credentials 사용 시 와일드카드 금지

```jsp
// 에러 발생
api.cors("*");
// + JavaScript에서 credentials: 'include' 사용

// 올바른 방법
api.cors("https://yourdomain.com");
// + JavaScript에서 credentials: 'include' 사용
```

### 3. 필요한 메소드만 허용

```jsp
// 읽기 전용 API
api.cors("https://yourdomain.com", "GET");

// 읽기/쓰기 API
api.cors("https://yourdomain.com", "GET, POST, PUT, DELETE");
```

---

## 관련 문서

- [REST API 개발](restapi.md) - REST API 기본 가이드
- [JWT 인증](restapi-jwt.md) - JWT 인증 가이드
- [응답 표준](restapi-response.md) - 에러/성공 응답 표준

---

[← 목차로 돌아가기](README.md)
