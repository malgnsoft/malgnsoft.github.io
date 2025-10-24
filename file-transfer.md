# 파일 전송 및 압축

맑은프레임워크는 FTP 파일 전송과 ZIP 압축/해제 기능을 제공합니다.

## 목차

- [FTP 파일 전송](#ftp-파일-전송)
  - [기본 연결](#기본-연결)
  - [파일 업로드](#파일-업로드)
  - [디렉토리 작업](#디렉토리-작업)
- [ZIP 압축 및 해제](#zip-압축-및-해제)
  - [파일 압축](#파일-압축)
  - [압축 해제](#압축-해제)
  - [다운로드 압축](#다운로드-압축)

---

## FTP 파일 전송

SimpleFTP 클래스는 간단하고 직관적인 FTP 클라이언트 기능을 제공합니다.

### 기본 연결

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

SimpleFTP ftp = new SimpleFTP();

try {
    // 기본 포트(21)로 연결
    ftp.connect("ftp.example.com");

    // 또는 포트 지정
    ftp.connect("ftp.example.com", 21);

    // 또는 사용자명/비밀번호 지정
    ftp.connect("ftp.example.com", 21, "username", "password");

    m.p("FTP 연결 성공");

} catch(Exception e) {
    m.p("FTP 연결 실패: " + e.getMessage());
}

%>
```

### 익명 로그인

```jsp
SimpleFTP ftp = new SimpleFTP();

// 기본적으로 anonymous/anonymous로 로그인
ftp.connect("ftp.example.com");
```

---

## 파일 업로드

### 단일 파일 업로드

```jsp
SimpleFTP ftp = new SimpleFTP();

try {
    // FTP 서버 연결
    ftp.connect("ftp.example.com", 21, "username", "password");

    // 바이너리 모드 설정 (이미지, 압축파일 등)
    ftp.bin();

    // 파일 업로드
    File file = new File("/local/path/document.pdf");
    boolean success = ftp.stor(file);

    if(success) {
        m.p("파일 업로드 성공");
    } else {
        m.p("파일 업로드 실패");
    }

    // 연결 종료
    ftp.disconnect();

} catch(Exception e) {
    m.p("오류: " + e.getMessage());
}
```

### ASCII 모드

```jsp
// 텍스트 파일 업로드 시
ftp.ascii();
File textFile = new File("/local/path/readme.txt");
ftp.stor(textFile);
```

### InputStream으로 업로드

```jsp
SimpleFTP ftp = new SimpleFTP();
ftp.connect("ftp.example.com", 21, "user", "pass");
ftp.bin();

// InputStream 사용
FileInputStream fis = new FileInputStream("/local/path/image.jpg");
boolean success = ftp.stor(fis, "uploaded_image.jpg");
fis.close();

ftp.disconnect();
```

---

## 디렉토리 작업

### 현재 디렉토리 확인

```jsp
SimpleFTP ftp = new SimpleFTP();
ftp.connect("ftp.example.com", 21, "user", "pass");

String currentDir = ftp.pwd();
m.p("현재 디렉토리: " + currentDir);
// 출력: /home/user
```

### 디렉토리 변경

```jsp
SimpleFTP ftp = new SimpleFTP();
ftp.connect("ftp.example.com", 21, "user", "pass");

// 디렉토리 변경
boolean success = ftp.cwd("/public_html/uploads");

if(success) {
    m.p("디렉토리 변경 성공");
    m.p("현재 위치: " + ftp.pwd());
} else {
    m.p("디렉토리 변경 실패");
}
```

---

## 실제 FTP 사용 예제

### 파일 업로드 폼

**upload_ftp.jsp**
```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

if(m.isPost()) {
    // 로컬에 업로드된 파일
    String uploadedFile = m.rs("uploaded_file_path");

    SimpleFTP ftp = new SimpleFTP();

    try {
        // FTP 연결
        ftp.connect(
            Config.get("ftp.host"),
            Config.getInt("ftp.port"),
            Config.get("ftp.user"),
            Config.get("ftp.pass")
        );

        // 업로드 디렉토리로 이동
        ftp.cwd("/uploads");

        // 바이너리 모드
        ftp.bin();

        // 파일 업로드
        File file = new File(uploadedFile);
        boolean success = ftp.stor(file);

        // 연결 종료
        ftp.disconnect();

        if(success) {
            m.jsAlert("FTP 업로드 성공");
            m.jsReplace("/list");
        } else {
            m.jsError("FTP 업로드 실패");
        }

    } catch(Exception e) {
        Malgn.errorLog("FTP 오류", e);
        m.jsError("FTP 오류: " + e.getMessage());
    }

    return;
}

p.setBody("upload.ftp");
p.display();

%>
```

### 배치 파일 업로드

```jsp
SimpleFTP ftp = new SimpleFTP();
ftp.connect("ftp.example.com", 21, "user", "pass");
ftp.cwd("/uploads");
ftp.bin();

// 여러 파일 업로드
String[] files = {
    "/local/path/file1.jpg",
    "/local/path/file2.jpg",
    "/local/path/file3.jpg"
};

int successCount = 0;
for(String filePath : files) {
    File file = new File(filePath);
    if(ftp.stor(file)) {
        successCount++;
        m.p(file.getName() + " 업로드 완료");
    }
}

ftp.disconnect();
m.p("총 " + successCount + "개 파일 업로드 완료");
```

---

## ZIP 압축 및 해제

Zip 클래스는 파일과 폴더의 압축 및 압축 해제 기능을 제공합니다.

### 파일 압축

#### 단일 파일 압축

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

Zip zip = new Zip();
zip.setDebug(out);

String sourceFile = Config.getDataDir() + "/document.pdf";
String zipFile = Config.getDataDir() + "/document.zip";

boolean success = zip.compress(sourceFile, zipFile);

if(success) {
    m.p("압축 완료: " + zipFile);
} else {
    m.p("압축 실패: " + zip.errMsg);
}

%>
```

#### 여러 파일 압축

```jsp
Zip zip = new Zip();

String[] files = {
    Config.getDataDir() + "/file1.txt",
    Config.getDataDir() + "/file2.txt",
    Config.getDataDir() + "/image.jpg"
};

String zipFile = Config.getDataDir() + "/archive.zip";

boolean success = zip.compress(files, zipFile);

if(success) {
    m.p("압축 완료");
} else {
    m.p("압축 실패: " + zip.errMsg);
}
```

#### 폴더 전체 압축

```jsp
Zip zip = new Zip();

// 폴더 압축
String folderPath = Config.getDataDir() + "/uploads";
String zipFile = Config.getDataDir() + "/uploads_backup.zip";

boolean success = zip.compress(folderPath, zipFile);

if(success) {
    m.p("폴더 압축 완료");
}
```

#### 파일명 변경하여 압축

```jsp
Zip zip = new Zip();

// "원본경로=>압축파일명" 형식
String[] files = {
    Config.getDataDir() + "/temp/report_2025.pdf=>report.pdf",
    Config.getDataDir() + "/temp/data_2025.xlsx=>data.xlsx"
};

String zipFile = Config.getDataDir() + "/files.zip";
zip.compress(files, zipFile);
```

---

## 압축 해제

### 기본 압축 해제

```jsp
Zip zip = new Zip();

String zipFile = Config.getDataDir() + "/archive.zip";
String extractFolder = Config.getDataDir() + "/extracted";

boolean success = zip.extract(zipFile, extractFolder);

if(success) {
    m.p("압축 해제 완료: " + extractFolder);
} else {
    m.p("압축 해제 실패: " + zip.errMsg);
}
```

### File 객체로 압축 해제

```jsp
Zip zip = new Zip();

File zipFile = new File(Config.getDataDir() + "/archive.zip");
String extractFolder = Config.getDataDir() + "/extracted";

boolean success = zip.extract(zipFile, extractFolder);
```

---

## 다운로드 압축

### 즉시 다운로드

브라우저로 직접 압축 파일을 전송할 수 있습니다:

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

Zip zip = new Zip();

String[] files = {
    Config.getDataDir() + "/report.pdf",
    Config.getDataDir() + "/data.xlsx",
    Config.getDataDir() + "/images/photo.jpg"
};

// 파일을 저장하지 않고 바로 다운로드
boolean success = zip.compress(files, "download.zip", response);

if(!success) {
    m.jsError("압축 실패: " + zip.errMsg);
}

%>
```

### 폴더 다운로드

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

Zip zip = new Zip();

String folderPath = Config.getDataDir() + "/uploads/user_123";

// 폴더를 압축하여 바로 다운로드
zip.compress(folderPath, "user_files.zip", response);

%>
```

---

## 실제 활용 예제

### 백업 시스템

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 매일 자정에 실행되는 백업 스크립트

Zip zip = new Zip();
SimpleFTP ftp = new SimpleFTP();

String today = Malgn.time("yyyyMMdd");
String backupFile = Config.getDataDir() + "/backup_" + today + ".zip";

// 1. 데이터 폴더 압축
String dataFolder = Config.getDataDir() + "/uploads";
boolean compressed = zip.compress(dataFolder, backupFile);

if(!compressed) {
    Malgn.errorLog("백업 압축 실패: " + zip.errMsg);
    return;
}

m.p("백업 파일 생성 완료: " + backupFile);

// 2. FTP로 백업 파일 업로드
try {
    ftp.connect("backup.example.com", 21, "backup_user", "backup_pass");
    ftp.cwd("/backups");
    ftp.bin();

    File file = new File(backupFile);
    boolean uploaded = ftp.stor(file);

    ftp.disconnect();

    if(uploaded) {
        m.p("FTP 백업 완료");

        // 로컬 백업 파일 삭제 (선택사항)
        file.delete();
    } else {
        m.p("FTP 업로드 실패");
    }

} catch(Exception e) {
    Malgn.errorLog("FTP 백업 오류", e);
}

%>
```

### 대용량 파일 다운로드

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 사용자가 선택한 파일들을 압축하여 다운로드

String[] fileIds = m.reqArr("file_id");

if(fileIds.length == 0) {
    m.jsError("파일을 선택해주세요");
    return;
}

// 파일 경로 조회
FileDao dao = new FileDao();
DataSet files = dao.query("WHERE id IN (?)", Malgn.implode(",", fileIds));

ArrayList<String> filePaths = new ArrayList<>();
while(files.next()) {
    String path = files.s("file_path");
    File file = new File(path);
    if(file.exists()) {
        filePaths.add(path);
    }
}

if(filePaths.isEmpty()) {
    m.jsError("다운로드 가능한 파일이 없습니다");
    return;
}

// 압축 및 다운로드
Zip zip = new Zip();
String downloadName = "files_" + Malgn.time("yyyyMMddHHmmss") + ".zip";

boolean success = zip.compress(
    filePaths.toArray(new String[0]),
    downloadName,
    response
);

if(!success) {
    m.jsError("압축 오류: " + zip.errMsg);
}

%>
```

### 이미지 갤러리 다운로드

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

int galleryId = m.ri("gallery_id");

// 갤러리의 모든 이미지 조회
ImageDao dao = new ImageDao();
DataSet images = dao.query("WHERE gallery_id = ?", galleryId);

String[] files = new String[images.size()];
int idx = 0;

while(images.next()) {
    String originalName = images.s("original_name");
    String filePath = images.s("file_path");

    // 압축 파일 내에서 원본 파일명 사용
    files[idx++] = filePath + "=>" + originalName;
}

// 압축 다운로드
Zip zip = new Zip();
zip.compress(files, "gallery_" + galleryId + ".zip", response);

%>
```

---

## 디버그 모드

### Zip 디버그

```jsp
Zip zip = new Zip();
zip.setDebug(out);  // 압축 과정 출력

String[] files = {"/path/file1.txt", "/path/file2.txt"};
zip.compress(files, "output.zip");

// 출력:
// /path/file1.txt
// /path/file2.txt
```

### 에러 확인

```jsp
Zip zip = new Zip();

boolean success = zip.compress("/path/file.txt", "output.zip");

if(!success) {
    // 에러 메시지 확인
    m.p("에러: " + zip.errMsg);
    Malgn.errorLog("압축 실패: " + zip.errMsg);
}
```

---

## 한글 파일명 처리

### ZIP 한글 파일명

Zip 클래스는 한글 파일명을 자동으로 처리합니다 (EUC-KR 인코딩):

```jsp
Zip zip = new Zip();

String[] files = {
    Config.getDataDir() + "/보고서.pdf",
    Config.getDataDir() + "/회의록.docx"
};

// 한글 파일명 자동 처리
zip.compress(files, "문서모음.zip", response);
```

### 압축 해제 시 한글 처리

```jsp
Zip zip = new Zip();

// EUC-KR 인코딩된 ZIP 파일 해제
zip.extract("한글파일명.zip", Config.getDataDir() + "/extracted");
```

---

## 주의사항

### FTP

1. **포트**: 기본 FTP 포트는 21
2. **패시브 모드**: SimpleFTP는 패시브 모드를 사용하여 방화벽 문제 최소화
3. **연결 종료**: 작업 완료 후 반드시 `disconnect()` 호출
4. **전송 모드**:
   - 바이너리 파일: `ftp.bin()` 사용
   - 텍스트 파일: `ftp.ascii()` 사용

### ZIP

1. **메모리**: 대용량 파일 압축 시 메모리 사용량 주의
2. **한글 인코딩**: EUC-KR 사용 (한국 환경 최적화)
3. **경로**: 절대 경로 사용 권장
4. **디렉토리 생성**: 압축 해제 시 필요한 디렉토리 자동 생성

---

## 관련 문서

- [유틸리티 메소드](utility-methods.md) - 파일 처리 유틸리티
- [Excel 처리](excel.md) - Excel 파일 생성 및 읽기
- [환경설정 및 캐시](configuration.md) - 경로 설정
