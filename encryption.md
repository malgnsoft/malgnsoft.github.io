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

// 서명 검증: equals로 직접 비교하지 말고 verifyHmacSha256을 쓴다.
// (equals는 비상수 시간이고, apiKey가 비어 있으면 hmacSha256이 빈 문자열을 돌려주므로 서명을 비워 보낸 요청이 통과한다.
//  verifyHmacSha256은 키·서명이 비어 있으면 false이고 상수 시간으로 비교한다.)
if(!Malgn.verifyHmacSha256(apiKey, requestBody, receivedSignature)) {
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

### KeyVault와 ENC: 암호문

`config.xml`의 비밀 값(S3 키, SMTP 비밀번호 등)이나 DB 비밀번호를 평문 대신 **암호문**으로 둘 수 있습니다(1.18.11~, 옵트인). 값이 `ENC:`로 시작하면 프레임워크가 읽을 때 풀어 쓰며, 키와 평문은 로그에 남지 않습니다.

| 형식 | 마스터 키 |
|------|-----------|
| `ENC:v1:{키id}:{용도}:…` | 로컬 마스터 키 (16진수 64자) |
| `ENC:kms:{용도}:…` | AWS KMS |
| `ENC:nhn:{용도}:…` | NHN Cloud KMS |

용도는 `config`(config.xml 값), `tenants`(멀티테넌시 DB 비밀번호), `aes`(`aes.key.enc`), `resin`(Resin 설정 파일), `dek`(개인정보 데이터 키)이며, 용도가 다른 자리에 쓴 값은 풀리지 않습니다.

```bash
# 로컬 마스터 키 만들기 (권한 600, 이미 있으면 거부, 키 값은 출력하지 않음)
bin/malgn vault gen-key /home/mysite/keys/vault.key

# 값 감싸기: 평문은 표준입력 또는 숨김 프롬프트로 받는다 (명령행 인자로 받지 않음)
export MALGN_VAULT_KEYFILE=/home/mysite/keys/vault.key
bin/malgn vault wrap config
```

출력된 `ENC:v1:…` 값을 `config.xml`에 그대로 넣고, 서버에는 `vaultKeyFile`(또는 환경변수 `MALGN_VAULT_KEY`)로 마스터 키를 알려 줍니다. KMS를 쓰려면 `keyProvider`(`awskms`/`nhnkms`)를 설정합니다. 설정 키의 정본은 `config.xml.sample`입니다.

### aes.key (SimpleAES·AES 키 파일)

`SimpleAES`와 `AES`는 키를 따로 정하지 않으면 `WEB-INF/aes.key`를 씁니다. **코드에 내장된 키는 없으므로** 파일이 없으면 `SimpleAES.encrypt/decrypt`, `AES.encrypt/decrypt`가 `IllegalStateException`을 던집니다. 파일은 자동으로 만들어지지 않으며 환경마다 새로 만들고(공유 금지) 서버가 여러 대면 모든 서버에 같은 파일을 둡니다. 권한은 600입니다.

| 상황 | 명령 |
|------|------|
| 새로 시작하는 사이트 (기존 암호문 없음) | `bin/malgn aes-key` |
| 옛 JAR의 내장 키로 만든 암호문을 계속 읽어야 함 | `bin/malgn aes-legacy-export` (옛 JAR에서 키를 꺼내 `aes.key`로 저장) |
| 확인 | `bin/malgn aes-check` |

`aes-key`로 만든 새 키로는 이전 키의 암호문을 읽을 수 없습니다. 파일이 이미 있으면 덮어쓰지 않습니다. 키 파일 자체를 평문으로 두고 싶지 않으면 위 KeyVault로 감싼 `aes.key.enc`(용도 `aes`)를 대신 둘 수 있습니다. `SimpleAES`(ECB)와 `AES`(고정 IV)는 무결성 검증이 없는 **전환 기간용**이므로 신규 코드에는 쓰지 않습니다.

### 개인정보 암호화 (고객사별 데이터 키)

이메일·휴대전화 같은 개인정보를 고객사별 데이터 키(DEK)로 암호화하는 방법입니다(1.19.0~). DEK는 마스터 키(KMS 또는 위 KeyVault)로 **감싼 값**으로 만들어 **앱이 자기 DB에 저장**하며, 저장 위치와 구조는 앱이 정합니다.

```java
// 최초 1회: 감싼 DEK를 만들어 앱의 DB에 저장
String wrapped = AES.newDek();

// 사용: DB에서 읽은 감싼 값으로
AES aes = AES.withDek(wrapped);
String enc = aes.pii("email", email);
String idx = aes.piiIndex("email", email);
String plain = aes.piiDecrypt(enc);

// 마스터 키를 교체한 뒤 다시 감싸기
String rewrapped = AES.rewrapDek(wrapped);
```

- 고객사 id와 DEK 버전은 감싼 값 안에 들어 있어 따로 줄 필요가 없습니다.
- [멀티테넌시](multitenancy.md)에서는 현재 사이트와 감싼 값 안의 고객사가 다르면 예외입니다(다른 고객사의 값을 끼워 넣는 것을 막음). 요청 밖 배치는 `TenantContext.run(사이트, …)` 안에서 호출합니다. 사이트가 지정되지 않은 단일 사이트의 `newDek()`은 고객사 `default`로 만듭니다.
- 푼 DEK는 `piiDekCacheSec`(기본 600초, 최대 3600, `0`/`off`면 끔) 동안 재사용합니다. KMS에서 키를 비활성화해도 이 시간 동안은 이미 푼 키가 쓰입니다. `AES.clearDekCache()`로 즉시 비웁니다.

관련 설정은 [보안 설정 모음](security-settings.md)을 참고하세요.

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
| Malgn.verifyHmacSha256(key, data, signature) | HMAC-SHA256 서명 검증 (상수 시간 비교, 키·서명이 비면 false) |
| Malgn.verifyHmacSha512(key, data, signature) | HMAC-SHA512 서명 검증 |
| Malgn.verifyHmac(key, data, signature, "HmacSHA256") | 알고리즘을 지정한 서명 검증 |

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
