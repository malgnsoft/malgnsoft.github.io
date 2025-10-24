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

p.setLoop("notice", list);
p.setBody("main.notice_latest");
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
