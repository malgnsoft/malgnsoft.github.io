# Claude Code 프로젝트 설정 완벽 가이드

> **Claude Code를 최대한 활용하기 위한 프로젝트 구조 및 설정 가이드**

---

## 목차

1. [`.claude/rules/` - 자동 로드 규칙](#1-clauderules---자동-로드-규칙)
2. [`CLAUDE.md` - 프로젝트 컨텍스트](#2-claudemd---프로젝트-컨텍스트)
3. [`.claude/templates/` - 코드 템플릿](#4-claudetemplates---코드-템플릿)
4. [`.claude/memory/` - 세션 기억](#5-claudememory---세션-기억)
5. [`.claude/config.json` - MCP 및 설정](#6-claudeconfigjson---mcp-및-설정)
6. [`.claudeignore` - 제외 파일](#7-claudeignore---제외-파일)
7. [전체 구조 예시](#8-전체-구조-예시)

---

## 1. `.claude/rules/` - 자동 로드 규칙

### 개요

**모든 대화에서 자동으로 로드되는 핵심 규칙 파일**

- **로드 시점**: 매 메시지마다 자동 로드
- **우선순위**: 가장 높음 (Claude가 반드시 따라야 할 규칙)
- **최적 크기**: 폴더 전체 합산 **2,000~5,000 토큰**
- **파일당 크기**: **500~2,000 토큰** 권장

### 토큰 계산 가이드

```
영문: 1단어 ≈ 1.3 토큰
한글: 1글자 ≈ 2~3 토큰
코드: 1줄 ≈ 10~20 토큰

예시:
- 100줄 코드 예제 ≈ 1,000~2,000 토큰
- 한글 500자 ≈ 1,000~1,500 토큰
```

### 무엇을 넣어야 하는가?

#### ✅ 포함해야 할 내용

1. **절대 규칙** (안티패턴)
   ```markdown
   - ❌ `m.getInt()` - 존재하지 않음
   - ✅ `m.ri()` - 정수 파라미터
   ```

2. **자주 반복되는 패턴**
   ```markdown
   if(m.isPost() && f.validate()) {
       // 처리 로직
       return;  // 필수!
   }
   ```

3. **간단한 코드 예제** (5~10줄)
   ```jsp
   UserDao user = new UserDao();
   DataSet info = user.get(id);
   if(info.next()) {
       p.setVar("name", info.s("name"));
   }
   ```

4. **핵심 원칙** (한 줄 설명)
   ```markdown
   - GET: m.rs() / POST: f.get()
   - DataSet은 next() 호출 필수
   ```

#### ❌ 포함하지 말아야 할 내용

1. **장황한 설명** → CLAUDE.md로 이동
2. **복잡한 예제** (50줄+) → `.claude/docs/`로 이동
3. **상세 매뉴얼** → docs 폴더로 이동
4. **히스토리/변경 로그** → 불필요
5. **"자세한 내용은 XXX 참고"** → 토큰 낭비

### 파일 분리 전략

#### 단일 파일 vs 분리

```
작은 프로젝트 (1~2개 규칙):
  .claude/rules/framework.md (1,500 토큰)

중간 프로젝트 (3~5개 규칙):
  .claude/rules/
    ├── framework.md (1,000 토큰)
    ├── ui-style.md (500 토큰)
    └── security.md (300 토큰)

큰 프로젝트 (6개+ 규칙):
  .claude/rules/
    ├── 01-framework-core.md (800 토큰)
    ├── 02-framework-templates.md (600 토큰)
    ├── 03-ui-bootstrap.md (500 토큰)
    ├── 04-security.md (400 토큰)
    └── 05-api-patterns.md (300 토큰)
```

**숫자 접두사 사용 이유**: 로드 순서 제어 (중요한 것부터)

### 파일 명명 규칙

```markdown
✅ 좋은 이름:
- framework-core.md
- ui-style.md
- api-patterns.md
- security-rules.md

❌ 나쁜 이름:
- rules.md (너무 일반적)
- temp.md (의미 불명)
- myfile.md (개인적)
```

### 실전 예시: 맑은프레임워크

```
.claude/rules/
├── malgn-framework.md (~1,000 토큰)
│   - GET/POST 파라미터 처리
│   - DAO CRUD 패턴
│   - DataSet 사용법
│   - 템플릿 문법
│
└── admin-style.md (~500 토큰)
    - Bootstrap 80/20 철학
    - 검색 필터 규칙
    - 테이블 규칙
    - 페이징 규칙

총합: ~1,500 토큰 (매우 효율적)
```

---

## 2. `CLAUDE.md` - 프로젝트 컨텍스트

### 개요

**프로젝트 전체 개요 및 컨텍스트 문서**

- **위치**: 프로젝트 루트 (`/CLAUDE.md`)
- **로드 시점**: 명시적으로 요청 시 또는 프로젝트 분석 시
- **최적 크기**: **5,000~15,000 토큰**
- **목적**: 프로젝트의 "큰 그림" 제공

### 무엇을 넣어야 하는가?

#### 필수 섹션

```markdown
# 프로젝트명

## 1. 프로젝트 개요
- 무엇을 만드는가?
- 주요 기능
- 사용자/고객

## 2. 기술 스택
- 언어/프레임워크
- 데이터베이스
- 주요 라이브러리

## 3. 프로젝트 구조
/src/              - Java 소스
/public_html/      - JSP/HTML

## 4. 개발 환경 설정
- 로컬 설정 방법
- 빌드 방법
- 실행 방법

## 5. 핵심 개념
- 도메인 용어
- 비즈니스 로직 개요
- 아키텍처 패턴

## 6. 주요 파일 위치
- 설정 파일: /WEB-INF/config.xml
- 초기화: /init.jsp
- DAO: /src/dao/

## 7. 팀 컨벤션
- 코딩 스타일
- Git 브랜치 전략
- PR 프로세스

## 8. 자주 묻는 질문
Q: 로컬 DB 연결은?
A: config.xml에서...

### 실전 예시: LMS 프로젝트

```markdown
# 스타트시스템 LMS

## 프로젝트 개요
온라인 교육 플랫폼 (Learning Management System)
- 강의 관리
- 수강생 관리
- 결제 시스템

## 기술 스택
- **프레임워크**: 맑은프레임워크 (JSP 기반)
- **데이터베이스**: MySQL 5.7
- **UI**: Bootstrap 5.3.0
- **서버**: Tomcat 9.0

## 핵심 개념
- **Course**: 강좌 (강의 묶음)
- **Lesson**: 개별 강의
- **Order**: 주문/결제
- **User**: 회원 (수강생 + 강사)

## 주요 DAO
- CourseDao - 강좌 관리
- LessonDao - 강의 관리
- OrderDao - 주문 관리
- UserDao - 회원 관리

## 개발 규칙
1. 모든 JSP는 /course/, /lesson/, /order/ 폴더별로 분리
2. DAO 명명: CourseDao course (courseDao ❌)
3. 템플릿 위치: /html/course/, /html/lesson/

## 로컬 실행
1. Tomcat 9.0 설치
2. config.xml DB 정보 수정
3. ant compile
4. http://localhost:8080
```

### CLAUDE.md vs .claude/rules/

| 항목 | CLAUDE.md | .claude/rules/ |
|------|-----------|----------------|
| 목적 | 프로젝트 이해 | 코딩 규칙 |
| 크기 | 5K~15K 토큰 | 2K~5K 토큰 |
| 로드 | 명시적 요청 | 자동 (매 메시지) |
| 내용 | 개요, 구조, 컨벤션 | 절대 규칙, 안티패턴 |
| 예시 | 아키텍처 설명 | 코드 스니펫 |

---

## 3. `.claude/templates/` - 코드 템플릿

### 개요

**자주 사용하는 코드 패턴 템플릿**

- **목적**: 빠른 코드 생성
- **형식**: 실제 코드 파일 또는 스니펫

### 디렉토리 구조

```
.claude/templates/
├── jsp/
│   ├── list.jsp              # 목록 페이지 템플릿
│   ├── insert.jsp            # 등록 페이지 템플릿
│   ├── modify.jsp            # 수정 페이지 템플릿
│   └── view.jsp              # 상세 페이지 템플릿
│
├── html/
│   ├── list.html             # 목록 HTML 템플릿
│   ├── form.html             # 폼 HTML 템플릿
│   └── view.html             # 상세 HTML 템플릿
│
├── dao/
│   └── DaoTemplate.java      # DAO 클래스 템플릿
│
└── sql/
    └── table.sql             # 테이블 생성 템플릿
```

### 템플릿 예시

#### `.claude/templates/jsp/list.jsp`

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// TODO: 테이블명 변경
{{TABLE_NAME}}Dao {{table}} = new {{TABLE_NAME}}Dao();

// 검색 폼 필드
f.addElement("keyword", null, null);

// 목록 조회
ListManager lm = new ListManager();
lm.setRequest(request);
lm.setListNum(20);
lm.setTable("tb_{{table}}");
lm.addSearch("{{SEARCH_FIELDS}}", f.get("keyword"), "LIKE");
lm.setOrderBy("id DESC");

DataSet list = lm.getDataSet();

p.setLayout("default");
p.setBody("{{folder}}.{{file}}_list");
p.setLoop("list", list);
p.setVar("total", lm.getTotalNum());
p.setVar("pager", lm.getPaging());
p.setVar("form_script", f.getScript());
p.display();

%>
```

### 활용 방법

```
사용자: "강의 목록 페이지 만들어줘"

Claude: ".claude/templates/jsp/list.jsp 템플릿을 사용하여 생성합니다."

        [생성된 코드]
        LessonDao lesson = new LessonDao();
        lm.setTable("tb_lesson");
        lm.addSearch("title,content", f.get("keyword"), "LIKE");
        ...
```

### 변수 치환 규칙

```markdown
템플릿 변수:
- {{TABLE_NAME}} → 테이블명 (대문자, 예: Lesson)
- {{table}} → 변수명 (소문자, 예: lesson)
- {{SEARCH_FIELDS}} → 검색 필드 (예: title,content)
- {{folder}} → 폴더명 (예: lesson)
- {{file}} → 파일명 (예: lesson_list)
```

---

## 4. `.claude/memory/` - 세션 기억

### 개요

**세션별 컨텍스트 및 진행 상황 저장**

- **목적**: 대화 세션 간 연속성 유지
- **자동 생성**: Claude가 중요한 정보 자동 저장
- **수동 관리**: 개발자가 명시적으로 메모

### 디렉토리 구조

```
.claude/memory/
├── session-2025-01-12.md     # 오늘 세션 메모
├── decisions.md               # 중요 결정사항
├── todos.md                   # 진행 중인 작업
└── learned.md                 # 학습한 내용
```

### 파일 예시

#### `session-2025-01-12.md`

```markdown
# 2025-01-12 세션

## 진행 사항
- [x] 강의 목록 페이지 완성
- [x] 강의 등록 폼 검증 추가
- [ ] 강의 수정 페이지 (진행 중)

## 발견한 이슈
- Bootstrap 모달에서 form_script 동작 안 됨
  → 해결: 모달 open 이벤트에서 수동 초기화

## 다음 세션 할 일
- 강의 수정 페이지 완성
- 강의 삭제 기능 추가
- 강사 권한 체크 로직
```

#### `decisions.md`

```markdown
# 주요 결정사항

## 2025-01-10: 파일 업로드 구조
**결정**: AWS S3 대신 로컬 파일 시스템 사용
**이유**: 비용 절감, 초기 단계에서 충분
**영향**: FileManager 클래스만 사용

## 2025-01-08: 결제 모듈
**결정**: 포트원(PortOne) 사용
**이유**: 간편 연동, 다양한 PG 지원
**참고**: docs/payment-guide.md
```

#### `learned.md`

```markdown
# 학습한 내용

## 맑은프레임워크 특이사항

### m.ri() vs m.getInt()
- ❌ m.getInt() 메소드는 존재하지 않음
- ✅ m.ri() 사용해야 함
- 이유: 프레임워크 설계 철학

### 템플릿 조건문
- ❌ <!--@if(status == 1)-->
- ✅ JSP에서 boolean 변수 생성 후 사용
- 이유: 템플릿은 출력만 담당
```

### 활용 방법

#### 자동 저장 (Claude Code 내장 기능)

```
사용자: "이 결정 사항 기억해줘"
Claude: ".claude/memory/decisions.md에 저장했습니다."
```

#### 수동 요청

```
사용자: "오늘 한 작업 요약해줘"
Claude: [.claude/memory/session-2025-01-12.md 확인]
        "오늘은 강의 목록 페이지와 등록 폼을 완성했습니다..."
```

#### 세션 재개

```
사용자: "어제 어디까지 했지?"
Claude: [.claude/memory/session-2025-01-11.md 확인]
        "어제는 강의 목록 페이지까지 완성했습니다.
         다음은 수정 페이지를 만들 예정이었습니다."
```

---

## 5. `.claude/config.json` - MCP 및 설정

### 개요

**Claude Code 및 MCP 서버 설정**

- **필수 여부**: MCP 서버 사용 시 필수
- **위치**: `.claude/config.json`

### 기본 구조

```json
{
  "mcpServers": {
    "malgn-framework": {
      "type": "http",
      "url": "http://your-server:8000",
      "description": "맑은프레임워크 검증 서버",
      "tools": [
        {
          "name": "validate_code",
          "description": "코드 검증",
          "endpoint": "/validate",
          "method": "POST"
        },
        {
          "name": "search_docs",
          "description": "문서 검색",
          "endpoint": "/search-docs",
          "method": "POST"
        }
      ]
    }
  },

  "autoValidate": {
    "enabled": true,
    "onSave": true,
    "fileTypes": ["jsp", "html"]
  },

  "contextRefresh": {
    "enabled": true,
    "messageThreshold": 20
  }
}
```

### 설정 옵션

#### MCP 서버 연결

```json
{
  "mcpServers": {
    "server-name": {
      "type": "http",              // http | stdio | sse
      "url": "http://localhost:8000",
      "timeout": 30000,            // 30초
      "retryAttempts": 3
    }
  }
}
```

#### 자동 검증

```json
{
  "autoValidate": {
    "enabled": true,
    "onSave": true,               // 저장 시 자동 검증
    "fileTypes": ["jsp", "html"],
    "showInlineErrors": true      // 에디터 내 오류 표시
  }
}
```

#### 컨텍스트 리프레시

```json
{
  "contextRefresh": {
    "enabled": true,
    "messageThreshold": 20,       // 20개 메시지마다
    "action": "remind"            // remind | reload
  }
}
```

---

## 6. `.claudeignore` - 제외 파일

### 개요

**Claude가 읽지 않아야 할 파일/폴더 지정**

- **형식**: `.gitignore`와 동일
- **목적**: 불필요한 파일 제외로 성능 향상

### 기본 설정

```
# .claudeignore

# 의존성
node_modules/
vendor/
*.jar
*.war

# 빌드 결과
dist/
build/
target/
*.class

# 로그
*.log
logs/

# 임시 파일
*.tmp
*.temp
.DS_Store
Thumbs.db

# IDE 설정
.idea/
.vscode/
*.swp

# 대용량 데이터
*.sql
*.dump
data/
uploads/

# 민감 정보
.env
config/secrets.yml
*.key
*.pem
```

### 프레임워크별 설정

#### JSP 프로젝트

```
# 컴파일된 클래스
WEB-INF/classes/

# 라이브러리
WEB-INF/lib/

# 업로드 파일
upload/
files/

# 캐시
cache/
```

### 주의사항

```markdown
✅ 제외해야 할 것:
- 외부 라이브러리 (node_modules, lib/)
- 빌드 결과물 (*.class, dist/)
- 대용량 파일 (*.sql, uploads/)
- 민감 정보 (.env, *.key)

❌ 제외하면 안 되는 것:
- 프로젝트 소스 코드
- 설정 파일 (config.xml)
- 문서 (docs/, .claude/)
- 초기화 스크립트 (init.jsp)
```

---

## 8. 전체 구조 예시

### 이상적인 프로젝트 구조

```
프로젝트/
├── .claude/
│   ├── rules/                          # 자동 로드 규칙 (2~5K 토큰)
│   │   ├── 01-framework-core.md        # 프레임워크 핵심 (1K)
│   │   ├── 02-ui-style.md              # UI 스타일 (500)
│   │   └── 03-security.md              # 보안 규칙 (300)
│   │
│   ├── templates/                      # 코드 템플릿
│   │   ├── jsp/
│   │   │   ├── list.jsp
│   │   │   ├── insert.jsp
│   │   │   └── modify.jsp
│   │   ├── html/
│   │   │   ├── list.html
│   │   │   └── form.html
│   │   └── dao/
│   │       └── DaoTemplate.java
│   │
│   ├── memory/                         # 세션 기억
│   │   ├── session-2025-01-12.md
│   │   ├── decisions.md
│   │   ├── todos.md
│   │   └── learned.md
│   │
│   └── config.json                     # MCP 설정
│
├── CLAUDE.md                           # 프로젝트 개요 (5~15K)
├── .claudeignore                       # 제외 파일
│
├── src/                                # Java 소스
├── public_html/                        # JSP/HTML
└── docs/                               # 원본 문서 (MCP 파싱용)
```

### 크기 가이드라인

| 파일/폴더 | 크기 | 목적 | 로드 방식 |
|-----------|------|------|-----------|
| `.claude/rules/` | 2~5K 토큰 | 핵심 규칙 | 자동 (매 메시지) |
| `CLAUDE.md` | 5~15K 토큰 | 프로젝트 개요 | 명시적 요청 |
| `.claude/templates/` | 무제한 | 코드 템플릿 | 필요 시 복사 |
| `.claude/memory/` | 자유 | 세션 메모 | 자동/수동 |

---

**작성일**: 2025-01-12
**버전**: 1.0
**작성자**: Claude Code 최적화 가이드