# REST API - JWT 인증

맑은프레임워크는 `RestAPI` 클래스에서 JWT(JSON Web Token) 기반 인증을 지원합니다. 세션 기반 인증과 달리 stateless API에 적합합니다.

---

## 1. JWT 토큰 생성

로그인 성공 시 JWT 토큰을 생성하여 클라이언트에 전달합니다.

### /api/auth/login.jsp

```jsp
<%@ include file="/api/init.jsp" %><%

// POST /api/auth/login - 로그인
api.post("/", () -> {
    String email = f.get("email");
    String password = f.get("password");

    // 사용자 인증 확인
    UserDao user = new UserDao();
    DataSet info = user.getByEmail(email);

    if(info.next() && user.checkPassword(password, info.s("password"))) {
        // RestAPI 객체에 사용자 정보 저장
        api.setData("user_id", info.i("id"));
        api.setData("user_name", info.s("name"));
        api.setData("user_level", info.i("level"));

        // JWT 토큰 생성 (60분 유효)
        String token = api.generateToken(60);

        j.put("token", token);
        j.put("user_id", info.i("id"));
        j.put("user_name", info.s("name"));
        j.put("user_level", info.i("level"));
        j.success("로그인되었습니다.");
    } else {
        j.error("INVALID_CREDENTIALS", "이메일 또는 비밀번호가 일치하지 않습니다.");
    }
});

%>
```

---

## 2. JWT 토큰 검증

`/api/init.jsp`에서 `api.auth()`로 자동 검증합니다:

```jsp
// JWT 토큰 인증 (퍼블릭 라우팅 자동 체크)
if(!api.auth()) return;

// 인증된 사용자 정보
int userId = api.getDataInt("user_id");
String userName = api.getData("user_name");
int userLevel = api.getDataInt("user_level");
```

### `api.auth()` 메소드 동작

1. 퍼블릭 라우팅이면 인증 불필요 (true 반환)
2. `Authorization: Bearer <token>` 헤더에서 토큰 추출
3. 토큰 서명 검증 및 만료 시간 확인
4. 토큰이 유효하면 사용자 정보 자동 로드 (true 반환)
5. 토큰이 없거나 유효하지 않으면 에러 응답 자동 출력 (false 반환)

---

## 3. 클라이언트 사용법

### Authorization 헤더 방식 (권장)

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

---

## 4. 권한 체크 예시

JWT 토큰으로 인증된 사용자의 권한을 체크합니다.

### /api/admin/stats.jsp

```jsp
<%@ include file="/api/init.jsp" %><%

// 관리자 권한 체크 (userLevel은 init.jsp의 api.auth()에서 자동 로드)
if(userLevel < 10) {
    j.error("FORBIDDEN", "관리자만 접근할 수 있습니다.");
    return;
}

// GET /api/admin/stats - 통계 조회
api.get("/", () -> {
    StatsDao stats = new StatsDao();
    DataSet data = stats.getMonthlyStats();

    j.put("stats", data);
    j.put("requested_by", userName);  // JWT 토큰에서 추출된 userName 사용
    j.success();
});

%>
```

**설명:**
- `init.jsp`의 `api.auth()`가 이미 로그인 체크를 완료
- userId, userName, userLevel이 자동으로 설정됨
- 추가 권한 체크만 수행하면 됨

---

## 5. 퍼블릭 라우팅

인증이 필요 없는 API 경로를 지정할 수 있습니다.

### init.jsp에서 설정

```jsp
// 퍼블릭 라우팅 설정 (인증 불필요)
api.publicRoute(
    "/api/auth/login",      // 로그인 API
    "/api/auth/register",   // 회원가입 API
    "/api/public/*"         // /api/public 하위 모든 경로
);
```

### 와일드카드 패턴

- `/api/auth/*` - `/api/auth` 하위 모든 경로
- `/api/public/*` - `/api/public` 하위 모든 경로
- 정확히 일치하는 경로는 와일드카드 없이 지정

### `api.auth()` 메소드 동작

1. 퍼블릭 라우팅이면 인증 불필요 (true 반환)
2. 퍼블릭이 아니면 JWT 토큰 검증
3. 토큰이 없거나 유효하지 않으면 에러 응답 자동 출력 (false 반환)
4. 토큰이 유효하면 사용자 정보 자동 로드 (true 반환)

### 장점

- 퍼블릭 API는 인증 없이 접근 가능
- 나머지 API는 자동으로 인증 체크
- 각 API 파일에서 인증 코드 작성 불필요

---

## 6. 보안 고려사항

1. **HTTPS 필수**: JWT 토큰은 반드시 HTTPS를 통해 전송해야 합니다.
2. **토큰 만료 시간**: 보안을 위해 짧은 만료 시간(30~60분)을 권장합니다.
3. **Refresh Token**: 장기 로그인이 필요한 경우 Refresh Token 패턴 구현을 고려하세요.
4. **비밀키 관리**: `Config.getSecretId()`로 반환되는 비밀키는 안전하게 보관하세요.
5. **토큰 저장**: XSS 공격 방지를 위해 `httpOnly` 쿠키 저장도 고려하세요.

---

## 관련 문서

- [REST API 개발](restapi.md) - REST API 기본 가이드
- [CORS 설정](restapi-cors.md) - CORS 설정 가이드
- [응답 표준](restapi-response.md) - 에러/성공 응답 표준

---

[← 목차로 돌아가기](README.md)
