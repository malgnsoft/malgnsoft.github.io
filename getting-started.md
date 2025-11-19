# 3. 시작하기

[← 목차로 돌아가기](README.md)

---

## init.jsp 작성

모든 프로그램에서 공통적으로 사용하는 초기화 파일입니다.

### 파일 위치

```
/public_html/init.jsp
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

**/public_html/main/index.jsp**

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
/public_html/main/index.jsp          - JSP 파일 (Controller)
/public_html/html/main/index.html    - HTML 템플릿 (View)
```

### JSP 파일

**/public_html/main/index.jsp**

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

p.setBody("main.index");
p.display();

%>
```

### HTML 템플릿

**/public_html/html/main/index.html**

```html
<html>
<body>
시작 페이지 입니다.
</body>
</html>
```

### 템플릿 경로 규칙

- `main.index` → `/public_html/html/main/index.html`
- 폴더는 `.`(점)으로 구분
- 확장자는 생략 (자동으로 `.html` 추가)

**예시**:
- `main.user` → `/public_html/html/main/user.html`
- `admin.member.list` → `/public_html/html/admin/member/list.html`

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
├── build.xml                   # Ant 빌드 스크립트
├── ivy.xml                     # Ivy 의존성 관리
├── src/                        # Java 소스 파일
│   └── dao/                   # DAO 클래스
│       └── UserDao.java
│
└── public_html/                # 웹 애플리케이션 루트
    ├── init.jsp               # 공통 초기화 파일
    ├── assets/                # 정적 파일
    │   ├── css/              # CSS 파일
    │   │   └── style.css
    │   ├── js/               # JavaScript 파일
    │   │   └── lib.validate.js
    │   └── images/           # 이미지 파일
    ├── main/                  # 메인 모듈
    │   ├── index.jsp         # 메인 페이지
    │   ├── user_list.jsp     # 사용자 목록
    │   ├── user_insert.jsp   # 사용자 등록
    │   └── user_modify.jsp   # 사용자 수정
    ├── data/                  # 데이터 폴더
    │   ├── file/             # 업로드 파일
    │   ├── tmp/              # 임시 파일
    │   └── log/              # 로그 파일
    ├── html/                  # 템플릿 루트
    │   ├── layout/           # 레이아웃 템플릿
    │   │   ├── layout_main.html
    │   │   └── layout_admin.html
    │   └── main/             # 모듈 템플릿
    │       ├── index.html
    │       ├── user_list.html
    │       ├── user_insert.html
    │       └── user_modify.html
    └── WEB-INF/
        ├── lib/
        │   └── malgn.jar
        ├── classes/          # 컴파일된 클래스 파일
        │   └── dao/
        │       └── UserDao.class
        ├── config.xml
        ├── web.xml
        └── resin-web.xml
```

---

## 첫 번째 모듈 만들기

### 1. DAO 클래스 작성

**/src/dao/UserDao.java**

```java
package dao;

import malgnsoft.db.*;

public class UserDao extends DataObject {
    public UserDao() {
        this.table = "tb_user";
    }
}
```

### 2. 의존성 관리 (Apache Ivy)

Apache Ivy를 사용하여 필요한 라이브러리를 자동으로 다운로드합니다.

**ivy.xml** (프로젝트 루트):

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ivy-module version="2.0">
    <info organisation="com.example" module="myproject" revision="1.0.0"/>

    <dependencies>
        <!-- Servlet API -->
        <dependency org="javax.servlet" name="javax.servlet-api" rev="4.0.1" conf="compile->default"/>

        <!-- Apache Commons Compress -->
        <dependency org="org.apache.commons" name="commons-compress" rev="1.21" conf="compile->default"/>

        <!-- MyBatis -->
        <dependency org="org.mybatis" name="mybatis" rev="3.5.11" conf="compile->default"/>

        <!-- Apache POI (Excel 처리) -->
        <dependency org="org.apache.poi" name="poi-ooxml" rev="3.17" conf="compile->default"/>

        <!-- JExcel API -->
        <dependency org="net.sourceforge.jexcelapi" name="jxl" rev="2.6.12" conf="compile->default"/>

        <!-- JXLS (Excel 템플릿) -->
        <dependency org="net.sf.jxls" name="jxls-core" rev="1.0.6" conf="compile->default"/>

        <!-- JWT (JSON Web Token) -->
        <dependency org="io.jsonwebtoken" name="jjwt" rev="0.9.1" conf="compile->default"/>

        <!-- MySQL Connector (필요시) -->
        <dependency org="mysql" name="mysql-connector-java" rev="8.0.33" conf="compile->default"/>

        <!-- Oracle JDBC (필요시 - 주석 제거 후 사용) -->
        <!-- <dependency org="com.oracle.database.jdbc" name="ojdbc8" rev="19.3.0.0" conf="compile->default"/> -->

        <!-- JSON 처리 -->
        <dependency org="com.google.code.gson" name="gson" rev="2.10.1" conf="compile->default"/>

        <!-- Apache Commons FileUpload -->
        <dependency org="commons-fileupload" name="commons-fileupload" rev="1.5" conf="compile->default"/>

        <!-- Apache Commons IO -->
        <dependency org="commons-io" name="commons-io" rev="2.11.0" conf="compile->default"/>
    </dependencies>
</ivy-module>
```

### 3. 빌드 설정 (build.xml)

Apache Ant와 Ivy를 이용하여 의존성 관리 및 컴파일을 수행합니다.

**build.xml** (프로젝트 루트):

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project name="MyProject" default="compile" basedir="."
         xmlns:ivy="antlib:org.apache.ivy.ant">

    <!-- 프로퍼티 설정 -->
    <property name="src.dir" value="src"/>
    <property name="build.dir" value="public_html/WEB-INF/classes"/>
    <property name="lib.dir" value="public_html/WEB-INF/lib"/>

    <!-- Ivy 초기화 -->
    <target name="ivy-init">
        <path id="ivy.lib.path">
            <fileset dir="${user.home}/.ant/lib" includes="ivy-*.jar"/>
        </path>
        <taskdef resource="org/apache/ivy/ant/antlib.xml"
                 uri="antlib:org.apache.ivy.ant"
                 classpathref="ivy.lib.path"/>
    </target>

    <!-- 의존성 다운로드 -->
    <target name="resolve" depends="ivy-init" description="Download dependencies">
        <ivy:retrieve pattern="${lib.dir}/[artifact]-[revision].[ext]"
                      conf="compile"
                      sync="false"/>
        <echo message="Dependencies downloaded to ${lib.dir}"/>
    </target>

    <!-- 클래스패스 설정 -->
    <path id="classpath">
        <fileset dir="${lib.dir}">
            <include name="*.jar"/>
        </fileset>
    </path>

    <!-- 컴파일 타겟 (기본) -->
    <target name="compile" depends="resolve" description="Compile Java sources">
        <mkdir dir="${build.dir}"/>
        <javac srcdir="${src.dir}"
               destdir="${build.dir}"
               encoding="UTF-8"
               includeantruntime="false">
            <classpath refid="classpath"/>
        </javac>
        <echo message="Compilation complete!"/>
    </target>

    <!-- 클린 타겟 -->
    <target name="clean" description="Clean build directory">
        <delete dir="${build.dir}"/>
        <echo message="Build directory cleaned!"/>
    </target>

    <!-- 리빌드 타겟 -->
    <target name="rebuild" depends="clean,compile" description="Clean and compile">
        <echo message="Rebuild complete!"/>
    </target>

</project>
```

**Ivy 설치**:

```bash
# Ivy jar 파일을 Ant의 lib 디렉토리에 복사
# https://ant.apache.org/ivy/download.cgi 에서 다운로드

# 또는 자동 설치 (Linux/Mac)
mkdir -p ~/.ant/lib
wget https://repo1.maven.org/maven2/org/apache/ivy/ivy/2.5.2/ivy-2.5.2.jar -P ~/.ant/lib/

# Windows
# https://repo1.maven.org/maven2/org/apache/ivy/ivy/2.5.2/ivy-2.5.2.jar
# 다운로드 후 %USERPROFILE%\.ant\lib\ 에 복사
```

**빌드 실행**:

```bash
# 의존성 다운로드 + 컴파일 (기본 타겟)
ant

# 의존성만 다운로드
ant resolve

# 컴파일만 (의존성이 이미 있는 경우)
ant compile

# 클린 (빌드 파일 삭제)
ant clean

# 리빌드 (클린 + 컴파일)
ant rebuild
```

**수동 컴파일** (Ant 없이):

```bash
javac -cp public_html/WEB-INF/lib/malgn.jar -d public_html/WEB-INF/classes src/dao/UserDao.java
```

### 4. JSP 파일 작성

**/public_html/main/user_list.jsp**

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

UserDao user = new UserDao();
DataSet list = user.query("SELECT * FROM tb_user ORDER BY id DESC");

p.setBody("main.user_list");
p.setLoop("list", list);
p.display();

%>
```

### 5. HTML 템플릿 작성

**/public_html/html/main/user_list.html**

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

### 4. 에러 메시지와 함께 뒤로 가기

```jsp
m.jsError("에러 메시지");  // 메시지 출력 후 history.back()
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
- `/public_html/html/main/index.html` 파일 존재 확인
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
