# 맑은프레임워크 v1.3 통합 매뉴얼

**Java JSP 웹 개발 프레임워크**

버전: 1.3
최종 수정일: 2025-10-24
공식 사이트: https://malgnsoft.github.io

---

## 목차

### 기본 가이드
1. [프레임워크 소개](#1-프레임워크-소개)
2. [설치 및 환경설정](#2-설치-및-환경설정)
3. [시작하기](#3-시작하기)

### 핵심 기능
4. [맑은템플릿](#4-맑은템플릿)
5. [데이터베이스 연동](#5-데이터베이스-연동)
5-1. [DataObject 클래스](#5-1-dataobject-클래스)
6. [데이터 입력 및 유효성 체크](#6-데이터-입력-및-유효성-체크)
7. [파일 업로드 및 다운로드](#7-파일-업로드-및-다운로드)
8. [목록 및 검색](#8-목록-및-검색)
9. [DataSet 활용](#9-dataset-활용)

### 데이터 처리
10. [JSON 처리](#10-json-처리)
11. [XML 처리](#11-xml-처리)
12. [Excel 처리](#12-excel-처리)

### 보안 및 인증
13. [암호화](#13-암호화)
14. [인증 처리](#14-인증-처리)
15. [OAuth 소셜 로그인](#15-oauth-소셜-로그인)

### 고급 기능
16. [HTTP 클라이언트](#16-http-클라이언트)
17. [이메일 발송](#17-이메일-발송)
18. [달력 및 날짜 선택](#18-달력-및-날짜-선택)
19. [유틸리티 메소드](#19-유틸리티-메소드)
20. [다국어 지원](#20-다국어-지원)
21. [OpenAI 통합](#21-openai-통합)
22. [파일 전송 및 압축](#22-파일-전송-및-압축)
23. [환경설정 및 캐시](#23-환경설정-및-캐시)

---

## 빠른 참조

### 핵심 클래스

| 클래스 | 변수명 | 주요 기능 |
|--------|--------|-----------|
| Malgn | m | 요청/응답 처리, 디버깅, 유틸리티 |
| Form | f | 폼 데이터 처리, 유효성 검증 |
| Page | p | 템플릿 렌더링, 변수 치환 |
| Json | j | JSON 파싱 및 생성 |
| Auth | auth | 인증 처리, 세션 관리 |
| DataObject | - | DAO 베이스 클래스 |
| DataSet | - | 데이터 컬렉션 |
| ListManager | - | 페이징 및 검색 |

### 빠른 시작

#### Hello World
```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%
m.p("Hello, World!");
%>
```

#### 템플릿 렌더링
```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%
p.setBody("main.index");
p.setVar("title", "환영합니다");
p.display();
%>
```

#### 데이터베이스 조회
```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%
UserDao user = new UserDao();
DataSet list = user.query("SELECT * FROM tb_user ORDER BY id DESC");
p.setBody("user.list");
p.setLoop("list", list);
p.display();
%>
```

---

## 중요 참고사항

### Page 메소드 호출 순서
Page 클래스 메소드는 반드시 다음 순서로 호출해야 합니다:

```jsp
1. p.setLayout()    // 레이아웃 설정 (선택사항)
2. p.setBody()      // 템플릿 파일 지정 (필수)
3. p.setVar()       // 변수 설정 (선택사항)
   p.setLoop()      // 루프 변수 설정 (선택사항)
4. p.display()      // 출력 (필수)
```

**올바른 예**:
```jsp
p.setBody("main.index");
p.setVar("title", "제목");
p.display();
```

**잘못된 예**:
```jsp
p.setVar("title", "제목");  // setBody() 전에 setVar() 호출 - 잘못됨!
p.setBody("main.index");
p.display();
```

### JSP에서 HTML 직접 출력 금지
JSP 파일 내에 HTML을 직접 작성하지 마세요. 반드시 별도의 템플릿 파일로 분리하고 `setBody()`와 `display()`를 사용하세요:

**잘못된 예**:
```jsp
<%
p.setVar("title", "제목");
%>
<html><body>{{title}}</body></html>
```

**올바른 예**:
```jsp
<%
p.setBody("main.index");
p.setVar("title", "제목");
p.display();
%>
```

### if(m.isPost()) 사용 시 주의
모든 `if(m.isPost())` 블록은 반드시 `return;`으로 종료해야 합니다:

```jsp
if(m.isPost()) {
    // 처리 로직
    m.jsAlert("저장되었습니다");
    m.jsReplace("list.jsp");
    return;  // 필수!
}
```

### 한글 인코딩
모든 JSP 파일 상단에 charset 지정:
```jsp
<%@ page contentType="text/html; charset=utf-8" %>
```

### 공백 제거
init.jsp와 다운로드 처리 JSP는 공백/개행 제거 필수:
```jsp
<%@ page ... %><%@ include ... %><%
// 코드
%>
```

---

# 1. 프레임워크 소개

[← 목차로 돌아가기](README.md)

---

## 개요

맑은프레임워크는 자바 웹개발 프레임워크입니다. WAS(Web Application Server) 안에서 구동되는 솔루션이며, 간결하고 효율적인 웹 애플리케이션 개발을 지원합니다.

---

## 주요 특징

### 1. 템플릿 엔진
- 프로그램 로직과 화면 출력을 완전히 분리
- 웹디자이너/퍼블리셔가 쉽게 수정 가능한 단순화된 문법
- 변수 치환, 루프, 조건문, 레이아웃 기능 제공

### 2. DataSet 객체
- 모든 데이터를 통일된 형태로 처리
- 데이터베이스, XML, Excel 등 다양한 소스 지원
- JSON 직렬화/역직렬화 지원

### 3. 데이터베이스 추상화
- Oracle, MySQL, MSSQL, DB2 등 다양한 DBMS 지원
- DAO 패턴을 통한 깔끔한 데이터베이스 처리
- Connection Pool 자동 관리

### 4. 유효성 검증
- 한 번의 설정으로 서버/클라이언트 양측 검증
- 다양한 기본 검증 규칙 제공
- 사용자 정의 검증 규칙 추가 가능

### 5. 간단한 설정
- 최소한의 환경설정 파일
- XML 기반의 직관적인 설정
- 운영 중 설정 변경 가능 (재시작 불필요)

---

## 아키텍처

### MVC 패턴 기반

```
[Browser]
    ↓ Request
[JSP Controller]
    ↓
[Model (DAO/DataSet)]
    ↓
[Database]
    ↑
[View (Template)]
    ↓ Response
[Browser]
```

### 주요 구성 요소

1. **Controller**: JSP 파일
   - 요청 처리
   - 비즈니스 로직 실행
   - 데이터 준비

2. **Model**: DAO 클래스 + DataSet
   - 데이터베이스 연동
   - 비즈니스 로직
   - 데이터 관리

3. **View**: HTML 템플릿
   - 화면 출력
   - 데이터 표시
   - 사용자 인터페이스

---

## 핵심 클래스

### Malgn
유틸리티 메소드를 제공하는 핵심 클래스

**주요 메소드**:
- `p()`: 디버깅 출력
- `isPost()`: POST 방식 체크
- `jsError()`: 에러 메시지 + 이전 페이지 이동
- `jsAlert()`: 알림 메시지
- `jsReplace()`: 페이지 이동
- `time()`: 날짜/시간 포맷팅

### Form
폼 데이터 처리 및 유효성 검증

**주요 메소드**:
- `addElement()`: 폼 항목 정의
- `validate()`: 유효성 검증
- `get()`: 폼 데이터 가져오기
- `getScript()`: 클라이언트 검증 스크립트 생성

### Page
템플릿 처리

**주요 메소드**:
- `setLayout()`: 레이아웃 지정
- `setBody()`: 템플릿 지정
- `setVar()`: 변수 설정
- `setLoop()`: 루프 데이터 설정
- `display()`: 화면 출력

### DataObject
데이터베이스 기본 연동

**주요 메소드**:
- `query()`: SELECT 쿼리 실행
- `execute()`: INSERT/UPDATE/DELETE 실행
- `find()`: 조건으로 검색
- `item()`: 데이터 항목 설정
- `insert()`: 데이터 삽입
- `update()`: 데이터 수정
- `delete()`: 데이터 삭제

### DataSet
데이터 저장 및 처리

**주요 메소드**:
- `addRow()`: 행 추가
- `next()`: 다음 행으로 이동
- `put()`: 데이터 설정
- `get()` / `s()` / `i()`: 데이터 가져오기
- `serialize()`: JSON 직렬화
- `unserialize()`: JSON 역직렬화

### ListManager
목록 및 페이징 처리

**주요 메소드**:
- `setTable()`: 테이블 지정
- `setListNum()`: 목록 개수 지정
- `addWhere()`: 조건 추가
- `addSearch()`: 검색 조건 추가
- `setOrderBy()`: 정렬 지정
- `getDataSet()`: 목록 데이터 가져오기
- `getTotalNum()`: 전체 개수 가져오기
- `getPaging()`: 페이지 네비게이션 생성

### Auth
인증 처리

**주요 메소드**:
- `isValid()`: 인증 유효성 체크
- `put()`: 인증 정보 설정
- `get()` / `getString()` / `getInt()`: 인증 정보 가져오기
- `save()`: 인증 정보 저장
- `delete()`: 인증 정보 삭제

### Config
환경설정 관리

**주요 메소드**:
- `get()`: 설정값 가져오기
- `getInt()`: 정수형 설정값 가져오기
- `getDocRoot()`: 문서 루트 경로
- `getDataDir()`: 데이터 디렉토리 경로
- `getTplRoot()`: 템플릿 루트 경로
- `reload()`: 설정 재로드

---

## 개발 프로세스

### 1. 환경 구축
1. WAS 설치 (Resin, Tomcat 등)
2. 맑은프레임워크 라이브러리 설치
3. 환경설정 파일 작성

### 2. 기본 구조 생성
1. init.jsp 작성
2. 폴더 구조 생성
3. config.xml 설정

### 3. 기능 개발
1. DAO 클래스 작성 (Model)
2. JSP 파일 작성 (Controller)
3. HTML 템플릿 작성 (View)

### 4. 테스트 및 배포
1. 기능 테스트
2. 디버깅
3. 배포

---

## 다음 단계

- [설치 및 환경설정](installation.md)으로 이동하여 프레임워크를 설치하세요.

---

[← 목차로 돌아가기](README.md) | [다음: 설치 및 환경설정 →](installation.md)
# 2. 설치 및 환경설정

[← 목차로 돌아가기](README.md)

---

## WAS 설치

맑은프레임워크는 다양한 WAS에서 구동 가능합니다. 여기서는 Resin을 기준으로 설명합니다.

### Resin 설치

#### 1. 다운로드

- 공식 사이트: http://www.caucho.com
- 권장 버전: Resin 4.0.67 이상
- **주의**: Pro 버전이 아닌 오픈소스 버전을 다운로드하세요

다운로드 URL:
```
http://www.caucho.com/download/resin-4.0.67.zip
```

#### 2. 설치

1. 압축 파일 해제
2. 원하는 위치에 복사 (예: `C:\resin-4.0.67`)
3. JDK 설치 확인

**JDK 다운로드**:
- http://www.oracle.com/technetwork/java/index.html
- Java SE 8 이상 필요

#### 3. 실행

1. `C:\resin-4.0.67\httpd.exe` 실행
2. 브라우저에서 확인: http://localhost:8080/
3. "Welcome to my homepage!!" 메시지 확인

**Document Root**:
```
C:\resin-4.0.67\webapps\Root
```

---

## 맑은프레임워크 설치

### 1. 라이브러리 복사

`malgn.jar` 파일을 `/WEB-INF/lib` 폴더에 복사합니다.

```
/WEB-INF/lib/malgn.jar
```

### 2. web.xml 설정

`/WEB-INF/web.xml` 파일에 다음 내용을 추가합니다:

```xml
<servlet>
    <servlet-name>ConfigServlet</servlet-name>
    <servlet-class>malgnsoft.util.Config</servlet-class>
    <load-on-startup>1</load-on-startup>
</servlet>
```

### 3. config.xml 작성

`/WEB-INF/config.xml` 파일을 생성합니다:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<config>
    <env>
        <jndi>jdbc/malgn</jndi>
    </env>
</config>
```

---

## 데이터베이스 설정

### WAS Connection Pool 설정

`/WEB-INF/resin-web.xml` 파일 생성:

```xml
<web-app xmlns="http://caucho.com/ns/resin">
    <class-loader>
        <library-loader path="WEB-INF/lib"/>
    </class-loader>
    <database>
        <jndi-name>jdbc/malgn</jndi-name>
        <driver type="com.mysql.cj.jdbc.Driver">
            <url>jdbc:mysql://localhost:3306/database</url>
            <user>userid</user>
            <password>password</password>
        </driver>
        <prepared-statement-cache-size>8</prepared-statement-cache-size>
        <max-connections>4</max-connections>
        <max-idle-time>30s</max-idle-time>
    </database>
</web-app>
```

### 데이터베이스별 드라이버

#### MySQL
```xml
<driver>com.mysql.cj.jdbc.Driver</driver>
<url>jdbc:mysql://localhost:3306/database</url>
```

#### Oracle
```xml
<driver>oracle.jdbc.driver.OracleDriver</driver>
<url>jdbc:oracle:thin:@localhost:1521:ORCL</url>
```

#### MSSQL
```xml
<driver>com.microsoft.sqlserver.jdbc.SQLServerDriver</driver>
<url>jdbc:sqlserver://localhost:1433;databaseName=database</url>
```

---

## 폴더 구조 생성

권장하는 폴더 구조를 생성합니다:

```
/                           - Document Root
├── main/                   - 모듈 폴더
├── data/                   - 데이터 폴더
│   ├── file/              - 업로드 파일
│   ├── tmp/               - 임시 파일
│   └── log/               - 로그 파일
├── assets/                 - 정적 파일
│   ├── css/               - CSS 파일
│   ├── js/                - JavaScript 파일
│   └── images/            - 이미지 파일
├── html/                   - 템플릿 루트
│   ├── layout/            - 레이아웃 템플릿
│   └── main/              - 모듈 템플릿
└── WEB-INF/
    ├── lib/               - 라이브러리
    ├── config.xml         - 환경설정
    ├── web.xml            - 웹 설정
    └── resin-web.xml      - DB 설정 (선택)
```

### 폴더 생성 스크립트 (Windows)

```batch
mkdir data
mkdir data\file
mkdir data\tmp
mkdir data\log
mkdir assets
mkdir assets\css
mkdir assets\js
mkdir assets\images
mkdir html
mkdir html\layout
mkdir html\main
mkdir main
```

### 폴더 생성 스크립트 (Linux/Mac)

```bash
mkdir -p data/{file,tmp,log}
mkdir -p assets/{css,js,images}
mkdir -p html/{layout,main}
mkdir main
```

---

## config.xml 환경설정 항목

### 기본 설정

| 항목 | 설명 | 기본값 |
|------|------|--------|
| tplRoot | 템플릿 기본 폴더 경로 | docRoot + /html |
| dataDir | 업로드파일 저장 폴더 경로 | docRoot + /data |
| logDir | 로그파일 저장 폴더 경로 | dataDir + /log |
| webUrl | 웹사이트 주소 | |
| dataUrl | 업로드파일 접근 URL | /data |
| encoding | 인코딩 정보 | UTF-8 |

### 데이터베이스 설정

| 항목 | 설명 | 기본값 |
|------|------|--------|
| jndi | JNDI 명 | jdbc/malgn |

### 메일 설정

| 항목 | 설명 | 예시 |
|------|------|------|
| mailFrom | 발송자 이메일 | `<webmaster@test.com>` |
| mailHost | 메일 서버 | 127.0.0.1 |

### 보안 설정

| 항목 | 설명 |
|------|------|
| secretId | 내부 암호화 인증키 |

### 전체 예시

```xml
<?xml version="1.0" encoding="UTF-8"?>
<config>
    <env>
        <jndi>jdbc/malgn</jndi>
        <tplRoot>/custom/template</tplRoot>
        <dataDir>/custom/data</dataDir>
        <webUrl>http://www.example.com</webUrl>
        <dataUrl>/data</dataUrl>
        <mailFrom>webmaster@example.com</mailFrom>
        <mailHost>127.0.0.1</mailHost>
        <secretId>your-secret-key</secretId>
        <encoding>UTF-8</encoding>
    </env>
</config>
```

---

## 설치 확인

### 1. init.jsp 작성

Document Root에 `init.jsp` 파일 생성:

```jsp
<%@ page import="java.util.*, java.io.*, malgnsoft.db.*, malgnsoft.util.*" %><%

Malgn m = new Malgn(request, response, out);

Form f = new Form();
f.setRequest(request);

Page p = new Page();
p.setRequest(request);
p.setWriter(out);
p.setPageContext(pageContext);

%>
```

### 2. 테스트 페이지 작성

`/main/test.jsp` 파일 생성:

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

m.p("맑은프레임워크 설치 성공!");

%>
```

### 3. 브라우저에서 확인

```
http://localhost:8080/main/test.jsp
```

"맑은프레임워크 설치 성공!" 메시지가 표시되면 설치 완료입니다.

---

## 문제 해결

### 500 에러 발생

**원인**:
- JDK가 설치되지 않음
- 라이브러리 경로 오류
- config.xml 문법 오류

**해결**:
1. JDK 설치 확인
2. malgn.jar 위치 확인
3. config.xml 문법 검사

### 데이터베이스 연결 실패

**원인**:
- JDBC 드라이버 누락
- 데이터베이스 정보 오류
- JNDI 설정 불일치

**해결**:
1. JDBC 드라이버를 `/WEB-INF/lib`에 복사
2. 데이터베이스 접속 정보 확인
3. config.xml과 resin-web.xml의 JNDI 일치 확인

### 템플릿 파일을 찾을 수 없음

**원인**:
- tplRoot 경로 설정 오류
- 템플릿 파일 경로 오류

**해결**:
1. config.xml의 tplRoot 확인
2. 템플릿 파일 위치 확인
3. 파일명 대소문자 확인

---

## 다음 단계

- [시작하기](getting-started.md)로 이동하여 첫 페이지를 만들어보세요.

---

[← 이전: 프레임워크 소개](introduction.md) | [목차로 돌아가기](README.md) | [다음: 시작하기 →](getting-started.md)
# 3. 시작하기

[← 목차로 돌아가기](README.md)

---

## init.jsp 작성

모든 프로그램에서 공통적으로 사용하는 초기화 파일입니다.

### 파일 위치

```
/{Document Root}/init.jsp
```

### 기본 코드

```jsp
<%@ page import="java.util.*, java.io.*, malgnsoft.db.*, malgnsoft.util.*" %><%

Malgn m = new Malgn(request, response, out);

Form f = new Form();
f.setRequest(request);

Page p = new Page();
p.setRequest(request);
p.setWriter(out);
p.setPageContext(pageContext);

%>
```

### 주의사항

1. **공백 제거**: 상단/하단에 공백이나 개행문자가 없어야 합니다
2. **연결 방식**: `%><%` 형태로 연결하여 공백 방지
3. **다운로드 파일**: 공백이 있으면 다운로드 파일이 깨질 수 있습니다

### 테스트

브라우저에서 접속:
```
http://localhost:8080/init.jsp
```

에러 없이 빈 페이지가 나타나면 성공입니다.

---

## 첫 번째 페이지 작성

### 1. 간단한 출력

**/main/index.jsp**

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

m.p("Hello, World");

%>
```

**접속**:
```
http://localhost:8080/main/index.jsp
```

### 2. 디버깅 메소드 m.p()

`m.p()` 메소드는 다양한 데이터를 출력할 수 있습니다:

```jsp
m.p("문자열");
m.p(123);
m.p(dataSet);
m.p(hashMap);
m.p(arrayList);
```

---

## 템플릿을 이용한 페이지

### 파일 구조

```
/main/index.jsp          - JSP 파일 (Controller)
/html/main/index.html    - HTML 템플릿 (View)
```

### JSP 파일

**/main/index.jsp**

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

p.setBody("main.index");
p.display();

%>
```

### HTML 템플릿

**/html/main/index.html**

```html
<html>
<body>
시작 페이지 입니다.
</body>
</html>
```

### 템플릿 경로 규칙

- `main.index` → `/html/main/index.html`
- 폴더는 `.`(점)으로 구분
- 확장자는 생략 (자동으로 `.html` 추가)

**예시**:
- `main.user` → `/html/main/user.html`
- `admin.member.list` → `/html/admin/member/list.html`

---

## 변수 치환

### 단일 변수

**JSP**:
```jsp
p.setBody("main.index");
p.setVar("title", "Hello World!!");
p.display();
```

**HTML**:
```html
<html>
<body>
<h1>{{title}}</h1>
</body>
</html>
```

### 여러 변수

**JSP**:
```jsp
p.setBody("main.index");
p.setVar("name", "Daniel");
p.setVar("email", "daniel@gmail.com");
p.setVar("age", 24);
p.display();
```

**HTML**:
```html
<html>
<body>
<p>Name : {{name}}</p>
<p>E-mail : {{email}}</p>
<p>Age : {{age}}</p>
</body>
</html>
```

---

## 디버깅

### 템플릿 디버깅

```jsp
p.setDebug(out);  // 브라우저에 템플릿 처리 과정 출력
p.setBody("main.index");
p.display();
```

### 데이터 출력

```jsp
DataSet info = user.find("name = '홍길동'");
m.p(info);  // DataSet 내용 출력
```

---

## 기본 프로젝트 구조

### 추천 구조

```
/
├── init.jsp                    # 공통 초기화 파일
├── assets/                     # 정적 파일
│   ├── css/                   # CSS 파일
│   │   └── style.css
│   ├── js/                    # JavaScript 파일
│   │   └── lib.validate.js
│   └── images/                # 이미지 파일
├── main/                       # 메인 모듈
│   ├── index.jsp              # 메인 페이지
│   ├── user_list.jsp          # 사용자 목록
│   ├── user_insert.jsp        # 사용자 등록
│   └── user_modify.jsp        # 사용자 수정
├── data/                       # 데이터 폴더
│   ├── file/                  # 업로드 파일
│   ├── tmp/                   # 임시 파일
│   └── log/                   # 로그 파일
├── html/                       # 템플릿 루트
│   ├── layout/                # 레이아웃 템플릿
│   │   ├── layout_main.html
│   │   └── layout_admin.html
│   └── main/                  # 모듈 템플릿
│       ├── index.html
│       ├── user_list.html
│       ├── user_insert.html
│       └── user_modify.html
└── WEB-INF/
    ├── lib/
    │   └── malgn.jar
    ├── classes/
    │   └── dao/               # DAO 클래스
    │       └── UserDao.java
    ├── config.xml
    ├── web.xml
    └── resin-web.xml
```

---

## 첫 번째 모듈 만들기

### 1. DAO 클래스 작성

**/WEB-INF/classes/dao/UserDao.java**

```java
package dao;

import malgnsoft.db.*;

public class UserDao extends DataObject {
    public UserDao() {
        this.table = "tb_user";
    }
}
```

### 2. JSP 파일 작성

**/main/user_list.jsp**

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

UserDao user = new UserDao();
DataSet list = user.query("SELECT * FROM tb_user ORDER BY id DESC");

p.setBody("main.user_list");
p.setLoop("list", list);
p.display();

%>
```

### 3. HTML 템플릿 작성

**/html/main/user_list.html**

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>사용자 목록</title>
</head>
<body>
    <h1>사용자 목록</h1>
    <table border="1">
        <thead>
            <tr>
                <th>ID</th>
                <th>이름</th>
                <th>이메일</th>
            </tr>
        </thead>
        <tbody>
            <!--@loop(list)-->
            <tr>
                <td>{{list.id}}</td>
                <td>{{list.name}}</td>
                <td>{{list.email}}</td>
            </tr>
            <!--/loop(list)-->
        </tbody>
    </table>
</body>
</html>
```

---

## 유용한 팁

### 1. 한글 인코딩 문제

JSP 파일 상단에 반드시 charset 지정:
```jsp
<%@ page contentType="text/html; charset=utf-8" %>
```

### 2. 에러 메시지 한글 처리

```jsp
m.jsError("에러 메시지");
m.jsAlert("알림 메시지");
```

### 3. 페이지 이동

```jsp
m.jsReplace("user_list.jsp");  // location.replace
m.jsUrl("user_list.jsp");      // location.href
```

### 4. 이전 페이지로 이동

```jsp
m.jsBack();  // history.back()
```

### 5. POST 방식 체크

```jsp
if(m.isPost()) {
    // POST로 전송된 경우에만 실행
}
```

---

## 자주 발생하는 오류

### 1. 템플릿을 찾을 수 없음

**오류**:
```
Template file not found: main.index
```

**원인**:
- 템플릿 파일이 없음
- 템플릿 경로가 잘못됨

**해결**:
- `/html/main/index.html` 파일 존재 확인
- 파일명 대소문자 확인

### 2. 변수가 치환되지 않음

**오류**: `{{title}}` 그대로 출력됨

**원인**:
- `setVar()`를 호출하지 않음
- 변수명이 틀림

**해결**:
```jsp
p.setVar("title", "값");  // 변수 설정 확인
```

### 3. 공백 문자 오류

**오류**: 다운로드 파일이 깨짐

**원인**:
- init.jsp나 JSP 파일에 공백이 있음

**해결**:
- 모든 JSP 파일에서 불필요한 공백/개행 제거
- `%><%` 형태로 연결

---

## 다음 단계

- [맑은템플릿](template.md)으로 이동하여 템플릿 기능을 자세히 알아보세요.

---

[← 이전: 설치 및 환경설정](installation.md) | [목차로 돌아가기](README.md) | [다음: 맑은템플릿 →](template.md)
# 4. 맑은템플릿

[← 목차로 돌아가기](README.md)

---

## 개요

맑은템플릿은 프로그램 로직과 화면 출력을 분리하기 위한 템플릿 엔진입니다. 웹디자이너나 퍼블리셔도 쉽게 수정할 수 있도록 단순화된 문법을 제공합니다.

---

## 기본 변수 치환

### 단일 변수

**JSP**:
```jsp
p.setBody("main.index");
p.setVar("title", "Hello World!!");
p.display();
```

**HTML**:
```html
<html>
<body>
<h1>{{title}}</h1>
</body>
</html>
```

**결과**:
```html
<html>
<body>
<h1>Hello World!!</h1>
</body>
</html>
```

### 여러 변수

**JSP**:
```jsp
p.setVar("name", "Daniel");
p.setVar("email", "daniel@gmail.com");
p.setVar("age", 24);
```

**HTML**:
```html
<p>Name : {{name}}</p>
<p>E-mail : {{email}}</p>
<p>Age : {{age}}</p>
```

### DataSet 변수

DataSet 객체를 setVar에 넣으면 첫 번째 레코드의 모든 필드가 자동으로 변수화됩니다:

```jsp
DataSet info = user.find("id = 'hong'");
if(info.next()) {
    p.setVar(info);  // 모든 필드를 변수로 설정
}
```

---

## 루프 변수 치환

### 기본 루프

**JSP**:
```jsp
DataSet list = new DataSet();

list.addRow();
list.put("name", "Daniel");
list.put("email", "daniel@gmail.com");

list.addRow();
list.put("name", "David");
list.put("email", "david@gmail.com");

p.setLoop("list", list);
```

**HTML**:
```html
<ul>
 <!--@loop(list)-->
 <li>{{list.name}} ({{list.email}})</li>
 <!--/loop(list)-->
</ul>
```

**결과**:
```html
<ul>
 <li>Daniel (daniel@gmail.com)</li>
 <li>David (david@gmail.com)</li>
</ul>
```

### 루프 주의사항

1. **주석 형식**: `<!--@loop(변수명)-->` ~ `<!--/loop(변수명)-->`
2. **공백 금지**: 주석 안에 공백이 없어야 합니다
3. **변수명 규칙**: 루프 변수명이 앞에 붙습니다 (`{{list.name}}`)
4. **대소문자**: 변수명은 대소문자를 구분합니다

### 중첩 루프

```jsp
DataSet category = ...;  // 카테고리 목록
DataSet product = ...;    // 상품 목록

p.setLoop("category", category);
p.setLoop("product", product);
```

```html
<!--@loop(category)-->
<h2>{{category.name}}</h2>
<ul>
    <!--@loop(product)-->
    <li>{{product.name}}</li>
    <!--/loop(product)-->
</ul>
<!--/loop(category)-->
```

---

## 조건 변수 치환

### if 문

조건이 true이거나 값이 존재하면 출력됩니다:

**JSP**:
```jsp
p.setVar("admin_block", true);
```

**HTML**:
```html
<!--@if(admin_block)-->
<p>관리자 메뉴</p>
<!--/if(admin_block)-->
```

### nif 문

조건이 false이거나 빈값이면 출력됩니다:

**JSP**:
```jsp
p.setVar("admin_block", false);
```

**HTML**:
```html
<!--@nif(admin_block)-->
<p>일반 사용자 메뉴</p>
<!--/nif(admin_block)-->
```

### 조건 예시

```jsp
// 문자열이 있으면 true
p.setVar("message", "Hello");  // true

// 빈 문자열은 false
p.setVar("message", "");  // false

// 숫자 0은 false, 그 외는 true
p.setVar("count", 0);  // false
p.setVar("count", 5);  // true

// boolean
p.setVar("is_admin", true);   // true
p.setVar("is_admin", false);  // false
```

---

## INCLUDE 기능

공통으로 사용되는 HTML을 포함시킵니다.

### 기본 사용법

```html
<html>
<body>
<!--@include(layout/header.html)-->

<div class="content">
    페이지 내용
</div>

<!--@include(layout/footer.html)-->
</body>
</html>
```

### 경로 규칙

- 반드시 `/`로 시작
- 템플릿 루트(`/html`) 기준 상대 경로
- 예: `/layout/header.html` → `/html/layout/header.html`

### 헤더/푸터 예시

**/html/layout/header.html**:
```html
<header>
    <h1>사이트 제목</h1>
    <nav>
        <a href="/">홈</a>
        <a href="/about">소개</a>
    </nav>
</header>
```

**/html/layout/footer.html**:
```html
<footer>
    <p>&copy; 2025 Company Name</p>
</footer>
```

---

## EXECUTE 기능

다른 JSP를 실행하고 결과를 포함시킵니다.

### 기본 사용법

```html
<!--@execute(/main/top.jsp)-->
```

### 파라미터 전달

```html
<!--@execute(/main/top.jsp?menu=3&type=ab&id=3)-->
```

### 활용 예시

메인 페이지에 여러 컴포넌트를 조합:

**/html/main/index.html**:
```html
<html>
<body>
    <!--@include(layout/header.html)-->

    <!--@execute(/main/notice_latest.jsp)-->

    <!--@execute(/main/product_best.jsp?limit=5)-->

    <!--@include(layout/footer.html)-->
</body>
</html>
```

**/main/notice_latest.jsp**:
```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

NoticeDao notice = new NoticeDao();
DataSet list = notice.query("SELECT * FROM tb_notice ORDER BY reg_date DESC", 5);

p.setBody("main.notice_latest");
p.setLoop("notice", list);
p.display();

%>
```

**/html/main/notice_latest.html**:
```html
<div class="notice-latest">
    <h3>최신 공지사항</h3>
    <ul>
    <!--@loop(notice)-->
        <li>{{notice.title}}</li>
    <!--/loop(notice)-->
    </ul>
</div>
```

---

## 레이아웃 기능

레이아웃 HTML과 콘텐츠 HTML을 조합하여 출력합니다.

### 레이아웃 파일 규칙

1. **위치**: `/html/layout/` 폴더
2. **파일명**: `layout_`로 시작, `.html`로 끝남
3. **BODY 태그**: 반드시 `<!--@include(BODY)-->` 포함

### 레이아웃 파일

**/html/layout/layout_main.html**:
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>사이트 제목</title>
    <link rel="stylesheet" href="/assets/css/style.css">
</head>
<body>
    <header>
        <h1>사이트 로고</h1>
        <nav>메뉴</nav>
    </header>

    <main>
        <!--@include(BODY)-->
    </main>

    <footer>
        <p>&copy; 2025 Company</p>
    </footer>
</body>
</html>
```

### JSP에서 레이아웃 사용

```jsp
p.setLayout("main");  // layout_main.html 사용
p.setBody("main.index");
p.display();
```

### 레이아웃 없이 출력

```jsp
p.setLayout(null);  // 레이아웃 사용 안 함
p.setBody("main.index");
p.display();
```

### 여러 레이아웃 사용

```jsp
// 메인 레이아웃
p.setLayout("main");

// 관리자 레이아웃
p.setLayout("admin");

// 팝업 레이아웃
p.setLayout("popup");
```

---

## 고급 기능

### 1. 변수명 규칙

- 영문, 숫자, 언더스코어(_), 한글 사용 가능
- 공백 사용 불가
- 대소문자 구분

```html
{{name}}          <!-- O -->
{{user_name}}     <!-- O -->
{{userName}}      <!-- O -->
{{사용자이름}}     <!-- O -->
{{user name}}     <!-- X 공백 불가 -->
```

### 2. 특수문자 출력

중괄호를 그대로 출력하려면:

```html
<!-- 방법 1: HTML 엔티티 -->
&#123;&#123;변수&#125;&#125;

<!-- 방법 2: 변수를 만들어 출력 -->
{{open_brace}}{{open_brace}}변수{{close_brace}}{{close_brace}}
```

```jsp
p.setVar("open_brace", "{");
p.setVar("close_brace", "}");
```

### 3. HTML 이스케이프

기본적으로 변수는 HTML 이스케이프되지 않습니다:

```jsp
p.setVar("content", "<b>Bold</b>");
```

```html
{{content}}  <!-- <b>Bold</b> 그대로 출력 -->
```

---

## 디버깅

### 템플릿 디버깅

```jsp
p.setDebug(out);  // 브라우저에 출력
p.setBody("main.index");
p.display();
```

**출력 내용**:
- 템플릿 파일 경로
- 변수 목록
- 루프 데이터
- 처리 시간

---

## 베스트 프랙티스

### 1. 레이아웃 활용

공통 레이아웃을 만들어 코드 중복을 줄입니다:

```
/html/layout/
├── layout_main.html       # 메인 레이아웃
├── layout_admin.html      # 관리자 레이아웃
└── layout_popup.html      # 팝업 레이아웃
```

### 2. 공통 요소 분리

자주 사용하는 요소는 별도 파일로:

```
/html/layout/
├── header.html
├── footer.html
├── nav.html
└── sidebar.html
```

### 3. 명확한 변수명

```jsp
// Bad
p.setVar("d", data);
p.setVar("l", list);

// Good
p.setVar("user_info", data);
p.setVar("product_list", list);
```

### 4. 주석 활용

```html
<!-- 사용자 정보 영역 -->
<div class="user-info">
    <p>{{user.name}}</p>
</div>

<!-- 상품 목록 -->
<!--@loop(product)-->
<div class="product">{{product.name}}</div>
<!--/loop(product)-->
```

---

## 다음 단계

- [데이터베이스 연동](database.md)으로 이동하여 DB 처리 방법을 배워보세요.

---

[← 이전: 시작하기](getting-started.md) | [목차로 돌아가기](README.md) | [다음: 데이터베이스 연동 →](database.md)
# 5. 데이터베이스 연동

[← 목차로 돌아가기](README.md)

---

## DataObject 클래스

데이터베이스 연동을 위한 기본 클래스입니다.

### 기본 사용법

```jsp
DataObject dao = new DataObject();
DataSet list = dao.query("SELECT name, email FROM tb_user");

p.setLoop("list", list);
```

### 디버깅

```jsp
dao.setDebug(out);  // 브라우저에 SQL 출력
dao.setDebug();     // 로그파일에 SQL 출력
```

---

## DAO 클래스 작성

### 기본 구조

**/WEB-INF/classes/dao/UserDao.java**:
```java
package dao;

import malgnsoft.db.*;

public class UserDao extends DataObject {
    public UserDao() {
        this.table = "tb_user";
    }
}
```

### 사용

```jsp
UserDao user = new UserDao();
DataSet info = user.find("name = '홍길동'");
```

---

## 데이터 조회 (SELECT)

### query() 메소드

```jsp
// 전체 조회
DataSet list = user.query("SELECT * FROM tb_user");

// 개수 제한
DataSet list = user.query("SELECT * FROM tb_user", 5);

// 조건 조회
DataSet list = user.query("SELECT * FROM tb_user WHERE status = 1");
```

### find() 메소드

```jsp
// 단순 조건
DataSet info = user.find("id = 'hong'");

// 복합 조건
DataSet list = user.find("status = 1 AND type = '02'");

// 정렬
user.setOrderBy("reg_date DESC");
DataSet list = user.find("status = 1");
```

### 단일 레코드 처리

```jsp
DataSet info = user.find("id = 'hong'");
if(!info.next()) {
    m.jsError("회원정보를 찾을 수 없습니다.");
    return;
}

String name = info.s("name");
String email = info.s("email");
```

---

## 데이터 등록 (INSERT)

### item() + insert()

```jsp
UserDao user = new UserDao();
user.item("name", "홍길동");
user.item("email", "hong@gmail.com");
user.item("reg_date", "NOW()", "function");

if(user.insert()) {
    m.jsAlert("등록되었습니다.");
} else {
    m.jsError("등록 실패");
}
```

### execute() 사용

```jsp
user.execute("INSERT INTO tb_user (name, email) VALUES ('홍길동', 'hong@gmail.com')");
```

---

## 데이터 수정 (UPDATE)

### item() + update()

```jsp
UserDao user = new UserDao();
user.item("email", "newemail@gmail.com");
user.item("mod_date", "NOW()", "function");

if(user.update("id = 'hong'")) {
    m.jsAlert("수정되었습니다.");
}
```

### execute() 사용

```jsp
user.execute("UPDATE tb_user SET email = 'newemail@gmail.com' WHERE id = 'hong'");
```

---

## 데이터 삭제 (DELETE)

### delete()

```jsp
UserDao user = new UserDao();
if(user.delete("id = 'hong'")) {
    m.jsAlert("삭제되었습니다.");
}
```

### execute() 사용

```jsp
user.execute("DELETE FROM tb_user WHERE id = 'hong'");
```

---

## 주요 메소드

| 메소드 | 설명 | 반환 |
|--------|------|------|
| query(sql) | SELECT 쿼리 실행 | DataSet |
| query(sql, limit) | 개수 제한 조회 | DataSet |
| execute(sql) | INSERT/UPDATE/DELETE 실행 | int (영향받은 행 수) |
| find(where) | 조건으로 검색 | DataSet |
| item(key, value) | 데이터 항목 설정 | void |
| insert() | 데이터 삽입 | boolean |
| update(where) | 데이터 수정 | boolean |
| delete(where) | 데이터 삭제 | boolean |
| setOrderBy(order) | 정렬 설정 | void |
| setDebug(out) | 디버깅 설정 | void |

---

## 트랜잭션

### 수동 트랜잭션

```jsp
DataObject dao = new DataObject();

try {
    dao.beginTransaction();

    // 작업 1
    dao.execute("INSERT INTO tb_user ...");

    // 작업 2
    dao.execute("UPDATE tb_point ...");

    dao.commit();

} catch(Exception e) {
    dao.rollback();
    m.jsError("처리 중 오류가 발생했습니다.");
}
```

---

[← 이전: 맑은템플릿](template.md) | [목차로 돌아가기](README.md) | [다음: 데이터 입력 및 유효성 체크 →](form-validation.md)
# 5-1. DataObject 클래스

[← 목차로 돌아가기](README.md)

---

## 개요

DataObject는 맑은프레임워크의 핵심 DAO(Data Access Object) 베이스 클래스입니다. 모든 데이터 접근 로직을 단순화하고 표준화하여 데이터베이스 작업을 쉽게 처리할 수 있습니다.

### 주요 특징

- **DAO 패턴 구현**: 테이블별 클래스 생성
- **CRUD 자동화**: insert, update, delete, find 메소드 제공
- **SQL 빌더**: 조건절, 정렬, 조인 등 동적 SQL 생성
- **트랜잭션 관리**: 간편한 트랜잭션 처리
- **PreparedStatement**: SQL Injection 방지
- **다중 DBMS 지원**: MySQL, Oracle, MSSQL, DB2 등

---

## DAO 클래스 작성

### 기본 구조

```java
package com.example.dao;

import malgnsoft.db.*;

public class UserDao extends DataObject {

    public UserDao() {
        this.table = "tb_user";    // 테이블명
        this.PK = "id";            // Primary Key (기본값: "id")
    }
}
```

### WEB-INF/src 폴더 구조

```
/WEB-INF/src/
└── com/example/dao/
    ├── UserDao.java
    ├── BoardDao.java
    └── ProductDao.java
```

---

## 데이터 조회

### find() - 단일 레코드 조회

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

UserDao user = new UserDao();

// Primary Key로 조회
DataSet info = user.get(123);
if(info.next()) {
    m.p("이름: " + info.s("name"));
    m.p("이메일: " + info.s("email"));
}

// 조건절로 조회
DataSet info2 = user.find("email = ?", new Object[] { "hong@example.com" });
if(info2.next()) {
    m.p("사용자 ID: " + info2.i("id"));
}

%>
```

### find() - 여러 레코드 조회

```jsp
UserDao user = new UserDao();

// 전체 조회
DataSet list = user.find("status = 1");
while(list.next()) {
    m.p(list.s("name"));
}

// 정렬 포함
DataSet list2 = user.find("status = 1", "*", "reg_date DESC");

// 개수 제한
DataSet list3 = user.find("status = 1", "*", "reg_date DESC", 10);
```

### find() 파라미터 바인딩

**PreparedStatement 사용으로 SQL Injection 방지**:

```jsp
UserDao user = new UserDao();

// 단일 조건
String email = f.get("email");
DataSet info = user.find("email = ?", new Object[] { email });

// 여러 조건
String name = f.get("name");
int age = f.getInt("age");
DataSet list = user.find(
    "name = ? AND age > ?",
    new Object[] { name, age }
);

// LIKE 검색
String keyword = f.get("keyword");
DataSet list2 = user.find(
    "name LIKE ? OR email LIKE ?",
    new Object[] { "%" + keyword + "%", "%" + keyword + "%" }
);
```

### query() - 직접 SQL 실행

```jsp
UserDao user = new UserDao();

// SQL 직접 작성
DataSet list = user.query(
    "SELECT * FROM tb_user WHERE status = ? ORDER BY id DESC",
    new Object[] { 1 }
);

// 복잡한 쿼리
DataSet list2 = user.query(
    "SELECT u.*, c.name AS city_name " +
    "FROM tb_user u " +
    "LEFT JOIN tb_city c ON u.city_id = c.id " +
    "WHERE u.status = ? AND u.age >= ?",
    new Object[] { 1, 18 }
);

// 개수 제한
DataSet list3 = user.query(
    "SELECT * FROM tb_user ORDER BY id DESC",
    null,
    10  // LIMIT 10
);
```

---

## 데이터 입력

### item() - 데이터 설정

```jsp
UserDao user = new UserDao();

// 필드별 설정
user.item("name", "홍길동");
user.item("email", "hong@example.com");
user.item("age", 25);
user.item("status", 1);

// insert 실행
if(user.insert()) {
    m.p("등록 성공");

    // 자동 생성된 ID 가져오기
    int newId = user.getInsertId();
    m.p("생성된 ID: " + newId);
}
```

### item() - 타입별 메소드

```jsp
UserDao user = new UserDao();

user.item("name", "홍길동");           // String
user.item("age", 25);                  // int
user.item("price", 10000L);            // long
user.item("rate", 4.5);                // double
user.item("score", 98.5f);             // float
user.item("reg_date", new Date());     // Date
```

### item() - Hashtable로 일괄 설정

```jsp
UserDao user = new UserDao();

// Form 데이터를 Hashtable로 받기
Hashtable<String, String> data = new Hashtable<>();
data.put("name", f.get("name"));
data.put("email", f.get("email"));
data.put("age", f.get("age"));

// 일괄 설정
user.item(data);

// 특정 필드 제외
user.item(data, "status,admin_flag");  // status, admin_flag는 제외

if(user.insert()) {
    m.p("등록 성공");
}
```

### insert() - 삽입 후 ID 반환

```jsp
UserDao user = new UserDao();

user.item("name", "홍길동");
user.item("email", "hong@example.com");

// insert 후 자동 생성된 ID 반환
int newId = user.insert(true);

if(newId > 0) {
    m.p("등록 성공, ID: " + newId);
}
```

### replace() - REPLACE INTO

```jsp
UserDao user = new UserDao();

user.item("id", 123);
user.item("name", "홍길동");
user.item("email", "new@example.com");

// 존재하면 UPDATE, 없으면 INSERT
if(user.replace()) {
    m.p("처리 완료");
}
```

---

## 데이터 수정

### update() - Primary Key로 수정

```jsp
UserDao user = new UserDao();

// 수정할 레코드 조회
DataSet info = user.get(123);
if(!info.next()) {
    m.jsError("사용자를 찾을 수 없습니다.");
    return;
}

// 데이터 설정
user.item("name", "김철수");
user.item("email", "kim@example.com");
user.item("age", 30);

// update 실행 (id = 123인 레코드 수정)
user.id = "123";
if(user.update()) {
    m.jsAlert("수정되었습니다.");
    m.jsReplace("list.jsp");
}
```

### update() - 조건절로 수정

```jsp
UserDao user = new UserDao();

user.item("status", 0);
user.item("block_date", new Date());

// 특정 조건의 레코드들 수정
if(user.update("age < ?", new Object[] { 18 })) {
    m.p("미성년자 계정 차단 완료");
}
```

### update() - Hashtable로 수정

```jsp
UserDao user = new UserDao();

Hashtable<String, String> data = new Hashtable<>();
data.put("id", "123");  // Primary Key
data.put("name", f.get("name"));
data.put("email", f.get("email"));

// Hashtable에서 id를 추출하여 해당 레코드 수정
if(user.update(data)) {
    m.p("수정 완료");
}

// 조건절 지정
if(user.update(data, "id = ? AND user_type = ?", new Object[] { 123, "admin" })) {
    m.p("관리자 정보 수정 완료");
}
```

---

## 데이터 삭제

### delete() - Primary Key로 삭제

```jsp
UserDao user = new UserDao();

// id 지정 후 삭제
user.id = "123";
if(user.delete()) {
    m.p("삭제 완료");
}

// 직접 id 전달
if(user.delete(123)) {
    m.p("삭제 완료");
}
```

### delete() - 조건절로 삭제

```jsp
UserDao user = new UserDao();

// 특정 조건 삭제
if(user.delete("status = 0 AND login_date < DATE_SUB(NOW(), INTERVAL 1 YEAR)")) {
    m.p("1년 이상 미접속 차단 계정 삭제 완료");
}

// 파라미터 바인딩
String email = f.get("email");
if(user.delete("email = ?", new Object[] { email })) {
    m.p("삭제 완료");
}
```

---

## 동적 SQL 빌더

### addWhere() - 고정 조건 추가

```jsp
UserDao user = new UserDao();

// 기본 조건 설정
user.addWhere("status = 1");
user.addWhere("type = ?", new Object[] { "premium" });

// find() 실행 시 자동으로 WHERE 절 생성
DataSet list = user.find();
// SQL: SELECT * FROM tb_user WHERE status = 1 AND type = 'premium'
```

### addSearch() - 동적 검색 조건

**값이 있을 때만 조건 추가**:

```jsp
UserDao user = new UserDao();

// 검색어가 있을 때만 조건 추가
String keyword = f.get("keyword");
user.addSearch("name,email", keyword, "LIKE");

int status = f.getInt("status", -1);
if(status >= 0) {
    user.addSearch("status", status);
}

int minAge = f.getInt("min_age", 0);
if(minAge > 0) {
    user.addSearch("age", minAge, ">=");
}

user.setOrderBy("id DESC");
DataSet list = user.find();
```

### addSearch() 연산자

```jsp
UserDao user = new UserDao();

// 같음 (기본)
user.addSearch("status", 1);
// WHERE status = 1

// LIKE 검색
user.addSearch("name", "홍길동", "LIKE");
// WHERE name LIKE '%홍길동%'

user.addSearch("name", "홍", "LIKE%");
// WHERE name LIKE '홍%'

user.addSearch("name", "동", "%LIKE");
// WHERE name LIKE '%동'

// 여러 필드 OR 검색
user.addSearch("name,email,phone", "홍길동", "LIKE");
// WHERE (name LIKE '%홍길동%' OR email LIKE '%홍길동%' OR phone LIKE '%홍길동%')

// 비교 연산자
user.addSearch("age", 18, ">=");
// WHERE age >= 18

user.addSearch("price", 10000, "<");
// WHERE price < 10000
```

### setOrderBy() - 정렬

```jsp
UserDao user = new UserDao();

// 내림차순
user.setOrderBy("id DESC");

// 오름차순
user.setOrderBy("name ASC");

// 여러 필드 정렬
user.setOrderBy("status DESC, reg_date DESC, name ASC");

DataSet list = user.find();
```

### setLimit() - 개수 제한

```jsp
UserDao user = new UserDao();

user.setLimit(10);  // 최대 10개
DataSet list = user.find();
```

---

## 조인 쿼리

### addJoin() - INNER JOIN

```jsp
UserDao user = new UserDao();

user.setTable("tb_user u");
user.addJoin("tb_city c", "INNER", "u.city_id = c.id");
user.setFields("u.*, c.name AS city_name");
user.addWhere("u.status = 1");

DataSet list = user.find();
while(list.next()) {
    m.p(list.s("name") + " - " + list.s("city_name"));
}
```

### addLeftJoin() - LEFT JOIN

```jsp
UserDao user = new UserDao();

user.setTable("tb_user u");
user.addLeftJoin("tb_profile p ON u.id = p.user_id");
user.addLeftJoin("tb_city c ON u.city_id = c.id");
user.setFields("u.*, p.bio, c.name AS city_name");

DataSet list = user.find();
```

### 복잡한 조인

```jsp
UserDao user = new UserDao();

user.setTable("tb_order o");
user.addJoin("tb_user u", "INNER", "o.user_id = u.id");
user.addJoin("tb_product p", "INNER", "o.product_id = p.id");
user.addLeftJoin("tb_coupon c ON o.coupon_id = c.id");

user.setFields("o.*, u.name AS user_name, p.name AS product_name, c.discount");
user.addWhere("o.status = ?", new Object[] { "completed" });
user.setOrderBy("o.order_date DESC");

DataSet list = user.find();
```

---

## 집계 함수

### findCount() - 개수 조회

```jsp
UserDao user = new UserDao();

// 전체 개수
int total = user.findCount("status = 1");
m.p("활성 사용자 수: " + total);

// 파라미터 바인딩
int count = user.findCount(
    "age >= ? AND status = ?",
    new Object[] { 18, 1 }
);
m.p("성인 활성 사용자 수: " + count);
```

### getOne() - 단일 값 조회

```jsp
UserDao user = new UserDao();

// 단일 값 가져오기
String email = user.getOne(
    "SELECT email FROM tb_user WHERE id = ?",
    new Object[] { 123 }
);
m.p("이메일: " + email);

// 최대값
String maxAge = user.getOne("SELECT MAX(age) FROM tb_user");
m.p("최고령: " + maxAge);
```

### getOneInt() - 정수 값 조회

```jsp
UserDao user = new UserDao();

// COUNT
int total = user.getOneInt("SELECT COUNT(*) FROM tb_user WHERE status = 1");
m.p("전체: " + total);

// SUM
int totalPoint = user.getOneInt(
    "SELECT SUM(point) FROM tb_user WHERE user_id = ?",
    new Object[] { 123 }
);
m.p("총 포인트: " + totalPoint);
```

---

## 트랜잭션

### 단일 DAO 트랜잭션

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

UserDao user = new UserDao();

try {
    user.startTrans();  // 트랜잭션 시작

    // 여러 작업 수행
    user.item("name", "홍길동");
    user.item("email", "hong@example.com");
    user.insert();

    user.item("status", 0);
    user.update("age < 18");

    if(user.endTrans()) {  // 커밋
        m.p("트랜잭션 완료");
    } else {
        m.p("트랜잭션 실패");
    }
} catch(Exception e) {
    m.p("오류 발생: " + e.getMessage());
}

%>
```

### 여러 DAO 트랜잭션

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

UserDao user = new UserDao();
OrderDao order = new OrderDao();
PointDao point = new PointDao();

try {
    // 여러 DAO를 하나의 트랜잭션으로 묶기
    user.startTransWith(order, point);

    // 사용자 등록
    user.item("name", "홍길동");
    user.item("email", "hong@example.com");
    user.insert();
    int userId = user.getInsertId();

    // 주문 등록
    order.item("user_id", userId);
    order.item("product_id", 100);
    order.item("price", 50000);
    order.insert();

    // 포인트 지급
    point.item("user_id", userId);
    point.item("point", 5000);
    point.item("memo", "가입 축하 포인트");
    point.insert();

    if(user.endTrans()) {
        m.jsAlert("회원가입이 완료되었습니다.");
        m.jsReplace("login.jsp");
    }
} catch(Exception e) {
    m.jsError("오류가 발생했습니다: " + e.getMessage());
}
return;

%>
```

---

## 시퀀스 관리

### getSequence() - 시퀀스 생성

**중복 없는 순차 번호 생성**:

```jsp
UserDao user = new UserDao();

// 다음 시퀀스 번호 가져오기
int seq = user.getSequence("tb_user");
m.p("시퀀스: " + seq);  // 1, 2, 3, ...

// 접두사와 자릿수 지정
String orderNo = user.getSequence("ORDER", 8);
m.p("주문번호: " + orderNo);  // ORDER00000001, ORDER00000002, ...
```

### getNextId() - 타임스탬프 기반 ID

```jsp
UserDao user = new UserDao();

// 밀리초 타임스탬프 기반 고유 ID
long id = user.getNextId();
m.p("ID: " + id);  // 1729843200123456

// 접두사 추가
String orderId = user.getNextId("ORD");
m.p("주문 ID: " + orderId);  // ORD1729843200123456
```

---

## SQL 함수 활용

### item() - SQL 함수 사용

```jsp
UserDao user = new UserDao();

// 현재 시각
user.item("reg_date", "", "NOW()");

// 수식 계산
user.item("total_price", "", "price * quantity");

// 조건부 값
user.item("status", "", "IF(age >= 18, 1, 0)");

if(user.insert()) {
    m.p("등록 완료");
}
```

### UPDATE에서 SQL 함수

```jsp
UserDao user = new UserDao();

// 조회수 증가
user.item("view_count", "", "view_count + 1");
user.update("id = ?", new Object[] { 123 });

// 포인트 차감
user.item("point", "", "point - 1000");
user.update("id = ?", new Object[] { userId });
```

---

## 고급 기능

### setInsertIgnore() - 중복 무시

```jsp
UserDao user = new UserDao();

user.setInsertIgnore(true);  // INSERT IGNORE 사용
user.item("email", "hong@example.com");
user.item("name", "홍길동");

// 중복되면 무시, 에러 없음
user.insert();
```

### setFields() - 조회 필드 지정

```jsp
UserDao user = new UserDao();

// 특정 필드만 조회
user.setFields("id, name, email");
DataSet list = user.find("status = 1");

// 집계 함수
user.setFields("COUNT(*) AS cnt, AVG(age) AS avg_age");
DataSet info = user.find();
```

### 디버그 모드

```jsp
UserDao user = new UserDao();

user.setDebug(out);  // SQL 쿼리를 브라우저에 출력

user.addWhere("status = 1");
DataSet list = user.find();

// 출력:
// SELECT * FROM tb_user WHERE status = 1
// Execution Time : 15 (1/1000 sec)
```

### getQuery() - 생성된 SQL 확인

```jsp
UserDao user = new UserDao();

user.item("name", "홍길동");
user.item("email", "hong@example.com");

String sql = user.getInsertQuery();
m.p("SQL: " + sql);
// INSERT INTO tb_user (name,email) VALUES (?,?)

user.insert();
```

---

## 실전 예제

### 게시판 목록 (검색 + 페이징)

**JSP**:
```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

BoardDao board = new BoardDao();

// 검색 조건
board.addWhere("status = 1");
board.addSearch("subject,content", f.get("keyword"), "LIKE");
board.addSearch("category", f.get("category"));

// 정렬
board.setOrderBy("id DESC");

// 목록 조회
board.setLimit(20);
DataSet list = board.find();

// 전체 개수
int total = board.findCount(board.where, board.params.toArray());

p.setBody("board.list");
p.setLoop("list", list);
p.setVar("total", total);
p.display();

%>
```

### 회원가입 (트랜잭션)

**JSP**:
```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

f.addElement("email", null, "required:'Y', type:'email'");
f.addElement("passwd", null, "required:'Y', minlength:4");
f.addElement("name", null, "required:'Y'");

if(m.isPost() && f.validate()) {

    UserDao user = new UserDao();
    PointDao point = new PointDao();

    try {
        user.startTransWith(point);

        // 이메일 중복 체크
        int exists = user.findCount("email = ?", new Object[] { f.get("email") });
        if(exists > 0) {
            m.jsError("이미 등록된 이메일입니다.");
            return;
        }

        // 사용자 등록
        user.item("email", f.get("email"));
        user.item("passwd", Malgn.sha256(f.get("passwd")));
        user.item("name", f.get("name"));
        user.item("status", 1);
        user.insert();

        int userId = user.getInsertId();

        // 가입 축하 포인트 지급
        point.item("user_id", userId);
        point.item("point", 5000);
        point.item("memo", "가입 축하 포인트");
        point.insert();

        if(user.endTrans()) {
            m.jsAlert("회원가입이 완료되었습니다.");
            m.jsReplace("login.jsp");
        } else {
            m.jsError("회원가입 중 오류가 발생했습니다.");
        }

    } catch(Exception e) {
        Malgn.errorLog("회원가입 오류", e);
        m.jsError("오류가 발생했습니다.");
    }
    return;
}

p.setBody("user.join");
p.setVar("form_script", f.getScript());
p.display();

%>
```

### 주문 처리 (복잡한 트랜잭션)

**JSP**:
```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

if(m.isPost()) {

    OrderDao order = new OrderDao();
    ProductDao product = new ProductDao();
    UserDao user = new UserDao();

    int productId = f.getInt("product_id");
    int userId = auth.i("id");
    int quantity = f.getInt("quantity", 1);

    try {
        order.startTransWith(product, user);

        // 상품 정보 조회
        DataSet productInfo = product.get(productId);
        if(!productInfo.next()) {
            m.jsError("상품을 찾을 수 없습니다.");
            return;
        }

        // 재고 확인
        int stock = productInfo.i("stock");
        if(stock < quantity) {
            m.jsError("재고가 부족합니다.");
            return;
        }

        // 주문 생성
        order.item("user_id", userId);
        order.item("product_id", productId);
        order.item("quantity", quantity);
        order.item("price", productInfo.i("price"));
        order.item("total", productInfo.i("price") * quantity);
        order.item("status", "pending");
        order.insert();

        int orderId = order.getInsertId();

        // 재고 차감
        product.item("stock", "", "stock - " + quantity);
        product.update("id = ?", new Object[] { productId });

        // 사용자 포인트 차감 (포인트 결제 시)
        int usePoint = f.getInt("use_point", 0);
        if(usePoint > 0) {
            user.item("point", "", "point - " + usePoint);
            user.update("id = ?", new Object[] { userId });
        }

        if(order.endTrans()) {
            m.jsAlert("주문이 완료되었습니다.");
            m.jsReplace("order_view.jsp?id=" + orderId);
        } else {
            m.jsError("주문 처리 중 오류가 발생했습니다.");
        }

    } catch(Exception e) {
        Malgn.errorLog("주문 처리 오류", e);
        m.jsError("오류가 발생했습니다.");
    }
    return;
}

%>
```

---

## 메소드 레퍼런스

### 조회 메소드

| 메소드 | 설명 |
|--------|------|
| `get(int id)` | Primary Key로 단일 레코드 조회 |
| `find(String where)` | 조건절로 레코드 조회 |
| `find(String where, Object[] args)` | 파라미터 바인딩 조회 |
| `find()` | 설정된 조건으로 조회 (addWhere 등) |
| `query(String sql)` | 직접 SQL 실행 |
| `query(String sql, Object[] args)` | 파라미터 바인딩 SQL 실행 |
| `findCount(String where)` | 조건에 맞는 레코드 개수 |
| `getOne(String sql)` | 단일 값 조회 (String) |
| `getOneInt(String sql)` | 단일 값 조회 (int) |

### 입력/수정/삭제 메소드

| 메소드 | 설명 |
|--------|------|
| `item(String name, Object value)` | 필드 값 설정 |
| `item(Hashtable data)` | Hashtable로 일괄 설정 |
| `insert()` | 레코드 삽입 |
| `insert(true)` | 삽입 후 ID 반환 |
| `update()` | Primary Key로 수정 |
| `update(String where)` | 조건절로 수정 |
| `delete()` | Primary Key로 삭제 |
| `delete(String where)` | 조건절로 삭제 |
| `replace()` | REPLACE INTO 실행 |

### SQL 빌더 메소드

| 메소드 | 설명 |
|--------|------|
| `addWhere(String where)` | 고정 조건 추가 |
| `addSearch(String field, String value, String oper)` | 동적 검색 조건 |
| `setOrderBy(String sort)` | 정렬 설정 |
| `setLimit(int limit)` | 개수 제한 |
| `setFields(String fields)` | 조회 필드 지정 |
| `addJoin(String cond)` | JOIN 추가 |
| `addLeftJoin(String cond)` | LEFT JOIN 추가 |

### 트랜잭션 메소드

| 메소드 | 설명 |
|--------|------|
| `startTrans()` | 트랜잭션 시작 |
| `startTransWith(dao1, dao2, ...)` | 여러 DAO 트랜잭션 묶기 |
| `endTrans()` | 트랜잭션 커밋 |

### 유틸리티 메소드

| 메소드 | 설명 |
|--------|------|
| `getInsertId()` | 마지막 INSERT ID |
| `getSequence()` | 시퀀스 번호 생성 |
| `getNextId()` | 타임스탬프 기반 ID |
| `setDebug(out)` | 디버그 모드 |
| `getQuery()` | 생성된 SQL 반환 |
| `clear()` | item 데이터 초기화 |

---

## 주의사항

### 1. 트랜잭션 후 반드시 return

```jsp
if(m.isPost()) {
    user.startTrans();
    user.insert();
    user.endTrans();
    return;  // 필수!
}
```

### 2. SQL Injection 방지

```jsp
// 나쁜 예 - SQL Injection 위험
String email = f.get("email");
user.find("email = '" + email + "'");

// 좋은 예 - PreparedStatement 사용
user.find("email = ?", new Object[] { email });
```

### 3. item() 데이터는 insert/update 후 유지됨

```jsp
user.item("name", "홍길동");
user.insert();

user.item("email", "new@example.com");
user.insert();  // name="홍길동", email="new@example.com" 둘 다 삽입됨

// 초기화 필요
user.clear();
user.item("name", "김철수");
user.insert();  // name="김철수"만 삽입됨
```

---

## 다음 단계

- [데이터베이스 연동](database.md)에서 기본 개념 확인
- [목록 및 검색](list-search.md)에서 ListManager와 함께 사용하기
- [DataSet 활용](dataset.md)에서 조회 결과 처리 방법 배우기

---

[← 이전: 데이터베이스 연동](database.md) | [목차로 돌아가기](README.md) | [다음: 데이터 입력 및 유효성 체크 →](form-validation.md)
# 6. 데이터 입력 및 유효성 체크

[← 목차로 돌아가기](README.md)

---

## Form 클래스 소개

Form 클래스는 웹에서 입력된 데이터를 받고 유효성 검증을 수행합니다.

### 주요 기능

- 폼 데이터 수신 및 처리
- 서버/클라이언트 양측 유효성 검증
- 파일 업로드 처리
- 데이터 타입 변환

---

## 기본 사용법

### 1. 간단한 폼 처리

**JSP**:
```jsp
f.addElement("name", null, "required:'Y'");
f.addElement("email", null, "required:'Y', type:'email'");

if(m.isPost() && f.validate()) {
    String name = f.get("name");
    String email = f.get("email");

    // 데이터 처리
    m.jsAlert("등록되었습니다.");
    return;
}
```

**HTML**:
```html
<form name="form1" method="POST">
    <input type="text" name="name">
    <input type="text" name="email">
    <input type="submit" value="전송">
</form>
<%= f.getScript() %>
```

### 2. addElement() 파라미터

```jsp
f.addElement("항목명", "기본값", "속성");
```

- **항목명**: 폼 필드 이름 (name 속성)
- **기본값**: 초기값 (없으면 null)
- **속성**: 검증 규칙 (JavaScript 객체 형식)

---

## 유효성 검증 속성

### 필수 입력

```jsp
f.addElement("name", null, "required:'Y'");
f.addElement("email", null, "required:true");
```

### 데이터 타입

```jsp
// 이메일
f.addElement("email", null, "type:'email', required:'Y'");

// 숫자
f.addElement("age", null, "type:'number', required:'Y'");

// URL
f.addElement("website", null, "type:'url'");

// 전화번호
f.addElement("phone", null, "type:'phone'");

// 한글만
f.addElement("name", null, "type:'hangul'");

// 영문만
f.addElement("userid", null, "type:'engonly'");

// 주민번호
f.addElement("jumin", null, "type:'jumin'");

// 사업자번호
f.addElement("bizno", null, "type:'bizno'");
```

### 길이 제한

```jsp
// 최소 길이
f.addElement("name", null, "minlength:2");

// 최대 길이
f.addElement("name", null, "maxlength:20");

// 최소/최대 길이
f.addElement("name", null, "minlength:2, maxlength:20");
```

### 숫자 범위

```jsp
// 최소값
f.addElement("age", null, "type:'number', min:1");

// 최대값
f.addElement("age", null, "type:'number', max:150");

// 범위
f.addElement("age", null, "type:'number', min:1, max:150");
```

### 패턴 매칭

```jsp
// 정규표현식
f.addElement("code", null, "pattern:'^[A-Z]{2}[0-9]{4}$'");
```

### 일치 확인

```jsp
// 비밀번호 확인
f.addElement("passwd", null, "required:'Y'");
f.addElement("passwd_confirm", null, "match:'passwd', required:'Y'");
```

### HTML 태그 허용

```jsp
// HTML 태그 허용
f.addElement("content", null, "allowhtml:'Y'");

// HTML 태그 제거
f.addElement("content", null, "allowhtml:'N'");  // 기본값
```

### 한글명 지정

```jsp
f.addElement("name", null, "title:'이름', required:'Y'");
```

---

## 전체 예제

### 회원가입 폼

**JSP** (`/main/user_join.jsp`):
```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

f.addElement("id", null, "title:'아이디', required:true, minlength:4, maxlength:12");
f.addElement("passwd", null, "title:'비밀번호', required:true, minlength:4");
f.addElement("passwd_confirm", null, "title:'비밀번호 확인', match:'passwd', required:true");
f.addElement("name", null, "title:'이름', required:true, maxlength:20");
f.addElement("email", null, "title:'이메일', type:'email', required:true");
f.addElement("phone", null, "title:'전화번호', type:'phone', required:true");
f.addElement("age", null, "title:'나이', type:'number', min:1, max:150");
f.addElement("url", null, "title:'홈페이지', type:'url'");
f.addElement("gender", "M", "title:'성별', required:true");
f.addElement("agree", null, "title:'약관동의', required:true");

if(m.isPost() && f.validate()) {

    UserDao user = new UserDao();
    user.item("id", f.get("id"));
    user.item("passwd", f.get("passwd"));
    user.item("name", f.get("name"));
    user.item("email", f.get("email"));
    user.item("phone", f.get("phone"));
    user.item("age", f.getInt("age"));
    user.item("url", f.get("url"));
    user.item("gender", f.get("gender"));

    if(user.insert()) {
        m.jsAlert("회원가입이 완료되었습니다.");
        m.jsReplace("login.jsp");
    } else {
        m.jsError("회원가입 중 오류가 발생했습니다.");
    }
    return;
}

p.setBody("main.user_join");
p.setVar("form_script", f.getScript());
p.display();

%>
```

**HTML** (`/html/main/user_join.html`):
```html
<form name="form1" method="post">
    <p>아이디 : <input type="text" name="id" placeholder="4-12자"></p>
    <p>비밀번호 : <input type="password" name="passwd" placeholder="4자 이상"></p>
    <p>비밀번호 확인 : <input type="password" name="passwd_confirm"></p>
    <p>이름 : <input type="text" name="name"></p>
    <p>이메일 : <input type="email" name="email"></p>
    <p>전화번호 : <input type="text" name="phone"></p>
    <p>나이 : <input type="number" name="age"></p>
    <p>홈페이지 : <input type="url" name="url"></p>
    <p>성별 :
        <select name="gender">
            <option value="M">남성</option>
            <option value="F">여성</option>
        </select>
    </p>
    <p><label><input type="checkbox" name="agree" value="Y"> 약관에 동의합니다</label></p>
    <p><input type="submit" value="회원가입"></p>
</form>
{{form_script}}
```

---

## 데이터 가져오기

### get() 메소드

```jsp
// 문자열
String name = f.get("name");

// 기본값 지정
String name = f.get("name", "기본값");
```

### 타입별 메소드

```jsp
// 정수
int age = f.getInt("age");
int age = f.getInt("age", 0);  // 기본값

// Long
long id = f.getLong("id");

// Double
double price = f.getDouble("price");

// Boolean
boolean agree = f.getBoolean("agree");
```

### 배열 데이터

```jsp
// 체크박스 등 여러 값
String[] hobbies = f.getArray("hobby");
```

---

## 포스트백 방식

등록/수정 폼과 처리를 하나의 JSP에서 처리합니다.

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

f.addElement("name", null, "required:'Y'");
f.addElement("email", null, "required:'Y', type:'email'");

if(m.isPost() && f.validate()) {
    // POST로 데이터가 전송된 경우
    UserDao user = new UserDao();
    user.item("name", f.get("name"));
    user.item("email", f.get("email"));

    if(user.insert()) {
        m.jsAlert("등록되었습니다.");
        m.jsReplace("list.jsp");
    } else {
        m.jsError("등록 실패");
    }
    return;
}

// GET 요청인 경우 폼 출력
p.setBody("main.user_form");
p.setVar("form_script", f.getScript());
p.display();

%>
```

---

## 수정 폼에 기존 데이터 입력

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

String id = f.get("id");

UserDao user = new UserDao();
DataSet info = user.find("id = ?", new Object[] { id });

if(!info.next()) {
    m.jsError("사용자를 찾을 수 없습니다.");
    return;
}

// 기존 값을 기본값으로 설정
f.addElement("name", info.s("name"), "required:'Y'");
f.addElement("email", info.s("email"), "required:'Y', type:'email'");
f.addElement("age", info.i("age"), "type:'number'");

if(m.isPost() && f.validate()) {
    user.item("name", f.get("name"));
    user.item("email", f.get("email"));
    user.item("age", f.getInt("age"));

    if(user.update("id = ?", new Object[] { id })) {
        m.jsAlert("수정되었습니다.");
        m.jsReplace("view.jsp?id=" + id);
    }
    return;
}

p.setBody("main.user_form");
p.setVar(info);  // 템플릿에 데이터 전달
p.setVar("form_script", f.getScript());
p.display();

%>
```

---

## 파일 업로드

> 파일 업로드 및 다운로드에 대한 상세한 내용은 [파일 업로드 및 다운로드](file-upload-download.md) 문서를 참조하세요.

### 단일 파일

**JSP** (`/main/file_upload.jsp`):
```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

f.addElement("name", null, "required:'Y'");
f.addElement("file", null, "required:'Y'");

if(m.isPost() && f.validate()) {

    File file = f.saveFile("file");
    if(file != null) {
        String fileName = f.getFileName("file");
        String filePath = file.getAbsolutePath();

        m.p("파일명: " + fileName);
        m.p("저장경로: " + filePath);

        // DB에 파일 정보 저장
        // ...
    }
    return;
}

p.setBody("main.file_upload");
p.setVar("form_script", f.getScript());
p.display();

%>
```

**HTML** (`/html/main/file_upload.html`):
```html
<form name="form1" method="post" enctype="multipart/form-data">
    <p>이름 : <input type="text" name="name"></p>
    <p>파일 : <input type="file" name="file"></p>
    <p><input type="submit" value="업로드"></p>
</form>
{{form_script}}
```

### 파일 확장자 제한

```jsp
// 허용 확장자
f.addElement("file", null, "allow:'jpg|png|gif'");

// 금지 확장자
f.addElement("file", null, "deny:'exe|jsp|php'");
```

### 여러 파일 업로드

```jsp
File[] files = f.saveFiles("files");
for(File file : files) {
    m.p(file.getName());
}
```

---

## 데이터 연결 (glue)

여러 필드를 하나로 연결합니다.

```jsp
// 전화번호 3개 필드를 하나로
f.addElement("phone1", "010", "glue:'phone2|phone3', delim:'-', required:'Y'");

// 사용
String phone = f.glue("-", "phone1", "phone2", "phone3");
// 결과: 010-1234-5678
```

---

## 유효성 검증 속성 전체 목록

| 속성 | 설명 | 예시 |
|------|------|------|
| title | 필드명 | `title:'이름'` |
| required | 필수 입력 | `required:'Y'` |
| type | 데이터 타입 | `type:'email'` |
| minlength | 최소 길이 | `minlength:4` |
| maxlength | 최대 길이 | `maxlength:20` |
| min | 최소값 | `min:1` |
| max | 최대값 | `max:100` |
| pattern | 정규표현식 | `pattern:'^[A-Z]+'` |
| match | 일치 확인 | `match:'passwd'` |
| allow | 허용 확장자 | `allow:'jpg\|png'` |
| deny | 금지 확장자 | `deny:'exe\|jsp'` |
| allowhtml | HTML 허용 | `allowhtml:'Y'` |
| glue | 필드 연결 | `glue:'field2\|field3'` |
| delim | 구분자 | `delim:'-'` |

---

## 클라이언트 검증

`f.getScript()`를 HTML에 포함하면 자동으로 클라이언트 검증 스크립트가 생성됩니다.

```html
<form name="form1" method="post">
    <!-- 폼 필드들 -->
</form>
<%= f.getScript() %>
```

**주의사항**:
- 폼 이름은 반드시 `form1`이어야 합니다
- `lib.validate.js` 파일이 필요합니다

---

## 베스트 프랙티스

### 1. 명확한 에러 메시지

```jsp
f.addElement("email", null, "title:'이메일 주소', type:'email', required:'Y'");
```

### 2. 적절한 타입 지정

```jsp
f.addElement("age", null, "type:'number', min:1, max:150");
```

### 3. 보안을 위한 HTML 필터링

```jsp
// 일반 텍스트는 HTML 제거 (기본)
f.addElement("name", null, "required:'Y'");

// 에디터 내용은 HTML 허용
f.addElement("content", null, "allowhtml:'Y'");
```

### 4. 서버 검증 필수

클라이언트 검증만으로는 부족합니다. 반드시 서버에서도 검증하세요:

```jsp
if(m.isPost() && f.validate()) {
    // 서버 검증 통과
    // 추가 비즈니스 로직 검증
    if(!isValidBusinessLogic()) {
        m.jsError("잘못된 요청입니다.");
        return;
    }
}
```

---

## 다음 단계

- [파일 업로드 및 다운로드](file-upload-download.md)에서 파일 처리에 대해 더 자세히 알아보세요.
- [목록 및 검색](list-search.md)으로 이동하여 데이터 목록 처리를 배워보세요.

---

[← 이전: 데이터베이스 연동](database.md) | [목차로 돌아가기](README.md) | [다음: 파일 업로드 및 다운로드 →](file-upload-download.md)
# 파일 업로드 및 다운로드

[← 목차로 돌아가기](README.md)

---

## 개요

맑은프레임워크는 Form 클래스와 Malgn 클래스를 통해 간편한 파일 업로드 및 다운로드 기능을 제공합니다.

### 주요 기능

- **파일 업로드**: Form 클래스를 통한 멀티파트 파일 업로드
- **파일 다운로드**: Malgn 클래스를 통한 파일 전송
- **확장자 검증**: 허용/금지 확장자 제한
- **여러 파일 처리**: 다중 파일 업로드 지원
- **대역폭 제어**: 다운로드 속도 제한
- **이미지 썸네일**: 자동 썸네일 생성

---

## 목차

1. [파일 업로드](#파일-업로드)
2. [파일 다운로드](#파일-다운로드)
3. [이미지 썸네일](#이미지-썸네일)
4. [파일 관리](#파일-관리)
5. [보안 가이드](#보안-가이드)

---

## 파일 업로드

### 1. 기본 업로드

**JSP**:
```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

f.addElement("title", null, "required:'Y'");
f.addElement("file", null, "required:'Y'");

if(m.isPost() && f.validate()) {

    // 파일 저장
    File file = f.saveFile("file");
    if(file != null) {
        String fileName = f.getFileName("file");
        String filePath = file.getAbsolutePath();
        long fileSize = file.length();

        m.p("파일명: " + fileName);
        m.p("저장경로: " + filePath);
        m.p("파일크기: " + fileSize);

        // DB에 파일 정보 저장
        FileDao dao = new FileDao();
        DataMap data = new DataMap();
        data.put("title", f.get("title"));
        data.put("file_name", fileName);
        data.put("file_path", filePath);
        data.put("file_size", fileSize);
        data.put("reg_date", m.time());
        dao.insert(data);

        m.jsAlert("업로드 완료");
        m.jsReplace("list.jsp");
    } else {
        m.jsError("파일 업로드 실패");
    }
    return;
}

%>
<form name="form1" method="post" enctype="multipart/form-data">
    <p>제목 : <input type="text" name="title"></p>
    <p>파일 : <input type="file" name="file"></p>
    <p><input type="submit" value="업로드"></p>
</form>
<%= f.getScript() %>
```

**HTML 폼 주의사항**:
- `method="post"` 필수
- `enctype="multipart/form-data"` 필수

### 2. 확장자 제한

#### 허용 확장자 지정

```jsp
// 이미지만 허용
f.addElement("photo", null, "allow:'jpg|jpeg|png|gif'");

// 문서 파일만 허용
f.addElement("document", null, "allow:'pdf|doc|docx|xls|xlsx'");
```

#### 금지 확장자 지정

```jsp
// 실행 파일 업로드 금지
f.addElement("file", null, "deny:'exe|bat|cmd|sh'");

// 서버 스크립트 금지
f.addElement("file", null, "deny:'jsp|php|asp|aspx'");
```

#### 조합 사용

```jsp
// 이미지만 허용하되, bmp는 제외
f.addElement("image", null, "allow:'jpg|jpeg|png|gif|bmp', deny:'bmp'");
```

### 3. 파일 크기 제한

```jsp
// web.xml에서 설정
<multipart-config>
    <max-file-size>10485760</max-file-size>       <!-- 10MB -->
    <max-request-size>52428800</max-request-size> <!-- 50MB -->
</multipart-config>
```

또는 코드에서 검증:

```jsp
if(m.isPost() && f.validate()) {
    File file = f.saveFile("file");
    if(file != null) {
        long fileSize = file.length();
        long maxSize = 10 * 1024 * 1024;  // 10MB

        if(fileSize > maxSize) {
            file.delete();
            m.jsError("파일 크기는 10MB 이하만 가능합니다.");
            return;
        }

        // 처리...
    }
    return;
}
```

### 4. 여러 파일 업로드

#### 다중 선택 (배열)

**HTML**:
```html
<input type="file" name="files" multiple>
```

**JSP**:
```jsp
f.addElement("files", null, "required:'Y'");

if(m.isPost() && f.validate()) {

    File[] files = f.saveFiles("files");
    if(files != null && files.length > 0) {
        for(int i = 0; i < files.length; i++) {
            File file = files[i];
            String fileName = f.getFileNames("files")[i];

            m.p("파일 " + (i+1) + ": " + fileName);
            m.p("경로: " + file.getAbsolutePath());

            // DB 저장...
        }

        m.jsAlert(files.length + "개 파일 업로드 완료");
        m.jsReplace("list.jsp");
    }
    return;
}
```

#### 개별 필드

**HTML**:
```html
<input type="file" name="file1">
<input type="file" name="file2">
<input type="file" name="file3">
```

**JSP**:
```jsp
f.addElement("file1", null, "");
f.addElement("file2", null, "");
f.addElement("file3", null, "");

if(m.isPost() && f.validate()) {

    File file1 = f.saveFile("file1");
    File file2 = f.saveFile("file2");
    File file3 = f.saveFile("file3");

    // 각 파일 처리...
    return;
}
```

### 5. 저장 경로 커스터마이징

```jsp
// 기본 저장 경로는 /data/file/

// 커스텀 경로로 저장
String customPath = Config.getDataDir() + "/uploads/2025/06/";
File dir = new File(customPath);
if(!dir.exists()) {
    dir.mkdirs();
}

File uploadedFile = f.saveFile("file");
if(uploadedFile != null) {
    String newPath = customPath + uploadedFile.getName();
    File newFile = new File(newPath);
    uploadedFile.renameTo(newFile);

    m.p("최종 경로: " + newFile.getAbsolutePath());
}
```

### 6. 파일명 변경하여 저장

```jsp
if(m.isPost() && f.validate()) {

    File file = f.saveFile("file");
    if(file != null) {
        String originalName = f.getFileName("file");
        String ext = Malgn.getFileExt(originalName);

        // 고유한 파일명 생성
        String newFileName = m.time() + "_" + Malgn.getUniqId() + "." + ext;
        String savePath = Config.getDataDir() + "/file/" + newFileName;

        File newFile = new File(savePath);
        file.renameTo(newFile);

        m.p("원본 파일명: " + originalName);
        m.p("저장 파일명: " + newFileName);

        // DB에 두 파일명 모두 저장
        DataMap data = new DataMap();
        data.put("original_name", originalName);
        data.put("saved_name", newFileName);
        data.put("file_path", savePath);
        // ...
    }
    return;
}
```

---

## 파일 다운로드

### 1. 기본 다운로드

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

int id = m.ri("id");

FileDao dao = new FileDao();
DataSet info = dao.find("id = ?", id);

if(!info.next()) {
    m.jsError("파일을 찾을 수 없습니다.");
    return;
}

String filePath = info.s("file_path");
String fileName = info.s("file_name");

// 파일 다운로드
m.download(filePath, fileName);

%>
```

### 2. 대역폭 제한 다운로드

대용량 파일 다운로드 시 서버 부하를 줄이기 위해 속도 제한:

```jsp
// 100 KB/s로 제한
m.download(filePath, fileName, 100);

// 500 KB/s로 제한
m.download(filePath, fileName, 500);
```

### 3. 파일 브라우저 표시 (미리보기)

다운로드가 아닌 브라우저에서 직접 표시:

```jsp
// PDF 파일 브라우저에 표시
String pdfPath = Config.getDataDir() + "/document.pdf";
m.output(pdfPath, "application/pdf");

// 이미지 표시
String imagePath = Config.getDataDir() + "/photo.jpg";
m.output(imagePath, "image/jpeg");

// MIME 타입 자동 감지
m.output(filePath, null);
```

### 4. 조건부 다운로드

권한 확인 후 다운로드:

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

int id = m.ri("id");

// 로그인 체크
if(!auth.isLogin()) {
    m.jsError("로그인이 필요합니다.");
    return;
}

FileDao dao = new FileDao();
DataSet info = dao.find("id = ?", id);

if(!info.next()) {
    m.jsError("파일을 찾을 수 없습니다.");
    return;
}

// 권한 체크
int uploadUserId = info.i("user_id");
int currentUserId = auth.getUserId();

if(uploadUserId != currentUserId && !auth.isAdmin()) {
    m.jsError("다운로드 권한이 없습니다.");
    return;
}

// 다운로드 카운트 증가
dao.execute("UPDATE tb_file SET download_count = download_count + 1 WHERE id = ?", id);

// 다운로드
String filePath = info.s("file_path");
String fileName = info.s("file_name");
m.download(filePath, fileName);

%>
```

### 5. 폼 방식 다운로드

GET 방식 대신 POST로 다운로드:

**JSP**:
```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

if(m.isPost()) {
    int id = m.ri("id");

    FileDao dao = new FileDao();
    DataSet info = dao.find("id = ?", id);

    if(info.next()) {
        String filePath = info.s("file_path");
        String fileName = info.s("file_name");
        m.download(filePath, fileName);
    }
    return;
}

%>
<form method="post">
    <input type="hidden" name="id" value="123">
    <button type="submit">다운로드</button>
</form>
```

### 6. 여러 파일 ZIP으로 다운로드

여러 파일을 ZIP으로 압축하여 한 번에 다운로드:

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 다운로드할 파일 목록
String[] fileIds = m.reqArr("file_ids");

if(fileIds.length == 0) {
    m.jsError("선택된 파일이 없습니다.");
    return;
}

FileDao dao = new FileDao();
ArrayList<String> filePaths = new ArrayList<>();

for(String id : fileIds) {
    DataSet info = dao.find("id = ?", id);
    if(info.next()) {
        filePaths.add(info.s("file_path"));
    }
}

// ZIP 압축 및 다운로드
Zip zip = new Zip();
String[] files = filePaths.toArray(new String[0]);
zip.compress(files, "files_" + m.time() + ".zip", response);

%>
```

---

## 이미지 썸네일

### 1. 이미지 업로드 시 썸네일 자동 생성

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

f.addElement("photo", null, "required:'Y', allow:'jpg|jpeg|png|gif'");

if(m.isPost() && f.validate()) {

    File file = f.saveFile("photo");
    if(file != null) {
        String filePath = file.getAbsolutePath();
        String fileName = f.getFileName("photo");
        String ext = Malgn.getFileExt(fileName);

        // 썸네일 생성
        String thumbPath = Config.getDataDir() + "/thumb/";
        File thumbDir = new File(thumbPath);
        if(!thumbDir.exists()) {
            thumbDir.mkdirs();
        }

        String thumbFileName = "thumb_" + m.time() + "." + ext;
        String thumbFullPath = thumbPath + thumbFileName;

        Thumbnail thumb = new Thumbnail();
        // 가로 200px로 리사이징 (비율 유지)
        thumb.createThumbnail(filePath, thumbFullPath, 200, 0);

        // DB 저장
        ImageDao dao = new ImageDao();
        DataMap data = new DataMap();
        data.put("title", f.get("title"));
        data.put("image_path", filePath);
        data.put("thumb_path", thumbFullPath);
        data.put("file_size", file.length());
        data.put("reg_date", m.time());
        dao.insert(data);

        m.jsAlert("이미지 업로드 완료");
        m.jsReplace("list.jsp");
    }
    return;
}

%>
<form name="form1" method="post" enctype="multipart/form-data">
    <p>제목 : <input type="text" name="title"></p>
    <p>이미지 : <input type="file" name="photo" accept="image/*"></p>
    <p><input type="submit" value="업로드"></p>
</form>
<%= f.getScript() %>
```

### 2. 다양한 크기의 썸네일 생성

```jsp
File file = f.saveFile("photo");
if(file != null) {
    String sourcePath = file.getAbsolutePath();
    String thumbDir = Config.getDataDir() + "/thumb/";

    Thumbnail thumb = new Thumbnail();

    // 작은 썸네일 (100x100)
    thumb.createThumbnail(sourcePath, thumbDir + "small_" + fileName, 100, 100);

    // 중간 썸네일 (300x300)
    thumb.createThumbnail(sourcePath, thumbDir + "medium_" + fileName, 300, 300);

    // 큰 썸네일 (800px 너비, 높이 자동)
    thumb.createThumbnail(sourcePath, thumbDir + "large_" + fileName, 800, 0);
}
```

### 3. 정사각형 크롭

```jsp
// 200x200 정사각형으로 중앙 크롭
Thumbnail thumb = new Thumbnail();
thumb.createThumbnail(sourcePath, thumbPath, 200, 200);
```

---

## 파일 관리

### 1. 파일 정보 조회

```jsp
// 파일 확장자
String ext = Malgn.getFileExt("document.pdf");  // "pdf"

// MIME 타입
String mime = Malgn.getMimeType("image.png");   // "image/png"

// 파일 크기 포맷
String size = Malgn.getFileSize(1024);          // "1KB"
String size2 = Malgn.getFileSize(1048576);      // "1MB"
String size3 = Malgn.getFileSize(1073741824);   // "1GB"
```

### 2. 파일 읽기/쓰기

```jsp
// 파일 읽기
String content = Malgn.readFile("/path/to/file.txt");
String content2 = Malgn.readFile("/path/to/file.txt", "EUC-KR");

// 파일 쓰기
Malgn.writeFile("/path/to/output.txt", "Hello World");
Malgn.writeFile("/path/to/output.txt", "안녕하세요", "EUC-KR");
```

### 3. 파일 복사/이동/삭제

```jsp
// 파일 복사
Malgn.copyFile("/source/file.txt", "/dest/file.txt");

// 파일 삭제
Malgn.delFile("/path/to/file.txt");

// 폴더 재귀 삭제
Malgn.delFile("/path/to/dir", true);
```

### 4. 업로드 경로 생성

```jsp
// MD5 기반 고유 경로 생성
String uploadPath = m.getUploadPath("myfile.jpg");
// 결과: /data/file/abc123def456.jpg

String uploadUrl = m.getUploadUrl("myfile.jpg");
// 결과: http://example.com/file/abc123def456.jpg
```

---

## 보안 가이드

### 1. 확장자 검증 필수

```jsp
// Bad - 확장자 검증 없음
f.addElement("file", null, "required:'Y'");

// Good - 허용 확장자 명시
f.addElement("file", null, "required:'Y', allow:'jpg|jpeg|png|gif|pdf|doc|docx'");

// Better - 위험한 확장자 명시적 차단
f.addElement("file", null, "required:'Y', allow:'jpg|jpeg|png|gif|pdf|doc|docx', deny:'exe|jsp|php|asp|sh|bat'");
```

### 2. 파일명 검증

```jsp
String fileName = f.getFileName("file");

// 위험한 문자 제거
fileName = fileName.replaceAll("[^a-zA-Z0-9._-]", "_");

// 경로 조작 방지
fileName = fileName.replace("..", "");
fileName = fileName.replace("/", "");
fileName = fileName.replace("\\", "");
```

### 3. 파일 크기 검증

```jsp
File file = f.saveFile("file");
if(file != null) {
    long maxSize = 10 * 1024 * 1024;  // 10MB
    if(file.length() > maxSize) {
        file.delete();
        m.jsError("파일 크기 초과");
        return;
    }
}
```

### 4. MIME 타입 검증

```jsp
String fileName = f.getFileName("file");
String ext = Malgn.getFileExt(fileName);

// 확장자와 MIME 타입 매칭 확인
String expectedMime = Malgn.getMimeType(fileName);
// 실제 파일 MIME 타입과 비교하여 검증
```

### 5. 업로드 디렉토리 권한

```jsp
// 업로드 디렉토리에 실행 권한 제거 (Linux/Unix)
String uploadDir = Config.getDataDir() + "/file/";
Malgn.chmod("755", uploadDir);

// 또는 web.xml에서 실행 차단
```

### 6. 다운로드 권한 체크

```jsp
// 항상 권한 검증 후 다운로드
if(!auth.isLogin()) {
    m.jsError("로그인이 필요합니다.");
    return;
}

// 소유자 또는 관리자만 다운로드
if(!isOwner && !auth.isAdmin()) {
    m.httpError(403, "권한이 없습니다.");
    return;
}

m.download(filePath, fileName);
```

### 7. 경로 조작 방지

```jsp
// Bad - 사용자 입력을 직접 경로로 사용
String fileName = m.rs("file");
m.download("/data/file/" + fileName, fileName);  // 위험!

// Good - DB에서 경로 조회
int fileId = m.ri("id");
FileDao dao = new FileDao();
DataSet info = dao.find("id = ?", fileId);
if(info.next()) {
    m.download(info.s("file_path"), info.s("file_name"));
}
```

---

## 전체 예제

### 파일 게시판 구현

**list.jsp** (목록):
```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

ListManager lm = new ListManager();
lm.setRequest(request);
lm.setTable("tb_file");
lm.setListNum(20);
lm.addSearch("title,file_name", f.get("keyword"), "LIKE");
lm.setSort("id DESC");

DataSet list = lm.getDataSet();

p.setBody("main.file_list");
p.setLoop("list", list);
p.setVar("pager", lm.getPaging());
p.display();

%>
```

**upload.jsp** (업로드):
```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

if(!auth.isLogin()) {
    m.jsError("로그인이 필요합니다.");
    return;
}

f.addElement("title", null, "required:'Y', maxlength:200");
f.addElement("content", null, "allowhtml:'Y'");
f.addElement("file", null, "required:'Y', allow:'jpg|jpeg|png|gif|pdf|doc|docx|xls|xlsx|zip', deny:'exe|jsp|php'");

if(m.isPost() && f.validate()) {

    File file = f.saveFile("file");
    if(file != null) {
        String fileName = f.getFileName("file");
        String filePath = file.getAbsolutePath();
        long fileSize = file.length();

        FileDao dao = new FileDao();
        DataMap data = new DataMap();
        data.put("user_id", auth.getUserId());
        data.put("title", f.get("title"));
        data.put("content", f.get("content"));
        data.put("file_name", fileName);
        data.put("file_path", filePath);
        data.put("file_size", fileSize);
        data.put("download_count", 0);
        data.put("reg_date", m.time());

        int newId = dao.insert(data);

        m.jsAlert("업로드 완료");
        m.jsReplace("view.jsp?id=" + newId);
    } else {
        m.jsError("파일 업로드 실패");
    }
    return;
}

p.setBody("main.file_upload");
p.setVar("form_script", f.getScript());
p.display();

%>
```

**download.jsp** (다운로드):
```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

int id = m.ri("id");

FileDao dao = new FileDao();
DataSet info = dao.find("id = ?", id);

if(!info.next()) {
    m.jsError("파일을 찾을 수 없습니다.");
    return;
}

// 다운로드 카운트 증가
dao.execute("UPDATE tb_file SET download_count = download_count + 1 WHERE id = ?", id);

// 다운로드
String filePath = info.s("file_path");
String fileName = info.s("file_name");
m.download(filePath, fileName);

%>
```

**delete.jsp** (삭제):
```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

if(!auth.isLogin()) {
    m.jsError("로그인이 필요합니다.");
    return;
}

int id = m.ri("id");

FileDao dao = new FileDao();
DataSet info = dao.find("id = ?", id);

if(!info.next()) {
    m.jsError("파일을 찾을 수 없습니다.");
    return;
}

// 권한 체크
int uploadUserId = info.i("user_id");
int currentUserId = auth.getUserId();

if(uploadUserId != currentUserId && !auth.isAdmin()) {
    m.jsError("삭제 권한이 없습니다.");
    return;
}

// 파일 삭제
String filePath = info.s("file_path");
File file = new File(filePath);
if(file.exists()) {
    file.delete();
}

// DB 삭제
dao.delete("id = ?", id);

m.jsAlert("삭제되었습니다.");
m.jsReplace("list.jsp");

%>
```

---

## 베스트 프랙티스

### 1. 항상 확장자 검증

```jsp
f.addElement("file", null, "allow:'jpg|png|pdf|doc', deny:'exe|jsp|php'");
```

### 2. 파일 크기 제한

```jsp
long maxSize = 10 * 1024 * 1024;  // 10MB
if(file.length() > maxSize) {
    file.delete();
    m.jsError("파일 크기는 10MB 이하만 가능합니다.");
    return;
}
```

### 3. 고유한 파일명 사용

```jsp
String uniqueName = m.time() + "_" + Malgn.getUniqId() + "." + ext;
```

### 4. DB에 메타데이터 저장

```jsp
data.put("original_name", fileName);  // 사용자가 업로드한 원본 파일명
data.put("saved_name", uniqueName);   // 서버에 저장된 파일명
data.put("file_size", file.length()); // 파일 크기
data.put("file_ext", ext);            // 확장자
data.put("mime_type", mimeType);      // MIME 타입
```

### 5. 권한 체크

```jsp
// 업로드
if(!auth.isLogin()) {
    m.jsError("로그인이 필요합니다.");
    return;
}

// 다운로드/삭제
if(!isOwner && !auth.isAdmin()) {
    m.jsError("권한이 없습니다.");
    return;
}
```

### 6. 에러 처리

```jsp
File file = f.saveFile("file");
if(file == null) {
    m.jsError("파일 업로드에 실패했습니다.");
    return;
}

if(!file.exists()) {
    m.jsError("파일이 존재하지 않습니다.");
    return;
}
```

---

## 참고 문서

- [Form 클래스 - 데이터 입력 및 유효성 체크](form-validation.md)
- [유틸리티 메소드 - 파일 처리](utility-methods.md#파일-처리)
- [FTP 및 ZIP - 파일 전송 및 압축](file-transfer.md)

---

[← 이전: 데이터 입력 및 유효성 체크](form-validation.md) | [목차로 돌아가기](README.md) | [다음: 목록 및 검색 →](list-search.md)
# 7. 목록 및 검색

[← 목차로 돌아가기](README.md)

---

## ListManager 클래스

페이징이 있는 목록을 작성하기 위한 클래스입니다.

### 주요 기능

- 목록 데이터 조회
- 전체 개수 계산
- 페이징 네비게이션 자동 생성
- 검색 조건 처리
- 다양한 DBMS 지원 (Oracle, MySQL, MSSQL, DB2 등)

---

## 기본 사용법

### 1. 간단한 목록

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

ListManager lm = new ListManager();
lm.setRequest(request);
lm.setListNum(20);  // 페이지당 20개
lm.setTable("tb_blog");

DataSet list = lm.getDataSet();
int total = lm.getTotalNum();
String pager = lm.getPaging();

p.setBody("main.list");
p.setLoop("list", list);
p.setVar("total", total);
p.setVar("pager", pager);
p.display();

%>
```

### 2. HTML 템플릿

```html
<h1>게시판 (전체: {{total}}개)</h1>

<table>
    <thead>
        <tr>
            <th>번호</th>
            <th>제목</th>
            <th>작성자</th>
            <th>날짜</th>
        </tr>
    </thead>
    <tbody>
        <!--@loop(list)-->
        <tr>
            <td>{{list.id}}</td>
            <td>{{list.subject}}</td>
            <td>{{list.name}}</td>
            <td>{{list.reg_date}}</td>
        </tr>
        <!--/loop(list)-->
    </tbody>
</table>

{{pager}}
```

---

## 조건 추가

### addWhere() - 고정 조건

항상 적용되는 조건입니다.

```jsp
ListManager lm = new ListManager();
lm.setRequest(request);
lm.setTable("tb_blog");
lm.addWhere("status = 1");  // 승인된 게시물만
lm.addWhere("type = '02'");  // 특정 타입만

DataSet list = lm.getDataSet();
```

**생성되는 SQL**:
```sql
SELECT * FROM tb_blog WHERE status = 1 AND type = '02'
```

### addSearch() - 동적 검색 조건

값이 있을 때만 조건이 추가됩니다.

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

ListManager lm = new ListManager();
lm.setRequest(request);
lm.setTable("tb_blog");

// 동적 검색 조건
lm.addSearch("subject,content", f.get("keyword"), "LIKE");
lm.addSearch("status", f.getInt("status"), ">");
lm.addSearch("type", f.get("type"));

lm.setOrderBy("id DESC");

DataSet list = lm.getDataSet();

p.setBody("main.search");
p.setLoop("list", list);
p.setVar("pager", lm.getPaging());
p.display();

%>
```

---

## addSearch() 연산자

### 기본 연산자

```jsp
// 같음 (기본값)
lm.addSearch("status", 1);
// SQL: status = 1

// 같지 않음
lm.addSearch("status", 0, "!=");
// SQL: status != 0

// 크다
lm.addSearch("age", 18, ">");
// SQL: age > 18

// 크거나 같다
lm.addSearch("age", 18, ">=");
// SQL: age >= 18

// 작다
lm.addSearch("age", 65, "<");
// SQL: age < 65

// 작거나 같다
lm.addSearch("age", 65, "<=");
// SQL: age <= 65
```

### LIKE 연산자

```jsp
// %keyword%
lm.addSearch("subject", "맑은", "LIKE");
// SQL: subject LIKE '%맑은%'

// keyword%
lm.addSearch("subject", "맑은", "LIKE%");
// SQL: subject LIKE '맑은%'

// %keyword
lm.addSearch("subject", "맑은", "%LIKE");
// SQL: subject LIKE '%맑은'
```

### 여러 필드 OR 검색

```jsp
// 제목 또는 내용에서 검색
lm.addSearch("subject,content", "맑은", "LIKE");
// SQL: (subject LIKE '%맑은%' OR content LIKE '%맑은%')

// 이름, 이메일, 전화번호 중 하나라도 일치
lm.addSearch("name,email,phone", "홍길동", "LIKE");
// SQL: (name LIKE '%홍길동%' OR email LIKE '%홍길동%' OR phone LIKE '%홍길동%')
```

---

## 정렬

### setOrderBy()

```jsp
// 내림차순
lm.setOrderBy("id DESC");

// 오름차순
lm.setOrderBy("reg_date ASC");

// 여러 필드 정렬
lm.setOrderBy("status DESC, reg_date DESC");
```

**주의**: MSSQL 사용 시 `setOrderBy()`는 필수입니다.

---

## 조인 테이블 목록

### 기본 조인

```jsp
ListManager lm = new ListManager();
lm.setRequest(request);
lm.setListNum(20);
lm.setTable("tb_blog a JOIN tb_user b ON a.user_id = b.id");
lm.setFields("a.*, b.name AS user_name, b.email");
lm.addWhere("a.status = 1");
lm.setOrderBy("a.id DESC");

DataSet list = lm.getDataSet();
```

### LEFT JOIN

```jsp
lm.setTable("tb_blog a LEFT JOIN tb_category b ON a.category_id = b.id");
lm.setFields("a.*, b.name AS category_name");
```

---

## 검색 페이지 전체 예제

### JSP

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 검색 폼 설정
f.addElement("keyword", null, null);
f.addElement("status", null, null);
f.addElement("category", null, null);

// 목록 관리자 설정
ListManager lm = new ListManager();
lm.setRequest(request);
lm.setListNum(20);
lm.setTable("tb_blog");

// 검색 조건
lm.addSearch("subject,content", f.get("keyword"), "LIKE");
lm.addSearch("status", f.getInt("status"));
lm.addSearch("category", f.get("category"));

// 정렬
lm.setOrderBy("id DESC");

// 데이터 조회
DataSet list = lm.getDataSet();

p.setBody("main.blog_list");
p.setLoop("list", list);
p.setVar("total", lm.getTotalNum());
p.setVar("pager", lm.getPaging());
p.setVar("form_script", f.getScript());
p.display();

%>
```

### HTML

```html
<h1>게시판 (전체: {{total}}개)</h1>

<!-- 검색 폼 -->
<form name="form1" method="get">
    <input type="text" name="keyword" placeholder="검색어" value="{{keyword}}">
    <select name="status">
        <option value="">전체</option>
        <option value="1">승인</option>
        <option value="0">대기</option>
    </select>
    <select name="category">
        <option value="">전체 카테고리</option>
        <option value="notice">공지사항</option>
        <option value="free">자유게시판</option>
    </select>
    <button type="submit">검색</button>
</form>

<!-- 목록 -->
<table>
    <thead>
        <tr>
            <th>번호</th>
            <th>제목</th>
            <th>작성자</th>
            <th>날짜</th>
            <th>조회</th>
        </tr>
    </thead>
    <tbody>
        <!--@loop(list)-->
        <tr>
            <td>{{list.id}}</td>
            <td><a href="view.jsp?id={{list.id}}">{{list.subject}}</a></td>
            <td>{{list.name}}</td>
            <td>{{list.reg_date}}</td>
            <td>{{list.hit}}</td>
        </tr>
        <!--/loop(list)-->
    </tbody>
</table>

<!-- 페이징 -->
{{pager}}

{{form_script}}
```

---

## 페이징 커스터마이징

### Pager 클래스 직접 사용

**JSP**:
```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

Pager pg = new Pager(request);
pg.setPageNum(f.getInt("page", 1));  // 현재 페이지
pg.setTotalNum(300);  // 전체 개수
pg.setListNum(20);  // 페이지당 개수
pg.setPageNum(10);  // 페이지 링크 개수
pg.setLinkType(9);  // 링크 타입
pg.setLink("/main/list.jsp");  // 기본 URL

p.setBody("main.custom_paging");
p.setVar("pager", pg.getPager());
p.display();

%>
```

**HTML** (`/html/main/custom_paging.html`):
```html
{{pager}}
```

### 링크 타입

| 타입 | 설명 |
|------|------|
| 1 | `[이전] 1 2 3 4 5 [다음]` |
| 2 | `[처음] [이전] 1 2 3 4 5 [다음] [마지막]` |
| 3 | `1 2 3 4 5` (링크만) |
| 9 | Bootstrap 스타일 |

---

## 주요 메소드

### ListManager

| 메소드 | 설명 |
|--------|------|
| setRequest(request) | Request 객체 설정 |
| setTable(table) | 테이블 설정 |
| setFields(fields) | 조회할 필드 설정 |
| setListNum(num) | 페이지당 목록 개수 |
| addWhere(condition) | 고정 조건 추가 |
| addSearch(field, value) | 검색 조건 추가 (=) |
| addSearch(field, value, op) | 검색 조건 추가 (연산자 지정) |
| setOrderBy(order) | 정렬 설정 |
| setGroupBy(group) | 그룹핑 설정 |
| getDataSet() | 목록 데이터 조회 |
| getTotalNum() | 전체 개수 조회 |
| getPaging() | 페이징 HTML 생성 |
| setDebug(out) | 디버깅 모드 |

---

## 고급 기능

### 1. 필드 선택

```jsp
lm.setFields("id, subject, name, reg_date");
```

### 2. 그룹핑

```jsp
lm.setFields("category, COUNT(*) AS cnt");
lm.setGroupBy("category");
```

### 3. HAVING 조건

```jsp
lm.setFields("user_id, COUNT(*) AS cnt");
lm.setGroupBy("user_id");
lm.setHaving("COUNT(*) > 10");
```

### 4. 서브쿼리

```jsp
lm.setTable("(SELECT * FROM tb_blog WHERE status = 1) AS a");
```

---

## 디버깅

```jsp
ListManager lm = new ListManager();
lm.setDebug(out);  // SQL 쿼리 및 파라미터 출력
```

**출력 내용**:
- 생성된 SQL 쿼리
- 검색 조건
- 전체 개수
- 실행 시간

---

## 베스트 프랙티스

### 1. 페이지 개수는 적절하게

```jsp
lm.setListNum(20);  // 일반적으로 10~50개
```

### 2. 필요한 필드만 조회

```jsp
// Bad
lm.setFields("*");

// Good
lm.setFields("id, subject, name, reg_date");
```

### 3. 인덱스 활용

```jsp
// 인덱스가 있는 필드로 정렬
lm.setOrderBy("id DESC");

// 인덱스가 있는 필드로 검색
lm.addSearch("user_id", userId);
```

### 4. 검색 조건 검증

```jsp
// 빈 값은 자동으로 무시됨
String keyword = f.get("keyword");
lm.addSearch("subject", keyword, "LIKE");  // keyword가 빈값이면 조건 추가 안됨
```

---

## 다음 단계

- [DataSet 활용](dataset.md)으로 이동하여 데이터 처리 방법을 배워보세요.

---

[← 이전: 파일 업로드 및 다운로드](file-upload-download.md) | [목차로 돌아가기](README.md) | [다음: DataSet 활용 →](dataset.md)
# 8. DataSet 활용

[← 목차로 돌아가기](README.md)

---

## DataSet이란?

DataSet은 MalgnFramework에서 테이블 형태의 데이터를 다루기 위한 핵심 클래스입니다.

### 주요 특징

- `List<Map<String, Object>>` 구조
- 데이터베이스 조회 결과를 담는 표준 컨테이너
- 행(Row) 단위 순회 및 조작
- 열(Column) 검증 기능
- JSON 변환 지원

---

## 기본 사용법

### 1. DataSet 생성

```jsp
DataSet ds = new DataSet();

// 행 추가
ds.addRow();
ds.put("id", 1);
ds.put("name", "홍길동");
ds.put("email", "hong@example.com");

// 다른 행 추가
ds.addRow();
ds.put("id", 2);
ds.put("name", "김철수");
ds.put("email", "kim@example.com");
```

### 2. 데이터 조회

```jsp
// 처음 행으로 이동
ds.first();

// 다음 행으로 이동하며 순회
while(ds.next()) {
    int id = ds.getInt("id");
    String name = ds.getString("name");
    String email = ds.getString("email");

    m.p(id + " : " + name + " : " + email);
}
```

---

## 행(Row) 조작

### 행 추가 및 이동

```jsp
DataSet ds = new DataSet();

// 새 행 추가
ds.addRow();
ds.put("subject", "첫 번째 게시물");
ds.put("content", "내용입니다.");

ds.addRow();
ds.put("subject", "두 번째 게시물");
ds.put("content", "또 다른 내용입니다.");

// 처음으로 이동
ds.first();

// 다음 행으로 이동
if(ds.next()) {
    m.p(ds.getString("subject"));  // "첫 번째 게시물"
}

if(ds.next()) {
    m.p(ds.getString("subject"));  // "두 번째 게시물"
}
```

### 특정 위치로 이동

```jsp
// 0부터 시작하는 인덱스
ds.moveRow(0);  // 첫 번째 행
ds.moveRow(2);  // 세 번째 행
```

### 현재 행 가져오기

```jsp
DataMap row = ds.getRow();
String subject = row.getString("subject");
int status = row.getInt("status");
```

---

## 데이터 가져오기

### 타입별 메소드

```jsp
// 문자열
String name = ds.getString("name");
String subject = ds.getString("subject", "기본값");

// 숫자
int age = ds.getInt("age");
long id = ds.getLong("id");
double price = ds.getDouble("price");

// 날짜/시간
String regDate = ds.getString("reg_date");

// null 체크가 필요한 경우
Object value = ds.get("field_name");
if(value != null) {
    // 처리
}
```

### 기본값 설정

```jsp
// 값이 null이면 기본값 반환
String title = ds.getString("title", "제목 없음");
int status = ds.getInt("status", 0);
```

---

## DataMap 클래스

DataSet의 각 행은 DataMap 객체입니다.

### 기본 사용

```jsp
DataMap map = new DataMap();
map.put("subject", "안녕하세요");
map.put("content", "반갑습니다");
map.put("reg_date", m.time());
map.put("status", 1);

// 값 가져오기
String subject = map.getString("subject");
int status = map.getInt("status");
```

### JSON 변환

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

DataMap map = new DataMap();
map.put("subject", "Nice to meet you.");
map.put("content", "Nice to meet you too.");
map.put("reg_date", Malgn.time());
map.put("status", 1);

// JSON 인코딩
String json = Json.encode(map);
m.p(json);
// 출력: {"subject":"Nice to meet you.","content":"Nice to meet you too.","reg_date":"20250623153000","status":1}

%>
```

---

## 열(Column) 검증

### setCol() - 검증 규칙 설정

```jsp
DataSet ds = new DataSet();
ds.addRow();

// 컬럼별 검증 규칙 설정
ds.setCol("email", "type:email, required:Y");
ds.setCol("age", "type:number, min:1, max:150");
ds.setCol("phone", "type:phone");

ds.put("email", "invalid-email");
ds.put("age", 200);

// 검증 실행
if(!ds.validate()) {
    m.p("검증 실패");
}
```

### 검증 속성

| 속성 | 설명 | 예제 |
|------|------|------|
| type | 데이터 타입 | `type:email`, `type:number` |
| required | 필수 여부 | `required:Y` |
| minlength | 최소 길이 | `minlength:8` |
| maxlength | 최대 길이 | `maxlength:100` |
| min | 최소값 | `min:0` |
| max | 최대값 | `max:999` |
| length | 정확한 길이 | `length:6` |

**지원하는 타입**:
- `email`, `url`, `number`, `phone`, `datetime`, `hangul` 등

---

## JSON 처리

### JSON 클래스 (j)

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="/init.jsp" %><%

DataMap map = new DataMap();
map.put("id", 123);
map.put("name", "홍길동");
map.put("email", "hong@example.com");

// 성공 응답
j.success("조회 성공", map);
// 출력: {"result":"success","message":"조회 성공","data":{...}}

%>
```

### JSON 응답 메소드

```jsp
// 에러 응답
j.error(100, "잘못된 요청입니다");
// 출력: {"result":"error","code":100,"message":"잘못된 요청입니다"}

// 성공 응답 (데이터 포함)
j.success("성공", dataMap);
// 출력: {"result":"success","message":"성공","data":{...}}

// 커스텀 응답
j.print(0, "완료되었습니다", dataMap);
// 출력: {"code":0,"message":"완료되었습니다","data":{...}}
```

---

## 데이터베이스 연동

### 조회 결과를 DataSet으로

```jsp
UserDao dao = new UserDao();
DataSet list = dao.query("WHERE status = 1 ORDER BY id DESC");

list.first();
while(list.next()) {
    m.p(list.getInt("id"));
    m.p(list.getString("name"));
    m.p(list.getString("email"));
}
```

### ListManager에서 DataSet 사용

```jsp
ListManager lm = new ListManager();
lm.setRequest(request);
lm.setTable("tb_blog");
lm.setListNum(20);
lm.addSearch("subject,content", keyword, "LIKE");

DataSet list = lm.getDataSet();

// 템플릿에 전달
p.setLoop("list", list);
```

---

## 유틸리티 메소드

### 날짜/시간 처리

```jsp
// 현재 시각
String now = m.time();  // "20250623153045"

// 포맷 지정
String today = m.time("yyyyMMdd");  // "20250623"
String formatted = m.time("yyyy-MM-dd HH:mm:ss");  // "2025-06-23 15:30:45"

// 날짜 계산
String nextMonth = m.addDate("M", 1, m.time(), "yyyyMMdd");  // 1개월 후
String yesterday = m.addDate("D", -1, m.time(), "yyyyMMdd");  // 1일 전

// 날짜 차이 계산
long days = m.diffDate("D", "20000101", m.time("yyyyMMdd"));
m.p("2000년 1월 1일부터 " + days + "일 지났습니다");

// Unix timestamp
long timestamp = m.getUnixTimeL("29991231");

// 시간대 변환
Date d = m.strToDate("yyyyMMddHHmmss", "20230425113600", "UTC");
String seoulTime = m.time("yyyyMMddHHmmss", d, "Asia/Seoul");
```

### 문자열 처리

```jsp
// 왼쪽 패딩
String padded = m.strpad("abc", 10, "0");  // "0000000abc"

// 오른쪽 패딩
String rpadded = m.strrpad("abc", 10, "0");  // "abc0000000"

// 문자열 자르기 (한글 고려)
String cut = m.cutString("우리는 자랑스러운 한국인이다", 10, "...");
// "우리는 자랑스..."

// Hex 변환
byte[] bytes = new byte[] {'a', 'b', 'c', 'd'};
String hex = Malgn.bytesToHex(bytes);  // "61626364"

byte[] decoded = Malgn.hexToBytes("61626364");
String original = new String(decoded);  // "abcd"

// 숫자 파싱
double price = Malgn.parseDouble("1.00");  // 1.0
```

---

## 실전 예제

### 게시판 목록 처리

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

ListManager lm = new ListManager();
lm.setRequest(request);
lm.setTable("tb_blog");
lm.setListNum(20);
lm.setOrderBy("id DESC");

DataSet list = lm.getDataSet();

// 날짜 포맷팅 추가
list.first();
while(list.next()) {
    String regDate = list.getString("reg_date");
    String formatted = m.time("yyyy-MM-dd", regDate);
    list.put("formatted_date", formatted);
}

p.setBody("main.blog_list");
p.setLoop("list", list);
p.display();

%>
```

### API 응답 처리

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="/init.jsp" %><%

try {
    UserDao dao = new UserDao();
    DataSet users = dao.query("WHERE status = 1 ORDER BY name ASC");

    DataMap result = new DataMap();
    result.put("total", users.size());
    result.put("users", users);

    j.success("사용자 목록 조회 성공", result);

} catch(Exception e) {
    j.error(500, "서버 오류: " + e.getMessage());
}

%>
```

### 데이터 검증 및 저장

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

f.addElement("name", null, "required:Y, minlength:2");
f.addElement("email", null, "type:email, required:Y");
f.addElement("age", null, "type:number, min:1, max:150");

if(m.isPost() && f.validate()) {

    DataMap data = new DataMap();
    data.put("name", f.get("name"));
    data.put("email", f.get("email"));
    data.put("age", f.getInt("age"));
    data.put("reg_date", m.time());

    UserDao dao = new UserDao();
    int newId = dao.insert(data);

    m.jsAlert("등록되었습니다. ID: " + newId);
    m.jsReplace("list.jsp");
    return;
}

%>
```

---

## 주요 메소드 정리

### DataSet

| 메소드 | 설명 |
|--------|------|
| addRow() | 새 행 추가 |
| first() | 첫 행으로 이동 (실제로 -1 위치) |
| next() | 다음 행으로 이동 및 존재 여부 반환 |
| moveRow(index) | 특정 인덱스로 이동 |
| getRow() | 현재 행의 DataMap 반환 |
| size() | 전체 행 개수 |
| put(key, value) | 현재 행에 값 설정 |
| get(key) | 현재 행에서 값 가져오기 |
| getString(key) | 문자열로 가져오기 |
| getInt(key) | 정수로 가져오기 |
| getLong(key) | Long으로 가져오기 |
| getDouble(key) | Double로 가져오기 |
| setCol(name, rules) | 컬럼 검증 규칙 설정 |
| validate() | 전체 행 검증 |

### DataMap

| 메소드 | 설명 |
|--------|------|
| put(key, value) | 값 설정 |
| get(key) | 값 가져오기 |
| getString(key) | 문자열로 가져오기 |
| getString(key, defaultValue) | 기본값을 가진 문자열 |
| getInt(key) | 정수로 가져오기 |
| getInt(key, defaultValue) | 기본값을 가진 정수 |
| getLong(key) | Long으로 가져오기 |
| getDouble(key) | Double로 가져오기 |

---

## 베스트 프랙티스

### 1. null 체크

```jsp
// Bad
String name = ds.getString("name");
if(name.equals("admin")) {  // NullPointerException 가능
    // ...
}

// Good
String name = ds.getString("name", "");
if("admin".equals(name)) {  // 안전
    // ...
}
```

### 2. 타입 명시

```jsp
// Bad
Object value = ds.get("age");
int age = (Integer)value;  // ClassCastException 가능

// Good
int age = ds.getInt("age");
```

### 3. 순회 패턴

```jsp
// 항상 first() 후 next()
ds.first();
while(ds.next()) {
    // 처리
}
```

---

## 다음 단계

- [XML 처리](xml.md)로 이동하여 XML 파싱 방법을 배워보세요.

---

[← 이전: 목록 및 검색](list-search.md) | [목차로 돌아가기](README.md) | [다음: XML 처리 →](xml.md)
# JSON 처리

맑은프레임워크의 Json 클래스는 JSON 데이터의 생성, 파싱, 변환을 위한 포괄적인 기능을 제공합니다.

## 목차

- [기본 사용법](#기본-사용법)
- [JSON 파싱](#json-파싱)
- [JSON 생성](#json-생성)
- [JSON 응답](#json-응답)
- [정적 메소드](#정적-메소드)

---

## 기본 사용법

### Json 객체 생성

```jsp
// 빈 JSON 객체 생성
Json j = new Json();

// JSON 문자열로 초기화
Json j = new Json("{\"name\":\"John\",\"age\":30}");

// URL에서 JSON 가져오기
Json j = new Json("https://api.example.com/data");

// Writer와 함께 생성 (응답 출력용)
Json j = new Json(out);
```

### 디버그 모드

```jsp
Json j = new Json();
j.setDebug(out);  // Writer와 함께
j.setDebug();     // 로그 파일로
```

---

## JSON 파싱

### JSON 문자열 파싱

```jsp
Json j = new Json();
j.setJson("{\"name\":\"John\",\"age\":30,\"address\":{\"city\":\"Seoul\"}}");

// 경로로 값 가져오기
String name = j.getString("//name");      // "John"
int age = j.getInt("//age");              // 30
String city = j.getString("//address/city"); // "Seoul"
```

### 배열 처리

```jsp
String jsonArray = "[{\"id\":1,\"name\":\"A\"},{\"id\":2,\"name\":\"B\"}]";
Json j = new Json(jsonArray);

// 배열 요소 접근
String name = j.getString("//0/name");  // "A"
int id = j.getInt("//1/id");            // 2

// JSONArray로 가져오기
JSONArray arr = j.getJSONArray("//");
```

### DataSet으로 변환

```jsp
String json = "{\"users\":[{\"name\":\"John\",\"age\":30},{\"name\":\"Jane\",\"age\":25}]}";
Json j = new Json(json);

// JSON 배열을 DataSet으로 변환
DataSet users = j.getDataSet("//users");
while(users.next()) {
    m.p(users.s("name") + ": " + users.i("age"));
}
```

### Map으로 변환

```jsp
Json j = new Json("{\"name\":\"John\",\"age\":30}");
Map<String, Object> map = j.getMap("//");
```

---

## JSON 생성

### 객체 생성

```jsp
Json j = new Json();

// 키-값 추가
j.put("name", "John");
j.put("age", 30);
j.put("active", true);

// Map으로 한번에 설정
DataMap map = new DataMap();
map.put("subject", "Hello");
map.put("content", "World");
j.put(map);

// 문자열로 변환
String json = j.toString();
// {"name":"John","age":30,"active":true}
```

### 배열 생성

```jsp
Json j = new Json();

// 배열 요소 추가
j.put("Apple");
j.put("Banana");
j.put("Cherry");

// List로 한번에 설정
List<Object> list = new ArrayList<>();
list.add("Apple");
list.add("Banana");
j.put(list);
```

---

## JSON 응답

### 기본 응답 형식

Json 클래스는 표준 API 응답 형식을 제공합니다:

```json
{
  "error": 0,
  "message": "success",
  "data": { ... }
}
```

### 성공 응답

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 간단한 성공 응답
j.success();
// {"error":0,"message":"success"}

// 데이터와 함께
DataMap data = new DataMap();
data.put("id", 123);
data.put("name", "John");
j.success(data);
// {"error":0,"message":"success","data":{"id":123,"name":"John"}}

// 사용자 정의 메시지
j.success("작업이 완료되었습니다", data);
// {"error":0,"message":"작업이 완료되었습니다","data":{...}}

%>
```

### 에러 응답

```jsp
// 에러 코드와 메시지
j.error(404, "데이터를 찾을 수 없습니다");
// {"error":404,"message":"데이터를 찾을 수 없습니다"}

// 간단한 에러 메시지
j.error("오류가 발생했습니다");
// {"_ERROR_":"오류가 발생했습니다"}
```

### 사용자 정의 응답

```jsp
// 완전한 제어
j.print(0, "success", data);
j.print(100, "validation error", null);
```

### 실제 사용 예제

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 폼 검증
f.addElement("email", null, "required:Y, email:Y");
f.addElement("password", null, "required:Y, minlength:8");

if(!f.validate()) {
    j.error(400, f.getErrorString());
    return;
}

// 데이터 처리
UserDao dao = new UserDao();
DataSet user = dao.query("WHERE email = ?", f.get("email"));

if(!user.next()) {
    j.error(404, "사용자를 찾을 수 없습니다");
    return;
}

// 성공 응답
DataMap result = new DataMap();
result.put("id", user.i("id"));
result.put("name", user.s("name"));
result.put("email", user.s("email"));
j.success("로그인 성공", result);

%>
```

---

## 정적 메소드

### 인코딩

```jsp
// Map을 JSON 문자열로
DataMap map = new DataMap();
map.put("subject", "Nice to meet you.");
map.put("content", "Nice to meet you too.");
map.put("reg_date", Malgn.time());
map.put("status", 1);

String json = Json.encode(map);
m.p(json);
// {"subject":"Nice to meet you.","content":"Nice to meet you too.","reg_date":"2025-10-23 14:30:00","status":1}

// List를 JSON 문자열로
List<String> list = new ArrayList<>();
list.add("Apple");
list.add("Banana");
String jsonArray = Json.encode(list);
// ["Apple","Banana"]
```

### 디코딩

```jsp
// JSON 문자열을 DataSet으로
String json = "{\"name\":\"John\",\"age\":30}";
DataSet rs = Json.decode(json);
rs.next();
m.p(rs.s("name"));  // "John"
m.p(rs.i("age"));   // 30

// 배열 JSON도 처리 가능
String jsonArray = "[{\"name\":\"John\"},{\"name\":\"Jane\"}]";
DataSet users = Json.decode(jsonArray);
while(users.next()) {
    m.p(users.s("name"));
}
```

### Map/List 변환

```jsp
// JSON 문자열을 HashMap으로
String json = "{\"name\":\"John\",\"age\":30}";
HashMap<String, Object> map = Json.toMap(json);

// JSONObject를 HashMap으로
JSONObject obj = new JSONObject(json);
HashMap<String, Object> map2 = Json.toMap(obj);

// JSON 배열 문자열을 ArrayList로
String jsonArray = "[\"Apple\",\"Banana\",\"Cherry\"]";
ArrayList<Object> list = Json.toList(jsonArray);

// JSONArray를 ArrayList로
JSONArray arr = new JSONArray(jsonArray);
ArrayList<Object> list2 = Json.toList(arr);
```

---

## 중첩된 객체 처리

```jsp
String json = "{\"user\":{\"name\":\"John\",\"address\":{\"city\":\"Seoul\",\"zip\":\"12345\"}}}";
Json j = new Json(json);

// 경로를 통한 깊은 접근
String name = j.getString("//user/name");           // "John"
String city = j.getString("//user/address/city");   // "Seoul"
String zip = j.getString("//user/address/zip");     // "12345"

// 중첩된 객체를 Map으로
Map<String, Object> address = j.getMap("//user/address");
// {"city":"Seoul","zip":"12345"}

// JSONObject로 직접 가져오기
JSONObject userObj = j.getJSONObject("//user");
```

---

## URL에서 JSON 가져오기

```jsp
// REST API 호출 및 파싱
Json j = new Json("https://api.example.com/users/123");

// 응답 데이터 접근
String name = j.getString("//name");
String email = j.getString("//email");

// 또는 setUrl 사용
Json j = new Json();
j.setUrl("https://api.example.com/data");
DataSet result = j.getDataSet("//items");
```

---

## 출력 및 디버깅

```jsp
// JSON을 Writer로 출력
Json j = new Json();
j.put("name", "John");
j.put("age", 30);
j.print(out);  // {"name":"John","age":30}

// 디버그 모드로 상세 로그
Json j = new Json();
j.setDebug(out);
j.setUrl("https://api.example.com/data");  // HTTP 요청/응답 로그 출력
```

---

## 주의사항

1. **경로 표현**: JSON 경로는 `//`로 시작해야 합니다
   ```jsp
   j.getString("//name")      // 올바름
   j.getString("name")        // 작동 안 함
   ```

2. **배열 인덱스**: 숫자로 배열 요소에 접근
   ```jsp
   j.getString("//items/0/name")  // 첫 번째 항목
   j.getString("//items/1/name")  // 두 번째 항목
   ```

3. **타입 변환**: 적절한 getter 메소드 사용
   ```jsp
   int age = j.getInt("//age");        // 숫자
   String name = j.getString("//name"); // 문자열
   double price = j.getDouble("//price"); // 실수
   ```

4. **에러 처리**: null 체크 필수
   ```jsp
   Object data = j.get("//data");
   if(data == null) {
       // 데이터가 없거나 경로가 잘못됨
   }
   ```

---

## 관련 문서

- [DataSet 활용](dataset.md) - DataSet과 JSON 변환
- [HTTP 클라이언트](http-client.md) - REST API 호출
- [데이터베이스 연동](database.md) - DB 데이터의 JSON 변환
# 9. XML 처리

[← 목차로 돌아가기](README.md)

---

## SimpleParser 클래스

MalgnFramework는 XML 파싱을 위한 SimpleParser 클래스를 제공합니다.

### 주요 기능

- XML 파일 파싱
- XPath를 사용한 노드 검색
- 단일 값 추출
- 여러 노드를 DataSet으로 변환
- 간편한 API

---

## 기본 사용법

### 1. XML 파일 로드

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

SimpleParser xml = new SimpleParser(Config.getDocRoot() + "/data/sample.xml");

%>
```

### 2. 노드 값 가져오기

```jsp
// XPath로 단일 값 추출
String logDir = xml.getNodeValue("//config/env/logDir");
m.p(logDir);
```

### 3. 여러 노드를 DataSet으로

```jsp
// XPath로 여러 노드 추출
DataSet users = xml.getDataSet("//config/users/user");

users.first();
while(users.next()) {
    m.p(users.getString("id"));
    m.p(users.getString("name"));
    m.p(users.getString("email"));
}
```

---

## XML 파일 예제

### sample.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<config>
    <env>
        <logDir>/var/log/app</logDir>
        <dataDir>/var/data</dataDir>
        <uploadDir>/var/upload</uploadDir>
    </env>

    <users>
        <user>
            <id>user001</id>
            <name>홍길동</name>
            <email>hong@example.com</email>
            <role>admin</role>
        </user>
        <user>
            <id>user002</id>
            <name>김철수</name>
            <email>kim@example.com</email>
            <role>user</role>
        </user>
        <user>
            <id>user003</id>
            <name>이영희</name>
            <email>lee@example.com</email>
            <role>user</role>
        </user>
    </users>

    <settings>
        <maxUploadSize>10485760</maxUploadSize>
        <sessionTimeout>3600</sessionTimeout>
        <debugMode>true</debugMode>
    </settings>
</config>
```

---

## 주요 메소드

### getNodeValue()

단일 노드의 값을 가져옵니다.

```jsp
SimpleParser xml = new SimpleParser(Config.getDocRoot() + "/data/sample.xml");

// 단일 값 추출
String logDir = xml.getNodeValue("//config/env/logDir");
m.p(logDir);  // "/var/log/app"

String dataDir = xml.getNodeValue("//config/env/dataDir");
m.p(dataDir);  // "/var/data"

String maxSize = xml.getNodeValue("//config/settings/maxUploadSize");
m.p(maxSize);  // "10485760"
```

### getDataSet()

여러 노드를 DataSet으로 변환합니다.

```jsp
SimpleParser xml = new SimpleParser(Config.getDocRoot() + "/data/sample.xml");

// 여러 user 노드를 DataSet으로
DataSet users = xml.getDataSet("//config/users/user");

m.p("총 " + users.size() + "명의 사용자");

users.first();
while(users.next()) {
    String id = users.getString("id");
    String name = users.getString("name");
    String email = users.getString("email");
    String role = users.getString("role");

    m.p(id + " - " + name + " (" + email + ") - " + role);
}
```

**출력**:
```
총 3명의 사용자
user001 - 홍길동 (hong@example.com) - admin
user002 - 김철수 (kim@example.com) - user
user003 - 이영희 (lee@example.com) - user
```

---

## XPath 표현식

### 기본 표현식

```jsp
// 루트부터 절대 경로
xml.getNodeValue("/config/env/logDir");

// 어디서든 찾기 (//)
xml.getNodeValue("//logDir");

// 특정 위치의 노드
xml.getNodeValue("//users/user[1]/name");  // 첫 번째 user의 name
xml.getNodeValue("//users/user[2]/email");  // 두 번째 user의 email
```

### 조건부 선택

```jsp
// 속성으로 선택
xml.getNodeValue("//user[@id='user001']/name");

// 값으로 선택
xml.getNodeValue("//user[role='admin']/name");
```

---

## 실전 예제

### 1. 설정 파일 읽기

**config.xml**:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<application>
    <database>
        <host>localhost</host>
        <port>3306</port>
        <name>myapp</name>
        <user>dbuser</user>
        <password>dbpass</password>
    </database>
    <email>
        <smtp>smtp.gmail.com</smtp>
        <port>587</port>
        <user>admin@example.com</user>
        <password>emailpass</password>
    </email>
</application>
```

**JSP**:
```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

SimpleParser xml = new SimpleParser(Config.getDocRoot() + "/data/config.xml");

// 데이터베이스 설정 읽기
String dbHost = xml.getNodeValue("//database/host");
String dbPort = xml.getNodeValue("//database/port");
String dbName = xml.getNodeValue("//database/name");
String dbUser = xml.getNodeValue("//database/user");

m.p("DB 접속: " + dbUser + "@" + dbHost + ":" + dbPort + "/" + dbName);

// 이메일 설정 읽기
String smtpHost = xml.getNodeValue("//email/smtp");
String smtpPort = xml.getNodeValue("//email/port");
String emailUser = xml.getNodeValue("//email/user");

m.p("SMTP: " + smtpHost + ":" + smtpPort + " (" + emailUser + ")");

%>
```

### 2. 메뉴 구조 읽기

**menu.xml**:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<menu>
    <item>
        <id>1</id>
        <title>홈</title>
        <url>/main/index.jsp</url>
        <order>1</order>
    </item>
    <item>
        <id>2</id>
        <title>게시판</title>
        <url>/board/list.jsp</url>
        <order>2</order>
    </item>
    <item>
        <id>3</id>
        <title>공지사항</title>
        <url>/notice/list.jsp</url>
        <order>3</order>
    </item>
</menu>
```

**JSP**:
```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

SimpleParser xml = new SimpleParser(Config.getDocRoot() + "/data/menu.xml");
DataSet menu = xml.getDataSet("//menu/item");

p.setBody("main.menu");
p.setLoop("menu", menu);
p.display();

%>
```

**HTML 템플릿**:
```html
<nav>
    <ul>
        <!--@loop(menu)-->
        <li><a href="{{menu.url}}">{{menu.title}}</a></li>
        <!--/loop(menu)-->
    </ul>
</nav>
```

### 3. 다국어 리소스 파일

**messages_ko.xml**:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<messages>
    <message>
        <key>welcome</key>
        <value>환영합니다</value>
    </message>
    <message>
        <key>login</key>
        <value>로그인</value>
    </message>
    <message>
        <key>logout</key>
        <value>로그아웃</value>
    </message>
    <message>
        <key>error.required</key>
        <value>필수 항목입니다</value>
    </message>
</messages>
```

**JSP**:
```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

SimpleParser xml = new SimpleParser(Config.getDocRoot() + "/data/messages_ko.xml");
DataSet messages = xml.getDataSet("//messages/message");

// 메시지 맵 생성
DataMap msgMap = new DataMap();
messages.first();
while(messages.next()) {
    String key = messages.getString("key");
    String value = messages.getString("value");
    msgMap.put(key, value);
}

// 사용
m.p(msgMap.getString("welcome"));  // "환영합니다"
m.p(msgMap.getString("login"));    // "로그인"

%>
```

---

## 디버깅

### setDebug()

파싱 과정을 출력합니다.

```jsp
SimpleParser xml = new SimpleParser(Config.getDocRoot() + "/data/sample.xml");
xml.setDebug(out);  // 디버그 정보 출력

String value = xml.getNodeValue("//config/env/logDir");
```

**출력 내용**:
- XML 파일 경로
- 파싱 결과
- XPath 쿼리 실행 과정

---

## XML 생성

MalgnFramework는 XML 읽기에 집중되어 있지만, 간단한 XML 생성은 문자열로 할 수 있습니다.

### 수동 XML 생성

```jsp
<%@ page contentType="text/xml; charset=utf-8" %><%

StringBuilder xml = new StringBuilder();
xml.append("<?xml version=\"1.0\" encoding=\"UTF-8\"?>\n");
xml.append("<response>\n");
xml.append("  <status>success</status>\n");
xml.append("  <message>처리 완료</message>\n");
xml.append("  <data>\n");
xml.append("    <id>123</id>\n");
xml.append("    <name>").append(Malgn.escapeXml("홍길동")).append("</name>\n");
xml.append("  </data>\n");
xml.append("</response>\n");

out.print(xml.toString());

%>
```

### DataSet을 XML로

```jsp
<%@ page contentType="text/xml; charset=utf-8" %><%@ include file="/init.jsp" %><%

UserDao dao = new UserDao();
DataSet users = dao.query("WHERE status = 1");

StringBuilder xml = new StringBuilder();
xml.append("<?xml version=\"1.0\" encoding=\"UTF-8\"?>\n");
xml.append("<users>\n");

users.first();
while(users.next()) {
    xml.append("  <user>\n");
    xml.append("    <id>").append(users.getInt("id")).append("</id>\n");
    xml.append("    <name>").append(Malgn.escapeXml(users.getString("name"))).append("</name>\n");
    xml.append("    <email>").append(Malgn.escapeXml(users.getString("email"))).append("</email>\n");
    xml.append("  </user>\n");
}

xml.append("</users>\n");
out.print(xml.toString());

%>
```

---

## JSON vs XML

### JSON 사용 (권장)

현대적인 웹 개발에서는 JSON이 더 선호됩니다.

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="/init.jsp" %><%

DataMap map = new DataMap();
map.put("id", 123);
map.put("name", "홍길동");
map.put("email", "hong@example.com");

j.success("조회 성공", map);

%>
```

**출력**:
```json
{
  "result": "success",
  "message": "조회 성공",
  "data": {
    "id": 123,
    "name": "홍길동",
    "email": "hong@example.com"
  }
}
```

### XML 사용 (레거시 시스템 연동)

XML은 주로 레거시 시스템이나 특정 표준(SOAP 등)을 따를 때 사용합니다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<response>
    <result>success</result>
    <message>조회 성공</message>
    <data>
        <id>123</id>
        <name>홍길동</name>
        <email>hong@example.com</email>
    </data>
</response>
```

---

## 베스트 프랙티스

### 1. 설정 파일은 XML보다 properties 사용

```jsp
// config.properties (더 간단)
db.host=localhost
db.port=3306
db.name=myapp

// 읽기
Config config = new Config();
String dbHost = config.getString("db.host");
```

### 2. API 응답은 JSON 사용

```jsp
// XML (X)
<%@ page contentType="text/xml; charset=utf-8" %>
<response><status>success</status></response>

// JSON (O)
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="/init.jsp" %><%
j.success("성공");
%>
```

### 3. XML 사용이 필요한 경우

- 레거시 시스템과 연동
- XML 기반 표준 프로토콜 (SOAP, RSS 등)
- 복잡한 계층 구조의 설정 파일
- 스키마 검증이 필요한 경우

---

## 주요 메소드 정리

### SimpleParser

| 메소드 | 설명 |
|--------|------|
| SimpleParser(filePath) | XML 파일 로드 |
| getNodeValue(xpath) | XPath로 단일 노드 값 가져오기 |
| getDataSet(xpath) | XPath로 여러 노드를 DataSet으로 |
| setDebug(out) | 디버깅 모드 활성화 |

---

## 다음 단계

- [Excel 처리](excel.md)로 이동하여 엑셀 파일 읽기/쓰기 방법을 배워보세요.

---

[← 이전: DataSet 활용](dataset.md) | [목차로 돌아가기](README.md) | [다음: Excel 처리 →](excel.md)
# 10. Excel 처리

[← 목차로 돌아가기](README.md)

---

## Excel 클래스 개요

MalgnFramework는 Excel 파일을 읽고 쓰기 위한 세 가지 클래스를 제공합니다.

### 지원 클래스

| 클래스 | 파일 형식 | 용도 |
|--------|----------|------|
| ExcelReader | .xls (Excel 97-2003) | 읽기 전용 |
| ExcelWriter | .xls (Excel 97-2003) | 쓰기 전용 |
| ExcelX | .xlsx (Excel 2007+) | 읽기/쓰기 |

---

## ExcelReader - Excel 읽기 (.xls)

### 기본 사용법

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

ExcelReader excel = new ExcelReader(Config.getDataDir() + "/sample.xls");

// 시트 선택 (0부터 시작)
excel.setSheet(0);

// DataSet으로 읽기
DataSet data = excel.getDataSet();

// 데이터 출력
data.first();
while(data.next()) {
    m.p(data.getString("col0"));  // 첫 번째 열
    m.p(data.getString("col1"));  // 두 번째 열
    m.p(data.getString("col2"));  // 세 번째 열
}

excel.close();

%>
```

### 여러 시트 읽기

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

ExcelReader excel = new ExcelReader(Config.getDataDir() + "/sample.xls");

int sheetCount = excel.getSheetCount();
m.p("총 " + sheetCount + "개의 시트");

for(int i = 0; i < sheetCount; i++) {
    excel.setSheet(i);
    String sheetName = excel.getSheetName(i);
    m.p("시트 이름: " + sheetName);

    DataSet data = excel.getDataSet();
    m.p("행 개수: " + data.size());
    m.p(data);
}

excel.close();

%>
```

---

## ExcelX - Excel 읽기/쓰기 (.xlsx)

### Excel 읽기

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

ExcelX excel = new ExcelX();

// 파일 읽기
DataSet data = excel.read(Config.getDataDir() + "/sample.xlsx");

// 데이터 검증
data.setCol("col0", "maxlength:12, type:datetime, length:9");
data.setCol("col5", "type:number, length:3");

if(data.validate()) {
    m.p("검증 성공");
} else {
    m.p("검증 실패");
}

// 데이터 출력
data.first();
while(data.next()) {
    m.p(data.getString("col0"));
    m.p(data.getString("col1"));
}

%>
```

### Excel 쓰기 (단일 시트)

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 데이터 준비
DataSet data = new DataSet();
data.addRow();
data.put("col0", "user001");
data.put("col1", "홍길동");
data.put("col2", "hong@example.com");
data.put("col3", "010-1234-5678");

data.addRow();
data.put("col0", "user002");
data.put("col1", "김철수");
data.put("col2", "kim@example.com");
data.put("col3", "010-2345-6789");

// Excel 생성
ExcelX excel = new ExcelX();

// 컬럼 정의 (col이름=>헤더명)
String[] columns = {"col0=>아이디", "col1=>이름", "col2=>이메일", "col3=>전화번호"};

data.first();
excel.setSheet("회원정보");
excel.setData(data, columns);

// 파일로 저장
excel.write(Config.getDataDir() + "/users.xlsx");

// 또는 다운로드로 전송
// excel.write(response, "회원정보.xlsx");

%>
```

### Excel 쓰기 (다중 시트)

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

ExcelX excel = new ExcelX();

// 첫 번째 시트 - 회원 정보
DataSet users = new DataSet();
users.addRow();
users.put("col0", "user001");
users.put("col1", "홍길동");
users.put("col2", "hong@example.com");

users.addRow();
users.put("col0", "user002");
users.put("col1", "김철수");
users.put("col2", "kim@example.com");

String[] userColumns = {"col0=>아이디", "col1=>이름", "col2=>이메일"};
users.first();
excel.setSheet("회원정보");
excel.setData(users, userColumns);

// 두 번째 시트 - 주문 정보
DataSet orders = new DataSet();
orders.addRow();
orders.put("col0", "20250101");
orders.put("col1", "ORD001");
orders.put("col2", "50000");

orders.addRow();
orders.put("col0", "20250102");
orders.put("col1", "ORD002");
orders.put("col2", "35000");

String[] orderColumns = {"col0=>주문일", "col1=>주문번호", "col2=>금액"};
orders.first();
excel.setSheet("주문내역");
excel.setData(orders, orderColumns);

// 다운로드
excel.write(response, "보고서.xlsx");

%>
```

### 비밀번호로 보호

```jsp
ExcelX excel = new ExcelX();
excel.setSheet("민감정보");
excel.setData(data, columns);

// 비밀번호 설정
excel.setPassword("1234");

excel.write(response, "보호문서.xlsx");
```

---

## 실전 예제

### 1. 데이터베이스 조회 결과를 Excel로

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 데이터베이스 조회
UserDao dao = new UserDao();
DataSet users = dao.query("WHERE status = 1 ORDER BY reg_date DESC");

// Excel 생성
ExcelX excel = new ExcelX();

String[] columns = {
    "id=>번호",
    "user_id=>아이디",
    "name=>이름",
    "email=>이메일",
    "phone=>전화번호",
    "reg_date=>가입일"
};

users.first();
excel.setSheet("회원목록");
excel.setData(users, columns);

// 다운로드
excel.write(response, "회원목록_" + m.time("yyyyMMdd") + ".xlsx");

%>
```

### 2. Excel 업로드 및 일괄 처리

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

f.addElement("excel_file", null, "required:Y, accept:'.xlsx'");

if(m.isPost() && f.validate()) {

    String filePath = f.getFileDir("excel_file") + "/" + f.getFileName("excel_file");

    // Excel 읽기
    ExcelX excel = new ExcelX();
    DataSet data = excel.read(filePath);

    // 데이터 검증
    data.setCol("col0", "required:Y, maxlength:50");  // 아이디
    data.setCol("col1", "required:Y");  // 이름
    data.setCol("col2", "type:email");  // 이메일

    if(!data.validate()) {
        m.jsError("Excel 데이터 검증 실패");
        return;
    }

    // 데이터베이스에 일괄 저장
    UserDao dao = new UserDao();
    int successCount = 0;

    data.first();
    while(data.next()) {
        DataMap user = new DataMap();
        user.put("user_id", data.getString("col0"));
        user.put("name", data.getString("col1"));
        user.put("email", data.getString("col2"));
        user.put("reg_date", m.time());

        try {
            dao.insert(user);
            successCount++;
        } catch(Exception e) {
            m.p("오류: " + e.getMessage());
        }
    }

    m.jsAlert(successCount + "명의 회원이 등록되었습니다.");
    m.jsReplace("list.jsp");
}

p.setBody("main.upload_excel");
p.setVar("form_script", f.getScript());
p.display();

%>
```

### 3. 검색 결과를 Excel로 다운로드

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

String mode = m.rs("mode");

if("download".equals(mode)) {

    // 검색 조건으로 데이터 조회
    ListManager lm = new ListManager();
    lm.setRequest(request);
    lm.setTable("tb_blog");
    lm.addSearch("subject,content", f.get("keyword"), "LIKE");
    lm.addSearch("status", f.getInt("status"));

    // 페이징 없이 전체 조회
    lm.setListNum(100000);
    DataSet list = lm.getDataSet();

    // Excel 생성
    ExcelX excel = new ExcelX();

    String[] columns = {
        "id=>번호",
        "subject=>제목",
        "name=>작성자",
        "reg_date=>작성일",
        "hit=>조회수"
    };

    list.first();
    excel.setSheet("게시물목록");
    excel.setData(list, columns);

    excel.write(response, "게시물목록_" + m.time("yyyyMMdd") + ".xlsx");
    return;
}

// 일반 목록 화면
ListManager lm = new ListManager();
lm.setRequest(request);
lm.setTable("tb_blog");
lm.setListNum(20);
lm.addSearch("subject,content", f.get("keyword"), "LIKE");

DataSet list = lm.getDataSet();

p.setBody("main.blog_list");
p.setLoop("list", list);
p.setVar("pager", lm.getPaging());
p.display();

%>
```

**HTML 템플릿에 다운로드 버튼 추가**:
```html
<a href="list.jsp?mode=download&keyword={{keyword}}" class="btn">Excel 다운로드</a>
```

### 4. 월별 통계 Excel 생성

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 월별 집계 데이터 조회
DataObject dao = new DataObject();
DataSet monthly = dao.queryDataSet(
    "SELECT " +
    "  DATE_FORMAT(reg_date, '%Y-%m') AS month, " +
    "  COUNT(*) AS cnt, " +
    "  SUM(amount) AS total " +
    "FROM tb_order " +
    "WHERE reg_date >= '2025-01-01' " +
    "GROUP BY DATE_FORMAT(reg_date, '%Y-%m') " +
    "ORDER BY month DESC"
);

// Excel 생성
ExcelX excel = new ExcelX();

String[] columns = {
    "month=>월",
    "cnt=>주문건수",
    "total=>매출합계"
};

monthly.first();
excel.setSheet("월별통계");
excel.setData(monthly, columns);

excel.write(response, "월별통계_" + m.time("yyyyMMdd") + ".xlsx");

%>
```

---

## 디버깅

### ExcelReader 디버깅

```jsp
ExcelReader excel = new ExcelReader(Config.getDataDir() + "/sample.xls");
excel.setDebug(out);  // 디버그 정보 출력
```

### ExcelX 디버깅

```jsp
ExcelX excel = new ExcelX();
excel.setDebug(out);  // 디버그 정보 출력
```

**출력 내용**:
- 파일 경로
- 시트 정보
- 컬럼 매핑
- 처리 시간

---

## 성능 최적화

### 대용량 파일 처리

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

ExcelReader excel = new ExcelReader(Config.getDataDir() + "/large_file.xls");

long start = System.currentTimeMillis();

excel.setSheet(0);
DataSet data = excel.getDataSet();

long end = System.currentTimeMillis();
m.p("처리 시간: " + (end - start) + "ms");
m.p("행 개수: " + data.size());

excel.close();

%>
```

### 메모리 관리

```jsp
// 사용 후 반드시 닫기
ExcelReader excel = new ExcelReader(filePath);
try {
    DataSet data = excel.getDataSet();
    // 처리...
} finally {
    excel.close();  // 메모리 해제
}
```

---

## 주요 메소드 정리

### ExcelReader (.xls 읽기)

| 메소드 | 설명 |
|--------|------|
| ExcelReader(filePath) | Excel 파일 로드 |
| getSheetCount() | 시트 개수 |
| getSheetName(index) | 시트 이름 |
| setSheet(index) | 시트 선택 (0부터 시작) |
| getDataSet() | 현재 시트를 DataSet으로 |
| setDebug(out) | 디버깅 모드 |
| close() | 리소스 해제 |

### ExcelX (.xlsx 읽기/쓰기)

| 메소드 | 설명 |
|--------|------|
| ExcelX() | 생성자 |
| read(filePath) | Excel 파일 읽기 → DataSet |
| setSheet(name) | 시트 이름 설정 |
| setData(dataSet, columns) | 데이터 및 컬럼 설정 |
| setPassword(password) | 비밀번호 설정 |
| write(filePath) | 파일로 저장 |
| write(response, fileName) | 다운로드로 전송 |
| setDebug(out) | 디버깅 모드 |

### 컬럼 정의 형식

```jsp
String[] columns = {
    "col0=>헤더1",  // col0을 "헤더1"로 표시
    "col1=>헤더2",  // col1을 "헤더2"로 표시
    "col2=>헤더3"   // col2를 "헤더3"로 표시
};
```

---

## 베스트 프랙티스

### 1. 파일 형식 선택

```jsp
// .xls (구버전, 호환성 중요)
ExcelReader excel = new ExcelReader(filePath);

// .xlsx (신버전, 더 많은 기능)
ExcelX excel = new ExcelX();
```

### 2. 대용량 처리

```jsp
// Bad - 메모리 부족 가능
DataSet data = excel.read("large_file.xlsx");

// Good - 배치 처리
int batchSize = 1000;
for(int i = 0; i < totalRows; i += batchSize) {
    // 일부씩 처리
}
```

### 3. 에러 처리

```jsp
try {
    ExcelX excel = new ExcelX();
    DataSet data = excel.read(filePath);

    if(data.size() == 0) {
        throw new Exception("데이터가 없습니다");
    }

    // 처리...

} catch(Exception e) {
    m.jsError("Excel 처리 중 오류: " + e.getMessage());
}
```

### 4. 사용자 파일 업로드 검증

```jsp
// 파일 확장자 검증
f.addElement("excel_file", null, "required:Y, accept:'.xlsx,.xls'");

if(m.isPost() && f.validate()) {

    String fileName = f.getFileName("excel_file");
    String ext = fileName.substring(fileName.lastIndexOf("."));

    if(!".xlsx".equalsIgnoreCase(ext) && !".xls".equalsIgnoreCase(ext)) {
        m.jsError("Excel 파일만 업로드 가능합니다");
        return;
    }

    // 처리...
}
```

---

## 다음 단계

- [인증 처리](authentication.md)로 이동하여 로그인/로그아웃 구현 방법을 배워보세요.

---

[← 이전: XML 처리](xml.md) | [목차로 돌아가기](README.md) | [다음: 인증 처리 →](authentication.md)
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

    DataMap user = new DataMap();
    user.put("user_id", userId);
    user.put("password", hashedPassword);
    user.put("name", f.get("name"));
    user.put("reg_date", m.time());

    UserDao dao = new UserDao();
    dao.insert(user);

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
    UserDao dao = new UserDao();
    DataSet user = dao.query("WHERE user_id = ? AND password = ?", userId, hashedPassword);

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

    DataMap user = new DataMap();
    user.put("name", f.get("name"));
    user.put("ssn", encryptedSsn);
    user.put("phone", encryptedPhone);
    user.put("email", f.get("email"));

    UserDao dao = new UserDao();
    dao.insert(user);

    m.jsAlert("저장되었습니다");
    m.jsReplace("list.jsp");

    return;
}

%>
```

**조회 시 복호화**:
```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

UserDao dao = new UserDao();
DataSet users = dao.query("WHERE status = 1");

String secretKey = Config.get("personalDataKey");
AES aes = new AES(secretKey);

// 복호화
users.first();
while(users.next()) {
    String encryptedSsn = users.getString("ssn");
    String encryptedPhone = users.getString("phone");

    String ssn = aes.decrypt(encryptedSsn);
    String phone = aes.decrypt(encryptedPhone);

    // 마스킹하여 표시
    String maskedSsn = ssn.substring(0, 6) + "-*******";
    String maskedPhone = phone.substring(0, 3) + "-****-" + phone.substring(9);

    users.put("ssn_masked", maskedSsn);
    users.put("phone_masked", maskedPhone);
}

p.setLoop("users", users);
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
# 11. 인증 처리

[← 목차로 돌아가기](README.md)

---

## 인증 시스템 개요

MalgnFramework는 세션 및 쿠키 기반의 간단한 인증 시스템을 제공합니다.

### 주요 클래스

- **Auth 클래스**: 인증 정보 저장/조회
- **Malgn 클래스 (m)**: 세션/쿠키 관리 메소드

---

## Auth 클래스

### 기본 개념

Auth 클래스는 로그인 사용자 정보를 세션에 저장하고 관리합니다.

```jsp
// init.jsp에서 자동 생성됨
Auth auth = new Auth(request, response);
```

### 주요 메소드

| 메소드 | 설명 |
|--------|------|
| put(key, value) | 인증 정보 추가 |
| save() | 세션에 저장 |
| get(key) | 값 가져오기 |
| getInt(key) | 정수 값 가져오기 |
| getString(key) | 문자열 값 가져오기 |
| clear() | 인증 정보 삭제 (로그아웃) |
| isLogin() | 로그인 여부 확인 |

---

## 로그인 구현

### 기본 로그인 예제

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

f.addElement("id", null, "title:'아이디', required:'Y'");
f.addElement("passwd", null, "title:'비밀번호', required:'Y'");

if(m.isPost() && f.validate()) {

    String id = f.get("id");
    String passwd = f.get("passwd");

    // 데이터베이스에서 사용자 확인
    UserDao dao = new UserDao();
    DataSet user = dao.query("WHERE user_id = ? AND passwd = ?", id, Malgn.sha256(passwd));

    if(user.next()) {
        // 로그인 성공 - 인증 정보 저장
        auth.put("user_id", user.getInt("id"));
        auth.put("user_name", user.getString("name"));
        auth.put("user_level", user.getInt("level"));
        auth.save();

        m.jsAlert("로그인 성공");
        m.jsReplace("/main/index.jsp");
    } else {
        // 로그인 실패
        m.jsError("아이디 또는 비밀번호가 올바르지 않습니다.");
    }
    return;
}

p.setBody("main.login");
p.setVar("form_script", f.getScript());
p.display();

%>
```

### HTML 템플릿 (login.vm)

```html
<h2>로그인</h2>

<form name="form1" method="post">
    <div>
        <label>아이디</label>
        <input type="text" name="id" placeholder="아이디를 입력하세요">
    </div>
    <div>
        <label>비밀번호</label>
        <input type="password" name="passwd" placeholder="비밀번호를 입력하세요">
    </div>
    <button type="submit">로그인</button>
</form>

{{form_script}}
```

---

## 로그아웃 구현

### 기본 로그아웃

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 인증 정보 삭제
auth.clear();

m.jsAlert("로그아웃되었습니다.");
m.jsReplace("/main/login.jsp");

%>
```

---

## 로그인 체크

### 페이지별 로그인 체크

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 로그인 체크
if(!auth.isLogin()) {
    m.jsAlert("로그인이 필요합니다.");
    m.jsReplace("/main/login.jsp");
    return;
}

// 로그인한 사용자 정보
int userId = auth.getInt("user_id");
String userName = auth.getString("user_name");

m.p("로그인 사용자: " + userName + " (ID: " + userId + ")");

%>
```

### 권한 레벨 체크

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 로그인 체크
if(!auth.isLogin()) {
    m.jsAlert("로그인이 필요합니다.");
    m.jsReplace("/main/login.jsp");
    return;
}

// 관리자 권한 체크
int userLevel = auth.getInt("user_level");
if(userLevel < 10) {
    m.jsAlert("관리자만 접근할 수 있습니다.");
    m.jsBack();
    return;
}

// 관리자 페이지 처리
p.setBody("admin.dashboard");
p.display();

%>
```

---

## 세션 관리

### 세션 값 설정

```jsp
// 세션에 값 저장
m.setSession("user_name", "홍길동");
m.setSession("user_level", 5);
```

### 세션 값 가져오기

```jsp
// 문자열로 가져오기
String userName = m.getSession("user_name");

// 정수로 가져오기
int userLevel = m.getSessionInt("user_level");

// null 체크
String value = m.getSession("key");
if(value != null) {
    // 처리
}
```

### 세션 값 삭제

```jsp
// 특정 세션 값 삭제
m.removeSession("user_name");

// 모든 세션 삭제
session.invalidate();
```

---

## 쿠키 관리

### 쿠키 설정

```jsp
// 기본 쿠키 (브라우저 종료 시 삭제)
m.setCookie("remember_id", "user001");

// 유효기간이 있는 쿠키 (7일)
m.setCookie("remember_id", "user001", 60 * 60 * 24 * 7);
```

### 쿠키 읽기

```jsp
String rememberId = m.getCookie("remember_id");
if(rememberId != null) {
    m.p("저장된 아이디: " + rememberId);
}
```

### 쿠키 삭제

```jsp
// 유효기간을 0으로 설정하여 삭제
m.setCookie("remember_id", "", 0);
```

---

## 실전 예제

### 1. 아이디 저장 기능이 있는 로그인

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

f.addElement("id", null, "required:Y");
f.addElement("passwd", null, "required:Y");
f.addElement("remember_id", null, null);

if(m.isPost() && f.validate()) {

    String id = f.get("id");
    String passwd = f.get("passwd");
    String rememberId = f.get("remember_id");

    // 사용자 확인
    UserDao dao = new UserDao();
    DataSet user = dao.query("WHERE user_id = ? AND passwd = ?", id, Malgn.sha256(passwd));

    if(user.next()) {
        // 로그인 성공
        auth.put("user_id", user.getInt("id"));
        auth.put("user_name", user.getString("name"));
        auth.save();

        // 아이디 저장 체크박스 처리
        if("Y".equals(rememberId)) {
            m.setCookie("remember_id", id, 60 * 60 * 24 * 30);  // 30일
        } else {
            m.setCookie("remember_id", "", 0);  // 삭제
        }

        m.jsReplace("/main/index.jsp");
    } else {
        m.jsError("로그인 실패");
    }
    return;
}

// 저장된 아이디 불러오기
String savedId = m.getCookie("remember_id");

p.setBody("main.login");
p.setVar("saved_id", savedId != null ? savedId : "");
p.setVar("form_script", f.getScript());
p.display();

%>
```

**HTML**:
```html
<form name="form1" method="post">
    <input type="text" name="id" value="{{saved_id}}">
    <input type="password" name="passwd">
    <label>
        <input type="checkbox" name="remember_id" value="Y"> 아이디 저장
    </label>
    <button type="submit">로그인</button>
</form>
```

### 2. 자동 로그인 기능

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 자동 로그인 체크
String autoLoginToken = m.getCookie("auto_login_token");
if(autoLoginToken != null && !auth.isLogin()) {

    // 토큰으로 사용자 찾기
    UserDao dao = new UserDao();
    DataSet user = dao.query("WHERE auto_login_token = ?", autoLoginToken);

    if(user.next()) {
        // 자동 로그인 처리
        auth.put("user_id", user.getInt("id"));
        auth.put("user_name", user.getString("name"));
        auth.save();
    }
}

// 로그인 폼 처리
if(m.isPost() && f.validate()) {

    String id = f.get("id");
    String passwd = f.get("passwd");
    String autoLogin = f.get("auto_login");

    UserDao dao = new UserDao();
    DataSet user = dao.query("WHERE user_id = ? AND passwd = ?", id, Malgn.sha256(passwd));

    if(user.next()) {
        auth.put("user_id", user.getInt("id"));
        auth.put("user_name", user.getString("name"));
        auth.save();

        // 자동 로그인 체크박스 처리
        if("Y".equals(autoLogin)) {
            // 고유 토큰 생성
            String token = Malgn.uuid();

            // DB에 토큰 저장
            DataMap data = new DataMap();
            data.put("auto_login_token", token);
            dao.update("id = ?", data, user.getInt("id"));

            // 쿠키에 저장 (30일)
            m.setCookie("auto_login_token", token, 60 * 60 * 24 * 30);
        }

        m.jsReplace("/main/index.jsp");
    }
}

%>
```

### 3. 마지막 로그인 시간 기록

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

if(m.isPost() && f.validate()) {

    String id = f.get("id");
    String passwd = f.get("passwd");

    UserDao dao = new UserDao();
    DataSet user = dao.query("WHERE user_id = ? AND passwd = ?", id, Malgn.sha256(passwd));

    if(user.next()) {
        int userId = user.getInt("id");

        // 마지막 로그인 시간 업데이트
        DataMap data = new DataMap();
        data.put("last_login", m.time());
        data.put("login_count", user.getInt("login_count") + 1);
        dao.update("id = ?", data, userId);

        // 인증 정보 저장
        auth.put("user_id", userId);
        auth.put("user_name", user.getString("name"));
        auth.put("last_login", m.time());
        auth.save();

        m.jsReplace("/main/index.jsp");
    } else {
        m.jsError("로그인 실패");
    }
}

%>
```

### 4. init.jsp에 전역 인증 변수 설정

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ page import="malgnsoft.*" %><%@ page import="malgnsoft.db.*" %><%@ page import="malgnsoft.util.*" %><%

Malgn m = new Malgn(request, response);
Form f = new Form(request);
Page p = new Page(request, response);
Json j = new Json(request, response);
Auth auth = new Auth(request, response);

// 전역 변수로 사용자 ID 설정
int userId = auth.getInt("user_id");
String userName = auth.getString("user_name");

// 템플릿에서 사용할 수 있도록 설정
p.setVar("isLogin", auth.isLogin());
p.setVar("userId", userId);
p.setVar("userName", userName);

%>
```

**HTML 템플릿에서 사용**:
```html
<!--@if(isLogin)-->
<div class="user-info">
    <span>{{userName}}님 환영합니다</span>
    <a href="/main/logout.jsp">로그아웃</a>
</div>
<!--@else-->
<div class="login-buttons">
    <a href="/main/login.jsp">로그인</a>
    <a href="/main/join.jsp">회원가입</a>
</div>
<!--/if(isLogin)-->
```

### 5. API 인증

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="/init.jsp" %><%

// API 토큰 인증
String apiToken = m.rq("token");

if(apiToken == null || apiToken.isEmpty()) {
    j.error(401, "인증 토큰이 필요합니다");
    return;
}

// 토큰 검증
UserDao dao = new UserDao();
DataSet user = dao.query("WHERE api_token = ? AND status = 1", apiToken);

if(!user.next()) {
    j.error(401, "유효하지 않은 토큰입니다");
    return;
}

// API 처리
DataMap result = new DataMap();
result.put("user_id", user.getInt("id"));
result.put("user_name", user.getString("name"));

j.success("인증 성공", result);

%>
```

---

## 비밀번호 암호화

### SHA-256 해시

```jsp
// 회원가입 시 비밀번호 암호화
String plainPassword = f.get("passwd");
String hashedPassword = Malgn.sha256(plainPassword);

DataMap user = new DataMap();
user.put("user_id", f.get("id"));
user.put("passwd", hashedPassword);
user.put("name", f.get("name"));

UserDao dao = new UserDao();
dao.insert(user);
```

### 다양한 해시 알고리즘

```jsp
// MD5 (권장하지 않음)
String md5 = Malgn.md5("password");

// SHA-1
String sha1 = Malgn.sha1("password");

// SHA-256 (권장)
String sha256 = Malgn.sha256("password");

// SHA-512
String sha512 = Malgn.sha512("password");
```

---

## 세션 타임아웃

### web.xml에서 설정

```xml
<session-config>
    <session-timeout>30</session-timeout>  <!-- 30분 -->
</session-config>
```

### JSP에서 동적 설정

```jsp
// 세션 유지 시간 설정 (초 단위)
session.setMaxInactiveInterval(60 * 30);  // 30분
```

---

## 보안 고려사항

### 1. HTTPS 사용

프로덕션 환경에서는 반드시 HTTPS를 사용하세요.

### 2. 비밀번호 정책

```jsp
// 비밀번호 강도 체크
f.addElement("passwd", null, "required:Y, minlength:8, maxlength:20");

// 비밀번호 확인
f.addElement("passwd_confirm", null, "required:Y, equalTo:'passwd'");
```

### 3. SQL Injection 방지

```jsp
// Bad - SQL Injection 위험
String sql = "SELECT * FROM tb_user WHERE user_id = '" + id + "'";

// Good - Prepared Statement 사용
DataSet user = dao.query("WHERE user_id = ?", id);
```

### 4. XSS 방지

```jsp
// 출력 시 HTML 이스케이프
String safeName = Malgn.htt(userName);
m.p(safeName);
```

---

## 주요 메소드 정리

### Auth 클래스

| 메소드 | 설명 |
|--------|------|
| put(key, value) | 인증 정보 추가 |
| save() | 세션에 저장 |
| get(key) | 값 가져오기 |
| getInt(key) | 정수로 가져오기 |
| getString(key) | 문자열로 가져오기 |
| clear() | 인증 정보 삭제 |
| isLogin() | 로그인 여부 |

### 세션/쿠키 메소드 (Malgn)

| 메소드 | 설명 |
|--------|------|
| setSession(key, value) | 세션 값 설정 |
| getSession(key) | 세션 값 가져오기 |
| getSessionInt(key) | 세션 정수 가져오기 |
| removeSession(key) | 세션 값 삭제 |
| setCookie(name, value) | 쿠키 설정 |
| setCookie(name, value, age) | 쿠키 설정 (유효기간) |
| getCookie(name) | 쿠키 값 가져오기 |

---

## 다음 단계

- [환경 설정](configuration.md)로 이동하여 Config 클래스 사용법을 배워보세요.

---

[← 이전: Excel 처리](excel.md) | [목차로 돌아가기](README.md) | [다음: 환경 설정 →](configuration.md)
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
# HTTP 클라이언트

[← 목차로 돌아가기](README.md)

---

## Http 클래스 개요

Http 클래스는 외부 API를 호출하거나 HTTP 요청을 보내기 위한 클라이언트 클래스입니다.

### 주요 기능

- GET, POST, PUT, DELETE 요청
- 헤더 설정
- 파라미터 전송
- JSON/XML 데이터 전송
- SSL 인증서 검증 비활성화
- 타임아웃 설정
- 비동기 요청 지원

---

## 기본 사용법

### GET 요청

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

Http http = new Http("https://api.example.com/users");
http.setParam("page", "1");
http.setParam("limit", "10");

String response = http.send("GET");
m.p(response);

%>
```

### POST 요청

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

Http http = new Http("https://api.example.com/users");
http.setParam("name", "홍길동");
http.setParam("email", "hong@example.com");
http.setParam("age", "30");

String response = http.send("POST");
m.p(response);

%>
```

---

## 헤더 설정

### 커스텀 헤더 추가

```jsp
Http http = new Http("https://api.example.com/data");

// Authorization 헤더
http.setHeader("Authorization", "Bearer " + apiToken);

// Content-Type 헤더
http.setHeader("Content-Type", "application/json");

// 커스텀 헤더
http.setHeader("X-API-Key", "your_api_key");
http.setHeader("X-Request-ID", Malgn.uuid());

String response = http.send("GET");
```

---

## JSON 데이터 전송

### JSON 요청

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

Http http = new Http("https://api.example.com/users");

// JSON 데이터 생성
JSONObject json = new JSONObject();
json.put("name", "홍길동");
json.put("email", "hong@example.com");
json.put("age", 30);

// JSON 데이터 설정 (자동으로 Content-Type: application/json 설정됨)
http.setData(json.toString());

String response = http.send("POST");
m.p(response);

%>
```

### JSON 응답 파싱

```jsp
Http http = new Http("https://api.example.com/users/123");
String response = http.send("GET");

// JSON 파싱
JSONObject json = new JSONObject(response);
String name = json.getString("name");
String email = json.getString("email");
int age = json.getInt("age");

m.p("이름: " + name);
m.p("이메일: " + email);
m.p("나이: " + age);
```

---

## XML 데이터 전송

```jsp
Http http = new Http("https://api.example.com/data");

String xmlData = "<?xml version=\"1.0\" encoding=\"UTF-8\"?>" +
                "<user>" +
                "  <name>홍길동</name>" +
                "  <email>hong@example.com</email>" +
                "</user>";

// XML 데이터 설정 (자동으로 Content-Type: application/xml 설정됨)
http.setData(xmlData);

String response = http.send("POST");
```

---

## 타임아웃 설정

```jsp
Http http = new Http("https://api.example.com/slow-endpoint");

// 타임아웃 설정 (초 단위)
http.setTimeout(30);  // 30초

try {
    String response = http.send("GET");
    m.p(response);
} catch(Exception e) {
    m.p("요청 시간 초과");
}
```

---

## SSL 인증서 검증

### SSL 검증 비활성화 (개발 환경)

```jsp
Http http = new Http("https://self-signed-cert.example.com");

// SSL 인증서 검증 비활성화
http.disableSSLVerification();

String response = http.send("GET");

// 검증 복원
http.restoreSSLVerification();
```

**주의**: 운영 환경에서는 SSL 검증을 비활성화하지 마세요!

---

## 응답 코드 확인

```jsp
Http http = new Http("https://api.example.com/users");
String response = http.send("GET");

int statusCode = http.getResponseCode();

if(statusCode == 200) {
    m.p("성공: " + response);
} else if(statusCode == 404) {
    m.p("리소스를 찾을 수 없습니다");
} else if(statusCode >= 500) {
    m.p("서버 오류");
} else {
    m.p("오류 코드: " + statusCode);
}
```

---

## 비동기 요청

```jsp
Http http = new Http("https://api.example.com/process");
http.setParam("task", "heavy_task");

// 비동기 요청 (백그라운드에서 실행)
http.send("POST", new HttpListener() {
    @Override
    public void execute(String result) {
        // 요청 완료 후 실행
        Malgn.logger("http", "완료: " + result);
    }
});

// 즉시 다음 코드 실행
m.p("요청이 백그라운드에서 실행 중입니다");
```

---

## 실전 예제

### 1. REST API 호출

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// GitHub API 호출
Http http = new Http("https://api.github.com/users/octocat");
http.setHeader("User-Agent", "MalgnFramework");

String response = http.send("GET");

JSONObject user = new JSONObject(response);
m.p("사용자명: " + user.getString("login"));
m.p("이름: " + user.getString("name"));
m.p("팔로워: " + user.getInt("followers"));

%>
```

### 2. 날씨 API 호출

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

String apiKey = Config.get("weatherApiKey");
String city = "Seoul";

Http http = new Http("https://api.openweathermap.org/data/2.5/weather");
http.setParam("q", city);
http.setParam("appid", apiKey);
http.setParam("units", "metric");

String response = http.send("GET");

JSONObject weather = new JSONObject(response);
double temp = weather.getJSONObject("main").getDouble("temp");
String description = weather.getJSONArray("weather").getJSONObject(0).getString("description");

m.p("서울 날씨: " + description);
m.p("온도: " + temp + "°C");

%>
```

### 3. 슬랙(Slack) 웹훅

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

String webhookUrl = Config.get("slackWebhookUrl");

JSONObject payload = new JSONObject();
payload.put("text", "새로운 주문이 접수되었습니다!");
payload.put("username", "알림봇");
payload.put("icon_emoji", ":bell:");

Http http = new Http(webhookUrl);
http.setData(payload.toString());

String response = http.send("POST");
m.p("슬랙 전송: " + response);

%>
```

### 4. 결제 API 연동

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

String apiUrl = "https://api.payment.com/v1/payments";
String apiKey = Config.get("paymentApiKey");

// 결제 요청 데이터
JSONObject payment = new JSONObject();
payment.put("amount", 50000);
payment.put("currency", "KRW");
payment.put("order_id", "ORD" + m.time());
payment.put("customer_name", "홍길동");
payment.put("customer_email", "hong@example.com");

Http http = new Http(apiUrl);
http.setHeader("Authorization", "Bearer " + apiKey);
http.setData(payment.toString());

String response = http.send("POST");

JSONObject result = new JSONObject(response);
if("success".equals(result.getString("status"))) {
    String paymentId = result.getString("payment_id");
    m.p("결제 성공: " + paymentId);
} else {
    m.p("결제 실패: " + result.getString("message"));
}

%>
```

### 5. 파일 다운로드

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

Http http = new Http("https://example.com/files/document.pdf");
String content = http.send("GET");

// 파일로 저장
String filePath = Config.getDataDir() + "/downloaded_document.pdf";
Malgn.writeFile(filePath, content);

m.p("파일 다운로드 완료: " + filePath);

%>
```

### 6. POST with Form Data

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

Http http = new Http("https://api.example.com/form");
http.setHeader("Content-Type", "application/x-www-form-urlencoded");
http.setParam("username", "hong");
http.setParam("password", "secret123");
http.setParam("remember", "true");

String response = http.send("POST");
m.p(response);

%>
```

### 7. PUT 요청 (업데이트)

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

int userId = 123;

JSONObject updateData = new JSONObject();
updateData.put("name", "김철수");
updateData.put("email", "kim@example.com");

Http http = new Http("https://api.example.com/users/" + userId);
http.setHeader("Authorization", "Bearer " + apiToken);
http.setData(updateData.toString());

String response = http.send("PUT");
m.p("업데이트 완료: " + response);

%>
```

### 8. DELETE 요청

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

int userId = 123;

Http http = new Http("https://api.example.com/users/" + userId);
http.setHeader("Authorization", "Bearer " + apiToken);

String response = http.send("DELETE");

if(http.getResponseCode() == 204) {
    m.p("삭제 성공");
} else {
    m.p("삭제 실패: " + response);
}

%>
```

---

## 디버깅

```jsp
Http http = new Http("https://api.example.com/data");
http.setDebug(out);  // 디버그 정보 출력

http.setHeader("X-API-Key", "test123");
http.setParam("query", "search_term");

String response = http.send("GET");
```

**디버그 출력 내용**:
- 요청 URL
- 요청 헤더
- 요청 파라미터
- 응답 코드
- 응답 내용

---

## 주요 메소드 정리

| 메소드 | 설명 |
|--------|------|
| Http(url) | HTTP 클라이언트 생성 |
| setUrl(url) | URL 설정 |
| setHeader(key, value) | 헤더 추가 |
| setParam(name, value) | 파라미터 추가 |
| setData(data) | Body 데이터 설정 (JSON/XML) |
| setTimeout(seconds) | 타임아웃 설정 (초) |
| send() | GET 요청 실행 |
| send(method) | 지정한 메소드로 요청 실행 |
| send(method, listener) | 비동기 요청 실행 |
| getResponseCode() | HTTP 응답 코드 |
| disableSSLVerification() | SSL 검증 비활성화 |
| restoreSSLVerification() | SSL 검증 복원 |
| setDebug(out) | 디버깅 모드 |

---

## 베스트 프랙티스

### 1. API 키는 설정 파일에

```jsp
// Bad
String apiKey = "sk-1234567890abcdef";

// Good
String apiKey = Config.get("apiKey");
```

### 2. 에러 처리

```jsp
try {
    Http http = new Http(apiUrl);
    http.setTimeout(10);
    String response = http.send("GET");

    int statusCode = http.getResponseCode();
    if(statusCode >= 400) {
        throw new Exception("API 오류: " + statusCode);
    }

    // 처리...

} catch(Exception e) {
    Malgn.errorLog("API 호출 실패", e);
    m.jsError("일시적인 오류가 발생했습니다");
}
```

### 3. 타임아웃 설정

```jsp
// 외부 API는 항상 타임아웃 설정
Http http = new Http(externalApi);
http.setTimeout(30);  // 30초
```

### 4. 응답 캐싱

```jsp
Cache cache = new Cache();
String cacheKey = "api_data_" + param;
String response = cache.getString(cacheKey);

if(response == null) {
    Http http = new Http(apiUrl);
    response = http.send("GET");
    cache.save(cacheKey, response);
}
```

---

[← 목차로 돌아가기](README.md)
# 이메일 발송

[← 목차로 돌아가기](README.md)

---

## 개요

맑은프레임워크는 Gmail 클래스를 통해 간편하게 이메일을 발송할 수 있습니다. Gmail, 네이버 메일, AWS SES 등 다양한 SMTP 서버를 지원합니다.

### 주요 기능

- **다양한 SMTP 지원**: Gmail, 네이버, AWS SES, 자체 SMTP 서버
- **파일 첨부**: 여러 파일 첨부 가능
- **HTML 이메일**: HTML 형식의 이메일 본문 지원
- **여러 수신자**: 한 번에 여러 명에게 발송
- **간편한 설정**: Config를 통한 기본 설정

---

## 목차

1. [기본 이메일 발송](#기본-이메일-발송)
2. [Gmail 계정 사용](#gmail-계정-사용)
3. [네이버 메일 사용](#네이버-메일-사용)
4. [AWS SES 사용](#aws-ses-사용)
5. [파일 첨부](#파일-첨부)
6. [HTML 이메일](#html-이메일)
7. [여러 수신자](#여러-수신자)
8. [환경설정](#환경설정)

---

## 기본 이메일 발송

### 1. Malgn 클래스 사용 (가장 간단)

Config에 설정된 기본 메일 서버를 사용합니다:

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 가장 간단한 방법
m.mail("user@example.com", "제목", "내용");

%>
```

### 2. Gmail 클래스 사용 (기본 설정)

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

Gmail mail = new Gmail();
mail.send("user@example.com", "안녕하세요", "테스트 메일입니다.");

%>
```

---

## Gmail 계정 사용

Gmail SMTP를 통해 이메일을 발송할 수 있습니다.

### 1. Gmail 앱 비밀번호 발급

1. Google 계정 설정 → 보안
2. 2단계 인증 활성화
3. 앱 비밀번호 생성 (16자리)

### 2. Gmail로 발송

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// Gmail 계정 정보 설정
Gmail mail = new Gmail("your-email@gmail.com", "앱비밀번호");

// 이메일 발송
mail.send("recipient@example.com", "제목입니다", "본문 내용입니다.");

%>
```

### 3. 전체 예제

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

f.addElement("email", null, "required:'Y', type:'email'");
f.addElement("subject", null, "required:'Y'");
f.addElement("message", null, "required:'Y'");

if(m.isPost() && f.validate()) {

    String toEmail = f.get("email");
    String subject = f.get("subject");
    String message = f.get("message");

    try {
        Gmail mail = new Gmail("your-email@gmail.com", "앱비밀번호");
        mail.send(toEmail, subject, message);

        m.jsAlert("이메일이 발송되었습니다.");
        m.jsReplace("list.jsp");
    } catch(Exception e) {
        m.jsError("이메일 발송 실패: " + e.getMessage());
    }
    return;
}

%>
<form name="form1" method="post">
    <p>받는 사람 : <input type="text" name="email"></p>
    <p>제목 : <input type="text" name="subject"></p>
    <p>내용 : <textarea name="message" rows="10"></textarea></p>
    <p><input type="submit" value="발송"></p>
</form>
<%= f.getScript() %>
```

---

## 네이버 메일 사용

네이버 메일 SMTP를 통해 발송할 수 있습니다.

### 1. 네이버 메일 설정

```jsp
Gmail mail = new Gmail("smtp.naver.com", "네이버아이디", "비밀번호");
mail.setMailFrom("네이버아이디@naver.com");
mail.send("recipient@example.com", "제목", "내용");
```

### 2. 전체 예제

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

String naverId = "your_naver_id";
String naverPw = "your_password";

Gmail mail = new Gmail("smtp.naver.com", naverId, naverPw);
mail.setMailFrom(naverId + "@naver.com");

mail.send("recipient@example.com", "네이버 메일 테스트", "안녕하세요. 네이버 메일로 발송합니다.");

m.p("이메일이 발송되었습니다.");

%>
```

---

## AWS SES 사용

Amazon Simple Email Service를 통해 대량 이메일을 안정적으로 발송할 수 있습니다.

### 1. AWS SES 설정

1. AWS SES 콘솔에서 이메일 주소 인증
2. SMTP 자격 증명 생성
3. 리전별 SMTP 엔드포인트 확인

### 2. AWS SES로 발송

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

String sesHost = "email-smtp.ap-northeast-2.amazonaws.com";  // 서울 리전
String sesUsername = "AKIAXXXXXXXXXXXXXXXX";  // SMTP 사용자 이름
String sesPassword = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx";  // SMTP 비밀번호

Gmail mail = new Gmail(sesHost, sesUsername, sesPassword);
mail.setMailFrom("verified@yourdomain.com");  // SES에서 인증된 이메일

mail.send("recipient@example.com", "AWS SES 테스트", "AWS SES를 통해 발송합니다.");

m.p("이메일이 발송되었습니다.");

%>
```

### 3. 주요 리전별 엔드포인트

```jsp
// 서울 (ap-northeast-2)
String sesHost = "email-smtp.ap-northeast-2.amazonaws.com";

// 미국 동부 (us-east-1)
String sesHost = "email-smtp.us-east-1.amazonaws.com";

// 미국 서부 (us-west-2)
String sesHost = "email-smtp.us-west-2.amazonaws.com";

// 유럽 (eu-west-1)
String sesHost = "email-smtp.eu-west-1.amazonaws.com";
```

---

## 파일 첨부

### 1. 단일 파일 첨부

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

String filePath = Config.getDataDir() + "/file/document.pdf";

Gmail mail = new Gmail("your-email@gmail.com", "앱비밀번호");
mail.send(
    "recipient@example.com",
    "파일 첨부 메일",
    "첨부 파일을 확인해주세요.",
    new String[] { filePath }
);

m.p("첨부 파일과 함께 발송되었습니다.");

%>
```

### 2. 여러 파일 첨부

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

String[] files = {
    Config.getDataDir() + "/file/document1.pdf",
    Config.getDataDir() + "/file/document2.pdf",
    Config.getDataDir() + "/file/image.jpg"
};

Gmail mail = new Gmail("your-email@gmail.com", "앱비밀번호");
mail.send("recipient@example.com", "여러 파일 첨부", "파일들을 확인해주세요.", files);

m.p("여러 파일과 함께 발송되었습니다.");

%>
```

### 3. 업로드된 파일 첨부

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

f.addElement("email", null, "required:'Y', type:'email'");
f.addElement("subject", null, "required:'Y'");
f.addElement("message", null, "required:'Y'");
f.addElement("file", null, "");

if(m.isPost() && f.validate()) {

    String toEmail = f.get("email");
    String subject = f.get("subject");
    String message = f.get("message");

    // 파일 업로드
    File uploadedFile = f.saveFile("file");
    String[] files = null;

    if(uploadedFile != null) {
        files = new String[] { uploadedFile.getAbsolutePath() };
    }

    // 이메일 발송
    Gmail mail = new Gmail("your-email@gmail.com", "앱비밀번호");
    mail.send(toEmail, subject, message, files);

    m.jsAlert("이메일이 발송되었습니다.");
    m.jsReplace("list.jsp");
    return;
}

%>
<form name="form1" method="post" enctype="multipart/form-data">
    <p>받는 사람 : <input type="text" name="email"></p>
    <p>제목 : <input type="text" name="subject"></p>
    <p>내용 : <textarea name="message" rows="10"></textarea></p>
    <p>첨부 파일 : <input type="file" name="file"></p>
    <p><input type="submit" value="발송"></p>
</form>
<%= f.getScript() %>
```

---

## HTML 이메일

HTML 형식의 이메일을 발송할 수 있습니다.

### 1. HTML 본문

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

String htmlBody = "<html>" +
    "<body>" +
    "<h1 style='color: #333;'>안녕하세요!</h1>" +
    "<p>이것은 <strong>HTML</strong> 이메일입니다.</p>" +
    "<p style='color: #666;'>HTML 태그를 사용할 수 있습니다.</p>" +
    "<hr>" +
    "<p><a href='https://example.com' style='color: #007bff;'>링크 보기</a></p>" +
    "</body>" +
    "</html>";

Gmail mail = new Gmail("your-email@gmail.com", "앱비밀번호");
mail.send("recipient@example.com", "HTML 이메일", htmlBody);

m.p("HTML 이메일이 발송되었습니다.");

%>
```

### 2. 템플릿을 사용한 HTML 이메일

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 사용자 정보 조회
int userId = m.ri("user_id");
UserDao dao = new UserDao();
DataSet user = dao.find("id = ?", userId);

if(user.next()) {

    // 템플릿 로드
    Page emailTemplate = new Page(request, response, session, application);
    emailTemplate.setBody("mail.welcome");
    emailTemplate.setVar("name", user.s("name"));
    emailTemplate.setVar("email", user.s("email"));
    emailTemplate.setVar("site_url", Config.getSiteUrl());

    String htmlBody = emailTemplate.fetch();

    // 이메일 발송
    Gmail mail = new Gmail();
    mail.send(user.s("email"), "회원가입을 환영합니다", htmlBody);

    m.p("회원가입 환영 이메일이 발송되었습니다.");
}

%>
```

**html/mail/welcome.html**:
```html
<html>
<head>
    <style>
        body { font-family: Arial, sans-serif; line-height: 1.6; color: #333; }
        .container { max-width: 600px; margin: 0 auto; padding: 20px; }
        .header { background-color: #007bff; color: white; padding: 20px; text-align: center; }
        .content { padding: 20px; background-color: #f9f9f9; }
        .button { display: inline-block; padding: 10px 20px; background-color: #007bff; color: white; text-decoration: none; border-radius: 5px; }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>회원가입을 환영합니다!</h1>
        </div>
        <div class="content">
            <p><strong>{{name}}</strong>님, 안녕하세요!</p>
            <p>회원가입이 완료되었습니다.</p>
            <p>이메일: {{email}}</p>
            <p style="text-align: center; margin-top: 30px;">
                <a href="{{site_url}}" class="button">사이트 방문하기</a>
            </p>
        </div>
    </div>
</body>
</html>
```

---

## 여러 수신자

### 1. 여러 명에게 발송

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

String[] recipients = {
    "user1@example.com",
    "user2@example.com",
    "user3@example.com"
};

Gmail mail = new Gmail("your-email@gmail.com", "앱비밀번호");
mail.send(recipients, "공지사항", "중요한 공지사항입니다.");

m.p("여러 명에게 발송되었습니다.");

%>
```

### 2. DB에서 조회하여 발송

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 이메일 수신 동의한 회원 조회
UserDao dao = new UserDao();
DataSet users = dao.find("email_agree = 'Y'");

ArrayList<String> emailList = new ArrayList<>();
while(users.next()) {
    emailList.add(users.s("email"));
}

if(emailList.size() > 0) {
    String[] recipients = emailList.toArray(new String[0]);

    String subject = "이벤트 안내";
    String message = "안녕하세요. 특별 이벤트를 안내드립니다...";

    Gmail mail = new Gmail();
    mail.send(recipients, subject, message);

    m.p(emailList.size() + "명에게 이메일을 발송했습니다.");
} else {
    m.p("발송할 이메일이 없습니다.");
}

%>
```

### 3. 개별 발송 (개인화)

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

UserDao dao = new UserDao();
DataSet users = dao.find("email_agree = 'Y'");

Gmail mail = new Gmail("your-email@gmail.com", "앱비밀번호");

int sentCount = 0;
while(users.next()) {
    String email = users.s("email");
    String name = users.s("name");

    String subject = name + "님께 특별 제안";
    String message = "<h1>" + name + "님, 안녕하세요!</h1>" +
                     "<p>" + name + "님을 위한 특별한 혜택을 준비했습니다.</p>";

    try {
        mail.send(email, subject, message);
        sentCount++;
    } catch(Exception e) {
        m.p("발송 실패: " + email + " - " + e.getMessage());
    }
}

m.p("총 " + sentCount + "명에게 개별 이메일을 발송했습니다.");

%>
```

---

## 환경설정

### 1. config.xml 설정

**WEB-INF/config/config.xml**:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<config>
    <!-- 메일 서버 설정 -->
    <mailHost>smtp.gmail.com</mailHost>
    <mailPort>587</mailPort>
    <mailFrom>your-email@gmail.com</mailFrom>
    <mailUser>your-email@gmail.com</mailUser>
    <mailPass>your-app-password</mailPass>
</config>
```

### 2. 기본 설정 사용

```jsp
// config.xml에 설정된 정보로 자동 초기화
Gmail mail = new Gmail();
mail.send("recipient@example.com", "제목", "내용");
```

### 3. 수동 설정

```jsp
Gmail mail = new Gmail();

// SMTP 서버 설정
mail.setHost("smtp.gmail.com");
mail.setPort(587);
mail.setSSL(true);

// 인증 정보
mail.setAuth("your-email@gmail.com", "your-app-password");

// 발신자 설정
mail.setMailFrom("your-email@gmail.com");

// 인코딩 설정 (기본: utf-8)
mail.setEncoding("utf-8");

// 이메일 발송
mail.send("recipient@example.com", "제목", "내용");
```

### 4. 포트 번호

```jsp
// 587 포트 (TLS/STARTTLS) - 권장
mail.setPort(587);
mail.setSSL(true);

// 465 포트 (SSL)
mail.setPort(465);

// 25 포트 (일반 SMTP)
mail.setPort(25);
```

---

## 전체 예제

### 문의하기 폼

**contact.jsp**:
```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

f.addElement("name", null, "required:'Y', maxlength:50");
f.addElement("email", null, "required:'Y', type:'email'");
f.addElement("subject", null, "required:'Y', maxlength:200");
f.addElement("message", null, "required:'Y'");

if(m.isPost() && f.validate()) {

    String name = f.get("name");
    String email = f.get("email");
    String subject = f.get("subject");
    String message = f.get("message");

    // DB에 저장
    ContactDao dao = new ContactDao();
    DataMap data = new DataMap();
    data.put("name", name);
    data.put("email", email);
    data.put("subject", subject);
    data.put("message", message);
    data.put("reg_date", m.time());
    int newId = dao.insert(data);

    // 관리자에게 이메일 발송
    String adminEmail = "admin@example.com";
    String emailSubject = "[문의] " + subject;
    String emailBody = "<h2>새로운 문의가 등록되었습니다</h2>" +
                       "<p><strong>이름:</strong> " + name + "</p>" +
                       "<p><strong>이메일:</strong> " + email + "</p>" +
                       "<p><strong>제목:</strong> " + subject + "</p>" +
                       "<p><strong>내용:</strong></p>" +
                       "<div style='padding: 10px; background: #f5f5f5;'>" + message + "</div>" +
                       "<hr>" +
                       "<p><a href='" + Config.getSiteUrl() + "/admin/contact/view.jsp?id=" + newId + "'>문의 보기</a></p>";

    try {
        Gmail mail = new Gmail();
        mail.send(adminEmail, emailSubject, emailBody);

        // 고객에게 자동 응답 발송
        String replySubject = "[자동응답] 문의가 접수되었습니다";
        String replyBody = "<h2>" + name + "님, 안녕하세요</h2>" +
                          "<p>문의가 정상적으로 접수되었습니다.</p>" +
                          "<p>빠른 시일 내에 답변 드리겠습니다.</p>" +
                          "<p>감사합니다.</p>";
        mail.send(email, replySubject, replyBody);

        m.jsAlert("문의가 접수되었습니다.");
        m.jsReplace("index.jsp");

    } catch(Exception e) {
        m.jsError("이메일 발송에 실패했습니다: " + e.getMessage());
    }
    return;
}

%>
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>문의하기</title>
    <style>
        .form-group { margin-bottom: 15px; }
        label { display: block; margin-bottom: 5px; font-weight: bold; }
        input[type="text"], input[type="email"], textarea {
            width: 100%;
            padding: 8px;
            border: 1px solid #ddd;
            border-radius: 4px;
        }
        textarea { min-height: 150px; }
        button { padding: 10px 20px; background: #007bff; color: white; border: none; border-radius: 4px; cursor: pointer; }
    </style>
</head>
<body>
    <h1>문의하기</h1>
    <form name="form1" method="post">
        <div class="form-group">
            <label>이름</label>
            <input type="text" name="name">
        </div>
        <div class="form-group">
            <label>이메일</label>
            <input type="email" name="email">
        </div>
        <div class="form-group">
            <label>제목</label>
            <input type="text" name="subject">
        </div>
        <div class="form-group">
            <label>내용</label>
            <textarea name="message"></textarea>
        </div>
        <button type="submit">문의하기</button>
    </form>
    <%= f.getScript() %>
</body>
</html>
```

---

## 베스트 프랙티스

### 1. 예외 처리

```jsp
try {
    Gmail mail = new Gmail();
    mail.send("user@example.com", "제목", "내용");
    m.p("이메일 발송 성공");
} catch(Exception e) {
    m.p("이메일 발송 실패: " + e.getMessage());
    Malgn.errorLog("Email send error", e);
}
```

### 2. 이메일 주소 검증

```jsp
String email = f.get("email");

// Form 클래스로 검증
f.addElement("email", null, "type:'email', required:'Y'");

// 또는 정규표현식으로 검증
if(!email.matches("^[A-Za-z0-9+_.-]+@(.+)$")) {
    m.jsError("올바른 이메일 주소를 입력하세요.");
    return;
}
```

### 3. 대량 발송 시 지연

```jsp
UserDao dao = new UserDao();
DataSet users = dao.find("email_agree = 'Y'");

Gmail mail = new Gmail();
int count = 0;

while(users.next()) {
    String email = users.s("email");

    try {
        mail.send(email, "제목", "내용");
        count++;

        // 100건마다 5초 대기 (SMTP 서버 부하 방지)
        if(count % 100 == 0) {
            Thread.sleep(5000);
        }

    } catch(Exception e) {
        Malgn.errorLog("Email send error: " + email, e);
    }
}

m.p("총 " + count + "건 발송 완료");
```

### 4. 비동기 발송

중요하지 않은 이메일은 별도 스레드로 발송:

```jsp
// 회원가입 처리
UserDao dao = new UserDao();
dao.insert(userData);

// 비동기로 환영 이메일 발송
new Thread(() -> {
    try {
        Gmail mail = new Gmail();
        mail.send(userEmail, "회원가입 환영", welcomeMessage);
    } catch(Exception e) {
        Malgn.errorLog("Welcome email error", e);
    }
}).start();

// 즉시 다음 페이지로 이동
m.jsReplace("login.jsp");
```

### 5. 로깅

```jsp
Gmail mail = new Gmail();

try {
    mail.send(email, subject, body);

    // 발송 로그 저장
    EmailLogDao logDao = new EmailLogDao();
    DataMap log = new DataMap();
    log.put("to_email", email);
    log.put("subject", subject);
    log.put("status", "success");
    log.put("sent_date", m.time());
    logDao.insert(log);

} catch(Exception e) {
    // 실패 로그 저장
    EmailLogDao logDao = new EmailLogDao();
    DataMap log = new DataMap();
    log.put("to_email", email);
    log.put("subject", subject);
    log.put("status", "failed");
    log.put("error_message", e.getMessage());
    log.put("sent_date", m.time());
    logDao.insert(log);
}
```

---

## 참고 문서

- [환경설정 및 캐시](configuration.md)
- [파일 업로드 및 다운로드](file-upload-download.md)
- [템플릿 - 이메일 템플릿](template.md)

---

[← 목차로 돌아가기](README.md)
# 달력 및 날짜 선택

[← 목차로 돌아가기](README.md)

---

## 개요

MCal 클래스는 달력 UI와 날짜 선택 기능을 제공합니다. 년/월/일/시/분 선택 UI를 쉽게 생성할 수 있습니다.

### 주요 기능

- **Select Box 생성**: 년/월/일/시/분 선택 옵션
- **달력 데이터**: 월별 달력 데이터 생성
- **주차 계산**: 요일 번호, 주의 첫날/마지막날
- **유연한 범위**: 년도 범위 설정 가능

---

## 목차

1. [년/월/일 선택](#년월일-선택)
2. [시/분 선택](#시분-선택)
3. [달력 데이터 생성](#달력-데이터-생성)
4. [날짜 범위 선택](#날짜-범위-선택)
5. [전체 예제](#전체-예제)

---

## 년/월/일 선택

### 1. 년도 선택 (Select Box)

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

MCal cal = new MCal();

// 현재 년도 기준 앞뒤 5년 (기본값)
DataSet years = cal.getYears();

p.setBody("main.date_form");
p.setLoop("years", years);
p.display();

%>
```

**html/main/date_form.html**:
```html
<select name="year">
    {{loop:years}}
    <option value="{{id}}">{{name}}년</option>
    {{endloop:years}}
</select>
```

**결과**:
```html
<select name="year">
    <option value="2020">2020년</option>
    <option value="2021">2021년</option>
    <option value="2022">2022년</option>
    <option value="2023">2023년</option>
    <option value="2024">2024년</option>
    <option value="2025">2025년</option>
    <option value="2026">2026년</option>
    <option value="2027">2027년</option>
    <option value="2028">2028년</option>
    <option value="2029">2029년</option>
</select>
```

### 2. 년도 범위 설정

```jsp
// 앞뒤 10년
MCal cal = new MCal(10);
DataSet years = cal.getYears();

// 또는
MCal cal = new MCal();
cal.setYearRange(10);
DataSet years = cal.getYears();

// 특정 년도 기준
DataSet years = cal.getYears(2020);  // 2020 기준 앞뒤 5년
DataSet years = cal.getYears("2020");
```

### 3. 월 선택

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

MCal cal = new MCal();
DataSet months = cal.getMonths();

p.setBody("main.month_form");
p.setLoop("months", months);
p.display();

%>
```

**html/main/month_form.html**:
```html
<select name="month">
    {{loop:months}}
    <option value="{{id}}">{{name}}월</option>
    {{endloop:months}}
</select>
```

**DataSet 구조**:
- `id`: "01", "02", ... "12" (2자리 문자열)
- `name`: "01", "02", ... "12" (2자리 문자열)
- `name2`: 1, 2, ... 12 (정수)

### 4. 일 선택

```jsp
MCal cal = new MCal();
DataSet days = cal.getDays();

// 1일 ~ 31일까지
```

**DataSet 구조**:
- `id`: "01", "02", ... "31"
- `name`: "01", "02", ... "31"
- `name2`: 1, 2, ... 31

### 5. 전체 날짜 선택 폼

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

MCal cal = new MCal();

DataSet years = cal.getYears();
DataSet months = cal.getMonths();
DataSet days = cal.getDays();

p.setBody("main.full_date_form");
p.setLoop("years", years);
p.setLoop("months", months);
p.setLoop("days", days);
p.display();

%>
```

**html/main/full_date_form.html**:
```html
<form>
    <select name="year">
        {{loop:years}}
        <option value="{{id}}">{{name}}년</option>
        {{endloop:years}}
    </select>

    <select name="month">
        {{loop:months}}
        <option value="{{id}}">{{name2}}월</option>
        {{endloop:months}}
    </select>

    <select name="day">
        {{loop:days}}
        <option value="{{id}}">{{name2}}일</option>
        {{endloop:days}}
    </select>
</form>
```

---

## 시/분 선택

### 1. 시간 선택 (0~23시)

```jsp
MCal cal = new MCal();
DataSet hours = cal.getHours();

// 00, 01, 02, ... 23
```

**DataSet 구조**:
- `id`: "00", "01", ... "23"
- `name`: "00", "01", ... "23"
- `name2`: 0, 1, ... 23

### 2. 분 선택

```jsp
MCal cal = new MCal();

// 1분 간격 (00 ~ 59)
DataSet minutes = cal.getMinutes();

// 5분 간격 (00, 05, 10, ... 55)
DataSet minutes5 = cal.getMinutes(5);

// 10분 간격 (00, 10, 20, ... 50)
DataSet minutes10 = cal.getMinutes(10);

// 15분 간격 (00, 15, 30, 45)
DataSet minutes15 = cal.getMinutes(15);
```

### 3. 시간/분 선택 폼

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

MCal cal = new MCal();
DataSet hours = cal.getHours();
DataSet minutes = cal.getMinutes(10);  // 10분 간격

p.setBody("main.time_form");
p.setLoop("hours", hours);
p.setLoop("minutes", minutes);
p.display();

%>
```

**html/main/time_form.html**:
```html
<form>
    <select name="hour">
        {{loop:hours}}
        <option value="{{id}}">{{name2}}시</option>
        {{endloop:hours}}
    </select>
    :
    <select name="minute">
        {{loop:minutes}}
        <option value="{{id}}">{{name2}}분</option>
        {{endloop:minutes}}
    </select>
</form>
```

---

## 달력 데이터 생성

### 1. 월별 달력 데이터

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

MCal cal = new MCal();

// 현재 월 달력 데이터
String currentDate = m.time("yyyy-MM-dd");
DataSet calendar = cal.getMonthDays(currentDate);

// 특정 월 달력 데이터
DataSet calendar2 = cal.getMonthDays("2025-06-01");

p.setBody("main.calendar");
p.setLoop("days", calendar);
p.display();

%>
```

**DataSet 구조**:
- `date`: 날짜 (yyyy-MM-dd 형식)
- `type`: "1" (이전달), "2" (현재달), "3" (다음달)
- `weeknum`: 요일 번호 (1=일요일, 2=월요일, ..., 7=토요일)
- `__last`: 마지막 행 여부 (Boolean)

### 2. 달력 HTML 생성

**html/main/calendar.html**:
```html
<style>
    .calendar { width: 100%; border-collapse: collapse; }
    .calendar th, .calendar td { padding: 10px; text-align: center; border: 1px solid #ddd; }
    .calendar th { background: #007bff; color: white; }
    .calendar .prev-month, .calendar .next-month { color: #ccc; }
    .calendar .sunday { color: red; }
    .calendar .saturday { color: blue; }
    .calendar .today { background: #fffacd; font-weight: bold; }
</style>

<table class="calendar">
    <thead>
        <tr>
            <th>일</th>
            <th>월</th>
            <th>화</th>
            <th>수</th>
            <th>목</th>
            <th>금</th>
            <th>토</th>
        </tr>
    </thead>
    <tbody>
        {{loop:days}}
        {{if:weeknum==1}}<tr>{{endif:weeknum==1}}

        <td class="
            {{if:type==1}}prev-month{{endif:type==1}}
            {{if:type==3}}next-month{{endif:type==3}}
            {{if:weeknum==1}}sunday{{endif:weeknum==1}}
            {{if:weeknum==7}}saturday{{endif:weeknum==7}}
            {{if:date==today}}today{{endif:date==today}}
        ">
            {{date|substr:-2}}
        </td>

        {{if:__last}}</tr>{{endif:__last}}
        {{endloop:days}}
    </tbody>
</table>
```

### 3. 일정이 있는 달력

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

String month = m.rs("month", m.time("yyyy-MM"));

MCal cal = new MCal();
DataSet calendar = cal.getMonthDays(month + "-01");

// 해당 월의 일정 조회
EventDao dao = new EventDao();
DataSet events = dao.find("event_date >= ? AND event_date < ?",
    new Object[] { month + "-01", Malgn.addMonth(1, month + "-01") });

// 날짜별로 일정 카운트
Hashtable<String, Integer> eventCounts = new Hashtable<>();
while(events.next()) {
    String date = events.s("event_date");
    eventCounts.put(date, eventCounts.getOrDefault(date, 0) + 1);
}

// 달력 데이터에 일정 카운트 추가
calendar.first();
while(calendar.next()) {
    String date = calendar.s("date");
    int count = eventCounts.getOrDefault(date, 0);
    calendar.put("event_count", count);
}

calendar.first();
p.setBody("main.event_calendar");
p.setLoop("days", calendar);
p.setVar("month", month);
p.setVar("prev_month", Malgn.addMonth(-1, month + "-01").substring(0, 7));
p.setVar("next_month", Malgn.addMonth(1, month + "-01").substring(0, 7));
p.display();

%>
```

**html/main/event_calendar.html**:
```html
<div style="text-align: center; margin-bottom: 20px;">
    <a href="?month={{prev_month}}">&laquo; 이전</a>
    <strong>{{month}}</strong>
    <a href="?month={{next_month}}">다음 &raquo;</a>
</div>

<table class="calendar">
    <!-- ... 달력 헤더 ... -->
    <tbody>
        {{loop:days}}
        {{if:weeknum==1}}<tr>{{endif:weeknum==1}}

        <td class="{{if:type==1}}prev-month{{endif:type==1}} {{if:type==3}}next-month{{endif:type==3}}">
            <div>{{date|substr:-2}}</div>
            {{if:event_count>0}}
            <div style="color: red; font-size: 11px;">{{event_count}}건</div>
            {{endif:event_count>0}}
        </td>

        {{if:__last}}</tr>{{endif:__last}}
        {{endloop:days}}
    </tbody>
</table>
```

---

## 날짜 범위 선택

### 1. 시작일/종료일 선택

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

MCal cal = new MCal();

DataSet years = cal.getYears();
DataSet months = cal.getMonths();
DataSet days = cal.getDays();

p.setBody("main.date_range_form");
p.setLoop("years", years);
p.setLoop("months", months);
p.setLoop("days", days);
p.display();

%>
```

**html/main/date_range_form.html**:
```html
<form method="get">
    <div>
        <label>시작일</label>
        <select name="start_year">
            {{loop:years}}
            <option value="{{id}}">{{name}}년</option>
            {{endloop:years}}
        </select>
        <select name="start_month">
            {{loop:months}}
            <option value="{{id}}">{{name2}}월</option>
            {{endloop:months}}
        </select>
        <select name="start_day">
            {{loop:days}}
            <option value="{{id}}">{{name2}}일</option>
            {{endloop:days}}
        </select>
    </div>

    <div>
        <label>종료일</label>
        <select name="end_year">
            {{loop:years}}
            <option value="{{id}}">{{name}}년</option>
            {{endloop:years}}
        </select>
        <select name="end_month">
            {{loop:months}}
            <option value="{{id}}">{{name2}}월</option>
            {{endloop:months}}
        </select>
        <select name="end_day">
            {{loop:days}}
            <option value="{{id}}">{{name2}}일</option>
            {{endloop:days}}
        </select>
    </div>

    <button type="submit">검색</button>
</form>
```

### 2. 날짜 범위로 데이터 조회

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

if(m.isPost()) {

    String startYear = f.get("start_year");
    String startMonth = f.get("start_month");
    String startDay = f.get("start_day");

    String endYear = f.get("end_year");
    String endMonth = f.get("end_month");
    String endDay = f.get("end_day");

    String startDate = startYear + "-" + startMonth + "-" + startDay;
    String endDate = endYear + "-" + endMonth + "-" + endDay;

    // 날짜 검증
    if(Malgn.strToDate("yyyy-MM-dd", startDate).after(Malgn.strToDate("yyyy-MM-dd", endDate))) {
        m.jsError("시작일이 종료일보다 늦을 수 없습니다.");
        return;
    }

    // 데이터 조회
    OrderDao dao = new OrderDao();
    DataSet orders = dao.find("order_date >= ? AND order_date <= ?",
        new Object[] { startDate, endDate });

    p.setBody("main.order_list");
    p.setLoop("orders", orders);
    p.setVar("start_date", startDate);
    p.setVar("end_date", endDate);
    p.display();
    return;
}

// 폼 표시
MCal cal = new MCal();
DataSet years = cal.getYears();
DataSet months = cal.getMonths();
DataSet days = cal.getDays();

p.setBody("main.date_range_form");
p.setLoop("years", years);
p.setLoop("months", months);
p.setLoop("days", days);
p.display();

%>
```

---

## 전체 예제

### 예약 시스템

**reservation_form.jsp**:
```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

f.addElement("name", null, "required:'Y'");
f.addElement("phone", null, "required:'Y', type:'phone'");
f.addElement("year", null, "required:'Y'");
f.addElement("month", null, "required:'Y'");
f.addElement("day", null, "required:'Y'");
f.addElement("hour", null, "required:'Y'");
f.addElement("minute", null, "required:'Y'");

if(m.isPost() && f.validate()) {

    String name = f.get("name");
    String phone = f.get("phone");

    String year = f.get("year");
    String month = f.get("month");
    String day = f.get("day");
    String hour = f.get("hour");
    String minute = f.get("minute");

    String reservationDate = year + "-" + month + "-" + day;
    String reservationTime = hour + ":" + minute;
    String reservationDatetime = reservationDate + " " + reservationTime + ":00";

    // 과거 날짜 체크
    if(Malgn.strToDate("yyyy-MM-dd HH:mm:ss", reservationDatetime)
        .before(new Date())) {
        m.jsError("과거 날짜는 예약할 수 없습니다.");
        return;
    }

    // 중복 예약 체크
    ReservationDao dao = new ReservationDao();
    DataSet existing = dao.find("reservation_datetime = ?", reservationDatetime);
    if(existing.next()) {
        m.jsError("해당 시간은 이미 예약되어 있습니다.");
        return;
    }

    // 예약 저장
    DataMap data = new DataMap();
    data.put("name", name);
    data.put("phone", phone);
    data.put("reservation_datetime", reservationDatetime);
    data.put("status", "pending");
    data.put("reg_date", m.time());
    int newId = dao.insert(data);

    m.jsAlert("예약이 완료되었습니다.");
    m.jsReplace("reservation_view.jsp?id=" + newId);
    return;
}

// 폼 표시
MCal cal = new MCal();

// 오늘부터 1년치
cal.setYearRange(1);
DataSet years = cal.getYears();
DataSet months = cal.getMonths();
DataSet days = cal.getDays();

// 09:00 ~ 18:00, 30분 간격
DataSet hours = new DataSet();
for(int i = 9; i <= 18; i++) {
    hours.addRow();
    hours.put("id", (i < 10 ? "0" : "") + i);
    hours.put("name", i + "시");
}
hours.first();

DataSet minutes = new DataSet();
minutes.addRow();
minutes.put("id", "00");
minutes.put("name", "00분");
minutes.addRow();
minutes.put("id", "30");
minutes.put("name", "30분");
minutes.first();

p.setBody("main.reservation_form");
p.setLoop("years", years);
p.setLoop("months", months);
p.setLoop("days", days);
p.setLoop("hours", hours);
p.setLoop("minutes", minutes);
p.setVar("form_script", f.getScript());
p.display();

%>
```

**html/main/reservation_form.html**:
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>예약하기</title>
    <style>
        .form-group { margin-bottom: 15px; }
        label { display: block; margin-bottom: 5px; font-weight: bold; }
        input, select { padding: 8px; border: 1px solid #ddd; border-radius: 4px; }
        button { padding: 10px 20px; background: #007bff; color: white; border: none; border-radius: 4px; cursor: pointer; }
    </style>
</head>
<body>
    <h1>예약하기</h1>
    <form name="form1" method="post">
        <div class="form-group">
            <label>이름</label>
            <input type="text" name="name">
        </div>

        <div class="form-group">
            <label>전화번호</label>
            <input type="text" name="phone">
        </div>

        <div class="form-group">
            <label>예약 날짜</label>
            <select name="year">
                {{loop:years}}
                <option value="{{id}}">{{name}}년</option>
                {{endloop:years}}
            </select>
            <select name="month">
                {{loop:months}}
                <option value="{{id}}">{{name2}}월</option>
                {{endloop:months}}
            </select>
            <select name="day">
                {{loop:days}}
                <option value="{{id}}">{{name2}}일</option>
                {{endloop:days}}
            </select>
        </div>

        <div class="form-group">
            <label>예약 시간</label>
            <select name="hour">
                {{loop:hours}}
                <option value="{{id}}">{{name}}</option>
                {{endloop:hours}}
            </select>
            <select name="minute">
                {{loop:minutes}}
                <option value="{{id}}">{{name}}</option>
                {{endloop:minutes}}
            </select>
        </div>

        <button type="submit">예약하기</button>
    </form>
    {{form_script}}
</body>
</html>
```

---

## 요일 및 주차 계산

### 1. 요일 번호 구하기

```jsp
MCal cal = new MCal();

// 요일 번호 (1=일요일, 2=월요일, ..., 7=토요일)
int weekNum = cal.getWeekNum("20250601");
int weekNum2 = cal.getWeekNum("2025-06-01", "yyyy-MM-dd");

m.p("요일 번호: " + weekNum);
```

### 2. 주의 첫날/마지막날 구하기

```jsp
MCal cal = new MCal();

// 해당 주의 첫날 (일요일)
Date weekFirst = cal.getWeekFirstDate("20250601");
m.p("주의 첫날: " + Malgn.getTimeString("yyyy-MM-dd", weekFirst));

// 해당 주의 마지막날 (토요일)
Date weekLast = cal.getWeekLastDate("20250601");
m.p("주의 마지막날: " + Malgn.getTimeString("yyyy-MM-dd", weekLast));

// 월의 마지막날
Date monthLast = cal.getMonthLastDate("20250601");
m.p("월의 마지막날: " + Malgn.getTimeString("yyyy-MM-dd", monthLast));
```

---

## 베스트 프랙티스

### 1. 현재 날짜 선택 상태로 표시

```html
<select name="year">
    {{loop:years}}
    <option value="{{id}}" {{if:id==current_year}}selected{{endif:id==current_year}}>{{name}}년</option>
    {{endloop:years}}
</select>
```

```jsp
p.setVar("current_year", m.time("yyyy"));
p.setVar("current_month", m.time("MM"));
p.setVar("current_day", m.time("dd"));
```

### 2. JavaScript로 동적 처리

```html
<script>
// 월에 따라 일자 선택 동적 변경
document.querySelector('select[name="month"]').addEventListener('change', function() {
    var year = document.querySelector('select[name="year"]').value;
    var month = this.value;
    var daySelect = document.querySelector('select[name="day"]');

    // 해당 월의 마지막 날 계산
    var lastDay = new Date(year, month, 0).getDate();

    // 옵션 재생성
    daySelect.innerHTML = '';
    for(var i = 1; i <= lastDay; i++) {
        var option = document.createElement('option');
        option.value = (i < 10 ? '0' : '') + i;
        option.text = i + '일';
        daySelect.appendChild(option);
    }
});
</script>
```

### 3. 오늘 날짜 하이라이트

```jsp
p.setVar("today", m.time("yyyy-MM-dd"));
```

```html
<td class="{{if:date==today}}today{{endif:date==today}}">
    {{date|substr:-2}}
</td>
```

---

## 참고 문서

- [템플릿](template.md)
- [DataSet 활용](dataset.md)
- [유틸리티 메소드 - 날짜/시간](utility-methods.md#날짜시간-처리)

---

[← 목차로 돌아가기](README.md)
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
# 다국어 지원 (Internationalization)

맑은프레임워크의 Message 클래스는 다국어 웹 애플리케이션을 구축하기 위한 국제화(i18n) 기능을 제공합니다.

## 목차

- [기본 개념](#기본-개념)
- [메시지 파일 구조](#메시지-파일-구조)
- [Message 클래스 사용](#message-클래스-사용)
- [동적 메시지](#동적-메시지)
- [캐시 및 성능](#캐시-및-성능)

---

## 기본 개념

### 다국어 처리 방식

Message 클래스는 properties 파일 기반의 다국어 메시지 시스템을 제공합니다:

- **메시지 파일**: 키-값 쌍으로 메시지 저장
- **로케일 지원**: 언어별 메시지 파일 관리
- **폴백 처리**: 해당 언어에 메시지가 없으면 기본 언어로 대체
- **메모리 캐싱**: 성능을 위한 자동 캐싱

---

## 메시지 파일 구조

### 파일 위치

기본 경로: `/WEB-INF/message/`

설정 변경:
```jsp
Config.set("msgDir", "/custom/message/path");
```

### 파일 명명 규칙

```
/WEB-INF/message/
  ├── message.properties          (기본 언어)
  ├── message_en.properties       (영어)
  ├── message_ko.properties       (한국어)
  ├── message_ja.properties       (일본어)
  └── message_zh.properties       (중국어)
```

### 메시지 파일 형식

**message.properties** (기본 - 한국어)
```properties
# 사용자 인터페이스
welcome=환영합니다
login=로그인
logout=로그아웃
name=이름
email=이메일
password=비밀번호

# 에러 메시지
error.required=필수 항목입니다
error.invalid_email=올바른 이메일 주소를 입력하세요
error.password_mismatch=비밀번호가 일치하지 않습니다

# 시스템 메시지
sysop.book_insert.specify_books=도서를 선택해주세요
main.log=사용자 {0}이(가) {1}에 로그인했습니다
```

**message_en.properties** (영어)
```properties
welcome=Welcome
login=Login
logout=Logout
name=Name
email=Email
password=Password

error.required=This field is required
error.invalid_email=Please enter a valid email address
error.password_mismatch=Passwords do not match

sysop.book_insert.specify_books=Please select books
main.log=User {0} logged in at {1}
```

**message_ko.properties** (한국어)
```properties
welcome=환영합니다
login=로그인
logout=로그아웃
```

---

## Message 클래스 사용

### 기본 사용법

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 기본 로케일로 Message 객체 생성
Message msg = new Message();

// 메시지 가져오기
String welcome = msg.get("welcome");      // "환영합니다"
String login = msg.get("login");          // "로그인"

m.p(welcome);
m.p(login);

%>
```

### 로케일 지정

```jsp
// 영어로 설정
Message msg = new Message("en");
m.p(msg.get("welcome"));  // "Welcome"

// 또는 setLocale 사용
Message msg = new Message();
msg.setLocale("en");
m.p(msg.get("welcome"));  // "Welcome"

msg.setLocale("ko");
m.p(msg.get("welcome"));  // "환영합니다"
```

### 기본값 처리

```jsp
Message msg = new Message();

// 메시지가 없으면 키를 반환
String unknown = msg.get("unknown.key");  // "unknown.key"

// 사용자 정의 기본값
String custom = msg.get("unknown.key", "기본 메시지");  // "기본 메시지"
```

---

## 동적 메시지

### 매개변수가 있는 메시지

**message.properties**
```properties
user.greeting=안녕하세요, {0}님!
order.confirm=주문 {0}개 상품이 {1}원에 결제되었습니다
```

**JSP 코드**
```jsp
Message msg = new Message();

// MessageFormat 스타일 ({0}, {1}, ...)
String greeting = msg.format("user.greeting", "홍길동");
// "안녕하세요, 홍길동님!"

String confirm = msg.format("order.confirm", 3, 50000);
// "주문 3개 상품이 50000원에 결제되었습니다"
```

### 플레이스홀더 치환

**message.properties**
```properties
email.template=안녕하세요 {{name}}님, {{product}} 주문이 완료되었습니다.
```

**JSP 코드**
```jsp
Message msg = new Message();

String[] params = {
    "name=>홍길동",
    "product=>맥북 프로"
};

String email = msg.get("email.template", params);
// "안녕하세요 홍길동님, 맥북 프로 주문이 완료되었습니다."
```

---

## 배열과 Map 처리

### 배열 메시지

```jsp
Message msg = new Message();

String[] keys = {
    "status.0=>pending",
    "status.1=>approved",
    "status.2=>rejected"
};

String[] messages = msg.getArray(keys);
// ["status.0=>대기중", "status.1=>승인됨", "status.2=>거부됨"]
```

### Map 메시지

```jsp
Map<String, String> map = new HashMap<>();
map.put("title", "welcome");
map.put("subtitle", "login");

Map<String, String> translated = msg.getMap(map);
// {"title":"환영합니다", "subtitle":"로그인"}
```

---

## 캐시 및 성능

### 메모리 캐싱

Message 클래스는 메시지 파일을 메모리에 캐싱하여 성능을 최적화합니다.

```jsp
Message msg = new Message();

// 첫 번째 호출: 파일에서 로드 및 캐싱
String text1 = msg.get("welcome");

// 두 번째 호출: 캐시에서 가져옴 (빠름)
String text2 = msg.get("login");
```

### 캐시 새로고침

메시지 파일을 수정한 후 변경사항을 반영하려면:

```jsp
Message msg = new Message();

// 특정 로케일 캐시 재로드
msg.reload("en");

// 기본 로케일 캐시 재로드
msg.reload();

// 모든 로케일 캐시 재로드
msg.reloadAll();
```

---

## 실제 사용 예제

### 다국어 웹페이지

**user_profile.jsp**
```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 사용자 언어 설정 가져오기 (쿠키, 세션, 또는 파라미터에서)
String lang = m.cookie("lang");
if(lang == null || lang.isEmpty()) lang = "default";

Message msg = new Message(lang);

p.setBody("user.profile");
p.setVar("msg", msg);
p.display();

%>
```

**html/user/profile.html**
```html
<!DOCTYPE html>
<html>
<head>
    <title>{{msg.get("page.title")}}</title>
</head>
<body>
    <h1>{{msg.get("user.profile")}}</h1>

    <form>
        <label>{{msg.get("name")}}</label>
        <input type="text" name="name" />

        <label>{{msg.get("email")}}</label>
        <input type="email" name="email" />

        <label>{{msg.get("password")}}</label>
        <input type="password" name="password" />

        <button>{{msg.get("save")}}</button>
    </form>
</body>
</html>
```

### 폼 검증 메시지

**message.properties**
```properties
validation.required={0}은(는) 필수 항목입니다
validation.email=올바른 이메일 주소를 입력하세요
validation.minlength={0}은(는) 최소 {1}자 이상이어야 합니다
```

**login.jsp**
```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

Message msg = new Message();

f.addElement("email", null, "required:Y, email:Y");
f.addElement("password", null, "required:Y, minlength:8");

if(m.isPost() && !f.validate()) {
    // 에러 메시지를 다국어로 변환
    String emailError = msg.format("validation.email");
    String passwordError = msg.format("validation.minlength", "비밀번호", 8);

    m.jsError(f.getErrorString());
    return;
}

// 로그인 처리...

%>
```

### 동적 알림 메시지

**message.properties**
```properties
notification.new_message={0}님으로부터 새 메시지가 도착했습니다
notification.order_shipped=주문 #{0}이(가) 발송되었습니다. 운송장번호: {1}
notification.password_changed=비밀번호가 {0}에 변경되었습니다
```

**notification.jsp**
```jsp
Message msg = new Message();

// 새 메시지 알림
String sender = "홍길동";
String notification1 = msg.format("notification.new_message", sender);
// "홍길동님으로부터 새 메시지가 도착했습니다"

// 배송 알림
String notification2 = msg.format("notification.order_shipped", "A123456", "1234-5678-9012");
// "주문 #A123456이(가) 발송되었습니다. 운송장번호: 1234-5678-9012"

// 비밀번호 변경 알림
String notification3 = msg.format("notification.password_changed", Malgn.time("yyyy-MM-dd HH:mm"));
// "비밀번호가 2025-10-23 14:30에 변경되었습니다"
```

---

## 언어 전환 구현

### 언어 선택 UI

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

String currentLang = m.cookie("lang");
if(currentLang == null || currentLang.isEmpty()) currentLang = "default";

Message msg = new Message(currentLang);

%>
<div class="language-selector">
    <select onchange="changeLanguage(this.value)">
        <option value="default" <%=currentLang.equals("default") ? "selected" : ""%>>한국어</option>
        <option value="en" <%=currentLang.equals("en") ? "selected" : ""%>>English</option>
        <option value="ja" <%=currentLang.equals("ja") ? "selected" : ""%>>日本語</option>
        <option value="zh" <%=currentLang.equals("zh") ? "selected" : ""%>>中文</option>
    </select>
</div>

<script>
function changeLanguage(lang) {
    document.cookie = "lang=" + lang + "; path=/; max-age=31536000";
    location.reload();
}
</script>
```

### 언어 설정 API

**set_language.jsp**
```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="/init.jsp" %><%

String lang = m.rs("lang", "default");

// 쿠키에 저장 (1년)
m.setCookie("lang", lang, 365);

// 또는 세션에 저장
session.setAttribute("lang", lang);

j.success("언어가 변경되었습니다");

%>
```

---

## 모범 사례

### 1. 메시지 키 네이밍

계층적 구조 사용:
```properties
# 좋은 예
user.profile.title=프로필
user.profile.edit=프로필 수정
user.settings.password=비밀번호 변경

# 나쁜 예
title=프로필
edit=수정
password=비밀번호
```

### 2. 공통 메시지 재사용

```properties
# 공통 액션
common.save=저장
common.cancel=취소
common.delete=삭제
common.confirm=확인

# 공통 상태
status.active=활성
status.inactive=비활성
status.pending=대기중
```

### 3. UTF-8 인코딩

메시지 파일은 반드시 UTF-8로 저장:
- Eclipse: File > Properties > Text file encoding > UTF-8
- IntelliJ: File > Settings > Editor > File Encodings > UTF-8

### 4. 성능 최적화

```jsp
// 좋은 예: Message 객체 재사용
Message msg = new Message();
for(int i = 0; i < 100; i++) {
    m.p(msg.get("key" + i));
}

// 나쁜 예: 매번 새로 생성
for(int i = 0; i < 100; i++) {
    Message msg = new Message();
    m.p(msg.get("key" + i));
}
```

---

## 주의사항

1. **파일 인코딩**: 메시지 파일은 반드시 UTF-8로 저장
2. **키 일관성**: 모든 언어 파일에 동일한 키 사용
3. **기본 언어**: `message.properties`는 항상 유지 (폴백용)
4. **특수 문자**: `=`, `:`, `#` 등은 이스케이프 필요
   ```properties
   # 올바른 예
   message.with.colon=값\: 설명
   ```

---

## 관련 문서

- [환경설정 및 캐시](configuration.md) - Config 클래스
- [유틸리티 메소드](utility-methods.md) - 문자열 처리
- [템플릿](template.md) - 템플릿에서 다국어 사용
# OpenAI 통합

맑은프레임워크의 OpenAI 클래스는 OpenAI API(ChatGPT)를 쉽게 통합하여 AI 기능을 웹 애플리케이션에 추가할 수 있게 합니다.

## 목차

- [기본 설정](#기본-설정)
- [텍스트 생성](#텍스트-생성)
- [대화형 챗봇](#대화형-챗봇)
- [시스템 프롬프트](#시스템-프롬프트)
- [고급 설정](#고급-설정)

---

## 기본 설정

### OpenAI 객체 생성

```jsp
OpenAI model = new OpenAI();

// API 키 설정 (필수)
model.apiKey("sk-your-api-key-here");

// 모델 선택 (기본값: gpt-3.5-turbo)
model.modelName("gpt-3.5-turbo");
// 또는 더 강력한 모델
model.modelName("gpt-4o-mini");
model.modelName("gpt-4");
```

### 디버그 모드

```jsp
OpenAI model = new OpenAI();
model.setDebug(out);  // HTTP 요청/응답 로그 출력
model.setDebug();     // 로그 파일로 출력
```

---

## 텍스트 생성

### 간단한 질문-응답

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

OpenAI model = new OpenAI();
model.apiKey("sk-your-api-key-here");
model.modelName("gpt-3.5-turbo");

// 질문하고 응답 받기
String answer = model.generate("대한민국의 수도는 어디인가?");
m.p(answer);
// "대한민국의 수도는 서울입니다."

%>
```

### 구조화된 응답

JSON 형식으로 응답을 받을 수 있습니다:

```jsp
OpenAI model = new OpenAI();
model.apiKey("sk-your-api-key-here");

String response = model.generate(
    "서울의 인구는 몇명인가?\n\n"
    + "output-format:{\"data\":\"xxx\"}"
);

m.p(response);
// {"data":"약 970만명"}

// JSON 파싱
Json j = new Json(response);
String population = j.getString("//data");
```

### 복잡한 데이터 생성

```jsp
OpenAI model = new OpenAI();
model.apiKey("sk-your-api-key-here");
model.modelName("gpt-4o-mini");

m.startTimer();

String response = model.generate(
    "AI와 관련된 문제 10개만 만들어줘. 보기는 4개로 하고, 정답과 해설도 작성해줘. 한글로 해줘.\n\n"
    + "output-format:[{\"question\":\"xxx\", \"choice1\":\"-\", \"choice2\":\"-\", \"choice3\":\"-\", \"choice4\":\"-\", \"answer\":\"1\", \"description\":\"-\"}]"
);

m.p(response);
m.p("소요 시간: " + m.stopTimer() + "ms");

// JSON 파싱하여 데이터베이스에 저장
Json j = new Json(response);
DataSet questions = j.getDataSet("//");
QuizDao dao = new QuizDao();

while(questions.next()) {
    DataMap data = new DataMap();
    data.put("question", questions.s("question"));
    data.put("choice1", questions.s("choice1"));
    data.put("choice2", questions.s("choice2"));
    data.put("choice3", questions.s("choice3"));
    data.put("choice4", questions.s("choice4"));
    data.put("answer", questions.s("answer"));
    data.put("description", questions.s("description"));
    dao.insert(data);
}
```

---

## 대화형 챗봇

### 대화 기록 유지

`chat()` 메소드를 사용하면 대화 컨텍스트가 자동으로 유지됩니다:

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 세션에서 OpenAI 객체 가져오기 (또는 새로 생성)
OpenAI model = (OpenAI)session.getAttribute("chatModel");
if(model == null) {
    model = new OpenAI();
    model.apiKey("sk-your-api-key-here");
    model.modelName("gpt-4o-mini");
    session.setAttribute("chatModel", model);
}

// 사용자 메시지
String userMessage = m.rs("message");
if(!userMessage.isEmpty()) {
    String response = model.chat(userMessage);

    j.put("message", response);
    j.print(out);
}

%>
```

### 대화 예제

```jsp
OpenAI model = new OpenAI();
model.apiKey("sk-your-api-key-here");
model.modelName("gpt-4o-mini");

// 첫 번째 메시지
String r1 = model.chat("내 이름은 임꺽정이야");
m.p(r1);
// "안녕하세요, 임꺽정님! 무엇을 도와드릴까요?"

// 두 번째 메시지 - 이전 대화 기억
String r2 = model.chat("내가 혹시 누군지 아니? 내 나이 37세");
m.p(r2);
// "네, 임꺽정님이시죠. 37세시라고 말씀하셨습니다."

// 세 번째 메시지 - 계속 대화 컨텍스트 유지
String r3 = model.chat("나이도 알아?");
m.p(r3);
// "네, 37세라고 하셨습니다."
```

### 수동으로 대화 기록 추가

```jsp
OpenAI model = new OpenAI();
model.apiKey("sk-your-api-key-here");

// 이전 대화 수동 추가
model.addChat("안녕하세요", "안녕하세요! 무엇을 도와드릴까요?");
model.addChat("날씨 좋네요", "네, 좋은 날씨입니다!");

// 새로운 메시지 (이전 컨텍스트 포함)
String response = model.chat("지금 몇 시야?");
```

---

## 시스템 프롬프트

### 시스템 메시지 설정

시스템 메시지로 AI의 역할과 행동을 정의할 수 있습니다:

```jsp
OpenAI model = new OpenAI();
model.apiKey("sk-your-api-key-here");
model.modelName("gpt-4o-mini");

// AI의 페르소나 설정
model.system("당신은 친절한 고객 서비스 담당자입니다. 항상 공손하고 도움이 되는 답변을 제공하세요.");

String response = model.generate("제품이 불량인 것 같아요");
// "죄송합니다. 불편을 드려 죄송합니다. 어떤 문제가 있으신지 자세히 말씀해주시겠어요? 최대한 빠르게 해결해드리겠습니다."
```

### 전문가 시스템

```jsp
// 프로그래밍 전문가
model.system("당신은 숙련된 Java 프로그래머입니다. 코드 리뷰와 최적화 제안을 제공하세요.");

String codeReview = model.generate(
    "다음 코드를 리뷰해주세요:\n\n"
    + "for(int i=0; i<list.size(); i++) {\n"
    + "    System.out.println(list.get(i));\n"
    + "}"
);

// 의료 정보 제공자
model.system("당신은 의료 정보를 제공하는 AI입니다. 정확하고 근거 있는 정보만 제공하며, 진단은 하지 마세요.");

// 언어 튜터
model.system("당신은 영어 선생님입니다. 학생의 영어 문장을 교정하고 더 나은 표현을 제안하세요.");

// 창의적 작가
model.system("당신은 창의적인 소설가입니다. 흥미진진하고 상상력 넘치는 이야기를 만들어주세요.");
```

### 응답 스타일 제어

```jsp
// 장황하게 답변
model.system("아주 장황하게 대답해줘. 가능한 한 많은 세부 정보를 포함하세요.");
String detailed = model.generate("서울의 인구는 몇명인가?");

// 간결하게 답변
model.system("아주 간결하게 핵심만 대답해줘. 한 문장으로만 답하세요.");
String brief = model.generate("서울의 인구는 몇명인가?");

// 특정 형식으로 답변
model.system("모든 답변을 JSON 형식으로만 제공하세요.");
String jsonResponse = model.generate("대한민국의 수도와 인구를 알려줘");
```

---

## 고급 설정

### Temperature 조절

Temperature는 응답의 무작위성을 제어합니다 (0.0 ~ 2.0):

```jsp
OpenAI model = new OpenAI();
model.apiKey("sk-your-api-key-here");

// 낮은 temperature (0.0 ~ 0.3): 일관되고 예측 가능한 응답
model.temperature(0.2);
String factual = model.generate("물의 끓는점은?");
// "100°C입니다" (매번 동일한 답변)

// 중간 temperature (0.7): 균형잡힌 응답 (기본값)
model.temperature(0.7);

// 높은 temperature (1.0 ~ 2.0): 창의적이고 다양한 응답
model.temperature(1.5);
String creative = model.generate("우주에 대한 시를 써줘");
// 매번 다른 창의적인 시
```

### 사용 예시

```jsp
// 팩트 기반 Q&A (낮은 temperature)
model.temperature(0.1);
String fact = model.generate("2 + 2 = ?");

// 창작 콘텐츠 (높은 temperature)
model.temperature(1.2);
String story = model.generate("우주 해적에 대한 짧은 이야기를 써줘");

// 일반 대화 (중간 temperature)
model.temperature(0.7);
String chat = model.generate("오늘 기분이 어때?");
```

---

## 실제 활용 예제

### 1. 자동 요약 기능

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="/init.jsp" %><%

String content = m.rs("content");

OpenAI model = new OpenAI();
model.apiKey("sk-your-api-key-here");
model.modelName("gpt-3.5-turbo");
model.system("당신은 전문 요약가입니다. 핵심 내용만 3-5문장으로 요약하세요.");

String summary = model.generate("다음 글을 요약해주세요:\n\n" + content);

j.success("요약 완료", new DataMap().put("summary", summary));

%>
```

### 2. 감정 분석

```jsp
OpenAI model = new OpenAI();
model.apiKey("sk-your-api-key-here");
model.system("사용자 리뷰의 감정을 분석하세요. positive, negative, neutral 중 하나로만 답하세요.");

String review = "이 제품 정말 최고예요! 강력 추천합니다.";
String sentiment = model.generate(review);
// "positive"
```

### 3. 콘텐츠 생성기

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

String topic = m.rs("topic");
String tone = m.rs("tone", "전문적");

OpenAI model = new OpenAI();
model.apiKey("sk-your-api-key-here");
model.modelName("gpt-4o-mini");
model.system("당신은 블로그 작가입니다. " + tone + "인 톤으로 글을 작성하세요.");

String article = model.generate(
    topic + "에 대한 블로그 글을 작성해주세요. 500자 내외로 작성하세요."
);

p.setBody("blog.article");
p.setVar("content", article);
p.display();

%>
```

### 4. 챗봇 서비스

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 세션에서 대화 모델 가져오기
OpenAI model = (OpenAI)session.getAttribute("chatbot");
if(model == null) {
    model = new OpenAI();
    model.apiKey("sk-your-api-key-here");
    model.modelName("gpt-4o-mini");
    model.system(
        "당신은 맑은프레임워크 기술 지원 챗봇입니다. " +
        "사용자의 기술적 질문에 정확하고 친절하게 답변하세요. " +
        "모르는 것은 솔직하게 모른다고 말하세요."
    );
    session.setAttribute("chatbot", model);
}

String userMsg = m.rs("message");
String botResponse = model.chat(userMsg);

j.success(new DataMap().put("response", botResponse));

%>
```

### 5. 번역 서비스

```jsp
OpenAI model = new OpenAI();
model.apiKey("sk-your-api-key-here");
model.system("당신은 전문 번역가입니다. 한국어를 영어로 정확하게 번역하세요.");

String korean = "안녕하세요. 만나서 반갑습니다.";
String english = model.generate(korean);
// "Hello. Nice to meet you."
```

---

## 에러 처리

```jsp
OpenAI model = new OpenAI();
model.apiKey("sk-your-api-key-here");

try {
    String response = model.generate("질문");

    if(response.isEmpty()) {
        // 응답이 비어있음
        m.jsError("AI 응답을 받지 못했습니다");
        return;
    }

    m.p(response);

} catch(Exception e) {
    // API 에러
    Malgn.errorLog("OpenAI API 에러", e);
    m.jsError("AI 서비스 오류가 발생했습니다");
}

// 에러 메시지 확인
if(model.errMsg != null) {
    m.p("Error: " + model.errMsg);
}
```

---

## 비용 최적화 팁

### 1. 적절한 모델 선택

```jsp
// 간단한 작업: gpt-3.5-turbo (저렴, 빠름)
model.modelName("gpt-3.5-turbo");

// 복잡한 작업: gpt-4o-mini (균형)
model.modelName("gpt-4o-mini");

// 고급 작업: gpt-4 (비쌈, 정확)
model.modelName("gpt-4");
```

### 2. 프롬프트 최적화

```jsp
// 비효율적: 장황한 프롬프트
String bad = model.generate(
    "안녕하세요. 저는 학생입니다. 제가 궁금한게 있는데요, " +
    "혹시 물의 끓는점이 몇 도인지 알려주실 수 있나요? " +
    "가능하면 자세히 설명해주세요."
);

// 효율적: 간결한 프롬프트
String good = model.generate("물의 끓는점은?");
```

### 3. 캐싱 활용

```jsp
// 자주 묻는 질문은 캐싱
Cache cache = new Cache();
String question = "맑은프레임워크란?";
String answer = cache.getString("faq_" + Malgn.md5(question));

if(answer == null) {
    // 캐시에 없으면 AI에게 질문
    answer = model.generate(question);
    cache.save("faq_" + Malgn.md5(question), answer);
}

m.p(answer);
```

---

## 주의사항

1. **API 키 보안**: API 키를 소스코드에 직접 넣지 말고 환경설정 파일 사용
   ```jsp
   model.apiKey(Config.get("openai.apiKey"));
   ```

2. **비용 관리**: OpenAI API는 사용량에 따라 과금됩니다
3. **응답 시간**: AI 응답은 수 초가 걸릴 수 있으므로 로딩 UI 필요
4. **콘텐츠 필터링**: 생성된 콘텐츠는 검토가 필요할 수 있음
5. **개인정보**: 민감한 개인정보를 AI에 전송하지 않도록 주의

---

## 관련 문서

- [HTTP 클라이언트](http-client.md) - OpenAI가 내부적으로 사용
- [JSON 처리](json.md) - AI 응답 파싱
- [환경설정 및 캐시](configuration.md) - API 키 관리 및 응답 캐싱
# 파일 전송 및 압축

맑은프레임워크는 FTP 파일 전송과 ZIP 압축/해제 기능을 제공합니다.

## 목차

- [FTP 파일 전송](#ftp-파일-전송)
  - [기본 연결](#기본-연결)
  - [파일 업로드](#파일-업로드)
  - [디렉토리 작업](#디렉토리-작업)
- [ZIP 압축 및 해제](#zip-압축-및-해제)
  - [파일 압축](#파일-압축)
  - [압축 해제](#압축-해제)
  - [다운로드 압축](#다운로드-압축)

---

## FTP 파일 전송

SimpleFTP 클래스는 간단하고 직관적인 FTP 클라이언트 기능을 제공합니다.

### 기본 연결

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

SimpleFTP ftp = new SimpleFTP();

try {
    // 기본 포트(21)로 연결
    ftp.connect("ftp.example.com");

    // 또는 포트 지정
    ftp.connect("ftp.example.com", 21);

    // 또는 사용자명/비밀번호 지정
    ftp.connect("ftp.example.com", 21, "username", "password");

    m.p("FTP 연결 성공");

} catch(Exception e) {
    m.p("FTP 연결 실패: " + e.getMessage());
}

%>
```

### 익명 로그인

```jsp
SimpleFTP ftp = new SimpleFTP();

// 기본적으로 anonymous/anonymous로 로그인
ftp.connect("ftp.example.com");
```

---

## 파일 업로드

### 단일 파일 업로드

```jsp
SimpleFTP ftp = new SimpleFTP();

try {
    // FTP 서버 연결
    ftp.connect("ftp.example.com", 21, "username", "password");

    // 바이너리 모드 설정 (이미지, 압축파일 등)
    ftp.bin();

    // 파일 업로드
    File file = new File("/local/path/document.pdf");
    boolean success = ftp.stor(file);

    if(success) {
        m.p("파일 업로드 성공");
    } else {
        m.p("파일 업로드 실패");
    }

    // 연결 종료
    ftp.disconnect();

} catch(Exception e) {
    m.p("오류: " + e.getMessage());
}
```

### ASCII 모드

```jsp
// 텍스트 파일 업로드 시
ftp.ascii();
File textFile = new File("/local/path/readme.txt");
ftp.stor(textFile);
```

### InputStream으로 업로드

```jsp
SimpleFTP ftp = new SimpleFTP();
ftp.connect("ftp.example.com", 21, "user", "pass");
ftp.bin();

// InputStream 사용
FileInputStream fis = new FileInputStream("/local/path/image.jpg");
boolean success = ftp.stor(fis, "uploaded_image.jpg");
fis.close();

ftp.disconnect();
```

---

## 디렉토리 작업

### 현재 디렉토리 확인

```jsp
SimpleFTP ftp = new SimpleFTP();
ftp.connect("ftp.example.com", 21, "user", "pass");

String currentDir = ftp.pwd();
m.p("현재 디렉토리: " + currentDir);
// 출력: /home/user
```

### 디렉토리 변경

```jsp
SimpleFTP ftp = new SimpleFTP();
ftp.connect("ftp.example.com", 21, "user", "pass");

// 디렉토리 변경
boolean success = ftp.cwd("/public_html/uploads");

if(success) {
    m.p("디렉토리 변경 성공");
    m.p("현재 위치: " + ftp.pwd());
} else {
    m.p("디렉토리 변경 실패");
}
```

---

## 실제 FTP 사용 예제

### 파일 업로드 폼

**upload_ftp.jsp**
```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

if(m.isPost()) {
    // 로컬에 업로드된 파일
    String uploadedFile = m.rs("uploaded_file_path");

    SimpleFTP ftp = new SimpleFTP();

    try {
        // FTP 연결
        ftp.connect(
            Config.get("ftp.host"),
            Config.getInt("ftp.port"),
            Config.get("ftp.user"),
            Config.get("ftp.pass")
        );

        // 업로드 디렉토리로 이동
        ftp.cwd("/uploads");

        // 바이너리 모드
        ftp.bin();

        // 파일 업로드
        File file = new File(uploadedFile);
        boolean success = ftp.stor(file);

        // 연결 종료
        ftp.disconnect();

        if(success) {
            m.jsAlert("FTP 업로드 성공");
            m.jsReplace("/list");
        } else {
            m.jsError("FTP 업로드 실패");
        }

    } catch(Exception e) {
        Malgn.errorLog("FTP 오류", e);
        m.jsError("FTP 오류: " + e.getMessage());
    }

    return;
}

p.setBody("upload.ftp");
p.display();

%>
```

### 배치 파일 업로드

```jsp
SimpleFTP ftp = new SimpleFTP();
ftp.connect("ftp.example.com", 21, "user", "pass");
ftp.cwd("/uploads");
ftp.bin();

// 여러 파일 업로드
String[] files = {
    "/local/path/file1.jpg",
    "/local/path/file2.jpg",
    "/local/path/file3.jpg"
};

int successCount = 0;
for(String filePath : files) {
    File file = new File(filePath);
    if(ftp.stor(file)) {
        successCount++;
        m.p(file.getName() + " 업로드 완료");
    }
}

ftp.disconnect();
m.p("총 " + successCount + "개 파일 업로드 완료");
```

---

## ZIP 압축 및 해제

Zip 클래스는 파일과 폴더의 압축 및 압축 해제 기능을 제공합니다.

### 파일 압축

#### 단일 파일 압축

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

Zip zip = new Zip();
zip.setDebug(out);

String sourceFile = Config.getDataDir() + "/document.pdf";
String zipFile = Config.getDataDir() + "/document.zip";

boolean success = zip.compress(sourceFile, zipFile);

if(success) {
    m.p("압축 완료: " + zipFile);
} else {
    m.p("압축 실패: " + zip.errMsg);
}

%>
```

#### 여러 파일 압축

```jsp
Zip zip = new Zip();

String[] files = {
    Config.getDataDir() + "/file1.txt",
    Config.getDataDir() + "/file2.txt",
    Config.getDataDir() + "/image.jpg"
};

String zipFile = Config.getDataDir() + "/archive.zip";

boolean success = zip.compress(files, zipFile);

if(success) {
    m.p("압축 완료");
} else {
    m.p("압축 실패: " + zip.errMsg);
}
```

#### 폴더 전체 압축

```jsp
Zip zip = new Zip();

// 폴더 압축
String folderPath = Config.getDataDir() + "/uploads";
String zipFile = Config.getDataDir() + "/uploads_backup.zip";

boolean success = zip.compress(folderPath, zipFile);

if(success) {
    m.p("폴더 압축 완료");
}
```

#### 파일명 변경하여 압축

```jsp
Zip zip = new Zip();

// "원본경로=>압축파일명" 형식
String[] files = {
    Config.getDataDir() + "/temp/report_2025.pdf=>report.pdf",
    Config.getDataDir() + "/temp/data_2025.xlsx=>data.xlsx"
};

String zipFile = Config.getDataDir() + "/files.zip";
zip.compress(files, zipFile);
```

---

## 압축 해제

### 기본 압축 해제

```jsp
Zip zip = new Zip();

String zipFile = Config.getDataDir() + "/archive.zip";
String extractFolder = Config.getDataDir() + "/extracted";

boolean success = zip.extract(zipFile, extractFolder);

if(success) {
    m.p("압축 해제 완료: " + extractFolder);
} else {
    m.p("압축 해제 실패: " + zip.errMsg);
}
```

### File 객체로 압축 해제

```jsp
Zip zip = new Zip();

File zipFile = new File(Config.getDataDir() + "/archive.zip");
String extractFolder = Config.getDataDir() + "/extracted";

boolean success = zip.extract(zipFile, extractFolder);
```

---

## 다운로드 압축

### 즉시 다운로드

브라우저로 직접 압축 파일을 전송할 수 있습니다:

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

Zip zip = new Zip();

String[] files = {
    Config.getDataDir() + "/report.pdf",
    Config.getDataDir() + "/data.xlsx",
    Config.getDataDir() + "/images/photo.jpg"
};

// 파일을 저장하지 않고 바로 다운로드
boolean success = zip.compress(files, "download.zip", response);

if(!success) {
    m.jsError("압축 실패: " + zip.errMsg);
}

%>
```

### 폴더 다운로드

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

Zip zip = new Zip();

String folderPath = Config.getDataDir() + "/uploads/user_123";

// 폴더를 압축하여 바로 다운로드
zip.compress(folderPath, "user_files.zip", response);

%>
```

---

## 실제 활용 예제

### 백업 시스템

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 매일 자정에 실행되는 백업 스크립트

Zip zip = new Zip();
SimpleFTP ftp = new SimpleFTP();

String today = Malgn.time("yyyyMMdd");
String backupFile = Config.getDataDir() + "/backup_" + today + ".zip";

// 1. 데이터 폴더 압축
String dataFolder = Config.getDataDir() + "/uploads";
boolean compressed = zip.compress(dataFolder, backupFile);

if(!compressed) {
    Malgn.errorLog("백업 압축 실패: " + zip.errMsg);
    return;
}

m.p("백업 파일 생성 완료: " + backupFile);

// 2. FTP로 백업 파일 업로드
try {
    ftp.connect("backup.example.com", 21, "backup_user", "backup_pass");
    ftp.cwd("/backups");
    ftp.bin();

    File file = new File(backupFile);
    boolean uploaded = ftp.stor(file);

    ftp.disconnect();

    if(uploaded) {
        m.p("FTP 백업 완료");

        // 로컬 백업 파일 삭제 (선택사항)
        file.delete();
    } else {
        m.p("FTP 업로드 실패");
    }

} catch(Exception e) {
    Malgn.errorLog("FTP 백업 오류", e);
}

%>
```

### 대용량 파일 다운로드

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 사용자가 선택한 파일들을 압축하여 다운로드

String[] fileIds = m.reqArr("file_id");

if(fileIds.length == 0) {
    m.jsError("파일을 선택해주세요");
    return;
}

// 파일 경로 조회
FileDao dao = new FileDao();
DataSet files = dao.query("WHERE id IN (?)", Malgn.implode(",", fileIds));

ArrayList<String> filePaths = new ArrayList<>();
while(files.next()) {
    String path = files.s("file_path");
    File file = new File(path);
    if(file.exists()) {
        filePaths.add(path);
    }
}

if(filePaths.isEmpty()) {
    m.jsError("다운로드 가능한 파일이 없습니다");
    return;
}

// 압축 및 다운로드
Zip zip = new Zip();
String downloadName = "files_" + Malgn.time("yyyyMMddHHmmss") + ".zip";

boolean success = zip.compress(
    filePaths.toArray(new String[0]),
    downloadName,
    response
);

if(!success) {
    m.jsError("압축 오류: " + zip.errMsg);
}

%>
```

### 이미지 갤러리 다운로드

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

int galleryId = m.ri("gallery_id");

// 갤러리의 모든 이미지 조회
ImageDao dao = new ImageDao();
DataSet images = dao.query("WHERE gallery_id = ?", galleryId);

String[] files = new String[images.size()];
int idx = 0;

while(images.next()) {
    String originalName = images.s("original_name");
    String filePath = images.s("file_path");

    // 압축 파일 내에서 원본 파일명 사용
    files[idx++] = filePath + "=>" + originalName;
}

// 압축 다운로드
Zip zip = new Zip();
zip.compress(files, "gallery_" + galleryId + ".zip", response);

%>
```

---

## 디버그 모드

### Zip 디버그

```jsp
Zip zip = new Zip();
zip.setDebug(out);  // 압축 과정 출력

String[] files = {"/path/file1.txt", "/path/file2.txt"};
zip.compress(files, "output.zip");

// 출력:
// /path/file1.txt
// /path/file2.txt
```

### 에러 확인

```jsp
Zip zip = new Zip();

boolean success = zip.compress("/path/file.txt", "output.zip");

if(!success) {
    // 에러 메시지 확인
    m.p("에러: " + zip.errMsg);
    Malgn.errorLog("압축 실패: " + zip.errMsg);
}
```

---

## 한글 파일명 처리

### ZIP 한글 파일명

Zip 클래스는 한글 파일명을 자동으로 처리합니다 (EUC-KR 인코딩):

```jsp
Zip zip = new Zip();

String[] files = {
    Config.getDataDir() + "/보고서.pdf",
    Config.getDataDir() + "/회의록.docx"
};

// 한글 파일명 자동 처리
zip.compress(files, "문서모음.zip", response);
```

### 압축 해제 시 한글 처리

```jsp
Zip zip = new Zip();

// EUC-KR 인코딩된 ZIP 파일 해제
zip.extract("한글파일명.zip", Config.getDataDir() + "/extracted");
```

---

## 주의사항

### FTP

1. **포트**: 기본 FTP 포트는 21
2. **패시브 모드**: SimpleFTP는 패시브 모드를 사용하여 방화벽 문제 최소화
3. **연결 종료**: 작업 완료 후 반드시 `disconnect()` 호출
4. **전송 모드**:
   - 바이너리 파일: `ftp.bin()` 사용
   - 텍스트 파일: `ftp.ascii()` 사용

### ZIP

1. **메모리**: 대용량 파일 압축 시 메모리 사용량 주의
2. **한글 인코딩**: EUC-KR 사용 (한국 환경 최적화)
3. **경로**: 절대 경로 사용 권장
4. **디렉토리 생성**: 압축 해제 시 필요한 디렉토리 자동 생성

---

## 관련 문서

- [유틸리티 메소드](utility-methods.md) - 파일 처리 유틸리티
- [Excel 처리](excel.md) - Excel 파일 생성 및 읽기
- [환경설정 및 캐시](configuration.md) - 경로 설정
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
m.jsBack();

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
posts.first();
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
