# 맑은프레임워크

**Java JSP 웹 개발 프레임워크**

간결하고 효율적인 웹 애플리케이션 개발을 위한 경량 프레임워크입니다.

[![Version](https://img.shields.io/badge/version-1.3-blue.svg)](https://github.com/malgnsoft/malgnsoft.github.io)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![Java](https://img.shields.io/badge/java-8%2B-orange.svg)](https://www.oracle.com/java/)

---

## 빠른 시작

맑은프레임워크를 처음 사용하신다면 다음 순서로 진행하세요:

1. [프레임워크 소개](introduction.md) - 주요 특징 및 아키텍처
2. [설치 및 환경설정](installation.md) - WAS 설치 및 데이터베이스 연결
3. [시작하기](getting-started.md) - Hello World부터 시작하기
4. [코딩 원칙](coding-principles.md) - 설계 철학과 개발 가이드 (필독)

## 주요 특징

### 템플릿 엔진
프로그램 로직과 화면 출력을 완전히 분리합니다.

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%
p.setBody("main.index");
p.setVar("message", "Hello World!");
p.display();
%>
```

### 데이터베이스 추상화
Oracle, MySQL, MSSQL 등 다양한 DBMS를 통합 인터페이스로 제공합니다.

```java
public class UserDao extends DataObject {
    public UserDao() {
        this.table = "tb_user";
    }
}
```

### 폼 유효성 검증
서버/클라이언트 양측 검증을 한 번의 설정으로 구현합니다.

```jsp
f.addElement("email", null, "required:'Y', type:'email'");
f.addElement("age", null, "type:'number', min:1, max:150");
```

### 파일 처리
업로드, 다운로드, 썸네일 생성을 간단하게 처리합니다.

```jsp
File file = f.saveFile("file");
m.download(filePath, fileName);
```

## 핵심 클래스

| 클래스 | 변수 | 설명 |
|--------|------|------|
| Malgn | m | 요청/응답 처리, 유틸리티 메소드 |
| Form | f | 폼 데이터 처리 및 유효성 검증 |
| Page | p | 템플릿 렌더링 및 변수 치환 |
| Json | j | JSON 생성, 파싱, API 응답 |
| Auth | auth | 인증 처리 및 세션 관리 |
| DataObject | - | DAO 베이스 클래스 (CRUD) |
| DataSet | - | 데이터 컬렉션 |
| ListManager | lm | 목록 및 페이징 처리 |

## 문서 구성

### 기본 가이드
프레임워크를 처음 접하는 개발자를 위한 가이드입니다.

- [프레임워크 소개](introduction.md) - 주요 특징과 장점
- [설치 및 환경설정](installation.md) - WAS 설치, DB 연결
- [시작하기](getting-started.md) - Hello World부터 첫 CRUD까지
- [코딩 원칙](coding-principles.md) - 설계 철학, 베스트 프랙티스 (필독)

### 핵심 기능
웹 애플리케이션 개발의 핵심 기능들입니다.

- [맑은템플릿](template.md) - 템플릿 엔진 사용법
- [데이터베이스 연동](database.md) - 기본 연결 및 쿼리
- [DataObject 클래스](dataobject.md) - DAO 패턴, CRUD 메소드
- [데이터 입력 및 유효성 체크](form-validation.md) - Form 클래스 활용
- [파일 업로드 및 다운로드](file-upload-download.md) - 파일 처리
- [목록 및 검색](list-search.md) - 페이징, 검색, 정렬
- [DataSet 활용](dataset.md) - 데이터 구조 이해

### 데이터 처리
다양한 데이터 포맷을 처리하는 방법입니다.

- [JSON 처리](json.md) - JSON 파싱, 생성, API 응답
- [XML 처리](xml.md) - XML 파싱, XPath 조회
- [Excel 처리](excel.md) - 엑셀 읽기/쓰기, 다운로드

### 보안 및 인증
보안과 사용자 인증을 구현하는 방법입니다.

- [암호화](encryption.md) - AES 암호화, 해시
- [인증 처리](authentication.md) - 로그인, 세션, 권한
- [OAuth 소셜 로그인](oauth.md) - Google, Naver, Kakao 연동

### 고급 기능
고급 개발을 위한 추가 기능들입니다.

- [HTTP 클라이언트](http-client.md) - REST API 호출
- [이메일 발송](email.md) - Gmail, SMTP 메일 발송
- [달력 및 날짜 선택](calendar.md) - 달력 UI, 날짜 처리
- [유틸리티 메소드](utility-methods.md) - 날짜, 문자열, 파일 처리
- [다국어 지원](i18n.md) - 다국어 메시지 관리
- [OpenAI 통합](openai.md) - ChatGPT API 연동
- [파일 전송 및 압축](file-transfer.md) - FTP, ZIP
- [환경설정 및 캐시](configuration.md) - Config, Cache

## 개발 예제

### Hello World

**JSP 파일** (`/main/index.jsp`):
```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%
p.setBody("main.index");
p.setVar("message", "안녕하세요, 맑은프레임워크!");
p.display();
%>
```

**템플릿 파일** (`/html/main/index.html`):
```html
<html>
<body>
    <h1>{{message}}</h1>
</body>
</html>
```

### 게시판 목록

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

ListManager lm = new ListManager();
lm.setRequest(request);
lm.setTable("tb_board");
lm.setListNum(20);
lm.addSearch("subject,content", f.get("keyword"), "LIKE");
lm.setSort("id DESC");

DataSet list = lm.getDataSet();

p.setBody("main.board_list");
p.setLoop("list", list);
p.setVar("pager", lm.getPaging());
p.display();

%>
```

### REST API 응답

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

BoardDao board = new BoardDao();
board.setOrderBy("id DESC");
board.setLimit(10);
DataSet list = board.find();

if(list != null) {
    j.success("조회 성공", list);
} else {
    j.error("조회 실패: " + board.getErrMsg());
}

%>
```

## 다운로드

- [통합 매뉴얼](manual-v1.3.md) - 모든 문서를 하나의 파일로

## 링크

- [GitHub](https://github.com/malgnsoft)
- [맑은소프트](https://malgnsoft.com)
- 문의: support@malgnsoft.com

## 기여하기

문서 개선 제안 및 오류 수정은 언제나 환영합니다.

1. Fork this repository
2. Create a feature branch (`git checkout -b feature/improvement`)
3. Commit your changes (`git commit -am 'Add improvement'`)
4. Push to the branch (`git push origin feature/improvement`)
5. Create a Pull Request

## 라이선스

MIT License

---

© 2025 Malgn Software. All rights reserved.
