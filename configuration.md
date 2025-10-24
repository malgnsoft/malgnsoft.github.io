# 12. 환경 설정

[← 목차로 돌아가기](README.md)

---

## Config 클래스

Config 클래스는 애플리케이션의 환경 설정을 관리하는 정적 클래스입니다.

### 주요 기능

- 설정 파일 읽기 (config.properties)
- 경로 정보 제공
- 환경 변수 관리
- 설정 리로드

---

## 기본 경로 메소드

### 주요 경로 가져오기

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 문서 루트 경로
String docRoot = Config.getDocRoot();
m.p(docRoot);  // 예: G:\workspace\malgn\public_html

// 템플릿 루트 경로
String tplRoot = Config.getTplRoot();
m.p(tplRoot);  // 예: G:\workspace\malgn\public_html\html

// 데이터 디렉토리
String dataDir = Config.getDataDir();
m.p(dataDir);  // 예: G:\workspace\malgn\data

// 업로드 디렉토리
String uploadDir = Config.getUploadDir();
m.p(uploadDir);  // 예: G:\workspace\malgn\public_html\upload

%>
```

### 경로 활용 예제

```jsp
// 파일 읽기
String filePath = Config.getDataDir() + "/sample.txt";
BufferedReader reader = new BufferedReader(new FileReader(filePath));

// Excel 파일 읽기
ExcelX excel = new ExcelX();
DataSet data = excel.read(Config.getDataDir() + "/users.xlsx");

// XML 파일 읽기
SimpleParser xml = new SimpleParser(Config.getDocRoot() + "/data/config.xml");

// 템플릿 파일 지정
p.setBody("main.list");  // Config.getTplRoot() + "/main/list.vm"
```

---

## 설정 파일

### config.properties

프로젝트 루트의 `config.properties` 파일에서 설정을 관리합니다.

```properties
# 경로 설정
docRoot=/var/www/html
tplRoot=/var/www/html/html
dataDir=/var/data
uploadDir=/var/www/html/upload

# 이메일 설정
mailFrom=noreply@example.com
mailHost=smtp.gmail.com
mailPort=587
mailUser=admin@example.com
mailPassword=your_password

# API 키
googleApiKey=YOUR_GOOGLE_API_KEY
naverApiKey=YOUR_NAVER_API_KEY

# 기타 설정
siteName=My Website
siteUrl=https://example.com
debugMode=false
```

### 설정 값 읽기

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 문자열 값
String mailFrom = Config.get("mailFrom");
m.p(mailFrom);  // "noreply@example.com"

String siteName = Config.get("siteName");
m.p(siteName);  // "My Website"

// 기본값과 함께
String apiKey = Config.get("apiKey", "default_key");

%>
```

### 정수 값 읽기

```jsp
// 정수로 읽기
int mailPort = Config.getInt("mailPort");
m.p(mailPort);  // 587

// 기본값과 함께
int timeout = Config.getInt("timeout", 30);
```

### 불리언 값 읽기

```jsp
// true/false 값
String debugModeStr = Config.get("debugMode");
boolean debugMode = "true".equalsIgnoreCase(debugModeStr);

if(debugMode) {
    m.p("디버그 모드 활성화");
}
```

---

## 설정 리로드

### Config.reload()

설정 파일을 다시 읽어옵니다.

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 설정 파일 리로드
Config.reload();

m.jsAlert("설정이 다시 로드되었습니다.");
m.jsReplace("admin.jsp");

%>
```

**사용 시나리오**:
- config.properties 파일을 수정한 후
- 서버 재시작 없이 설정 반영이 필요할 때

---

## Cache 클래스

Cache 클래스는 데이터를 메모리에 캐시하여 성능을 향상시킵니다.

### 기본 사용법

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

Cache cache = new Cache();

// 캐시에서 데이터 가져오기
DataSet posts = cache.getDataSet("posts_cache");

if(posts == null) {
    // 캐시에 데이터가 없으면 DB에서 조회
    BlogDao dao = new BlogDao();
    posts = dao.query("WHERE status = 1 ORDER BY id DESC");

    // 캐시에 저장
    cache.save("posts_cache", posts);
    m.p("데이터베이스에서 조회");
} else {
    m.p("캐시에서 조회");
}

// 데이터 사용
while(posts.next()) {
    m.p(posts.getString("subject"));
}

%>
```

### 캐시 저장

```jsp
Cache cache = new Cache();

// DataSet 저장
DataSet data = new DataSet();
data.addRow();
data.put("key", "value");
cache.save("my_data", data);

// 문자열 저장
cache.save("site_name", "My Website");

// 숫자 저장
cache.save("view_count", 1234);
```

### 캐시 조회

```jsp
Cache cache = new Cache();

// DataSet 조회
DataSet data = cache.getDataSet("my_data");

// 문자열 조회
String siteName = cache.getString("site_name");

// 객체 조회
Object value = cache.get("key");
```

### 캐시 삭제

```jsp
Cache cache = new Cache();

// 특정 캐시 삭제
cache.remove("posts_cache");

// 모든 캐시 삭제
cache.clear();
```

---

## 실전 예제

### 1. 이메일 발송 설정

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 이메일 설정 읽기
String mailHost = Config.get("mailHost");
int mailPort = Config.getInt("mailPort");
String mailUser = Config.get("mailUser");
String mailPassword = Config.get("mailPassword");
String mailFrom = Config.get("mailFrom");

// 이메일 발송 (가상 코드)
Email email = new Email();
email.setHost(mailHost);
email.setPort(mailPort);
email.setAuth(mailUser, mailPassword);
email.setFrom(mailFrom);
email.setTo("user@example.com");
email.setSubject("환영합니다");
email.setBody("회원가입을 환영합니다.");
email.send();

%>
```

### 2. 메뉴 캐싱

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

Cache cache = new Cache();
DataSet menu = cache.getDataSet("main_menu");

if(menu == null) {
    // DB에서 메뉴 조회
    MenuDao dao = new MenuDao();
    menu = dao.query("WHERE status = 1 ORDER BY order_num ASC");

    // 캐시에 저장 (1시간)
    cache.save("main_menu", menu);
}

p.setLoop("menu", menu);
p.setBody("main.index");
p.display();

%>
```

### 3. 사이트 통계 캐싱

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

Cache cache = new Cache();
DataMap stats = cache.getDataMap("site_stats");

if(stats == null) {
    stats = new DataMap();

    // 회원 수
    UserDao userDao = new UserDao();
    DataSet users = userDao.query("WHERE status = 1");
    stats.put("user_count", users.size());

    // 게시물 수
    BlogDao blogDao = new BlogDao();
    DataSet posts = blogDao.query("WHERE status = 1");
    stats.put("post_count", posts.size());

    // 방문자 수 (오늘)
    String today = m.time("yyyyMMdd");
    VisitDao visitDao = new VisitDao();
    DataSet visits = visitDao.query("WHERE visit_date = ?", today);
    stats.put("today_visit", visits.size());

    // 캐시에 저장 (10분)
    cache.save("site_stats", stats);
}

p.setVar("user_count", stats.getInt("user_count"));
p.setVar("post_count", stats.getInt("post_count"));
p.setVar("today_visit", stats.getInt("today_visit"));
p.display();

%>
```

### 4. API 키 관리

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// Google Maps API 호출
String googleApiKey = Config.get("googleApiKey");
String mapUrl = "https://maps.googleapis.com/maps/api/geocode/json?address=Seoul&key=" + googleApiKey;

Http http = new Http(mapUrl);
String response = http.send("GET");
m.p(response);

// Naver API 호출
String naverApiKey = Config.get("naverApiKey");
// API 호출 코드...

%>
```

### 5. 환경별 설정

**config.properties (개발)**:
```properties
docRoot=/workspace/myapp/public_html
dataDir=/workspace/myapp/data
debugMode=true
siteUrl=http://localhost:8080
```

**config.properties (운영)**:
```properties
docRoot=/var/www/html
dataDir=/var/data
debugMode=false
siteUrl=https://www.example.com
```

**사용 예제**:
```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

String siteUrl = Config.get("siteUrl");
boolean isDebug = "true".equalsIgnoreCase(Config.get("debugMode"));

if(isDebug) {
    // 개발 환경
    m.p("개발 모드");
    // 상세한 에러 메시지 출력
} else {
    // 운영 환경
    m.p("운영 모드");
    // 간단한 에러 메시지만 출력
}

p.setVar("siteUrl", siteUrl);

%>
```

### 6. 캐시 무효화

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 관리자가 메뉴를 수정한 후
if(m.isPost() && f.validate()) {

    MenuDao dao = new MenuDao();
    DataMap menu = new DataMap();
    menu.put("title", f.get("title"));
    menu.put("url", f.get("url"));
    dao.insert(menu);

    // 메뉴 캐시 삭제
    Cache cache = new Cache();
    cache.remove("main_menu");

    m.jsAlert("메뉴가 등록되었습니다.");
    m.jsReplace("list.jsp");
}

%>
```

---

## 베스트 프랙티스

### 1. 민감한 정보는 설정 파일에

```jsp
// Bad - 코드에 하드코딩
String apiKey = "abc123def456";

// Good - 설정 파일에서 읽기
String apiKey = Config.get("apiKey");
```

### 2. 캐시 키 명명 규칙

```jsp
// 명확하고 일관된 이름 사용
cache.save("menu_main", data);          // 메인 메뉴
cache.save("menu_footer", data);        // 푸터 메뉴
cache.save("stats_today", data);        // 오늘 통계
cache.save("banner_home", data);        // 홈 배너
```

### 3. 캐시 갱신 전략

```jsp
// 시간 기반 갱신
long cacheTime = cache.getLong("menu_cache_time");
long now = System.currentTimeMillis();

if(cacheTime == 0 || (now - cacheTime) > 3600000) {  // 1시간
    // 캐시 갱신
    DataSet menu = dao.query("...");
    cache.save("main_menu", menu);
    cache.save("menu_cache_time", now);
}
```

### 4. 기본값 설정

```jsp
// 설정이 없을 때 기본값 사용
int pageSize = Config.getInt("pageSize", 20);  // 기본 20개
String siteName = Config.get("siteName", "My Site");  // 기본 이름
```

---

## 주요 메소드 정리

### Config 클래스

| 메소드 | 설명 |
|--------|------|
| getDocRoot() | 문서 루트 경로 |
| getTplRoot() | 템플릿 루트 경로 |
| getDataDir() | 데이터 디렉토리 |
| getUploadDir() | 업로드 디렉토리 |
| get(key) | 설정 값 가져오기 |
| get(key, defaultValue) | 기본값과 함께 가져오기 |
| getInt(key) | 정수로 가져오기 |
| getInt(key, defaultValue) | 기본값과 함께 정수로 |
| reload() | 설정 파일 다시 읽기 |

### Cache 클래스

| 메소드 | 설명 |
|--------|------|
| save(key, value) | 캐시에 저장 |
| get(key) | 캐시에서 가져오기 |
| getDataSet(key) | DataSet으로 가져오기 |
| getDataMap(key) | DataMap으로 가져오기 |
| getString(key) | 문자열로 가져오기 |
| getInt(key) | 정수로 가져오기 |
| getLong(key) | Long으로 가져오기 |
| remove(key) | 특정 캐시 삭제 |
| clear() | 모든 캐시 삭제 |

---

## 성능 최적화

### 1. 자주 조회되는 데이터 캐싱

```jsp
// 카테고리 목록 (거의 변경되지 않음)
Cache cache = new Cache();
DataSet categories = cache.getDataSet("categories");
if(categories == null) {
    categories = dao.query("WHERE status = 1");
    cache.save("categories", categories);
}
```

### 2. 복잡한 쿼리 결과 캐싱

```jsp
// 통계 데이터 (매번 계산하면 느림)
DataMap stats = cache.getDataMap("monthly_stats");
if(stats == null) {
    stats = dao.getMonthlyStats();  // 복잡한 집계 쿼리
    cache.save("monthly_stats", stats);
}
```

### 3. API 응답 캐싱

```jsp
// 외부 API 호출 결과 캐싱
String weatherData = cache.getString("weather_data");
if(weatherData == null) {
    Http http = new Http("https://api.weather.com/...");
    weatherData = http.send("GET");
    cache.save("weather_data", weatherData);
}
```

---

## 마무리

축하합니다! MalgnFramework의 모든 핵심 기능을 학습하셨습니다.

### 학습한 내용

1. [프레임워크 소개](introduction.md) - 기본 개념과 구조
2. [설치 및 환경설정](installation.md) - 설치 및 초기 설정
3. [시작하기](getting-started.md) - 첫 페이지 만들기
4. [템플릿](template.md) - 템플릿 시스템
5. [데이터베이스](database.md) - CRUD 작업
6. [데이터 입력 및 유효성 체크](form-validation.md) - 폼 처리
7. [목록 및 검색](list-search.md) - 페이징과 검색
8. [DataSet 활용](dataset.md) - 데이터 처리
9. [XML 처리](xml.md) - XML 파싱
10. [Excel 처리](excel.md) - 엑셀 읽기/쓰기
11. [인증 처리](authentication.md) - 로그인/로그아웃
12. [환경 설정](configuration.md) - 설정 및 캐시

### 다음 단계

- 실제 프로젝트에 적용해보기
- 공식 매뉴얼 참고하기
- 커뮤니티에서 도움 받기

---

[← 이전: 인증 처리](authentication.md) | [목차로 돌아가기](README.md)
