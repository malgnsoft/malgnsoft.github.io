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
├── html/                   - 템플릿 루트
│   ├── css/               - CSS 파일
│   ├── js/                - JavaScript 파일
│   ├── images/            - 이미지 파일
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
mkdir html
mkdir html\css
mkdir html\js
mkdir html\images
mkdir html\layout
mkdir html\main
mkdir main
```

### 폴더 생성 스크립트 (Linux/Mac)

```bash
mkdir -p data/{file,tmp,log}
mkdir -p html/{css,js,images,layout,main}
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
