# 8. DataSet 활용

[← 목차로 돌아가기](README.md)

---

## DataSet이란?

DataSet은 MalgnFramework에서 테이블 형태의 데이터를 다루기 위한 핵심 클래스입니다.

### 주요 특징

- `List<Map<String, Object>>` 구조
- 데이터베이스 조회 결과를 담는 표준 컨테이너
- 행(Row) 단위 순회 및 조작
- 열(Column) 검증 기능
- JSON 변환 지원

---

## 기본 사용법

### 1. DataSet 생성

```jsp
DataSet ds = new DataSet();

// 행 추가
ds.addRow();
ds.put("id", 1);
ds.put("name", "홍길동");
ds.put("email", "hong@example.com");

// 다른 행 추가
ds.addRow();
ds.put("id", 2);
ds.put("name", "김철수");
ds.put("email", "kim@example.com");
```

### 2. 데이터 조회

```jsp
// 처음 행으로 이동
ds.first();

// 다음 행으로 이동하며 순회
while(ds.next()) {
    int id = ds.getInt("id");
    String name = ds.getString("name");
    String email = ds.getString("email");

    m.p(id + " : " + name + " : " + email);
}
```

---

## 행(Row) 조작

### 행 추가 및 이동

```jsp
DataSet ds = new DataSet();

// 새 행 추가
ds.addRow();
ds.put("subject", "첫 번째 게시물");
ds.put("content", "내용입니다.");

ds.addRow();
ds.put("subject", "두 번째 게시물");
ds.put("content", "또 다른 내용입니다.");

// 처음으로 이동
ds.first();

// 다음 행으로 이동
if(ds.next()) {
    m.p(ds.getString("subject"));  // "첫 번째 게시물"
}

if(ds.next()) {
    m.p(ds.getString("subject"));  // "두 번째 게시물"
}
```

### 특정 위치로 이동

```jsp
// 0부터 시작하는 인덱스
ds.moveRow(0);  // 첫 번째 행
ds.moveRow(2);  // 세 번째 행
```

### 현재 행 가져오기

```jsp
DataMap row = ds.getRow();
String subject = row.getString("subject");
int status = row.getInt("status");
```

---

## 데이터 가져오기

### 타입별 메소드

```jsp
// 문자열
String name = ds.getString("name");
String subject = ds.getString("subject", "기본값");

// 숫자
int age = ds.getInt("age");
long id = ds.getLong("id");
double price = ds.getDouble("price");

// 날짜/시간
String regDate = ds.getString("reg_date");

// null 체크가 필요한 경우
Object value = ds.get("field_name");
if(value != null) {
    // 처리
}
```

### 기본값 설정

```jsp
// 값이 null이면 기본값 반환
String title = ds.getString("title", "제목 없음");
int status = ds.getInt("status", 0);
```

---

## DataMap 클래스

DataSet의 각 행은 DataMap 객체입니다.

### 기본 사용

```jsp
DataMap map = new DataMap();
map.put("subject", "안녕하세요");
map.put("content", "반갑습니다");
map.put("reg_date", m.time());
map.put("status", 1);

// 값 가져오기
String subject = map.getString("subject");
int status = map.getInt("status");
```

### JSON 변환

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

DataMap map = new DataMap();
map.put("subject", "Nice to meet you.");
map.put("content", "Nice to meet you too.");
map.put("reg_date", Malgn.time());
map.put("status", 1);

// JSON 인코딩
String json = Json.encode(map);
m.p(json);
// 출력: {"subject":"Nice to meet you.","content":"Nice to meet you too.","reg_date":"20250623153000","status":1}

%>
```

---

## 열(Column) 검증

### setCol() - 검증 규칙 설정

```jsp
DataSet ds = new DataSet();
ds.addRow();

// 컬럼별 검증 규칙 설정
ds.setCol("email", "type:email, required:Y");
ds.setCol("age", "type:number, min:1, max:150");
ds.setCol("phone", "type:phone");

ds.put("email", "invalid-email");
ds.put("age", 200);

// 검증 실행
if(!ds.validate()) {
    m.p("검증 실패");
}
```

### 검증 속성

| 속성 | 설명 | 예제 |
|------|------|------|
| type | 데이터 타입 | `type:email`, `type:number` |
| required | 필수 여부 | `required:Y` |
| minlength | 최소 길이 | `minlength:8` |
| maxlength | 최대 길이 | `maxlength:100` |
| min | 최소값 | `min:0` |
| max | 최대값 | `max:999` |
| length | 정확한 길이 | `length:6` |

**지원하는 타입**:
- `email`, `url`, `number`, `phone`, `datetime`, `hangul` 등

---

## JSON 처리

### JSON 클래스 (j)

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="/init.jsp" %><%

DataMap map = new DataMap();
map.put("id", 123);
map.put("name", "홍길동");
map.put("email", "hong@example.com");

// 성공 응답
j.success("조회 성공", map);
// 출력: {"result":"success","message":"조회 성공","data":{...}}

%>
```

### JSON 응답 메소드

```jsp
// 에러 응답
j.error(100, "잘못된 요청입니다");
// 출력: {"result":"error","code":100,"message":"잘못된 요청입니다"}

// 성공 응답 (데이터 포함)
j.success("성공", dataMap);
// 출력: {"result":"success","message":"성공","data":{...}}

// 커스텀 응답
j.print(0, "완료되었습니다", dataMap);
// 출력: {"code":0,"message":"완료되었습니다","data":{...}}
```

---

## 데이터베이스 연동

### 조회 결과를 DataSet으로

```jsp
UserDao dao = new UserDao();
DataSet list = dao.query("WHERE status = 1 ORDER BY id DESC");

list.first();
while(list.next()) {
    m.p(list.getInt("id"));
    m.p(list.getString("name"));
    m.p(list.getString("email"));
}
```

### ListManager에서 DataSet 사용

```jsp
ListManager lm = new ListManager();
lm.setRequest(request);
lm.setTable("tb_blog");
lm.setListNum(20);
lm.addSearch("subject,content", keyword, "LIKE");

DataSet list = lm.getDataSet();

// 템플릿에 전달
p.setLoop("list", list);
```

---

## 유틸리티 메소드

### 날짜/시간 처리

```jsp
// 현재 시각
String now = m.time();  // "20250623153045"

// 포맷 지정
String today = m.time("yyyyMMdd");  // "20250623"
String formatted = m.time("yyyy-MM-dd HH:mm:ss");  // "2025-06-23 15:30:45"

// 날짜 계산
String nextMonth = m.addDate("M", 1, m.time(), "yyyyMMdd");  // 1개월 후
String yesterday = m.addDate("D", -1, m.time(), "yyyyMMdd");  // 1일 전

// 날짜 차이 계산
long days = m.diffDate("D", "20000101", m.time("yyyyMMdd"));
m.p("2000년 1월 1일부터 " + days + "일 지났습니다");

// Unix timestamp
long timestamp = m.getUnixTimeL("29991231");

// 시간대 변환
Date d = m.strToDate("yyyyMMddHHmmss", "20230425113600", "UTC");
String seoulTime = m.time("yyyyMMddHHmmss", d, "Asia/Seoul");
```

### 문자열 처리

```jsp
// 왼쪽 패딩
String padded = m.strpad("abc", 10, "0");  // "0000000abc"

// 오른쪽 패딩
String rpadded = m.strrpad("abc", 10, "0");  // "abc0000000"

// 문자열 자르기 (한글 고려)
String cut = m.cutString("우리는 자랑스러운 한국인이다", 10, "...");
// "우리는 자랑스..."

// Hex 변환
byte[] bytes = new byte[] {'a', 'b', 'c', 'd'};
String hex = Malgn.bytesToHex(bytes);  // "61626364"

byte[] decoded = Malgn.hexToBytes("61626364");
String original = new String(decoded);  // "abcd"

// 숫자 파싱
double price = Malgn.parseDouble("1.00");  // 1.0
```

---

## 실전 예제

### 게시판 목록 처리

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

ListManager lm = new ListManager();
lm.setRequest(request);
lm.setTable("tb_blog");
lm.setListNum(20);
lm.setOrderBy("id DESC");

DataSet list = lm.getDataSet();

// 날짜 포맷팅 추가
list.first();
while(list.next()) {
    String regDate = list.getString("reg_date");
    String formatted = m.time("yyyy-MM-dd", regDate);
    list.put("formatted_date", formatted);
}

p.setBody("main.blog_list");
p.setLoop("list", list);
p.display();

%>
```

### API 응답 처리

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="/init.jsp" %><%

try {
    UserDao dao = new UserDao();
    DataSet users = dao.query("WHERE status = 1 ORDER BY name ASC");

    DataMap result = new DataMap();
    result.put("total", users.size());
    result.put("users", users);

    j.success("사용자 목록 조회 성공", result);

} catch(Exception e) {
    j.error(500, "서버 오류: " + e.getMessage());
}

%>
```

### 데이터 검증 및 저장

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

f.addElement("name", null, "required:Y, minlength:2");
f.addElement("email", null, "type:email, required:Y");
f.addElement("age", null, "type:number, min:1, max:150");

if(m.isPost() && f.validate()) {

    DataMap data = new DataMap();
    data.put("name", f.get("name"));
    data.put("email", f.get("email"));
    data.put("age", f.getInt("age"));
    data.put("reg_date", m.time());

    UserDao dao = new UserDao();
    int newId = dao.insert(data);

    m.jsAlert("등록되었습니다. ID: " + newId);
    m.jsReplace("list.jsp");
    return;
}

%>
```

---

## 주요 메소드 정리

### DataSet

| 메소드 | 설명 |
|--------|------|
| addRow() | 새 행 추가 |
| first() | 첫 행으로 이동 (실제로 -1 위치) |
| next() | 다음 행으로 이동 및 존재 여부 반환 |
| moveRow(index) | 특정 인덱스로 이동 |
| getRow() | 현재 행의 DataMap 반환 |
| size() | 전체 행 개수 |
| put(key, value) | 현재 행에 값 설정 |
| get(key) | 현재 행에서 값 가져오기 |
| getString(key) | 문자열로 가져오기 |
| getInt(key) | 정수로 가져오기 |
| getLong(key) | Long으로 가져오기 |
| getDouble(key) | Double로 가져오기 |
| setCol(name, rules) | 컬럼 검증 규칙 설정 |
| validate() | 전체 행 검증 |

### DataMap

| 메소드 | 설명 |
|--------|------|
| put(key, value) | 값 설정 |
| get(key) | 값 가져오기 |
| getString(key) | 문자열로 가져오기 |
| getString(key, defaultValue) | 기본값을 가진 문자열 |
| getInt(key) | 정수로 가져오기 |
| getInt(key, defaultValue) | 기본값을 가진 정수 |
| getLong(key) | Long으로 가져오기 |
| getDouble(key) | Double로 가져오기 |

---

## 베스트 프랙티스

### 1. null 체크

```jsp
// Bad
String name = ds.getString("name");
if(name.equals("admin")) {  // NullPointerException 가능
    // ...
}

// Good
String name = ds.getString("name", "");
if("admin".equals(name)) {  // 안전
    // ...
}
```

### 2. 타입 명시

```jsp
// Bad
Object value = ds.get("age");
int age = (Integer)value;  // ClassCastException 가능

// Good
int age = ds.getInt("age");
```

### 3. 순회 패턴

```jsp
// 항상 first() 후 next()
ds.first();
while(ds.next()) {
    // 처리
}
```

---

## 다음 단계

- [XML 처리](xml.md)로 이동하여 XML 파싱 방법을 배워보세요.

---

[← 이전: 목록 및 검색](list-search.md) | [목차로 돌아가기](README.md) | [다음: XML 처리 →](xml.md)
