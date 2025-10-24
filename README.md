# 맑은프레임워크 개발자 문서

> 자바 웹개발 프레임워크 - 간결하고 효율적인 웹 애플리케이션 개발

[![Version](https://img.shields.io/badge/version-1.3-blue.svg)](https://github.com/malgnsoft/malgnsoft.github.io)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![Java](https://img.shields.io/badge/java-6%2B-orange.svg)](https://www.oracle.com/java/)

---

## 🚀 빠른 시작

맑은프레임워크를 처음 사용하신다면 다음 순서로 진행하세요:

1. **[프레임워크 소개](introduction.md)** - 맑은프레임워크가 무엇인지 알아보세요
2. **[설치 및 환경설정](installation.md)** - WAS 설치부터 프레임워크 설정까지
3. **[시작하기](getting-started.md)** - 첫 번째 페이지 만들기

## 📚 주요 기능

### 🎨 템플릿 엔진
프로그램 로직과 화면 출력을 완전히 분리하여 개발 효율을 높입니다.
```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%
p.setBody("main.index");
p.setVar("message", "Hello World!");
p.display();
%>
```

### 💾 데이터베이스 추상화
Oracle, MySQL, MSSQL 등 다양한 DBMS를 하나의 인터페이스로 처리합니다.
```java
public class UserDao extends DataObject {
    public UserDao() {
        this.table = "tb_user";
    }
}
```

### ✅ 폼 유효성 검증
서버/클라이언트 양측에서 동작하는 강력한 유효성 검증 기능을 제공합니다.
```jsp
f.addElement("email", null, "required:'Y', type:'email'");
f.addElement("age", null, "type:'number', min:1, max:150");
```

### 📁 파일 처리
파일 업로드, 다운로드, 썸네일 생성을 간단하게 처리합니다.
```jsp
File file = f.saveFile("file");
m.download(filePath, fileName);
```

## 🎯 핵심 클래스

| 클래스 | 변수 | 설명 |
|--------|------|------|
| **Malgn** | m | 유틸리티 메소드 (날짜, 문자열, 파일 등) |
| **Form** | f | 폼 데이터 처리 및 유효성 검증 |
| **Page** | p | 템플릿 처리 및 화면 출력 |
| **Json** | j | JSON 생성, 파싱, API 응답 |
| **Auth** | auth | 로그인, 로그아웃, 권한 관리 |
| **DataObject** | dao | 데이터베이스 기본 연동 |
| **DataSet** | - | 데이터 저장 및 처리 |
| **ListManager** | lm | 목록 및 페이징 처리 |

## 📖 문서 구성

### 기본 가이드
프레임워크를 처음 접하는 개발자를 위한 가이드입니다.

- [프레임워크 소개](introduction.md) - 주요 특징과 장점
- [설치 및 환경설정](installation.md) - WAS 설치, DB 연결
- [시작하기](getting-started.md) - Hello World부터 첫 CRUD까지

### 핵심 기능
웹 애플리케이션 개발의 핵심 기능들입니다.

- [맑은템플릿](template.md) - 템플릿 엔진 사용법
- [데이터베이스 연동](database.md) - DAO 패턴, CRUD 작업
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

## 💡 개발 예제

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

try {
    BoardDao dao = new BoardDao();
    DataSet list = dao.findAll("ORDER BY id DESC LIMIT 10");

    j.success("조회 성공", list);
} catch(Exception e) {
    j.error("조회 실패: " + e.getMessage());
}

%>
```

## 🔗 링크

- **GitHub 저장소**: [github.com/malgnsoft](https://github.com/malgnsoft)
- **개발사**: [맑은소프트](https://malgnsoft.com)
- **문의**: support@malgnsoft.com

## 📄 다운로드

- [통합 매뉴얼 (Markdown)](manual-v1.3.md) - 전체 문서 하나로
- [매뉴얼 뷰어](view-manual.html) - 단일 페이지 HTML 뷰어

## 🤝 기여하기

문서 개선 제안이나 오류 수정은 언제나 환영합니다!

1. 이 저장소를 Fork 하세요
2. 새 브랜치를 만드세요 (`git checkout -b feature/improvement`)
3. 변경사항을 커밋하세요 (`git commit -am 'Add improvement'`)
4. 브랜치에 Push 하세요 (`git push origin feature/improvement`)
5. Pull Request를 생성하세요

## 📜 라이선스

이 문서는 MIT 라이선스 하에 배포됩니다.

---

**💻 즐거운 개발 되세요!**
