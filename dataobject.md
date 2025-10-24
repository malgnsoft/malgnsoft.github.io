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
