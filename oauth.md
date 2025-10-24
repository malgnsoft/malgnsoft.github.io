# OAuth 소셜 로그인

맑은프레임워크의 OAuthClient 클래스는 구글, 네이버, 카카오, 페이스북 등의 소셜 로그인을 간편하게 구현할 수 있게 합니다.

## 목차

- [OAuth 개요](#oauth-개요)
- [기본 설정](#기본-설정)
- [인증 플로우](#인증-플로우)
- [지원 플랫폼](#지원-플랫폼)
- [프로필 정보](#프로필-정보)
- [실제 구현 예제](#실제-구현-예제)

---

## OAuth 개요

### OAuth 2.0이란?

OAuth 2.0은 사용자가 비밀번호를 공유하지 않고도 다른 웹사이트에서 자신의 정보에 접근할 수 있도록 하는 개방형 표준 프로토콜입니다.

### 인증 프로세스

1. **인증 요청**: 사용자를 OAuth 제공자의 로그인 페이지로 리다이렉트
2. **사용자 승인**: 사용자가 로그인하고 권한 승인
3. **인증 코드 수신**: 콜백 URL로 인증 코드 반환
4. **액세스 토큰 교환**: 인증 코드를 액세스 토큰으로 교환
5. **프로필 조회**: 액세스 토큰으로 사용자 정보 가져오기

---

## 기본 설정

### OAuth 클라이언트 생성

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// OAuthClient 객체 생성
OAuthClient oauth = new OAuthClient(request, session);

// 디버그 모드 (선택사항)
oauth.setDebug(out);

%>
```

### 클라이언트 정보 설정

```jsp
// 방법 1: 수동 설정
oauth.setClient("google", "your-client-id", "your-client-secret");

// 방법 2: 콜백 URL 지정
oauth.setClient("naver", "client-id", "client-secret", "https://yoursite.com/callback");

// 방법 3: 개별 설정
oauth.setClientId("your-client-id");
oauth.setAuthorizeUrl("https://accounts.google.com/o/oauth2/v2/auth");
oauth.setTokenUrl("https://oauth2.googleapis.com/token");
oauth.setProfileUrl("https://www.googleapis.com/oauth2/v2/userinfo");
oauth.setScope("email profile");
```

---

## 인증 플로우

### 1단계: 로그인 버튼

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

OAuthClient oauth = new OAuthClient(request, session);

// 구글 로그인
oauth.setClient(
    "google",
    "your-google-client-id",
    "your-google-client-secret"
);
String googleAuthUrl = oauth.getAuthUrl();

// 네이버 로그인
oauth.setClient(
    "naver",
    "your-naver-client-id",
    "your-naver-client-secret"
);
String naverAuthUrl = oauth.getAuthUrl();

p.setBody("member.login");
p.setVar("googleAuthUrl", googleAuthUrl);
p.setVar("naverAuthUrl", naverAuthUrl);
p.display();

%>
```

**html/member/login.html**
```html
<!DOCTYPE html>
<html>
<body>
    <h1>로그인</h1>

    <a href="{{googleAuthUrl}}" class="btn-google">
        구글로 로그인
    </a>

    <a href="{{naverAuthUrl}}" class="btn-naver">
        네이버로 로그인
    </a>
</body>
</html>
```

### 2단계: 콜백 처리

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

OAuthClient oauth = new OAuthClient(request, session);

// 클라이언트 정보 설정 (1단계와 동일하게)
oauth.setClient(
    "google",
    "your-google-client-id",
    "your-google-client-secret"
);

// 인증 코드 가져오기
String code = m.rs("code");
String state = m.rs("state");

// State 검증 (CSRF 공격 방지)
if(!oauth.isValidState(state)) {
    m.jsError("잘못된 요청입니다");
    return;
}

// 프로필 정보 가져오기
HashMap<String, Object> profile = oauth.getProfile(code);

if(profile == null) {
    m.jsError("로그인에 실패했습니다: " + oauth.errMsg);
    return;
}

// 사용자 정보 추출
String email = (String)profile.get("email");
String name = (String)profile.get("name");
String id = (String)profile.get("id");

// 회원 가입 또는 로그인 처리
UserDao dao = new UserDao();
DataSet user = dao.query("WHERE email = ?", email);

if(!user.next()) {
    // 신규 회원 가입
    DataMap newUser = new DataMap();
    newUser.put("email", email);
    newUser.put("name", name);
    newUser.put("oauth_provider", "google");
    newUser.put("oauth_id", id);
    newUser.put("reg_date", Malgn.time());
    dao.insert(newUser);

    user = dao.query("WHERE email = ?", email);
    user.next();
}

// 세션에 로그인 정보 저장
auth.login(user.i("id"), user.s("name"));

m.jsAlert("로그인되었습니다");
m.jsReplace("/");

%>
```

---

## 지원 플랫폼

### 환경설정 파일

**config.xml**
```xml
<config>
    <oauth>
        <vendor>
            <vendorName>google</vendorName>
            <authorize>https://accounts.google.com/o/oauth2/v2/auth</authorize>
            <token>https://oauth2.googleapis.com/token</token>
            <profile>https://www.googleapis.com/oauth2/v2/userinfo</profile>
            <scope>email profile</scope>
        </vendor>
        <vendor>
            <vendorName>naver</vendorName>
            <authorize>https://nid.naver.com/oauth2.0/authorize</authorize>
            <token>https://nid.naver.com/oauth2.0/token</token>
            <profile>https://openapi.naver.com/v1/nid/me</profile>
            <scope></scope>
        </vendor>
        <vendor>
            <vendorName>kakao</vendorName>
            <authorize>https://kauth.kakao.com/oauth/authorize</authorize>
            <token>https://kauth.kakao.com/oauth/token</token>
            <profile>https://kapi.kakao.com/v2/user/me</profile>
            <scope></scope>
        </vendor>
        <vendor>
            <vendorName>facebook</vendorName>
            <authorize>https://www.facebook.com/v12.0/dialog/oauth</authorize>
            <token>https://graph.facebook.com/v12.0/oauth/access_token</token>
            <profile>https://graph.facebook.com/me</profile>
            <scope>email,public_profile</scope>
        </vendor>
    </oauth>
</config>
```

### 플랫폼별 클라이언트 ID 발급

#### 구글
1. [Google Cloud Console](https://console.cloud.google.com/) 접속
2. 프로젝트 생성
3. API 및 서비스 > 사용자 인증 정보
4. OAuth 2.0 클라이언트 ID 생성
5. 승인된 리디렉션 URI 설정: `https://yoursite.com/member/login_google.jsp`

#### 네이버
1. [네이버 개발자 센터](https://developers.naver.com/) 접속
2. 애플리케이션 등록
3. 서비스 URL 및 Callback URL 설정

#### 카카오
1. [카카오 개발자](https://developers.kakao.com/) 접속
2. 내 애플리케이션 > 애플리케이션 추가
3. 플랫폼 > Web 플랫폼 등록
4. Redirect URI 설정

---

## 프로필 정보

### 프로필 데이터 구조

각 플랫폼마다 반환하는 프로필 정보가 다릅니다:

#### 구글
```jsp
{
    "id": "123456789",
    "email": "user@gmail.com",
    "name": "홍길동",
    "picture": "https://..."
}
```

#### 네이버
```jsp
{
    "id": "naver-user-id",
    "email": "user@naver.com",
    "name": "홍길동",
    "profile_image": "https://...",
    "age": "30-39",
    "gender": "M",
    "birthday": "01-01",
    "birthyear": "1990"
}
```

#### 카카오
```jsp
{
    "id": "kakao-user-id",
    "kakao_account": {
        "email": "user@kakao.com",
        "profile": {
            "nickname": "홍길동",
            "profile_image_url": "https://..."
        }
    }
}
```

### 프로필 데이터 접근

```jsp
HashMap<String, Object> profile = oauth.getProfile(code);

// 공통 필드
String id = (String)profile.get("id");
String email = (String)profile.get("email");
String name = (String)profile.get("name");

// 플랫폼별 추가 정보
if(vendor.equals("naver")) {
    String gender = (String)profile.get("gender");
    String age = (String)profile.get("age");
    String birthday = (String)profile.get("birthday");
}

if(vendor.equals("facebook")) {
    String gender = (String)profile.get("gender");
}

// 원본 JSON 데이터
String rawData = oauth.getProfileData();
m.p(rawData);
```

---

## 실제 구현 예제

### 다중 소셜 로그인

**login.jsp** (로그인 페이지)
```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %>

<!DOCTYPE html>
<html>
<head>
    <title>로그인</title>
    <style>
        .social-login { margin: 20px 0; }
        .btn-social { display: block; margin: 10px 0; padding: 15px; text-decoration: none; }
        .btn-google { background: #4285f4; color: white; }
        .btn-naver { background: #03c75a; color: white; }
        .btn-kakao { background: #fee500; color: #000; }
    </style>
</head>
<body>
    <h1>로그인</h1>

    <div class="social-login">
        <a href="oauth_request.jsp?vendor=google" class="btn-social btn-google">
            구글로 로그인
        </a>

        <a href="oauth_request.jsp?vendor=naver" class="btn-social btn-naver">
            네이버로 로그인
        </a>

        <a href="oauth_request.jsp?vendor=kakao" class="btn-social btn-kakao">
            카카오로 로그인
        </a>
    </div>
</body>
</html>
```

**oauth_request.jsp** (인증 요청)
```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

String vendor = m.rs("vendor");

OAuthClient oauth = new OAuthClient(request, session);

// 각 플랫폼별 설정 (실제 값으로 교체)
if(vendor.equals("google")) {
    oauth.setClient("google", Config.get("oauth.google.clientId"), Config.get("oauth.google.clientSecret"));
} else if(vendor.equals("naver")) {
    oauth.setClient("naver", Config.get("oauth.naver.clientId"), Config.get("oauth.naver.clientSecret"));
} else if(vendor.equals("kakao")) {
    oauth.setClient("kakao", Config.get("oauth.kakao.clientId"), Config.get("oauth.kakao.clientSecret"));
} else {
    m.jsError("지원하지 않는 플랫폼입니다");
    return;
}

// 인증 URL 생성 및 리다이렉트
String authUrl = oauth.getAuthUrl();
response.sendRedirect(authUrl);

%>
```

**oauth_callback.jsp** (콜백 처리)
```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

String vendor = m.rs("vendor", "google");
String code = m.rs("code");
String state = m.rs("state");

OAuthClient oauth = new OAuthClient(request, session);

// 플랫폼별 설정 (oauth_request.jsp와 동일)
if(vendor.equals("google")) {
    oauth.setClient("google", Config.get("oauth.google.clientId"), Config.get("oauth.google.clientSecret"));
} else if(vendor.equals("naver")) {
    oauth.setClient("naver", Config.get("oauth.naver.clientId"), Config.get("oauth.naver.clientSecret"));
} else if(vendor.equals("kakao")) {
    oauth.setClient("kakao", Config.get("oauth.kakao.clientId"), Config.get("oauth.kakao.clientSecret"));
}

// State 검증
if(!oauth.isValidState(state)) {
    m.jsError("잘못된 요청입니다");
    return;
}

// 프로필 정보 가져오기
HashMap<String, Object> profile = oauth.getProfile(code);

if(profile == null) {
    m.jsError("로그인에 실패했습니다");
    return;
}

// 공통 정보 추출
String oauthId = (String)profile.get("id");
String email = (String)profile.get("email");
String name = (String)profile.get("name");

// 회원 확인
UserDao dao = new UserDao();
DataSet user = dao.query("WHERE oauth_provider = ? AND oauth_id = ?", vendor, oauthId);

if(!user.next()) {
    // 신규 회원 가입
    DataMap newUser = new DataMap();
    newUser.put("email", email);
    newUser.put("name", name);
    newUser.put("oauth_provider", vendor);
    newUser.put("oauth_id", oauthId);
    newUser.put("reg_date", Malgn.time());

    int userId = dao.insert(newUser);

    // 세션 로그인
    auth.login(userId, name);

    m.jsAlert(name + "님, 환영합니다!");
    m.jsReplace("/");
} else {
    // 기존 회원 로그인
    auth.login(user.i("id"), user.s("name"));

    m.jsAlert(user.s("name") + "님, 환영합니다!");
    m.jsReplace("/");
}

%>
```

### 데이터베이스 스키마

```sql
CREATE TABLE tb_user (
    id INT PRIMARY KEY AUTO_INCREMENT,
    email VARCHAR(100),
    name VARCHAR(50),
    password VARCHAR(255),  -- 일반 로그인용 (소셜 로그인은 NULL)
    oauth_provider VARCHAR(20),  -- google, naver, kakao, facebook
    oauth_id VARCHAR(100),       -- 플랫폼별 사용자 ID
    profile_image VARCHAR(255),
    reg_date DATETIME,
    INDEX idx_email (email),
    INDEX idx_oauth (oauth_provider, oauth_id)
);
```

---

## 보안 고려사항

### CSRF 공격 방지

```jsp
// State 파라미터 자동 생성 및 검증
String authUrl = oauth.getAuthUrl();  // state 자동 생성

// 콜백에서 검증
if(!oauth.isValidState(state)) {
    m.jsError("잘못된 요청입니다");
    return;
}
```

### 클라이언트 시크릿 보호

```jsp
// 나쁜 예: 소스코드에 직접 입력
oauth.setClient("google", "client-id", "client-secret");

// 좋은 예: 환경설정 파일 사용
oauth.setClient(
    "google",
    Config.get("oauth.google.clientId"),
    Config.get("oauth.google.clientSecret")
);
```

**config.xml**
```xml
<config>
    <oauth>
        <google>
            <clientId>your-google-client-id</clientId>
            <clientSecret>your-google-client-secret</clientSecret>
        </google>
    </oauth>
</config>
```

---

## 에러 처리

```jsp
OAuthClient oauth = new OAuthClient(request, session);
oauth.setDebug(out);

String code = m.rs("code");
String error = m.rs("error");

// OAuth 제공자가 에러를 반환한 경우
if(!error.isEmpty()) {
    String errorDesc = m.rs("error_description");
    m.jsError("로그인 실패: " + errorDesc);
    return;
}

// 프로필 조회 실패
HashMap<String, Object> profile = oauth.getProfile(code);
if(profile == null) {
    Malgn.errorLog("OAuth 에러: " + oauth.errMsg);
    m.jsError("로그인 처리 중 오류가 발생했습니다");
    return;
}
```

---

## 추가 기능

### 연동 해제

```jsp
// 사용자의 OAuth 정보 제거
UserDao dao = new UserDao();
DataMap updates = new DataMap();
updates.put("oauth_provider", null);
updates.put("oauth_id", null);
dao.update("id = ?", updates, userId);
```

### 계정 연동

```jsp
// 기존 계정에 OAuth 연동 추가
if(auth.isLogin()) {
    HashMap<String, Object> profile = oauth.getProfile(code);

    UserDao dao = new UserDao();
    DataMap updates = new DataMap();
    updates.put("oauth_provider", vendor);
    updates.put("oauth_id", profile.get("id"));
    dao.update("id = ?", updates, auth.id());

    m.jsAlert("계정이 연동되었습니다");
    m.jsReplace("/mypage");
}
```

---

## 관련 문서

- [인증 처리](authentication.md) - Auth 클래스
- [HTTP 클라이언트](http-client.md) - OAuth가 내부적으로 사용
- [JSON 처리](json.md) - 프로필 데이터 파싱
- [환경설정 및 캐시](configuration.md) - OAuth 설정 관리
