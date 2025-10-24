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
