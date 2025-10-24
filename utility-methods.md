# Malgn 유틸리티 메소드

[← 목차로 돌아가기](README.md)

---

## 개요

Malgn 클래스(`m`)는 맑은프레임워크의 핵심 유틸리티 클래스로, 웹 개발에 필요한 다양한 편의 메소드를 제공합니다.

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// init.jsp에서 자동으로 생성됨
Malgn m = new Malgn(request, response, out);

%>
```

---

## 목차

1. [요청 파라미터 처리](#요청-파라미터-처리)
2. [날짜/시간 처리](#날짜시간-처리)
3. [문자열 처리](#문자열-처리)
4. [파일 처리](#파일-처리)
5. [자바스크립트 출력](#자바스크립트-출력)
6. [쿠키/세션](#쿠키세션)
7. [배열/컬렉션 처리](#배열컬렉션-처리)
8. [로깅](#로깅)
9. [기타 유틸리티](#기타-유틸리티)

---

## 요청 파라미터 처리

### 기본 파라미터 받기

```jsp
// 문자열 파라미터 (XSS 방어 포함)
String name = m.rs("name");                    // 기본값 ""
String email = m.rs("email", "default@example.com");  // 기본값 지정

// 정수 파라미터
int page = m.ri("page");                       // 기본값 0
int limit = m.ri("limit", 20);                 // 기본값 20

// 배열 파라미터 (체크박스 등)
String[] hobbies = m.reqArr("hobby");

// Enum 방식 (허용된 값만 받기)
String[] allowedStatus = {"active", "inactive", "pending"};
String status = m.reqEnum("status", allowedStatus);  // 배열에 없으면 첫 번째 값 반환

// POST Body 전체 읽기 (JSON API 등)
String jsonBody = m.reqBody();

// 파라미터 맵으로 받기 (접두사로 시작하는 모든 파라미터)
Hashtable items = m.reqMap("item_");  // item_1, item_2, item_3... 을 모두 받아옴
```

**주요 메소드**:
- `rs(name)`, `rs(name, default)` - 문자열 파라미터 (XSS 방어 자동 적용)
- `ri(name)`, `ri(name, default)` - 정수 파라미터
- `reqArr(name)` - 배열 파라미터
- `reqEnum(name, allowedValues)` - Enum 검증
- `reqBody()` - POST Body 전체
- `reqMap(prefix)` - 접두사로 그룹핑된 파라미터

### 메소드 체크

```jsp
if(m.isPost()) {
    // POST 요청 처리
}
```

---

## 날짜/시간 처리

### 현재 시간

```jsp
// 기본 포맷 (yyyyMMddHHmmss)
String now = m.time();  // "20250623153045"

// 포맷 지정
String today = m.time("yyyyMMdd");              // "20250623"
String datetime = m.time("yyyy-MM-dd HH:mm:ss"); // "2025-06-23 15:30:45"
String time = m.time("HH:mm");                  // "15:30"
```

### 날짜 포맷 변환

```jsp
// 문자열 → 다른 포맷
String date = "20250623";
String formatted = m.time("yyyy-MM-dd", date);  // "2025-06-23"

// Date 객체 → 문자열
Date d = new Date();
String str = m.time("yyyy-MM-dd", d);

// Unix timestamp → 문자열
int unixTime = 1687516800;
String dateStr = m.time("yyyy-MM-dd", unixTime);

// 시간대 변환
String utcTime = "20230425113600";
String koreaTime = m.time("yyyy-MM-dd HH:mm:ss", utcTime, "Asia/Seoul");
```

### 날짜 계산

```jsp
// 날짜 더하기/빼기
String tomorrow = m.addDate("D", 1, m.time(), "yyyyMMdd");   // 1일 후
String nextMonth = m.addDate("M", 1, m.time(), "yyyyMMdd");  // 1개월 후
String nextYear = m.addDate("Y", 1, m.time(), "yyyyMMdd");   // 1년 후
String yesterday = m.addDate("D", -1, m.time(), "yyyyMMdd"); // 1일 전

// 날짜 차이 계산
long days = m.diffDate("D", "20250101", "20251231");    // 일 차이
long months = m.diffDate("M", "202501", "202512");      // 월 차이
long hours = m.diffDate("H", start, end);                // 시간 차이
long minutes = m.diffDate("I", start, end);              // 분 차이
long seconds = m.diffDate("S", start, end);              // 초 차이
```

**타입 코드**:
- `Y` - 년 (Year)
- `M` - 월 (Month)
- `W` - 주 (Week)
- `D` - 일 (Day)
- `H` - 시 (Hour)
- `I` - 분 (mInute)
- `S` - 초 (Second)

### Unix Timestamp

```jsp
// 현재 Unix timestamp
int unixTime = m.getUnixTime();       // 초 단위
long unixTimeL = m.getUnixTimeL();    // 초 단위 (long)

// 날짜 → Unix timestamp
int timestamp = m.getUnixTime("20251231");
```

### 문자열 → Date 변환

```jsp
Date d1 = Malgn.strToDate("20250623");                        // yyyyMMdd 자동 인식
Date d2 = Malgn.strToDate("2025-06-23");                      // yyyy-MM-dd 자동 인식
Date d3 = Malgn.strToDate("yyyy년 MM월 dd일", "2025년 06월 23일");  // 커스텀 포맷
```

---

## 문자열 처리

### 문자열 자르기

```jsp
// 한글 고려하여 자르기
String text = "우리는 자랑스러운 대한민국 국민입니다";
String cut = m.cutString(text, 10, "...");  // "우리는 자랑스..."

// 반복
String repeated = Malgn.repeatString("*", 10);  // "**********"

// 패딩
String padded = Malgn.strpad("abc", 10, "0");   // "abc0000000"
String rpadded = Malgn.strrpad("abc", 10, "0"); // "0000000abc"
```

### 문자열 치환

```jsp
// 단일 치환
String result = Malgn.replace("Hello World", "World", "Java");

// 다중 치환
String html = "<div>'test'</div>";
String safe = Malgn.replace(html,
    new String[]{"<", ">", "'"},
    new String[]{"&lt;", "&gt;", "&#39;"}
);

// HTML 엔티티로 변환
String escaped = Malgn.htmlentities("<script>alert('xss')</script>");
String escaped2 = Malgn.htt("<div>content</div>");  // 축약형

// HTML to Text
String text = Malgn.htmlToText("<b>Bold</b>\nNew line");  // <b>Bold</b><br />New line

// HTML 태그 제거
String plain = Malgn.stripTags("<div>Hello <b>World</b></div>");  // "Hello World"

// 개행을 <br>로
String html = Malgn.nl2br("Line1\nLine2\nLine3");
```

### 문자열 분할/결합

```jsp
// 분할
String[] parts = Malgn.split(",", "apple,banana,orange");
String[] parts2 = Malgn.split(",", "a,b,c", 5);  // 5개로 고정 (부족하면 "" 채움)

// 결합
String joined = Malgn.join(",", new String[]{"a", "b", "c"});  // "a,b,c"
```

### URL 인코딩

```jsp
// URL 인코딩/디코딩
String encoded = Malgn.urlencode("한글 테스트");
String decoded = Malgn.urldecode(encoded);

// Base64 인코딩/디코딩
String encoded64 = Malgn.encode("Hello World");
String decoded64 = Malgn.decode(encoded64);
```

### 문자열 검증

```jsp
// 배열에 포함 여부
boolean exists = Malgn.inArray("apple", new String[]{"apple", "banana"});  // true
boolean exists2 = Malgn.inArray("cherry", "apple,banana,orange");          // false

// SQL 안전 문자열
String safeSql = m.qstr("O'Reilly");  // "O''Reilly"
```

### 마스킹

```jsp
// 개인정보 마스킹
String masked1 = m.masking("홍길동");      // "홍*동"
String masked2 = m.masking("1234567890");  // "1********0"
String masked3 = m.masking("AB");          // "A*"
```

### Hex 변환

```jsp
// 바이트 → Hex
byte[] bytes = new byte[]{'a', 'b', 'c'};
String hex = Malgn.bytesToHex(bytes);  // "616263"

// Hex → 바이트
byte[] decoded = Malgn.hexToBytes("616263");
```

---

## 파일 처리

### 파일 읽기/쓰기

```jsp
// 파일 읽기
String content = Malgn.readFile("/path/to/file.txt");
String content2 = Malgn.readFile("/path/to/file.txt", "EUC-KR");

// 파일 쓰기
Malgn.writeFile("/path/to/output.txt", "Hello World");
Malgn.writeFile("/path/to/output.txt", "안녕하세요", "EUC-KR");
```

### 파일 복사/삭제

```jsp
// 파일/폴더 복사
Malgn.copyFile("/source/file.txt", "/dest/file.txt");
Malgn.copyFile("/source/dir", "/dest/dir");  // 폴더 전체 복사

// 파일/폴더 삭제
Malgn.delFile("/path/to/file.txt");                // 파일 삭제
Malgn.delFile("/path/to/dir", true);               // 폴더 재귀 삭제
Malgn.delFileRoot("/path/to/dir");                 // 폴더 내용물만 삭제
```

### 파일 다운로드

```jsp
// 파일 다운로드
String filePath = "/data/document.pdf";
String fileName = "문서.pdf";
m.download(filePath, fileName);

// 대역폭 제한 (KB/s)
m.download(filePath, fileName, 100);  // 100 KB/s로 제한

// 파일 출력 (브라우저에 직접 표시)
m.output(filePath, "application/pdf");
m.output(filePath, null);  // MIME 타입 자동 감지
```

### 파일 정보

```jsp
// 확장자 추출
String ext = Malgn.getFileExt("document.pdf");  // "pdf"

// MIME 타입
String mime = Malgn.getMimeType("image.png");   // "image/png"

// 파일 크기 포맷
String size = Malgn.getFileSize(1024);          // "1KB"
String size2 = Malgn.getFileSize(1048576);      // "1MB"
String size3 = Malgn.getFileSize(1073741824);   // "1GB"
```

### 업로드 경로

```jsp
// 업로드 파일 경로 (MD5 기반)
String uploadPath = m.getUploadPath("myfile.jpg");
// /data/file/abc123def456.jpg

String uploadUrl = m.getUploadUrl("myfile.jpg");
// http://example.com/file/abc123def456.jpg
```

---

## 자바스크립트 출력

### 기본 알림 및 이동

```jsp
// alert
m.jsAlert("저장되었습니다");

// alert + 뒤로가기
m.jsError("오류가 발생했습니다");

// alert + 창 닫기
m.jsErrClose("완료되었습니다");
m.jsErrClose("완료되었습니다", "opener");  // 부모창에서 알림

// 페이지 이동 (location.replace)
m.jsReplace("/main/list.jsp");
m.jsReplace("/main/list.jsp", "parent");  // 부모창 이동

// 리다이렉트 (서버측)
m.redirect("/main/list.jsp");

// 커스텀 자바스크립트
m.js("console.log('Hello');");
m.js("alert('Test'); location.href='/home';");
```

### HTTP 에러

```jsp
// HTTP 에러 응답
m.httpError(404, "Page Not Found");
m.httpError(403, "Forbidden");
m.httpError(500, "Internal Server Error");
```

---

## 쿠키/세션

### 쿠키

```jsp
// 쿠키 설정
m.setCookie("userId", "user001");                     // 세션 쿠키
m.setCookie("remember", "Y", 60 * 60 * 24 * 30);    // 30일 유지

// 쿠키 읽기
String userId = m.getCookie("userId");

// 쿠키 삭제
m.delCookie("userId");
```

### 세션

```jsp
// 세션 설정
m.setSession("userName", "홍길동");
m.setSession("userLevel", 10);

// 세션 읽기
String userName = m.getSession("userName");
int userLevel = m.getSessionInt("userLevel");

// 세션 삭제
m.removeSession("userName");
```

---

## 배열/컬렉션 처리

### 배열을 DataSet으로 변환

```jsp
// 배열을 템플릿 루프용 DataSet으로 변환
String[] status = {"active=>활성", "inactive=>비활성", "pending=>대기"};
DataSet loop = Malgn.arr2loop(status);

p.setLoop("statusList", loop);
// 템플릿에서: {{statusList.id}}, {{statusList.value}}

// Map을 DataSet으로
Hashtable<String, String> map = new Hashtable<>();
map.put("KR", "한국");
map.put("US", "미국");
map.put("JP", "일본");
DataSet countries = Malgn.arr2loop(map);
```

### 배열/Map 조작

```jsp
// 배열에서 값 찾기
String[] items = {"1=>사과", "2=>바나나", "3=>오렌지"};
String value = Malgn.getItem("2", items);  // "바나나"

// Map에서 값 찾기
String value2 = Malgn.getItem("key", hashtable);

// 키 추출
String[] keys = Malgn.getKeys(items);      // ["1", "2", "3"]
String[] keys2 = Malgn.getKeys(map);       // Map의 모든 키
```

### 직렬화

```jsp
// 객체 직렬화 (파일 저장)
DataSet data = new DataSet();
boolean success = Malgn.serialize("/path/to/data.ser", data);

// 역직렬화 (파일 읽기)
DataSet loaded = (DataSet)Malgn.unserialize("/path/to/data.ser");

// Map ↔ String 변환
Hashtable<String, String> map = new Hashtable<>();
map.put("name", "홍길동");
map.put("age", "30");
String mapStr = Malgn.mapToString(map);  // "name:홍길동,age:30"

Hashtable<String, String> restored = Malgn.strToMap(mapStr);
```

---

## 로깅

### 기본 로깅

```jsp
// 일반 로그 (debug_YYYYMMDD.log)
m.log("사용자 로그인");

// 접두사 지정 (prefix_YYYYMMDD.log)
m.log("access", "페이지 접근: " + m.getThisURI());

// 정적 메소드 (호출 위치 자동 기록)
Malgn.logger("info", "중요한 이벤트 발생");

// 에러 로그 (error_YYYYMMDD.log)
try {
    // 처리
} catch(Exception e) {
    Malgn.errorLog("처리 중 오류 발생", e);
}
```

---

## 기타 유틸리티

### 요청 정보

```jsp
// 쿼리 스트링
String qs = m.qs();              // 전체 쿼리 스트링
String qs2 = m.qs("page,id");    // 특정 파라미터 제외

// 현재 URI/URL
String uri = m.getThisURI();     // /main/list.jsp?page=1
String url = m.getThisURL();     // http://example.com/main/list.jsp?page=1

// 웹 URL (도메인까지만)
String webUrl = m.getWebUrl();   // http://example.com

// 실제 IP 주소 (프록시 고려)
String ip = m.getRemoteAddr();
```

### 모바일 감지

```jsp
if(m.isMobile()) {
    // 모바일 처리
} else {
    // PC 처리
}
```

### 숫자 포맷

```jsp
// 천단위 콤마
String formatted = Malgn.numberFormat(1234567);      // "1,234,567"
String formatted2 = Malgn.nf(1234567);               // 축약형

// 소수점 포맷
String decimal = Malgn.nf(123.456, 2);               // "123.45" (버림)
String decimal2 = Malgn.nf(123.456, 2, true);        // "123.46" (반올림)

// 반올림
double rounded = Malgn.round(123.456, 2);            // 123.46

// 퍼센트 계산
double percent = Malgn.getPercent(25, 100);          // 25.0
```

### 랜덤/UUID

```jsp
// 랜덤 정수
int rand = Malgn.getRandInt(1, 100);      // 1~100 사이

// 랜덤 문자열 (UUID)
String uuid = Malgn.getUniqId();          // 10자 (기본)
String uuid20 = Malgn.getUniqId(20);      // 20자
```

### URL 필터

```jsp
// 현재 URL이 패턴과 매치되는지 확인
if(m.urlFilter("/admin/*", "/manager/*")) {
    // 관리자 영역
}

if(m.urlFilter("/api/v1/*")) {
    // API 처리
}
```

### 시스템 명령

```jsp
// 외부 명령 실행
String output = Malgn.exec("ls -la");

// chmod (Linux/Unix)
Malgn.chmod("755", "/path/to/script.sh");
```

### 메일 발송

```jsp
// 동기 메일 발송
m.mail("user@example.com", "제목", "<h1>내용</h1>");
m.mail("user@example.com", "제목", "내용", "/path/to/attachment.pdf");

// 비동기 메일 발송 (스레드 풀 사용)
m.mailer("user@example.com", "제목", "내용");
m.mailer("user@example.com", "제목", "내용", "/path/to/file.pdf");
```

### 디버깅

```jsp
// 변수 출력 (개발 중)
m.p(dataSet);           // DataSet 예쁘게 출력
m.p(object);            // Object 출력
m.p(array);             // 배열 출력
m.p();                  // 모든 파라미터 출력

// 타이머
m.startTimer();
// ... 처리 ...
String elapsedTime = m.stopTimer();  // "1.234" (초 단위)
m.p("실행 시간: " + elapsedTime + "초");
```

### JSON 변환

```jsp
// JSON → Map
String json = "{\"name\":\"홍길동\",\"age\":30}";
HashMap<String, Object> map = m.jsonToMap(json);

// JSON → List
String jsonArray = "[1, 2, 3, 4, 5]";
ArrayList<Object> list = m.jsonToList(jsonArray);
```

### 기타

```jsp
// CRC32 체크섬
long checksum = Malgn.crc32("data");

// 디렉토리 경로
String dir = Malgn.dirname("/path/to/file.txt");  // "/path/to"

// 스크립트 디렉토리
String scriptDir = m.getScriptDir();

// MX 레코드 조회
String mx = m.getMX("gmail.com");  // "gmail-smtp-in.l.google.com"
```

---

## 정적 vs 인스턴스 메소드

### 정적 메소드 (static)
어디서나 `Malgn.` 접두사로 호출 가능

```jsp
String now = Malgn.time();
String hashed = Malgn.sha256("password");
String formatted = Malgn.numberFormat(1234567);
```

### 인스턴스 메소드
`m` 객체를 통해 호출 (request/response 필요)

```jsp
String param = m.rs("name");
m.jsAlert("메시지");
m.redirect("/home");
String ip = m.getRemoteAddr();
```

---

## 베스트 프랙티스

### 1. XSS 방어

```jsp
// Bad - XSS 취약
String name = request.getParameter("name");
out.print(name);

// Good - m.rs()는 자동으로 XSS 방어
String name = m.rs("name");
out.print(name);

// 또는 명시적 이스케이프
out.print(Malgn.htt(name));
```

### 2. 파라미터 타입 검증

```jsp
// Bad
String pageStr = request.getParameter("page");
int page = Integer.parseInt(pageStr);  // NumberFormatException 가능

// Good
int page = m.ri("page", 1);  // 안전하게 파싱, 실패시 기본값
```

### 3. 날짜 처리

```jsp
// 일관된 포맷 사용
String regDate = m.time();  // yyyyMMddHHmmss

// 화면 표시용 포맷 변환
String displayDate = m.time("yyyy-MM-dd HH:mm", regDate);
```

### 4. 로깅

```jsp
// 개발 중에는 m.p()로 디버깅
m.p(dataSet);

// 운영에서는 logger로 로그 파일에 기록
Malgn.logger("debug", "데이터: " + dataSet.size() + "건");
```

---

## 다음 단계

Malgn 클래스의 유틸리티 메소드를 활용하여 효율적인 개발을 진행하세요!

---

[← 목차로 돌아가기](README.md)
