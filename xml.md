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

UserDao user = new UserDao();
DataSet userList = user.query("WHERE status = 1");

StringBuilder xml = new StringBuilder();
xml.append("<?xml version=\"1.0\" encoding=\"UTF-8\"?>\n");
xml.append("<users>\n");

while(userList.next()) {
    xml.append("  <user>\n");
    xml.append("    <id>").append(userList.getInt("id")).append("</id>\n");
    xml.append("    <name>").append(Malgn.escapeXml(userList.getString("name"))).append("</name>\n");
    xml.append("    <email>").append(Malgn.escapeXml(userList.getString("email"))).append("</email>\n");
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
