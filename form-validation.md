# 6. 데이터 입력 및 유효성 체크

[← 목차로 돌아가기](README.md)

---

## Form 클래스 소개

Form 클래스는 웹에서 입력된 데이터를 받고 유효성 검증을 수행합니다.

### 주요 기능

- 폼 데이터 수신 및 처리
- 서버/클라이언트 양측 유효성 검증
- 파일 업로드 처리
- 데이터 타입 변환

---

## 기본 사용법

### 1. 간단한 폼 처리

**JSP**:
```jsp
f.addElement("name", null, "required:'Y'");
f.addElement("email", null, "required:'Y', type:'email'");

if(m.isPost() && f.validate()) {
    String name = f.get("name");
    String email = f.get("email");

    // 데이터 처리
    m.jsAlert("등록되었습니다.");
    return;
}
```

**HTML**:
```html
<form name="form1" method="POST">
    <input type="text" name="name">
    <input type="text" name="email">
    <input type="submit" value="전송">
</form>
{{form_script}}
```

### 2. addElement() 파라미터

```jsp
f.addElement("항목명", "기본값", "속성");
```

- **항목명**: 폼 필드 이름 (name 속성)
- **기본값**: 초기값 (없으면 null)
- **속성**: 검증 규칙 (JavaScript 객체 형식)

---

## 유효성 검증 속성

### 필수 입력

```jsp
f.addElement("name", null, "required:'Y'");
f.addElement("email", null, "required:true");
```

### 데이터 타입

```jsp
// 이메일
f.addElement("email", null, "type:'email', required:'Y'");

// 숫자
f.addElement("age", null, "type:'number', required:'Y'");

// URL
f.addElement("website", null, "type:'url'");

// 전화번호
f.addElement("phone", null, "type:'phone'");

// 한글만
f.addElement("name", null, "type:'hangul'");

// 영문만
f.addElement("userid", null, "type:'engonly'");

// 주민번호
f.addElement("jumin", null, "type:'jumin'");

// 사업자번호
f.addElement("bizno", null, "type:'bizno'");
```

### 길이 제한

```jsp
// 최소 길이
f.addElement("name", null, "minlength:2");

// 최대 길이
f.addElement("name", null, "maxlength:20");

// 최소/최대 길이
f.addElement("name", null, "minlength:2, maxlength:20");
```

### 숫자 범위

```jsp
// 최소값
f.addElement("age", null, "type:'number', min:1");

// 최대값
f.addElement("age", null, "type:'number', max:150");

// 범위
f.addElement("age", null, "type:'number', min:1, max:150");
```

### 패턴 매칭

```jsp
// 정규표현식
f.addElement("code", null, "pattern:'^[A-Z]{2}[0-9]{4}$'");
```

### 일치 확인

```jsp
// 비밀번호 확인
f.addElement("passwd", null, "required:'Y'");
f.addElement("passwd_confirm", null, "match:'passwd', required:'Y'");
```

### HTML 태그 허용

```jsp
// HTML 태그 허용
f.addElement("content", null, "allowhtml:'Y'");

// HTML 태그 제거
f.addElement("content", null, "allowhtml:'N'");  // 기본값
```

### 한글명 지정

```jsp
f.addElement("name", null, "title:'이름', required:'Y'");
```

---

## 전체 예제

### 회원가입 폼

**JSP** (`/main/user_join.jsp`):
```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

f.addElement("id", null, "title:'아이디', required:true, minlength:4, maxlength:12");
f.addElement("passwd", null, "title:'비밀번호', required:true, minlength:4");
f.addElement("passwd_confirm", null, "title:'비밀번호 확인', match:'passwd', required:true");
f.addElement("name", null, "title:'이름', required:true, maxlength:20");
f.addElement("email", null, "title:'이메일', type:'email', required:true");
f.addElement("phone", null, "title:'전화번호', type:'phone', required:true");
f.addElement("age", null, "title:'나이', type:'number', min:1, max:150");
f.addElement("url", null, "title:'홈페이지', type:'url'");
f.addElement("gender", "M", "title:'성별', required:true");
f.addElement("agree", null, "title:'약관동의', required:true");

if(m.isPost() && f.validate()) {

    UserDao user = new UserDao();
    user.item("id", f.get("id"));
    user.item("passwd", f.get("passwd"));
    user.item("name", f.get("name"));
    user.item("email", f.get("email"));
    user.item("phone", f.get("phone"));
    user.item("age", f.getInt("age"));
    user.item("url", f.get("url"));
    user.item("gender", f.get("gender"));

    if(user.insert()) {
        m.jsAlert("회원가입이 완료되었습니다.");
        m.jsReplace("login.jsp");
    } else {
        m.jsError("회원가입 중 오류가 발생했습니다.");
    }
    return;
}

p.setBody("main.user_join");
p.setVar("form_script", f.getScript());
p.display();

%>
```

**HTML** (`/html/main/user_join.html`):
```html
<form name="form1" method="post">
    <p>아이디 : <input type="text" name="id" placeholder="4-12자"></p>
    <p>비밀번호 : <input type="password" name="passwd" placeholder="4자 이상"></p>
    <p>비밀번호 확인 : <input type="password" name="passwd_confirm"></p>
    <p>이름 : <input type="text" name="name"></p>
    <p>이메일 : <input type="email" name="email"></p>
    <p>전화번호 : <input type="text" name="phone"></p>
    <p>나이 : <input type="number" name="age"></p>
    <p>홈페이지 : <input type="url" name="url"></p>
    <p>성별 :
        <select name="gender">
            <option value="M">남성</option>
            <option value="F">여성</option>
        </select>
    </p>
    <p><label><input type="checkbox" name="agree" value="Y"> 약관에 동의합니다</label></p>
    <p><input type="submit" value="회원가입"></p>
</form>
{{form_script}}
```

---

## 데이터 가져오기

### get() 메소드

```jsp
// 문자열
String name = f.get("name");

// 기본값 지정
String name = f.get("name", "기본값");
```

### 타입별 메소드

```jsp
// 정수
int age = f.getInt("age");
int age = f.getInt("age", 0);  // 기본값

// Long
long id = f.getLong("id");

// Double
double price = f.getDouble("price");

// Boolean
boolean agree = f.getBoolean("agree");
```

### 배열 데이터

```jsp
// 체크박스 등 여러 값
String[] hobbies = f.getArray("hobby");
```

---

## 포스트백 방식

등록/수정 폼과 처리를 하나의 JSP에서 처리합니다.

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

f.addElement("name", null, "required:'Y'");
f.addElement("email", null, "required:'Y', type:'email'");

if(m.isPost() && f.validate()) {
    // POST로 데이터가 전송된 경우
    UserDao user = new UserDao();
    user.item("name", f.get("name"));
    user.item("email", f.get("email"));

    if(user.insert()) {
        m.jsAlert("등록되었습니다.");
        m.jsReplace("list.jsp");
    } else {
        m.jsError("등록 실패");
    }
    return;
}

// GET 요청인 경우 폼 출력
p.setBody("main.user_form");
p.setVar("form_script", f.getScript());
p.display();

%>
```

---

## 수정 폼에 기존 데이터 입력

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

String id = f.get("id");

UserDao user = new UserDao();
DataSet info = user.find("id = ?", new Object[] { id });

if(!info.next()) {
    m.jsError("사용자를 찾을 수 없습니다.");
    return;
}

// 기존 값을 기본값으로 설정
f.addElement("name", info.s("name"), "required:'Y'");
f.addElement("email", info.s("email"), "required:'Y', type:'email'");
f.addElement("age", info.i("age"), "type:'number'");

if(m.isPost() && f.validate()) {
    user.item("name", f.get("name"));
    user.item("email", f.get("email"));
    user.item("age", f.getInt("age"));

    if(user.update("id = ?", new Object[] { id })) {
        m.jsAlert("수정되었습니다.");
        m.jsReplace("view.jsp?id=" + id);
    }
    return;
}

p.setBody("main.user_form");
p.setVar(info);  // 템플릿에 데이터 전달
p.setVar("form_script", f.getScript());
p.display();

%>
```

---

## 파일 업로드

> 파일 업로드 및 다운로드에 대한 상세한 내용은 [파일 업로드 및 다운로드](file-upload-download.md) 문서를 참조하세요.

### 단일 파일

**JSP** (`/main/file_upload.jsp`):
```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

f.addElement("name", null, "required:'Y'");
f.addElement("file", null, "required:'Y'");

if(m.isPost() && f.validate()) {

    File file = f.saveFile("file");
    if(file != null) {
        String fileName = f.getFileName("file");
        String filePath = file.getAbsolutePath();

        m.p("파일명: " + fileName);
        m.p("저장경로: " + filePath);

        // DB에 파일 정보 저장
        // ...
    }
    return;
}

p.setBody("main.file_upload");
p.setVar("form_script", f.getScript());
p.display();

%>
```

**HTML** (`/html/main/file_upload.html`):
```html
<form name="form1" method="post" enctype="multipart/form-data">
    <p>이름 : <input type="text" name="name"></p>
    <p>파일 : <input type="file" name="file"></p>
    <p><input type="submit" value="업로드"></p>
</form>
{{form_script}}
```

### 파일 확장자 제한

```jsp
// 허용 확장자
f.addElement("file", null, "allow:'jpg|png|gif'");

// 금지 확장자
f.addElement("file", null, "deny:'exe|jsp|php'");
```

### 여러 파일 업로드

```jsp
File[] files = f.saveFiles("files");
for(File file : files) {
    m.p(file.getName());
}
```

---

## 데이터 연결 (glue)

여러 필드를 하나로 연결합니다.

```jsp
// 전화번호 3개 필드를 하나로
f.addElement("phone1", "010", "glue:'phone2|phone3', delim:'-', required:'Y'");

// 사용
String phone = f.glue("-", "phone1", "phone2", "phone3");
// 결과: 010-1234-5678
```

---

## 유효성 검증 속성 전체 목록

| 속성 | 설명 | 예시 |
|------|------|------|
| title | 필드명 | `title:'이름'` |
| required | 필수 입력 | `required:'Y'` |
| type | 데이터 타입 | `type:'email'` |
| minlength | 최소 길이 | `minlength:4` |
| maxlength | 최대 길이 | `maxlength:20` |
| min | 최소값 | `min:1` |
| max | 최대값 | `max:100` |
| pattern | 정규표현식 | `pattern:'^[A-Z]+'` |
| match | 일치 확인 | `match:'passwd'` |
| allow | 허용 확장자 | `allow:'jpg\|png'` |
| deny | 금지 확장자 | `deny:'exe\|jsp'` |
| allowhtml | HTML 허용 | `allowhtml:'Y'` |
| glue | 필드 연결 | `glue:'field2\|field3'` |
| delim | 구분자 | `delim:'-'` |

---

## 클라이언트 검증

`f.getScript()`를 HTML에 포함하면 자동으로 클라이언트 검증 스크립트가 생성됩니다.

```html
<form name="form1" method="post">
    <!-- 폼 필드들 -->
</form>
{{form_script}}
```

**주의사항**:
- 폼 이름은 반드시 `form1`이어야 합니다
- `lib.validate.js` 파일이 필요합니다

---

## 베스트 프랙티스

### 1. 명확한 에러 메시지

```jsp
f.addElement("email", null, "title:'이메일 주소', type:'email', required:'Y'");
```

### 2. 적절한 타입 지정

```jsp
f.addElement("age", null, "type:'number', min:1, max:150");
```

### 3. 보안을 위한 HTML 필터링

```jsp
// 일반 텍스트는 HTML 제거 (기본)
f.addElement("name", null, "required:'Y'");

// 에디터 내용은 HTML 허용
f.addElement("content", null, "allowhtml:'Y'");
```

### 4. 서버 검증 필수

클라이언트 검증만으로는 부족합니다. 반드시 서버에서도 검증하세요:

```jsp
if(m.isPost() && f.validate()) {
    // 서버 검증 통과
    // 추가 비즈니스 로직 검증
    if(!isValidBusinessLogic()) {
        m.jsError("잘못된 요청입니다.");
        return;
    }
}
```

---

## 다음 단계

- [파일 업로드 및 다운로드](file-upload-download.md)에서 파일 처리에 대해 더 자세히 알아보세요.
- [목록 및 검색](list-search.md)으로 이동하여 데이터 목록 처리를 배워보세요.

---

[← 이전: 데이터베이스 연동](database.md) | [목차로 돌아가기](README.md) | [다음: 파일 업로드 및 다운로드 →](file-upload-download.md)
