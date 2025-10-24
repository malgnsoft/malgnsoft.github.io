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
