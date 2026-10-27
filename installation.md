# 2. 설치 및 환경설정

[← 목차로 돌아가기](README.md)

---

> 보안 설정은 [보안 설정 모음](security-settings.md), 멀티테넌시는 [멀티테넌시](multitenancy.md)를 참고한다.

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

#### 운영 권장값

Resin 풀은 기본값이 느슨해서 `max-connections`만 줄여도 상한이 지켜지지 않는다. 운영에서는 아래 값을 시작값으로 쓰고 동시 접속량과 DB 서버의 `max_connections`에 맞춰 조정한다. 전체 주석본은 `public_html/WEB-INF/resin-web.xml.sample`이다.

```xml
<web-app xmlns="http://caucho.com/ns/resin" xmlns:malgn="urn:java:malgnsoft.util">
    <database>
        <jndi-name>jdbc/malgn</jndi-name>
        <driver type="com.mysql.cj.jdbc.Driver">
            <url>jdbc:mysql://DB호스트:3306/DB이름?useSSL=false&amp;serverTimezone=Asia/Seoul&amp;characterEncoding=UTF-8</url>
            <user>DB사용자</user>
            <password><malgn:ResinPassword value="ENC:v1:..."/></password>
        </driver>
        <max-connections>10</max-connections>
        <max-overflow-connections>0</max-overflow-connections>
        <max-create-connections>1</max-create-connections>
        <connection-wait-time>5s</connection-wait-time>
        <max-active-time>10m</max-active-time>
        <max-idle-time>30s</max-idle-time>
    </database>
</web-app>
```

| 설정 | Resin 기본 | 권장 |
|------|-----------|------|
| `max-connections` | 1024 | 5~10부터 시작 |
| `max-overflow-connections` | 1024 | **0** (상한을 넘는 연결을 만들지 않음) |
| `max-create-connections` | 5 | **1** (첫 요청이 몰려도 상한 유지) |
| `connection-wait-time` | 30s | 3~5s (풀이 차면 빨리 실패) |
| `max-active-time` | 6h | 10m (엑셀·배치가 더 걸리면 늘림) |
| `max-idle-time` | 60s | 30s |

- 기본값에서는 풀이 찬 뒤 대기 시간이 지나면 상한을 넘어 연결을 더 만들어, 느린 쿼리가 몰릴 때 DB 연결이 폭증한다.
- 대기 시간이 길면 요청이 스레드를 쥔 채 기다려 Resin 스레드가 고갈된다.
- 반납하지 않은 연결(누수·멈춘 요청)이 `max-active-time` 동안 풀을 막는다.
- 시간 표기에서 소문자 `m`은 분, 대문자 `M`은 월이다.

**DB 비밀번호 암호화(ResinPassword)**: 평문 대신 `xmlns:malgn="urn:java:malgnsoft.util"`을 선언하고 `<malgn:ResinPassword value="ENC:v1:..."/>`를 쓴다. 값은 `MALGN_VAULT_KEYFILE=키파일경로 bin/malgn vault wrap resin`으로 만든다(비밀번호는 명령행 인자가 아니라 프롬프트/표준입력으로 입력). malgn JAR가 Resin 공통 lib에 있어야 하며, 복호화 키는 시스템 속성·환경변수·키 파일에서 읽는다. 풀지 못하면 시작 때 예외가 난다.

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

## 운영 도구 bin/malgn

저장소의 `bin/malgn`은 설치 후 점검과 키 관리를 돕는 명령행 도구이다(`bin/malgn <명령>`).

| 명령 | 설명 |
|------|------|
| `doctor [웹앱폴더] [--uses-aes]` | 설정·키·라이브러리 점검. 종료 코드 0 정상, 1 실패, 2 경고 |
| `auth-keys new [파일]` | 새 `auth.keys` 생성 |
| `auth-keys rotate [파일] [보관개수]` | 키 교체(새 키를 맨 위에 추가) |
| `auth-keys check [파일]` | `auth.keys` 점검 |
| `vault gen-key <파일>` | 무작위 마스터 키 파일 생성(권한 600, 이미 있으면 거부) |
| `vault wrap <용도>` | 평문을 `ENC:` 값으로 감쌈. 용도: config, tenants, aes, resin, dek |
| `aes-key [파일] [비트]` | 새 무작위 `aes.key` (전환 기간 전용) |
| `aes-legacy-export <옛 JAR> [파일]` | 옛 JAR의 내장 키를 `aes.key`로 기록 (전환 기간 전용) |
| `aes-check [파일] [--uses-aes] [--old-jar <옛 JAR>]` | `aes.key` 점검 (전환 기간 전용) |
| `jsp-compile <웹앱> ...` | JSP 병렬 사전 컴파일 (아래 절 참고) |

`vault wrap`의 마스터 키는 `MALGN_VAULT_KEY` 또는 `MALGN_VAULT_KEYFILE`로 지정한다. `aes-*` 명령은 SimpleAES·AES 전환 기간용이며 신규 코드에서는 쓰지 않는다.

---

## JSP 사전 컴파일과 배포

빌드 서버에서 JSP를 미리 컴파일해 두면 첫 요청 지연이 없고, 운영 서버에서 컴파일을 끌 수 있다. 근거는 `RESIN-SECURITY-GUIDE.md` 6절이다.

```bash
bin/malgn jsp-compile public_html --jobs 8 --skip-excluded
```

- `<웹앱>`만 주면 출력 폴더는 `<웹앱>/WEB-INF/work`이다. `RESIN_HOME`과 JDK 8이 필요하다.
- `--jobs N`: 동시 JVM 수(기본 CPU 수, 최대 8).
- `--changed`: 이전 컴파일 이후 바뀐 JSP만 컴파일한다. jar·클래스·태그·설정이 바뀌었으면 링크 오류를 막으려고 자동으로 전체를 다시 컴파일한다.
- `--skip-excluded`: 컴파일이 안 되는 것으로 기록된 JSP(`excluded.txt`)는 건너뛴다.

### 불변 배포

클래스만 배포하고 운영 서버는 자동 컴파일을 끄는 방식이다.

```xml
<!-- resin-web.xml -->
<jsp auto-compile="false"/>
```

- JSP 소스 없이 클래스만으로 동작하는 것을 시험으로 확인했다(`jsp:include`, `jsp:forward`, 정적 `<%@ include %>` 포함).
- 클래스가 없는 JSP는 404가 된다. 서버가 대신 컴파일하지 않으므로 `.jsp`가 업로드되어도 실행되지 않는다.
- `_jsp`는 `WEB-INF/work`에 둔다. `WEB-INF/classes` 아래에 두면 소스를 고쳐도 옛 클래스가 계속 쓰인다.
- 파일 단위로 덮어쓰지 말고 폴더째 교체한 뒤 롤링 재시작한다. `classes` 아래가 바뀌면 Resin이 웹앱을 재시작한다.
- 컴파일이 안 되는 JSP는 먼저 고치거나 없앤다. 서버에서 JSP를 생성·편집하는 기능은 쓸 수 없다.

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
