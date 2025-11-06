# 파일 업로드 및 다운로드

[← 목차로 돌아가기](README.md)

---

## 개요

맑은프레임워크는 Form 클래스와 Malgn 클래스를 통해 간편한 파일 업로드 및 다운로드 기능을 제공합니다.

### 주요 기능

- **파일 업로드**: Form 클래스를 통한 멀티파트 파일 업로드
- **파일 다운로드**: Malgn 클래스를 통한 파일 전송
- **확장자 검증**: 허용/금지 확장자 제한
- **여러 파일 처리**: 다중 파일 업로드 지원
- **대역폭 제어**: 다운로드 속도 제한
- **이미지 썸네일**: 자동 썸네일 생성

---

## 목차

1. [파일 업로드](#파일-업로드)
2. [파일 다운로드](#파일-다운로드)
3. [이미지 썸네일](#이미지-썸네일)
4. [파일 관리](#파일-관리)
5. [보안 가이드](#보안-가이드)

---

## 파일 업로드

### 1. 기본 업로드

**JSP**:
```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

f.addElement("title", null, "required:'Y'");
f.addElement("file", null, "required:'Y'");

if(m.isPost() && f.validate()) {

    // 파일 저장
    File file = f.saveFile("file");
    if(file != null) {
        String fileName = f.getFileName("file");
        String filePath = file.getAbsolutePath();
        long fileSize = file.length();

        m.p("파일명: " + fileName);
        m.p("저장경로: " + filePath);
        m.p("파일크기: " + fileSize);

        // DB에 파일 정보 저장
        FileDao dao = new FileDao();
        dao.item("title", f.get("title"));
        dao.item("file_name", fileName);
        dao.item("file_path", filePath);
        dao.item("file_size", fileSize);
        dao.item("reg_date", m.time());
        dao.insert();

        m.jsAlert("업로드 완료");
        m.jsReplace("list.jsp");
    } else {
        m.jsError("파일 업로드 실패");
    }
    return;
}

%>
<form name="form1" method="post" enctype="multipart/form-data">
    <p>제목 : <input type="text" name="title"></p>
    <p>파일 : <input type="file" name="file"></p>
    <p><input type="submit" value="업로드"></p>
</form>
{{form_script}}
```

**HTML 폼 주의사항**:
- `method="post"` 필수
- `enctype="multipart/form-data"` 필수

### 2. 확장자 제한

#### 허용 확장자 지정

```jsp
// 이미지만 허용
f.addElement("photo", null, "allow:'jpg|jpeg|png|gif'");

// 문서 파일만 허용
f.addElement("document", null, "allow:'pdf|doc|docx|xls|xlsx'");
```

#### 금지 확장자 지정

```jsp
// 실행 파일 업로드 금지
f.addElement("file", null, "deny:'exe|bat|cmd|sh'");

// 서버 스크립트 금지
f.addElement("file", null, "deny:'jsp|php|asp|aspx'");
```

#### 조합 사용

```jsp
// 이미지만 허용하되, bmp는 제외
f.addElement("image", null, "allow:'jpg|jpeg|png|gif|bmp', deny:'bmp'");
```

### 3. 파일 크기 제한

```jsp
// web.xml에서 설정
<multipart-config>
    <max-file-size>10485760</max-file-size>       <!-- 10MB -->
    <max-request-size>52428800</max-request-size> <!-- 50MB -->
</multipart-config>
```

또는 코드에서 검증:

```jsp
if(m.isPost() && f.validate()) {
    File file = f.saveFile("file");
    if(file != null) {
        long fileSize = file.length();
        long maxSize = 10 * 1024 * 1024;  // 10MB

        if(fileSize > maxSize) {
            file.delete();
            m.jsError("파일 크기는 10MB 이하만 가능합니다.");
            return;
        }

        // 처리...
    }
    return;
}
```

### 4. 여러 파일 업로드

#### 다중 선택 (배열)

**HTML**:
```html
<input type="file" name="files" multiple>
```

**JSP**:
```jsp
f.addElement("files", null, "required:'Y'");

if(m.isPost() && f.validate()) {

    File[] files = f.saveFiles("files");
    if(files != null && files.length > 0) {
        for(int i = 0; i < files.length; i++) {
            File file = files[i];
            String fileName = f.getFileNames("files")[i];

            m.p("파일 " + (i+1) + ": " + fileName);
            m.p("경로: " + file.getAbsolutePath());

            // DB 저장...
        }

        m.jsAlert(files.length + "개 파일 업로드 완료");
        m.jsReplace("list.jsp");
    }
    return;
}
```

#### 개별 필드

**HTML**:
```html
<input type="file" name="file1">
<input type="file" name="file2">
<input type="file" name="file3">
```

**JSP**:
```jsp
f.addElement("file1", null, "");
f.addElement("file2", null, "");
f.addElement("file3", null, "");

if(m.isPost() && f.validate()) {

    File file1 = f.saveFile("file1");
    File file2 = f.saveFile("file2");
    File file3 = f.saveFile("file3");

    // 각 파일 처리...
    return;
}
```

### 5. 저장 경로 커스터마이징

```jsp
// 기본 저장 경로는 /data/file/

// 커스텀 경로로 저장
String customPath = Config.getDataDir() + "/uploads/2025/06/";
File dir = new File(customPath);
if(!dir.exists()) {
    dir.mkdirs();
}

File uploadedFile = f.saveFile("file");
if(uploadedFile != null) {
    String newPath = customPath + uploadedFile.getName();
    File newFile = new File(newPath);
    uploadedFile.renameTo(newFile);

    m.p("최종 경로: " + newFile.getAbsolutePath());
}
```

### 6. 파일명 변경하여 저장

```jsp
if(m.isPost() && f.validate()) {

    File file = f.saveFile("file");
    if(file != null) {
        String originalName = f.getFileName("file");
        String ext = Malgn.getFileExt(originalName);

        // 고유한 파일명 생성
        String newFileName = m.time() + "_" + Malgn.getUniqId() + "." + ext;
        String savePath = Config.getDataDir() + "/file/" + newFileName;

        File newFile = new File(savePath);
        file.renameTo(newFile);

        m.p("원본 파일명: " + originalName);
        m.p("저장 파일명: " + newFileName);

        // DB에 두 파일명 모두 저장
        FileDao dao = new FileDao();
        dao.item("original_name", originalName);
        dao.item("saved_name", newFileName);
        dao.item("file_path", savePath);
        dao.insert();
    }
    return;
}
```

---

## 파일 다운로드

### 1. 기본 다운로드

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

int id = m.ri("id");

FileDao dao = new FileDao();
DataSet info = dao.find("id = ?", new Object[]{id});

if(!info.next()) {
    m.jsError("파일을 찾을 수 없습니다.");
    return;
}

String filePath = info.s("file_path");
String fileName = info.s("file_name");

// 파일 다운로드
m.download(filePath, fileName);

%>
```

### 2. 대역폭 제한 다운로드

대용량 파일 다운로드 시 서버 부하를 줄이기 위해 속도 제한:

```jsp
// 100 KB/s로 제한
m.download(filePath, fileName, 100);

// 500 KB/s로 제한
m.download(filePath, fileName, 500);
```

### 3. 파일 브라우저 표시 (미리보기)

다운로드가 아닌 브라우저에서 직접 표시:

```jsp
// PDF 파일 브라우저에 표시
String pdfPath = Config.getDataDir() + "/document.pdf";
m.output(pdfPath, "application/pdf");

// 이미지 표시
String imagePath = Config.getDataDir() + "/photo.jpg";
m.output(imagePath, "image/jpeg");

// MIME 타입 자동 감지
m.output(filePath, null);
```

### 4. 조건부 다운로드

권한 확인 후 다운로드:

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

int id = m.ri("id");

// 로그인 체크
if(!auth.isLogin()) {
    m.jsError("로그인이 필요합니다.");
    return;
}

FileDao dao = new FileDao();
DataSet info = dao.find("id = ?", new Object[]{id});

if(!info.next()) {
    m.jsError("파일을 찾을 수 없습니다.");
    return;
}

// 권한 체크
int uploadUserId = info.i("user_id");
int currentUserId = auth.getUserId();

if(uploadUserId != currentUserId && !auth.isAdmin()) {
    m.jsError("다운로드 권한이 없습니다.");
    return;
}

// 다운로드 카운트 증가
dao.execute("UPDATE tb_file SET download_count = download_count + 1 WHERE id = ?", id);

// 다운로드
String filePath = info.s("file_path");
String fileName = info.s("file_name");
m.download(filePath, fileName);

%>
```

### 5. 폼 방식 다운로드

GET 방식 대신 POST로 다운로드:

**JSP**:
```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

if(m.isPost()) {
    int id = m.ri("id");

    FileDao dao = new FileDao();
    DataSet info = dao.find("id = ?", new Object[]{id});

    if(info.next()) {
        String filePath = info.s("file_path");
        String fileName = info.s("file_name");
        m.download(filePath, fileName);
    }
    return;
}

%>
<form method="post">
    <input type="hidden" name="id" value="123">
    <button type="submit">다운로드</button>
</form>
```

### 6. 여러 파일 ZIP으로 다운로드

여러 파일을 ZIP으로 압축하여 한 번에 다운로드:

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 다운로드할 파일 목록
String[] fileIds = m.reqArr("file_ids");

if(fileIds.length == 0) {
    m.jsError("선택된 파일이 없습니다.");
    return;
}

FileDao dao = new FileDao();
ArrayList<String> filePaths = new ArrayList<>();

for(String id : fileIds) {
    DataSet info = dao.find("id = ?", new Object[]{id});
    if(info.next()) {
        filePaths.add(info.s("file_path"));
    }
}

// ZIP 압축 및 다운로드
Zip zip = new Zip();
String[] files = filePaths.toArray(new String[0]);
zip.compress(files, "files_" + m.time() + ".zip", response);

%>
```

---

## 이미지 썸네일

### 1. 이미지 업로드 시 썸네일 자동 생성

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

f.addElement("photo", null, "required:'Y', allow:'jpg|jpeg|png|gif'");

if(m.isPost() && f.validate()) {

    File file = f.saveFile("photo");
    if(file != null) {
        String filePath = file.getAbsolutePath();
        String fileName = f.getFileName("photo");
        String ext = Malgn.getFileExt(fileName);

        // 썸네일 생성
        String thumbPath = Config.getDataDir() + "/thumb/";
        File thumbDir = new File(thumbPath);
        if(!thumbDir.exists()) {
            thumbDir.mkdirs();
        }

        String thumbFileName = "thumb_" + m.time() + "." + ext;
        String thumbFullPath = thumbPath + thumbFileName;

        Thumbnail thumb = new Thumbnail();
        // 가로 200px로 리사이징 (비율 유지)
        thumb.createThumbnail(filePath, thumbFullPath, 200, 0);

        // DB 저장
        ImageDao dao = new ImageDao();
        dao.item("title", f.get("title"));
        dao.item("image_path", filePath);
        dao.item("thumb_path", thumbFullPath);
        dao.item("file_size", file.length());
        dao.item("reg_date", m.time());
        dao.insert();

        m.jsAlert("이미지 업로드 완료");
        m.jsReplace("list.jsp");
    }
    return;
}

%>
<form name="form1" method="post" enctype="multipart/form-data">
    <p>제목 : <input type="text" name="title"></p>
    <p>이미지 : <input type="file" name="photo" accept="image/*"></p>
    <p><input type="submit" value="업로드"></p>
</form>
{{form_script}}
```

### 2. 다양한 크기의 썸네일 생성

```jsp
File file = f.saveFile("photo");
if(file != null) {
    String sourcePath = file.getAbsolutePath();
    String thumbDir = Config.getDataDir() + "/thumb/";

    Thumbnail thumb = new Thumbnail();

    // 작은 썸네일 (100x100)
    thumb.createThumbnail(sourcePath, thumbDir + "small_" + fileName, 100, 100);

    // 중간 썸네일 (300x300)
    thumb.createThumbnail(sourcePath, thumbDir + "medium_" + fileName, 300, 300);

    // 큰 썸네일 (800px 너비, 높이 자동)
    thumb.createThumbnail(sourcePath, thumbDir + "large_" + fileName, 800, 0);
}
```

### 3. 정사각형 크롭

```jsp
// 200x200 정사각형으로 중앙 크롭
Thumbnail thumb = new Thumbnail();
thumb.createThumbnail(sourcePath, thumbPath, 200, 200);
```

---

## 파일 관리

### 1. 파일 정보 조회

```jsp
// 파일 확장자
String ext = Malgn.getFileExt("document.pdf");  // "pdf"

// MIME 타입
String mime = Malgn.getMimeType("image.png");   // "image/png"

// 파일 크기 포맷
String size = Malgn.getFileSize(1024);          // "1KB"
String size2 = Malgn.getFileSize(1048576);      // "1MB"
String size3 = Malgn.getFileSize(1073741824);   // "1GB"
```

### 2. 파일 읽기/쓰기

```jsp
// 파일 읽기
String content = Malgn.readFile("/path/to/file.txt");
String content2 = Malgn.readFile("/path/to/file.txt", "EUC-KR");

// 파일 쓰기
Malgn.writeFile("/path/to/output.txt", "Hello World");
Malgn.writeFile("/path/to/output.txt", "안녕하세요", "EUC-KR");
```

### 3. 파일 복사/이동/삭제

```jsp
// 파일 복사
Malgn.copyFile("/source/file.txt", "/dest/file.txt");

// 파일 삭제
Malgn.delFile("/path/to/file.txt");

// 폴더 재귀 삭제
Malgn.delFile("/path/to/dir", true);
```

### 4. 업로드 경로 생성

```jsp
// MD5 기반 고유 경로 생성
String uploadPath = m.getUploadPath("myfile.jpg");
// 결과: /data/file/abc123def456.jpg

String uploadUrl = m.getUploadUrl("myfile.jpg");
// 결과: http://example.com/file/abc123def456.jpg
```

---

## 보안 가이드

### 1. 확장자 검증 필수

```jsp
// Bad - 확장자 검증 없음
f.addElement("file", null, "required:'Y'");

// Good - 허용 확장자 명시
f.addElement("file", null, "required:'Y', allow:'jpg|jpeg|png|gif|pdf|doc|docx'");

// Better - 위험한 확장자 명시적 차단
f.addElement("file", null, "required:'Y', allow:'jpg|jpeg|png|gif|pdf|doc|docx', deny:'exe|jsp|php|asp|sh|bat'");
```

### 2. 파일명 검증

```jsp
String fileName = f.getFileName("file");

// 위험한 문자 제거
fileName = fileName.replaceAll("[^a-zA-Z0-9._-]", "_");

// 경로 조작 방지
fileName = fileName.replace("..", "");
fileName = fileName.replace("/", "");
fileName = fileName.replace("\\", "");
```

### 3. 파일 크기 검증

```jsp
File file = f.saveFile("file");
if(file != null) {
    long maxSize = 10 * 1024 * 1024;  // 10MB
    if(file.length() > maxSize) {
        file.delete();
        m.jsError("파일 크기 초과");
        return;
    }
}
```

### 4. MIME 타입 검증

```jsp
String fileName = f.getFileName("file");
String ext = Malgn.getFileExt(fileName);

// 확장자와 MIME 타입 매칭 확인
String expectedMime = Malgn.getMimeType(fileName);
// 실제 파일 MIME 타입과 비교하여 검증
```

### 5. 업로드 디렉토리 권한

```jsp
// 업로드 디렉토리에 실행 권한 제거 (Linux/Unix)
String uploadDir = Config.getDataDir() + "/file/";
Malgn.chmod("755", uploadDir);

// 또는 web.xml에서 실행 차단
```

### 6. 다운로드 권한 체크

```jsp
// 항상 권한 검증 후 다운로드
if(!auth.isLogin()) {
    m.jsError("로그인이 필요합니다.");
    return;
}

// 소유자 또는 관리자만 다운로드
if(!isOwner && !auth.isAdmin()) {
    m.httpError(403, "권한이 없습니다.");
    return;
}

m.download(filePath, fileName);
```

### 7. 경로 조작 방지

```jsp
// Bad - 사용자 입력을 직접 경로로 사용
String fileName = m.rs("file");
m.download("/data/file/" + fileName, fileName);  // 위험!

// Good - DB에서 경로 조회
int fileId = m.ri("id");
FileDao dao = new FileDao();
DataSet info = dao.find("id = ?", new Object[]{fileId});
if(info.next()) {
    m.download(info.s("file_path"), info.s("file_name"));
}
```

---

## 전체 예제

### 파일 게시판 구현

**list.jsp** (목록):
```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

ListManager lm = new ListManager();
lm.setRequest(request);
lm.setTable("tb_file");
lm.setListNum(20);
lm.addSearch("title,file_name", f.get("keyword"), "LIKE");
lm.setSort("id DESC");

DataSet list = lm.getDataSet();

p.setBody("main.file_list");
p.setLoop("list", list);
p.setVar("pager", lm.getPaging());
p.display();

%>
```

**upload.jsp** (업로드):
```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

if(!auth.isLogin()) {
    m.jsError("로그인이 필요합니다.");
    return;
}

f.addElement("title", null, "required:'Y', maxlength:200");
f.addElement("content", null, "allowhtml:'Y'");
f.addElement("file", null, "required:'Y', allow:'jpg|jpeg|png|gif|pdf|doc|docx|xls|xlsx|zip', deny:'exe|jsp|php'");

if(m.isPost() && f.validate()) {

    File file = f.saveFile("file");
    if(file != null) {
        String fileName = f.getFileName("file");
        String filePath = file.getAbsolutePath();
        long fileSize = file.length();

        FileDao dao = new FileDao();
        dao.item("user_id", auth.getUserId());
        dao.item("title", f.get("title"));
        dao.item("content", f.get("content"));
        dao.item("file_name", fileName);
        dao.item("file_path", filePath);
        dao.item("file_size", fileSize);
        dao.item("download_count", 0);
        dao.item("reg_date", m.time());

        dao.insert();
        int newId = dao.getInsertId();

        m.jsAlert("업로드 완료");
        m.jsReplace("view.jsp?id=" + newId);
    } else {
        m.jsError("파일 업로드 실패");
    }
    return;
}

p.setBody("main.file_upload");
p.setVar("form_script", f.getScript());
p.display();

%>
```

**download.jsp** (다운로드):
```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

int id = m.ri("id");

FileDao dao = new FileDao();
DataSet info = dao.find("id = ?", new Object[]{id});

if(!info.next()) {
    m.jsError("파일을 찾을 수 없습니다.");
    return;
}

// 다운로드 카운트 증가
dao.execute("UPDATE tb_file SET download_count = download_count + 1 WHERE id = ?", id);

// 다운로드
String filePath = info.s("file_path");
String fileName = info.s("file_name");
m.download(filePath, fileName);

%>
```

**delete.jsp** (삭제):
```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

if(!auth.isLogin()) {
    m.jsError("로그인이 필요합니다.");
    return;
}

int id = m.ri("id");

FileDao dao = new FileDao();
DataSet info = dao.find("id = ?", new Object[]{id});

if(!info.next()) {
    m.jsError("파일을 찾을 수 없습니다.");
    return;
}

// 권한 체크
int uploadUserId = info.i("user_id");
int currentUserId = auth.getUserId();

if(uploadUserId != currentUserId && !auth.isAdmin()) {
    m.jsError("삭제 권한이 없습니다.");
    return;
}

// 파일 삭제
String filePath = info.s("file_path");
File file = new File(filePath);
if(file.exists()) {
    file.delete();
}

// DB 삭제
dao.delete("id = ?", new Object[]{id});

m.jsAlert("삭제되었습니다.");
m.jsReplace("list.jsp");

%>
```

---

## 베스트 프랙티스

### 1. 항상 확장자 검증

```jsp
f.addElement("file", null, "allow:'jpg|png|pdf|doc', deny:'exe|jsp|php'");
```

### 2. 파일 크기 제한

```jsp
long maxSize = 10 * 1024 * 1024;  // 10MB
if(file.length() > maxSize) {
    file.delete();
    m.jsError("파일 크기는 10MB 이하만 가능합니다.");
    return;
}
```

### 3. 고유한 파일명 사용

```jsp
String uniqueName = m.time() + "_" + Malgn.getUniqId() + "." + ext;
```

### 4. DB에 메타데이터 저장

```jsp
data.put("original_name", fileName);  // 사용자가 업로드한 원본 파일명
data.put("saved_name", uniqueName);   // 서버에 저장된 파일명
data.put("file_size", file.length()); // 파일 크기
data.put("file_ext", ext);            // 확장자
data.put("mime_type", mimeType);      // MIME 타입
```

### 5. 권한 체크

```jsp
// 업로드
if(!auth.isLogin()) {
    m.jsError("로그인이 필요합니다.");
    return;
}

// 다운로드/삭제
if(!isOwner && !auth.isAdmin()) {
    m.jsError("권한이 없습니다.");
    return;
}
```

### 6. 에러 처리

```jsp
File file = f.saveFile("file");
if(file == null) {
    m.jsError("파일 업로드에 실패했습니다.");
    return;
}

if(!file.exists()) {
    m.jsError("파일이 존재하지 않습니다.");
    return;
}
```

---

## 참고 문서

- [Form 클래스 - 데이터 입력 및 유효성 체크](form-validation.md)
- [유틸리티 메소드 - 파일 처리](utility-methods.md#파일-처리)
- [FTP 및 ZIP - 파일 전송 및 압축](file-transfer.md)

---

[← 이전: 데이터 입력 및 유효성 체크](form-validation.md) | [목차로 돌아가기](README.md) | [다음: 목록 및 검색 →](list-search.md)
