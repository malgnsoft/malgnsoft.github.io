# 1. 프레임워크 소개

[← 목차로 돌아가기](README.md)

---

## 개요

맑은프레임워크는 자바 웹개발 프레임워크입니다. WAS(Web Application Server) 안에서 구동되는 솔루션이며, 간결하고 효율적인 웹 애플리케이션 개발을 지원합니다.

---

## 주요 특징

### 1. 템플릿 엔진
- 프로그램 로직과 화면 출력을 완전히 분리
- 웹디자이너/퍼블리셔가 쉽게 수정 가능한 단순화된 문법
- 변수 치환, 루프, 조건문, 레이아웃 기능 제공

### 2. DataSet 객체
- 모든 데이터를 통일된 형태로 처리
- 데이터베이스, XML, Excel 등 다양한 소스 지원
- JSON 직렬화/역직렬화 지원

### 3. 데이터베이스 추상화
- Oracle, MySQL, MSSQL, DB2 등 다양한 DBMS 지원
- DAO 패턴을 통한 깔끔한 데이터베이스 처리
- Connection Pool 자동 관리

### 4. 유효성 검증
- 한 번의 설정으로 서버/클라이언트 양측 검증
- 다양한 기본 검증 규칙 제공
- 사용자 정의 검증 규칙 추가 가능

### 5. 간단한 설정
- 최소한의 환경설정 파일
- XML 기반의 직관적인 설정
- 운영 중 설정 변경 가능 (재시작 불필요)

---

## 아키텍처

### MVC 패턴 기반

```
[Browser]
    ↓ Request
[JSP Controller]
    ↓
[Model (DAO/DataSet)]
    ↓
[Database]
    ↑
[View (Template)]
    ↓ Response
[Browser]
```

### 주요 구성 요소

1. **Controller**: JSP 파일
   - 요청 처리
   - 비즈니스 로직 실행
   - 데이터 준비

2. **Model**: DAO 클래스 + DataSet
   - 데이터베이스 연동
   - 비즈니스 로직
   - 데이터 관리

3. **View**: HTML 템플릿
   - 화면 출력
   - 데이터 표시
   - 사용자 인터페이스

---

## 핵심 클래스

### Malgn
유틸리티 메소드를 제공하는 핵심 클래스

**주요 메소드**:
- `p()`: 디버깅 출력
- `isPost()`: POST 방식 체크
- `jsError()`: 에러 메시지 + 이전 페이지 이동
- `jsAlert()`: 알림 메시지
- `jsReplace()`: 페이지 이동
- `time()`: 날짜/시간 포맷팅

### Form
폼 데이터 처리 및 유효성 검증

**주요 메소드**:
- `addElement()`: 폼 항목 정의
- `validate()`: 유효성 검증
- `get()`: 폼 데이터 가져오기
- `getScript()`: 클라이언트 검증 스크립트 생성

### Page
템플릿 처리

**주요 메소드**:
- `setLayout()`: 레이아웃 지정
- `setBody()`: 템플릿 지정
- `setVar()`: 변수 설정
- `setLoop()`: 루프 데이터 설정
- `display()`: 화면 출력

### DataObject
데이터베이스 기본 연동

**주요 메소드**:
- `query()`: SELECT 쿼리 실행
- `execute()`: INSERT/UPDATE/DELETE 실행
- `find()`: 조건으로 검색
- `item()`: 데이터 항목 설정
- `insert()`: 데이터 삽입
- `update()`: 데이터 수정
- `delete()`: 데이터 삭제

### DataSet
데이터 저장 및 처리

**주요 메소드**:
- `addRow()`: 행 추가
- `next()`: 다음 행으로 이동
- `put()`: 데이터 설정
- `get()` / `s()` / `i()`: 데이터 가져오기
- `serialize()`: JSON 직렬화
- `unserialize()`: JSON 역직렬화

### ListManager
목록 및 페이징 처리

**주요 메소드**:
- `setTable()`: 테이블 지정
- `setListNum()`: 목록 개수 지정
- `addWhere()`: 조건 추가
- `addSearch()`: 검색 조건 추가
- `setOrderBy()`: 정렬 지정
- `getDataSet()`: 목록 데이터 가져오기
- `getTotalNum()`: 전체 개수 가져오기
- `getPaging()`: 페이지 네비게이션 생성

### Auth
인증 처리

**주요 메소드**:
- `isValid()`: 인증 유효성 체크
- `put()`: 인증 정보 설정
- `get()` / `getString()` / `getInt()`: 인증 정보 가져오기
- `save()`: 인증 정보 저장
- `delete()`: 인증 정보 삭제

### Config
환경설정 관리

**주요 메소드**:
- `get()`: 설정값 가져오기
- `getInt()`: 정수형 설정값 가져오기
- `getDocRoot()`: 문서 루트 경로
- `getDataDir()`: 데이터 디렉토리 경로
- `getTplRoot()`: 템플릿 루트 경로
- `reload()`: 설정 재로드

---

## 개발 프로세스

### 1. 환경 구축
1. WAS 설치 (Resin, Tomcat 등)
2. 맑은프레임워크 라이브러리 설치
3. 환경설정 파일 작성

### 2. 기본 구조 생성
1. init.jsp 작성
2. 폴더 구조 생성
3. config.xml 설정

### 3. 기능 개발
1. DAO 클래스 작성 (Model)
2. JSP 파일 작성 (Controller)
3. HTML 템플릿 작성 (View)

### 4. 테스트 및 배포
1. 기능 테스트
2. 디버깅
3. 배포

---

## 다음 단계

- [설치 및 환경설정](installation.md)으로 이동하여 프레임워크를 설치하세요.

---

[← 목차로 돌아가기](README.md) | [다음: 설치 및 환경설정 →](installation.md)
