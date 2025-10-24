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

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

Pager pg = new Pager(request);
pg.setPageNum(f.getInt("page", 1));  // 현재 페이지
pg.setTotalNum(300);  // 전체 개수
pg.setListNum(20);  // 페이지당 개수
pg.setPageNum(10);  // 페이지 링크 개수
pg.setLinkType(9);  // 링크 타입
pg.setLink("/main/list.jsp");  // 기본 URL

%>
<%= pg.getPager() %>
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
