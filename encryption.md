# 암호화

[← 목차로 돌아가기](README.md)

---

## 암호화 개요

MalgnFramework는 다양한 암호화 기능을 제공합니다.

### 지원 기능

- **해시 함수**: MD5, SHA-1, SHA-256, SHA-512
- **HMAC**: HMAC-SHA256, HMAC-SHA512
- **대칭키 암호화**: AES-256
- **Base64 인코딩/디코딩**

---

## 해시 함수

### 기본 해시

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

String password = "mypassword123";

// MD5 (권장하지 않음 - 취약함)
String md5 = Malgn.md5(password);
m.p("MD5: " + md5);

// SHA-1 (권장하지 않음)
String sha1 = Malgn.sha1(password);
m.p("SHA-1: " + sha1);

// SHA-256 (권장)
String sha256 = Malgn.sha256(password);
m.p("SHA-256: " + sha256);

// SHA-512 (권장)
String sha512 = Malgn.sha512(password);
m.p("SHA-512: " + sha512);

%>
```

### Salt를 사용한 해시

```jsp
String password = "mypassword123";
String salt = "random_salt_value";

// SHA-256 with salt
String hashed = Malgn.sha256(password, salt);
m.p("SHA-256 with salt: " + hashed);

// SHA-512 with salt
String hashed512 = Malgn.sha512(password, salt);
m.p("SHA-512 with salt: " + hashed512);
```

---

## 비밀번호 해싱

### 회원가입 시 비밀번호 저장

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

f.addElement("user_id", null, "required:Y");
f.addElement("password", null, "required:Y, minlength:8");
f.addElement("name", null, "required:Y");

if(m.isPost() && f.validate()) {

    String userId = f.get("user_id");
    String plainPassword = f.get("password");

    // 비밀번호 해시화
    String hashedPassword = Malgn.sha256(plainPassword);

    UserDao user = new UserDao();
    user.item("user_id", userId);
    user.item("password", hashedPassword);
    user.item("name", f.get("name"));
    user.item("reg_date", m.time());
    user.insert();

    m.jsAlert("회원가입이 완료되었습니다");
    m.jsReplace("/login.jsp");
    
    return;
}

%>
```

### 로그인 시 비밀번호 확인

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

f.addElement("user_id", null, "required:Y");
f.addElement("password", null, "required:Y");

if(m.isPost() && f.validate()) {

    String userId = f.get("user_id");
    String plainPassword = f.get("password");

    // 입력된 비밀번호 해시화
    String hashedPassword = Malgn.sha256(plainPassword);

    // DB에서 사용자 확인
    UserDao user = new UserDao();
    DataSet info = user.query("WHERE user_id = ? AND password = ?", new Object[]{userId, hashedPassword});

    if(user.next()) {
        // 로그인 성공
        auth.put("user_id", user.getInt("id"));
        auth.put("user_name", user.getString("name"));
        auth.save();

        m.jsReplace("/main/index.jsp");
    } else {
        m.jsError("아이디 또는 비밀번호가 올바르지 않습니다");
    }

    return;
}

%>
```

---

## HMAC (Hash-based Message Authentication Code)

HMAC은 메시지 인증에 사용되며, API 서명이나 데이터 무결성 검증에 유용합니다.

### HMAC-SHA256

```jsp
String secretKey = "your_secret_key";
String message = "important_data";

String signature = Malgn.hmacSha256(secretKey, message);
m.p("HMAC-SHA256: " + signature);
```

### HMAC-SHA512

```jsp
String secretKey = "your_secret_key";
String message = "important_data";

String signature = Malgn.hmacSha512(secretKey, message);
m.p("HMAC-SHA512: " + signature);
```

### API 서명 검증

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="/init.jsp" %><%

String apiKey = Config.get("apiKey");
String requestBody = m.reqBody();
String receivedSignature = request.getHeader("X-Signature");

// 서명 생성
String expectedSignature = Malgn.hmacSha256(apiKey, requestBody);

// 서명 검증
if(!expectedSignature.equals(receivedSignature)) {
    j.error(401, "서명이 올바르지 않습니다");
    return;
}

// 처리...
j.success("검증 성공");

%>
```

---

## AES 암호화

AES는 대칭키 암호화 알고리즘으로, 데이터를 안전하게 암호화/복호화할 수 있습니다.

### 기본 사용법

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 16, 24, 또는 32바이트 키
String secretKey = "abcdefghijklmnop";  // 16바이트

AES aes = new AES(secretKey);

// 암호화
String plainText = "This is secret data. 중요한 데이터입니다.";
String encrypted = aes.encrypt(plainText);
m.p("암호화: " + encrypted);

// 복호화
String decrypted = aes.decrypt(encrypted);
m.p("복호화: " + decrypted);

%>
```

### IV(Initialization Vector)를 사용한 AES

```jsp
String secretKey = "abcdefghijklmnop";  // 16바이트
String iv = "1234567890123456";  // 16바이트

AES aes = new AES(secretKey, iv);

String encrypted = aes.encrypt("민감한 데이터");
String decrypted = aes.decrypt(encrypted);
```

### SimpleAES (간편 사용)

```jsp
// 키 파일에서 로드
SimpleAES.setKeyFile(Config.getDocRoot() + "/WEB-INF/key.dat");

// 암호화
String encrypted = SimpleAES.encrypt("비밀 정보");
m.p(encrypted);

// 복호화
String decrypted = SimpleAES.decrypt(encrypted);
m.p(decrypted);
```

---

## 실전 예제

### 1. 사용자 ID 암호화 (URL에 노출)

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 사용자 ID 암호화
int userId = 12345;
String secretKey = Config.get("encryptionKey");

AES aes = new AES(secretKey);
String encryptedId = aes.encrypt(String.valueOf(userId));

// URL에 암호화된 ID 사용
String url = "/user/profile.jsp?id=" + Malgn.urlencode(encryptedId);

m.p("<a href='" + url + "'>프로필 보기</a>");

%>
```

**profile.jsp**:
```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

String encryptedId = m.rs("id");
String secretKey = Config.get("encryptionKey");

AES aes = new AES(secretKey);
String decryptedId = aes.decrypt(encryptedId);

int userId = Integer.parseInt(decryptedId);
m.p("사용자 ID: " + userId);

%>
```

### 2. 개인정보 암호화 저장

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

if(m.isPost() && f.validate()) {

    String secretKey = Config.get("personalDataKey");
    AES aes = new AES(secretKey);

    // 주민등록번호 암호화
    String ssn = f.get("ssn");
    String encryptedSsn = aes.encrypt(ssn);

    // 전화번호 암호화
    String phone = f.get("phone");
    String encryptedPhone = aes.encrypt(phone);

    UserDao user = new UserDao();
    user.item("name", f.get("name"));
    user.item("ssn", encryptedSsn);
    user.item("phone", encryptedPhone);
    user.item("email", f.get("email"));
    user.insert();

    m.jsAlert("저장되었습니다");
    m.jsReplace("list.jsp");

    return;
}

%>
```

**조회 시 복호화**:
```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

UserDao user = new UserDao();
DataSet userList = user.query("WHERE status = 1");

String secretKey = Config.get("personalDataKey");
AES aes = new AES(secretKey);

// 복호화
while(userList.next()) {
    String encryptedSsn = userList.getString("ssn");
    String encryptedPhone = userList.getString("phone");

    String ssn = aes.decrypt(encryptedSsn);
    String phone = aes.decrypt(encryptedPhone);

    // 마스킹하여 표시
    String maskedSsn = ssn.substring(0, 6) + "-*******";
    String maskedPhone = phone.substring(0, 3) + "-****-" + phone.substring(9);

    userList.put("ssn_masked", maskedSsn);
    userList.put("phone_masked", maskedPhone);
}

p.setLoop("users", userList);
p.display();

%>
```

### 3. API 토큰 생성

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 로그인 성공 후 API 토큰 생성
if(auth.isLogin()) {

    int userId = auth.getInt("user_id");
    long timestamp = System.currentTimeMillis();

    // 토큰 데이터
    String tokenData = userId + "|" + timestamp;

    // 토큰 암호화
    String secretKey = Config.get("tokenKey");
    AES aes = new AES(secretKey);
    String token = aes.encrypt(tokenData);

    // 응답
    DataMap result = new DataMap();
    result.put("token", token);
    result.put("expires_in", 3600);

    j.success("토큰 발급 성공", result);
}

%>
```

**API 인증**:
```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="/init.jsp" %><%

String token = request.getHeader("Authorization");
if(token == null) {
    j.error(401, "인증 토큰이 필요합니다");
    return;
}

try {
    String secretKey = Config.get("tokenKey");
    AES aes = new AES(secretKey);
    String decrypted = aes.decrypt(token);

    String[] parts = Malgn.split("\\|", decrypted);
    int userId = Integer.parseInt(parts[0]);
    long timestamp = Long.parseLong(parts[1]);

    // 토큰 만료 확인 (1시간)
    long now = System.currentTimeMillis();
    if(now - timestamp > 3600000) {
        j.error(401, "토큰이 만료되었습니다");
        return;
    }

    // 처리...
    j.success("인증 성공");

} catch(Exception e) {
    j.error(401, "유효하지 않은 토큰");
}

%>
```

### 4. 설정 파일 암호화

**암호화 도구**:
```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// DB 비밀번호 암호화
String dbPassword = "db_secret_password";

String secretKey = Config.get("configKey");
AES aes = new AES(secretKey);

String encrypted = aes.encrypt(dbPassword);
m.p("암호화된 비밀번호: " + encrypted);
m.p("이 값을 config.properties에 저장하세요");

%>
```

**config.properties**:
```properties
db.host=localhost
db.user=dbuser
db.password.encrypted=XmZkL2pRQ3VHdz09
```

**사용 시**:
```jsp
String encryptedPassword = Config.get("db.password.encrypted");
String secretKey = Config.get("configKey");

AES aes = new AES(secretKey);
String dbPassword = aes.decrypt(encryptedPassword);

// DB 연결에 사용
```

---

## Base64 인코딩/디코딩

### 기본 사용

```jsp
String original = "Hello, World! 안녕하세요";

// 인코딩
String encoded = Malgn.encode(original);
m.p("인코딩: " + encoded);

// 디코딩
String decoded = Malgn.decode(encoded);
m.p("디코딩: " + decoded);
```

### 파일 인코딩

```jsp
// 파일을 Base64로 인코딩
String filePath = Config.getDataDir() + "/image.jpg";
String fileContent = Malgn.readFile(filePath);
String encoded = Malgn.encode(fileContent);

// 이메일에 첨부하거나 데이터 URL로 사용
String dataUrl = "data:image/jpeg;base64," + encoded;
```

---

## 로깅

### 안전한 로그

```jsp
// 민감한 정보는 로그에 남기지 않기
String password = f.get("password");

// Bad
Malgn.logger("info", "비밀번호: " + password);

// Good
Malgn.logger("info", "로그인 시도: " + f.get("user_id"));
```

---

## 주요 메소드 정리

### 해시 함수

| 메소드 | 설명 |
|--------|------|
| Malgn.md5(str) | MD5 해시 |
| Malgn.sha1(str) | SHA-1 해시 |
| Malgn.sha256(str) | SHA-256 해시 (권장) |
| Malgn.sha256(str, salt) | SHA-256 with salt |
| Malgn.sha512(str) | SHA-512 해시 (권장) |
| Malgn.sha512(str, salt) | SHA-512 with salt |

### HMAC

| 메소드 | 설명 |
|--------|------|
| Malgn.hmacSha256(key, data) | HMAC-SHA256 |
| Malgn.hmacSha512(key, data) | HMAC-SHA512 |

### AES 암호화

| 메소드 | 설명 |
|--------|------|
| new AES(key) | AES 인스턴스 생성 (16/24/32바이트) |
| new AES(key, iv) | IV를 사용한 AES |
| encrypt(plainText) | 암호화 |
| decrypt(encrypted) | 복호화 |
| setKeyFile(path) | 파일에서 키 로드 |

### Base64

| 메소드 | 설명 |
|--------|------|
| Malgn.encode(str) | Base64 인코딩 |
| Malgn.decode(str) | Base64 디코딩 |

---

## 보안 권장사항

### 1. 비밀번호 해싱

```jsp
// Good - SHA-256 or SHA-512
String hashed = Malgn.sha256(password);

// Better - with salt
String salt = Malgn.getUniqId(16);
String hashed = Malgn.sha256(password, salt);
```

### 2. 키 관리

```jsp
// Bad - 하드코딩
String secretKey = "my_secret_key_123";

// Good - 설정 파일
String secretKey = Config.get("encryptionKey");

// Better - 환경 변수 또는 외부 키 저장소
```

### 3. 암호화 키 길이

```jsp
// AES-128: 16바이트
// AES-192: 24바이트
// AES-256: 32바이트 (권장)

String key256 = "12345678901234567890123456789012";  // 32바이트
AES aes = new AES(key256);
```

### 4. HTTPS 사용

암호화된 데이터도 반드시 HTTPS를 통해 전송하세요.

---

## 다음 단계

암호화 기능을 활용하여 안전한 애플리케이션을 구축하세요.

---

[← 목차로 돌아가기](README.md)
