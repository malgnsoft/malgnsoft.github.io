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

```json
{
  "error": 0,
  "message": "success",
  "data": { ... }
}
```

### 성공 응답

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 간단한 성공 응답
j.success();
// {"error":0,"message":"success"}

// 데이터와 함께
DataMap data = new DataMap();
data.put("id", 123);
data.put("name", "John");
j.success(data);
// {"error":0,"message":"success","data":{"id":123,"name":"John"}}

// 사용자 정의 메시지
j.success("작업이 완료되었습니다", data);
// {"error":0,"message":"작업이 완료되었습니다","data":{...}}

%>
```

### 에러 응답

```jsp
// 에러 코드와 메시지
j.error(404, "데이터를 찾을 수 없습니다");
// {"error":404,"message":"데이터를 찾을 수 없습니다"}

// 간단한 에러 메시지
j.error("오류가 발생했습니다");
// {"_ERROR_":"오류가 발생했습니다"}
```

### 사용자 정의 응답

```jsp
// 완전한 제어
j.print(0, "success", data);
j.print(100, "validation error", null);
```

### 실제 사용 예제

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 폼 검증
f.addElement("email", null, "required:Y, email:Y");
f.addElement("password", null, "required:Y, minlength:8");

if(!f.validate()) {
    j.error(400, f.getErrorString());
    return;
}

// 데이터 처리
UserDao dao = new UserDao();
DataSet user = dao.query("WHERE email = ?", f.get("email"));

if(!user.next()) {
    j.error(404, "사용자를 찾을 수 없습니다");
    return;
}

// 성공 응답
DataMap result = new DataMap();
result.put("id", user.i("id"));
result.put("name", user.s("name"));
result.put("email", user.s("email"));
j.success("로그인 성공", result);

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

---

## 관련 문서

- [DataSet 활용](dataset.md) - DataSet과 JSON 변환
- [HTTP 클라이언트](http-client.md) - REST API 호출
- [데이터베이스 연동](database.md) - DB 데이터의 JSON 변환
