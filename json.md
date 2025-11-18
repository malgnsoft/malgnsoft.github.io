# JSON 처리

맑은프레임워크의 Json 클래스는 JSON 데이터의 생성, 파싱, 변환을 위한 포괄적인 기능을 제공합니다.

## 목차

- [기본 사용법](#기본-사용법)
- [JSON 파싱](#json-파싱)
- [JSON 생성](#json-생성)
- [JSON 응답](#json-응답)
- [정적 메소드](#정적-메소드)

---

## 기본 사용법

### Json 객체 생성

```jsp
// 빈 JSON 객체 생성
Json j = new Json();

// JSON 문자열로 초기화
Json j = new Json("{\"name\":\"John\",\"age\":30}");

// URL에서 JSON 가져오기
Json j = new Json("https://api.example.com/data");

// Writer와 함께 생성 (응답 출력용)
Json j = new Json(out);
```

### 디버그 모드

```jsp
Json j = new Json();
j.setDebug(out);  // Writer와 함께
j.setDebug();     // 로그 파일로
```

---

## JSON 파싱

### JSON 문자열 파싱

```jsp
Json j = new Json();
j.setJson("{\"name\":\"John\",\"age\":30,\"address\":{\"city\":\"Seoul\"}}");

// 경로로 값 가져오기
String name = j.getString("//name");      // "John"
int age = j.getInt("//age");              // 30
String city = j.getString("//address/city"); // "Seoul"
```

### 배열 처리

```jsp
String jsonArray = "[{\"id\":1,\"name\":\"A\"},{\"id\":2,\"name\":\"B\"}]";
Json j = new Json(jsonArray);

// 배열 요소 접근
String name = j.getString("//0/name");  // "A"
int id = j.getInt("//1/id");            // 2

// JSONArray로 가져오기
JSONArray arr = j.getJSONArray("//");
```

### DataSet으로 변환

```jsp
String json = "{\"users\":[{\"name\":\"John\",\"age\":30},{\"name\":\"Jane\",\"age\":25}]}";
Json j = new Json(json);

// JSON 배열을 DataSet으로 변환
DataSet users = j.getDataSet("//users");
while(users.next()) {
    m.p(users.s("name") + ": " + users.i("age"));
}
```

### Map으로 변환

```jsp
Json j = new Json("{\"name\":\"John\",\"age\":30}");
Map<String, Object> map = j.getMap("//");
```

---

## JSON 생성

### 객체 생성

```jsp
Json j = new Json();

// 키-값 추가
j.put("name", "John");
j.put("age", 30);
j.put("active", true);

// Map으로 한번에 설정
DataMap map = new DataMap();
map.put("subject", "Hello");
map.put("content", "World");
j.put(map);

// 문자열로 변환
String json = j.toString();
// {"name":"John","age":30,"active":true}
```

### 배열 생성

```jsp
Json j = new Json();

// 배열 요소 추가
j.put("Apple");
j.put("Banana");
j.put("Cherry");

// List로 한번에 설정
List<Object> list = new ArrayList<>();
list.add("Apple");
list.add("Banana");
j.put(list);
```

---

## JSON 응답

### 기본 응답 형식

Json 클래스는 표준 API 응답 형식을 제공합니다:

**성공 응답**:
```json
{
  "success": true,
  "message": "success",
  "data": { ... }
}
```

**에러 응답**:
```json
{
  "success": false,
  "error": "ERROR_CODE",
  "message": "에러 메시지",
  "details": { ... }
}
```

### 성공 응답

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 간단한 성공 응답
j.success();
// {"success":true,"message":"success"}

// 메시지와 함께
j.success("작업이 완료되었습니다");
// {"success":true,"message":"작업이 완료되었습니다"}

// 데이터와 함께 (Map)
DataMap data = new DataMap();
data.put("id", 123);
data.put("name", "John");
j.success("조회 성공", data);
// {"success":true,"message":"조회 성공","data":{"id":123,"name":"John"}}

// 데이터와 함께 (List)
List<DataMap> list = new ArrayList<>();
// ... list에 데이터 추가
j.success("목록 조회 성공", list);
// {"success":true,"message":"목록 조회 성공","data":[...]}

%>
```

### DataSet 자동 처리

DataSet을 전달하면 레코드 개수에 따라 자동으로 처리됩니다:

```jsp
UserDao user = new UserDao();

// 단일 레코드 - Map 형태로 반환
DataSet info = user.find("id = 1");
info.next();
j.success(info);
// {"success":true,"message":"success","data":{"id":1,"name":"John",...}}

// 복수 레코드 - Array 형태로 반환
DataSet list = user.find();
j.success(list);
// {"success":true,"message":"success","data":[{"id":1,"name":"John"},{"id":2,"name":"Jane"}]}
```

### 에러 응답

```jsp
// 에러 코드와 메시지
j.error("NOT_FOUND", "데이터를 찾을 수 없습니다");
// {"success":false,"error":"NOT_FOUND","message":"데이터를 찾을 수 없습니다"}

// 상세 정보 포함
DataMap details = new DataMap();
details.put("field", "email");
details.put("value", "test@example.com");
j.error("INVALID_INPUT", "유효하지 않은 이메일입니다", details);
// {"success":false,"error":"INVALID_INPUT","message":"유효하지 않은 이메일입니다","details":{...}}

// 간단한 에러 (구버전 호환)
j.error("오류가 발생했습니다");
// {"success":false,"error":"ERROR","message":"오류가 발생했습니다"}
```

### 에러 코드 예시

```jsp
// 일반적인 HTTP 상태 코드 스타일
j.error("BAD_REQUEST", "잘못된 요청입니다");
j.error("UNAUTHORIZED", "인증이 필요합니다");
j.error("FORBIDDEN", "권한이 없습니다");
j.error("NOT_FOUND", "데이터를 찾을 수 없습니다");
j.error("CONFLICT", "이미 존재하는 데이터입니다");
j.error("VALIDATION_ERROR", "입력값 검증 실패");
j.error("INTERNAL_ERROR", "서버 내부 오류");
```

### 실제 사용 예제

#### 로그인 API

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 폼 검증
f.addElement("email", null, "required:Y, email:Y");
f.addElement("password", null, "required:Y, minlength:8");

if(!f.validate()) {
    j.error("VALIDATION_ERROR", f.getErrorString());
    return;
}

// 사용자 조회
UserDao user = new UserDao();
DataSet info = user.query("WHERE email = ?", f.get("email"));

if(!info.next()) {
    j.error("NOT_FOUND", "사용자를 찾을 수 없습니다");
    return;
}

// 비밀번호 확인
if(!info.s("password").equals(Malgn.sha256(f.get("password")))) {
    j.error("UNAUTHORIZED", "비밀번호가 일치하지 않습니다");
    return;
}

// 성공 응답 - DataSet을 직접 전달 (자동으로 Map 변환)
j.success("로그인 성공", info);
// {"success":true,"message":"로그인 성공","data":{"id":1,"name":"John","email":"john@example.com"}}

%>
```

#### 목록 조회 API

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="/init.jsp" %><%

UserDao user = new UserDao();
DataSet list = user.find("status = 1");

// 복수 레코드는 자동으로 Array로 변환
j.success("사용자 목록 조회 성공", list);
// {"success":true,"message":"사용자 목록 조회 성공","data":[{"id":1,"name":"John"},{"id":2,"name":"Jane"}]}

%>
```

#### 데이터 생성 API

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="/init.jsp" %><%

f.addElement("name", null, "required:Y");
f.addElement("email", null, "required:Y, email:Y");

if(!f.validate()) {
    j.error("VALIDATION_ERROR", f.getErrorString());
    return;
}

UserDao user = new UserDao();
user.item("name", f.get("name"));
user.item("email", f.get("email"));
user.item("reg_date", m.time());

int newId = user.insert();

if(newId > 0) {
    DataMap result = new DataMap();
    result.put("id", newId);
    j.success("사용자가 등록되었습니다", result);
} else {
    j.error("INTERNAL_ERROR", "등록 중 오류가 발생했습니다");
}

%>
```

---

## 정적 메소드

### 인코딩

```jsp
// Map을 JSON 문자열로
DataMap map = new DataMap();
map.put("subject", "Nice to meet you.");
map.put("content", "Nice to meet you too.");
map.put("reg_date", Malgn.time());
map.put("status", 1);

String json = Json.encode(map);
m.p(json);
// {"subject":"Nice to meet you.","content":"Nice to meet you too.","reg_date":"2025-10-23 14:30:00","status":1}

// List를 JSON 문자열로
List<String> list = new ArrayList<>();
list.add("Apple");
list.add("Banana");
String jsonArray = Json.encode(list);
// ["Apple","Banana"]
```

### 디코딩

```jsp
// JSON 문자열을 DataSet으로
String json = "{\"name\":\"John\",\"age\":30}";
DataSet rs = Json.decode(json);
rs.next();
m.p(rs.s("name"));  // "John"
m.p(rs.i("age"));   // 30

// 배열 JSON도 처리 가능
String jsonArray = "[{\"name\":\"John\"},{\"name\":\"Jane\"}]";
DataSet users = Json.decode(jsonArray);
while(users.next()) {
    m.p(users.s("name"));
}
```

### Map/List 변환

```jsp
// JSON 문자열을 HashMap으로
String json = "{\"name\":\"John\",\"age\":30}";
HashMap<String, Object> map = Json.toMap(json);

// JSONObject를 HashMap으로
JSONObject obj = new JSONObject(json);
HashMap<String, Object> map2 = Json.toMap(obj);

// JSON 배열 문자열을 ArrayList로
String jsonArray = "[\"Apple\",\"Banana\",\"Cherry\"]";
ArrayList<Object> list = Json.toList(jsonArray);

// JSONArray를 ArrayList로
JSONArray arr = new JSONArray(jsonArray);
ArrayList<Object> list2 = Json.toList(arr);
```

---

## 중첩된 객체 처리

```jsp
String json = "{\"user\":{\"name\":\"John\",\"address\":{\"city\":\"Seoul\",\"zip\":\"12345\"}}}";
Json j = new Json(json);

// 경로를 통한 깊은 접근
String name = j.getString("//user/name");           // "John"
String city = j.getString("//user/address/city");   // "Seoul"
String zip = j.getString("//user/address/zip");     // "12345"

// 중첩된 객체를 Map으로
Map<String, Object> address = j.getMap("//user/address");
// {"city":"Seoul","zip":"12345"}

// JSONObject로 직접 가져오기
JSONObject userObj = j.getJSONObject("//user");
```

---

## URL에서 JSON 가져오기

```jsp
// REST API 호출 및 파싱
Json j = new Json("https://api.example.com/users/123");

// 응답 데이터 접근
String name = j.getString("//name");
String email = j.getString("//email");

// 또는 setUrl 사용
Json j = new Json();
j.setUrl("https://api.example.com/data");
DataSet result = j.getDataSet("//items");
```

---

## 출력 및 디버깅

```jsp
// JSON을 Writer로 출력
Json j = new Json();
j.put("name", "John");
j.put("age", 30);
j.print(out);  // {"name":"John","age":30}

// 디버그 모드로 상세 로그
Json j = new Json();
j.setDebug(out);
j.setUrl("https://api.example.com/data");  // HTTP 요청/응답 로그 출력
```

---

## 주의사항

1. **경로 표현**: JSON 경로는 `//`로 시작해야 합니다
   ```jsp
   j.getString("//name")      // 올바름
   j.getString("name")        // 작동 안 함
   ```

2. **배열 인덱스**: 숫자로 배열 요소에 접근
   ```jsp
   j.getString("//items/0/name")  // 첫 번째 항목
   j.getString("//items/1/name")  // 두 번째 항목
   ```

3. **타입 변환**: 적절한 getter 메소드 사용
   ```jsp
   int age = j.getInt("//age");        // 숫자
   String name = j.getString("//name"); // 문자열
   double price = j.getDouble("//price"); // 실수
   ```

4. **에러 처리**: null 체크 필수
   ```jsp
   Object data = j.get("//data");
   if(data == null) {
       // 데이터가 없거나 경로가 잘못됨
   }
   ```

5. **응답 형식 변경사항**: v1.13.7부터 표준 응답 형식이 변경되었습니다
   ```jsp
   // 이전 (deprecated)
   j.error(404, "Not Found");
   // {"error":404,"message":"Not Found"}

   // 현재 (권장)
   j.error("NOT_FOUND", "데이터를 찾을 수 없습니다");
   // {"success":false,"error":"NOT_FOUND","message":"데이터를 찾을 수 없습니다"}
   ```

6. **DataSet 자동 처리**: DataSet을 전달할 때는 next() 호출 후 전달
   ```jsp
   DataSet info = user.find("id = 1");
   info.next();  // 중요!
   j.success(info);
   ```

---

## 관련 문서

- [DataSet 활용](dataset.md) - DataSet과 JSON 변환
- [HTTP 클라이언트](http-client.md) - REST API 호출
- [데이터베이스 연동](database.md) - DB 데이터의 JSON 변환
