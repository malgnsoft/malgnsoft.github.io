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

## 읽기/쓰기 분리 (Read/Write Splitting)

### 개요

대형 사이트에서는 읽기 성능 향상을 위해 Master-Slave 구조의 데이터베이스 복제 환경을 사용합니다. 맑은프레임워크는 `rojndi` (Read-Only JNDI) 설정을 통해 자동으로 읽기/쓰기를 분리합니다.

### config.xml 설정

```xml
<?xml version="1.0" encoding="UTF-8"?>
<config>
    <env>
        <!-- 쓰기용 데이터베이스 (Master) -->
        <jndi>jdbc/myapp</jndi>

        <!-- 읽기 전용 데이터베이스 (Slave) -->
        <rojndi>jdbc/myapp_ro</rojndi>
    </env>
</config>
```

### 자동 분리 원리

- **INSERT, UPDATE, DELETE**: `jndi` (Master DB) 사용
- **SELECT**: `rojndi` (Slave DB) 사용
- `rojndi`가 설정되지 않은 경우: 모든 쿼리가 `jndi` 사용

### 사용 예제

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

UserDao user = new UserDao();

// 읽기 쿼리 - Slave DB(rojndi) 사용
DataSet userList = user.query("SELECT * FROM tb_user WHERE status = 1");

// 쓰기 작업 - Master DB(jndi) 사용
user.item("name", "홍길동");
user.item("email", "hong@example.com");
user.insert();

// 수정 작업 - Master DB(jndi) 사용
user.item("status", 1);
user.update("id = 123");

%>
```

### Slave DB가 여러 개인 경우

Slave DB가 여러 대인 경우, WAS의 JNDI 설정에서 로드 밸런싱을 구성할 수 있습니다.

**Tomcat의 context.xml 예시**:
```xml
<Resource name="jdbc/myapp_ro"
    auth="Container"
    type="javax.sql.DataSource"
    driverClassName="com.mysql.jdbc.Driver"
    url="jdbc:mysql:loadbalance://slave1:3306,slave2:3306,slave3:3306/mydb"
    username="readonly_user"
    password="password"
    maxTotal="20"
    maxIdle="10"
    maxWaitMillis="10000"/>
```

### 장점

1. **성능 향상**: 읽기 부하를 여러 Slave DB에 분산
2. **가용성 향상**: Slave DB 장애 시에도 Master DB로 fallback 가능
3. **코드 수정 불필요**: 설정만으로 읽기/쓰기 분리 자동 적용
4. **확장성**: Slave DB 추가로 읽기 성능 선형적 확장

### 주의사항

1. **복제 지연**: Master에서 쓰기 직후 Slave에서 읽을 때 데이터가 아직 복제되지 않을 수 있음
2. **일관성 필요 시**: 쓰기 직후 읽기가 필요한 경우 명시적으로 Master DB 사용 고려

---

## 다중 데이터베이스 연동

### 개요

하나의 애플리케이션에서 여러 데이터베이스를 동시에 사용할 수 있습니다. Dao와 ListManager에서 `setJndi()` 메소드를 사용하여 연결할 데이터베이스를 지정할 수 있습니다.

### config.xml 설정

```xml
<?xml version="1.0" encoding="UTF-8"?>
<config>
    <env>
        <!-- 메인 데이터베이스 -->
        <jndi>jdbc/myapp</jndi>
        <rojndi>jdbc/myapp_ro</rojndi>
    </env>
</config>
```

**Tomcat의 context.xml에 추가 데이터베이스 설정**:
```xml
<!-- 메인 데이터베이스 -->
<Resource name="jdbc/myapp"
    auth="Container"
    type="javax.sql.DataSource"
    driverClassName="com.mysql.jdbc.Driver"
    url="jdbc:mysql://localhost:3306/myapp"
    username="user"
    password="password"
    maxTotal="20"/>

<!-- 로그 데이터베이스 -->
<Resource name="jdbc/log_db"
    auth="Container"
    type="javax.sql.DataSource"
    driverClassName="com.mysql.jdbc.Driver"
    url="jdbc:mysql://log-server:3306/log_db"
    username="log_user"
    password="password"
    maxTotal="10"/>

<!-- 통계 데이터베이스 -->
<Resource name="jdbc/stats_db"
    auth="Container"
    type="javax.sql.DataSource"
    driverClassName="com.mysql.jdbc.Driver"
    url="jdbc:mysql://stats-server:3306/stats_db"
    username="stats_user"
    password="password"
    maxTotal="10"/>
```

### Dao에서 JNDI 지정

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 메인 데이터베이스의 사용자 정보 조회
UserDao userDao = new UserDao();
DataSet users = userDao.query("SELECT * FROM tb_user WHERE status = 1");

// 로그 데이터베이스에 로그 기록
LogDao log = new LogDao();
logDao.setJndi("jdbc/log_db");  // 로그 전용 DB 지정

logDao.item("user_id", userId);
logDao.item("action", "login");
logDao.item("ip_address", m.getRemoteAddr());
logDao.item("reg_date", m.time("yyyyMMddHHmmss"));
logDao.insert();

// 통계 데이터베이스에서 조회
StatsDao statsDao = new StatsDao();
statsDao.setJndi("jdbc/stats_db");  // 통계 전용 DB 지정
DataSet stats = statsDao.query("SELECT * FROM tb_daily_stats WHERE stat_date = ?", new Object[]{m.time("yyyyMMdd")});

%>
```

### ListManager에서 JNDI 지정

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 메인 DB의 게시판 목록
BoardDao boardDao = new BoardDao();
ListManager lm = new ListManager(boardDao, m.req("page"), 20);
lm.setSearchCond(searchCond);
DataSet boardList = lm.getDataSet();

// 로그 DB의 접속 기록 목록
LogDao log = new LogDao();
logDao.setJndi("jdbc/log_db");  // 로그 DB 지정

ListManager logLm = new ListManager(logDao, m.req("page"), 50);
logLm.setSearchField("action");
logLm.setSearchValue("login");
DataSet logList = logLm.getDataSet();

p.setBody("admin.dashboard");
p.setLoop("boardList", boardList);
p.setLoop("logList", logList);
p.display();

%>
```

### 실전 예제

#### 1. 로그 분리 아키텍처

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 메인 DB - 비즈니스 로직 처리
UserDao user = new UserDao();
DataSet info = user.find("id = ?", new Object[]{userId});

if(info.next()) {
    // 사용자 정보 업데이트
    user.item("last_login", m.time("yyyyMMddHHmmss"));
    user.update("id = ?", new Object[] { userId });

    // 로그 DB - 접속 이력 저장
    AccessLogDao accessLog = new AccessLogDao();
    accessLog.setJndi("jdbc/log_db");

    accessLog.item("user_id", userId);
    accessLog.item("login_time", m.time("yyyyMMddHHmmss"));
    accessLog.item("ip_address", m.getRemoteAddr());
    accessLog.item("user_agent", request.getHeader("User-Agent"));
    accessLog.insert();
}

%>
```

#### 2. 마이크로서비스 스타일 DB 분리

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 주문 DB
OrderDao orderDao = new OrderDao();
orderDao.setJndi("jdbc/order_db");
DataSet orders = orderDao.query("WHERE user_id = ?", new Object[]{userId});

// 상품 DB
ProductDao productDao = new ProductDao();
productDao.setJndi("jdbc/product_db");
DataSet products = productDao.query("WHERE id IN (SELECT product_id FROM tb_order WHERE user_id = ?)", new Object[]{userId});

// 결제 DB
PaymentDao paymentDao = new PaymentDao();
paymentDao.setJndi("jdbc/payment_db");
DataSet payments = paymentDao.query("WHERE user_id = ?", new Object[]{userId});

p.setBody("user.order_history");
p.setLoop("orders", orders);
p.setLoop("products", products);
p.setLoop("payments", payments);
p.display();

%>
```

#### 3. 레거시 시스템 통합

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 신규 시스템 DB
UserDao newUserDao = new UserDao();
DataSet newUsers = newUserDao.query("SELECT * FROM tb_user WHERE reg_date >= '20240101'");

// 레거시 시스템 DB
UserDao legacyUserDao = new UserDao();
legacyUserDao.setJndi("jdbc/legacy_db");
DataSet legacyUsers = legacyUserDao.query("SELECT * FROM old_user_table WHERE created_at < '2024-01-01'");

// 두 시스템의 데이터 병합
DataSet allUsers = new DataSet();
while(newUsers.next()) {
    allUsers.addRow();
    allUsers.put("id", newUsers.s("id"));
    allUsers.put("name", newUsers.s("name"));
    allUsers.put("source", "new");
}
while(legacyUsers.next()) {
    allUsers.addRow();
    allUsers.put("id", legacyUsers.s("user_id"));
    allUsers.put("name", legacyUsers.s("user_name"));
    allUsers.put("source", "legacy");
}

p.setBody("admin.user_list");
p.setLoop("users", allUsers);
p.display();

%>
```

### 사용 시나리오

1. **로그/감사 추적 분리**: 로그 데이터를 별도 DB에 저장하여 메인 DB 부하 감소
2. **읽기 전용 분석 DB**: 대용량 분석 쿼리를 별도 DB로 분리
3. **마이크로서비스**: 도메인별로 DB를 분리하여 독립적으로 운영
4. **멀티테넌시**: 고객사별로 독립된 DB 사용
5. **레거시 시스템 통합**: 기존 시스템 DB와 신규 시스템 DB 동시 사용

### 주의사항

1. **분산 트랜잭션**: 여러 DB에 걸친 트랜잭션은 복잡하므로 가능한 피하거나 별도 처리 필요
2. **연결 풀 관리**: 각 DB마다 적절한 연결 풀 크기 설정 필요
3. **성능 모니터링**: 여러 DB 쿼리 시 성능 영향 고려
4. **데이터 일관성**: DB 간 데이터 동기화가 필요한 경우 별도 전략 수립

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
