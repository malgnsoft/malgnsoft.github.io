# 10. Excel 처리

[← 목차로 돌아가기](README.md)

---

## Excel 클래스 개요

MalgnFramework는 Excel 파일을 읽고 쓰기 위한 세 가지 클래스를 제공합니다.

### 지원 클래스

| 클래스 | 파일 형식 | 용도 |
|--------|----------|------|
| ExcelReader | .xls (Excel 97-2003) | 읽기 전용 |
| ExcelWriter | .xls (Excel 97-2003) | 쓰기 전용 |
| ExcelX | .xlsx (Excel 2007+) | 읽기/쓰기 |

---

## ExcelReader - Excel 읽기 (.xls)

### 기본 사용법

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

ExcelReader excel = new ExcelReader(Config.getDataDir() + "/sample.xls");

// 시트 선택 (0부터 시작)
excel.setSheet(0);

// DataSet으로 읽기
DataSet data = excel.getDataSet();

// 데이터 출력
data.first();
while(data.next()) {
    m.p(data.getString("col0"));  // 첫 번째 열
    m.p(data.getString("col1"));  // 두 번째 열
    m.p(data.getString("col2"));  // 세 번째 열
}

excel.close();

%>
```

### 여러 시트 읽기

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

ExcelReader excel = new ExcelReader(Config.getDataDir() + "/sample.xls");

int sheetCount = excel.getSheetCount();
m.p("총 " + sheetCount + "개의 시트");

for(int i = 0; i < sheetCount; i++) {
    excel.setSheet(i);
    String sheetName = excel.getSheetName(i);
    m.p("시트 이름: " + sheetName);

    DataSet data = excel.getDataSet();
    m.p("행 개수: " + data.size());
    m.p(data);
}

excel.close();

%>
```

---

## ExcelX - Excel 읽기/쓰기 (.xlsx)

### Excel 읽기

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

ExcelX excel = new ExcelX();

// 파일 읽기
DataSet data = excel.read(Config.getDataDir() + "/sample.xlsx");

// 데이터 검증
data.setCol("col0", "maxlength:12, type:datetime, length:9");
data.setCol("col5", "type:number, length:3");

if(data.validate()) {
    m.p("검증 성공");
} else {
    m.p("검증 실패");
}

// 데이터 출력
data.first();
while(data.next()) {
    m.p(data.getString("col0"));
    m.p(data.getString("col1"));
}

%>
```

### Excel 쓰기 (단일 시트)

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 데이터 준비
DataSet data = new DataSet();
data.addRow();
data.put("col0", "user001");
data.put("col1", "홍길동");
data.put("col2", "hong@example.com");
data.put("col3", "010-1234-5678");

data.addRow();
data.put("col0", "user002");
data.put("col1", "김철수");
data.put("col2", "kim@example.com");
data.put("col3", "010-2345-6789");

// Excel 생성
ExcelX excel = new ExcelX();

// 컬럼 정의 (col이름=>헤더명)
String[] columns = {"col0=>아이디", "col1=>이름", "col2=>이메일", "col3=>전화번호"};

data.first();
excel.setSheet("회원정보");
excel.setData(data, columns);

// 파일로 저장
excel.write(Config.getDataDir() + "/users.xlsx");

// 또는 다운로드로 전송
// excel.write(response, "회원정보.xlsx");

%>
```

### Excel 쓰기 (다중 시트)

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

ExcelX excel = new ExcelX();

// 첫 번째 시트 - 회원 정보
DataSet users = new DataSet();
users.addRow();
users.put("col0", "user001");
users.put("col1", "홍길동");
users.put("col2", "hong@example.com");

users.addRow();
users.put("col0", "user002");
users.put("col1", "김철수");
users.put("col2", "kim@example.com");

String[] userColumns = {"col0=>아이디", "col1=>이름", "col2=>이메일"};
users.first();
excel.setSheet("회원정보");
excel.setData(users, userColumns);

// 두 번째 시트 - 주문 정보
DataSet orders = new DataSet();
orders.addRow();
orders.put("col0", "20250101");
orders.put("col1", "ORD001");
orders.put("col2", "50000");

orders.addRow();
orders.put("col0", "20250102");
orders.put("col1", "ORD002");
orders.put("col2", "35000");

String[] orderColumns = {"col0=>주문일", "col1=>주문번호", "col2=>금액"};
orders.first();
excel.setSheet("주문내역");
excel.setData(orders, orderColumns);

// 다운로드
excel.write(response, "보고서.xlsx");

%>
```

### 비밀번호로 보호

```jsp
ExcelX excel = new ExcelX();
excel.setSheet("민감정보");
excel.setData(data, columns);

// 비밀번호 설정
excel.setPassword("1234");

excel.write(response, "보호문서.xlsx");
```

---

## 실전 예제

### 1. 데이터베이스 조회 결과를 Excel로

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 데이터베이스 조회
UserDao dao = new UserDao();
DataSet users = dao.query("WHERE status = 1 ORDER BY reg_date DESC");

// Excel 생성
ExcelX excel = new ExcelX();

String[] columns = {
    "id=>번호",
    "user_id=>아이디",
    "name=>이름",
    "email=>이메일",
    "phone=>전화번호",
    "reg_date=>가입일"
};

users.first();
excel.setSheet("회원목록");
excel.setData(users, columns);

// 다운로드
excel.write(response, "회원목록_" + m.time("yyyyMMdd") + ".xlsx");

%>
```

### 2. Excel 업로드 및 일괄 처리

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

f.addElement("excel_file", null, "required:Y, accept:'.xlsx'");

if(m.isPost() && f.validate()) {

    String filePath = f.getFileDir("excel_file") + "/" + f.getFileName("excel_file");

    // Excel 읽기
    ExcelX excel = new ExcelX();
    DataSet data = excel.read(filePath);

    // 데이터 검증
    data.setCol("col0", "required:Y, maxlength:50");  // 아이디
    data.setCol("col1", "required:Y");  // 이름
    data.setCol("col2", "type:email");  // 이메일

    if(!data.validate()) {
        m.jsError("Excel 데이터 검증 실패");
        return;
    }

    // 데이터베이스에 일괄 저장
    UserDao dao = new UserDao();
    int successCount = 0;

    data.first();
    while(data.next()) {
        dao.item("user_id", data.getString("col0"));
        dao.item("name", data.getString("col1"));
        dao.item("email", data.getString("col2"));
        dao.item("reg_date", m.time());

        if(dao.insert()) {
            successCount++;
        } else {
            m.p("오류: " + dao.getErrMsg());
        }
        dao.clear();
    }

    m.jsAlert(successCount + "명의 회원이 등록되었습니다.");
    m.jsReplace("list.jsp");
    return;
}

p.setBody("main.upload_excel");
p.setVar("form_script", f.getScript());
p.display();

%>
```

### 3. 검색 결과를 Excel로 다운로드

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

String mode = m.rs("mode");

if("download".equals(mode)) {

    // 검색 조건으로 데이터 조회 (페이징 없으므로 DataObject 사용)
    BlogDao blog = new BlogDao();
    blog.addSearch("subject,content", f.get("keyword"), "LIKE");

    int status = f.getInt("status", -1);
    if(status >= 0) {
        blog.addSearch("status", status);
    }

    blog.setOrderBy("id DESC");
    DataSet list = blog.find();

    // Excel 생성
    ExcelX excel = new ExcelX();

    String[] columns = {
        "id=>번호",
        "subject=>제목",
        "name=>작성자",
        "reg_date=>작성일",
        "hit=>조회수"
    };

    list.first();
    excel.setSheet("게시물목록");
    excel.setData(list, columns);

    excel.write(response, "게시물목록_" + m.time("yyyyMMdd") + ".xlsx");
    return;
}

// 일반 목록 화면 (페이징이 필요하므로 ListManager 사용)
ListManager lm = new ListManager();
lm.setRequest(request);
lm.setTable("tb_blog");
lm.setListNum(20);
lm.addSearch("subject,content", f.get("keyword"), "LIKE");

DataSet list = lm.getDataSet();

p.setBody("main.blog_list");
p.setLoop("list", list);
p.setVar("pager", lm.getPaging());
p.display();

%>
```

**HTML 템플릿에 다운로드 버튼 추가**:
```html
<a href="list.jsp?mode=download&keyword={{keyword}}" class="btn">Excel 다운로드</a>
```

### 4. 월별 통계 Excel 생성

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 월별 집계 데이터 조회
DataObject dao = new DataObject();
DataSet monthly = dao.queryDataSet(
    "SELECT " +
    "  DATE_FORMAT(reg_date, '%Y-%m') AS month, " +
    "  COUNT(*) AS cnt, " +
    "  SUM(amount) AS total " +
    "FROM tb_order " +
    "WHERE reg_date >= '2025-01-01' " +
    "GROUP BY DATE_FORMAT(reg_date, '%Y-%m') " +
    "ORDER BY month DESC"
);

// Excel 생성
ExcelX excel = new ExcelX();

String[] columns = {
    "month=>월",
    "cnt=>주문건수",
    "total=>매출합계"
};

monthly.first();
excel.setSheet("월별통계");
excel.setData(monthly, columns);

excel.write(response, "월별통계_" + m.time("yyyyMMdd") + ".xlsx");

%>
```

---

## 디버깅

### ExcelReader 디버깅

```jsp
ExcelReader excel = new ExcelReader(Config.getDataDir() + "/sample.xls");
excel.setDebug(out);  // 디버그 정보 출력
```

### ExcelX 디버깅

```jsp
ExcelX excel = new ExcelX();
excel.setDebug(out);  // 디버그 정보 출력
```

**출력 내용**:
- 파일 경로
- 시트 정보
- 컬럼 매핑
- 처리 시간

---

## 성능 최적화

### 대용량 파일 처리

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

ExcelReader excel = new ExcelReader(Config.getDataDir() + "/large_file.xls");

long start = System.currentTimeMillis();

excel.setSheet(0);
DataSet data = excel.getDataSet();

long end = System.currentTimeMillis();
m.p("처리 시간: " + (end - start) + "ms");
m.p("행 개수: " + data.size());

excel.close();

%>
```

### 메모리 관리

```jsp
// 사용 후 반드시 닫기
ExcelReader excel = new ExcelReader(filePath);
try {
    DataSet data = excel.getDataSet();
    // 처리...
} finally {
    excel.close();  // 메모리 해제
}
```

---

## 주요 메소드 정리

### ExcelReader (.xls 읽기)

| 메소드 | 설명 |
|--------|------|
| ExcelReader(filePath) | Excel 파일 로드 |
| getSheetCount() | 시트 개수 |
| getSheetName(index) | 시트 이름 |
| setSheet(index) | 시트 선택 (0부터 시작) |
| getDataSet() | 현재 시트를 DataSet으로 |
| setDebug(out) | 디버깅 모드 |
| close() | 리소스 해제 |

### ExcelX (.xlsx 읽기/쓰기)

| 메소드 | 설명 |
|--------|------|
| ExcelX() | 생성자 |
| read(filePath) | Excel 파일 읽기 → DataSet |
| setSheet(name) | 시트 이름 설정 |
| setData(dataSet, columns) | 데이터 및 컬럼 설정 |
| setPassword(password) | 비밀번호 설정 |
| write(filePath) | 파일로 저장 |
| write(response, fileName) | 다운로드로 전송 |
| setDebug(out) | 디버깅 모드 |

### 컬럼 정의 형식

```jsp
String[] columns = {
    "col0=>헤더1",  // col0을 "헤더1"로 표시
    "col1=>헤더2",  // col1을 "헤더2"로 표시
    "col2=>헤더3"   // col2를 "헤더3"로 표시
};
```

---

## 베스트 프랙티스

### 1. 파일 형식 선택

```jsp
// .xls (구버전, 호환성 중요)
ExcelReader excel = new ExcelReader(filePath);

// .xlsx (신버전, 더 많은 기능)
ExcelX excel = new ExcelX();
```

### 2. 대용량 처리

```jsp
// Bad - 메모리 부족 가능
DataSet data = excel.read("large_file.xlsx");

// Good - 배치 처리
int batchSize = 1000;
for(int i = 0; i < totalRows; i += batchSize) {
    // 일부씩 처리
}
```

### 3. 에러 처리

```jsp
try {
    ExcelX excel = new ExcelX();
    DataSet data = excel.read(filePath);

    if(data.size() == 0) {
        throw new Exception("데이터가 없습니다");
    }

    // 처리...

} catch(Exception e) {
    m.jsError("Excel 처리 중 오류: " + e.getMessage());
}
```

### 4. 사용자 파일 업로드 검증

```jsp
// 파일 확장자 검증
f.addElement("excel_file", null, "required:Y, accept:'.xlsx,.xls'");

if(m.isPost() && f.validate()) {

    String fileName = f.getFileName("excel_file");
    String ext = fileName.substring(fileName.lastIndexOf("."));

    if(!".xlsx".equalsIgnoreCase(ext) && !".xls".equalsIgnoreCase(ext)) {
        m.jsError("Excel 파일만 업로드 가능합니다");
        return;
    }

    // 처리...
}
```

---

## 다음 단계

- [인증 처리](authentication.md)로 이동하여 로그인/로그아웃 구현 방법을 배워보세요.

---

[← 이전: XML 처리](xml.md) | [목차로 돌아가기](README.md) | [다음: 인증 처리 →](authentication.md)
