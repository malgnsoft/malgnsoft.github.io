# 이메일 발송

[← 목차로 돌아가기](README.md)

---

## 개요

맑은프레임워크는 Gmail 클래스를 통해 간편하게 이메일을 발송할 수 있습니다. Gmail, 네이버 메일, AWS SES 등 다양한 SMTP 서버를 지원합니다.

### 주요 기능

- **다양한 SMTP 지원**: Gmail, 네이버, AWS SES, 자체 SMTP 서버
- **파일 첨부**: 여러 파일 첨부 가능
- **HTML 이메일**: HTML 형식의 이메일 본문 지원
- **여러 수신자**: 한 번에 여러 명에게 발송
- **간편한 설정**: Config를 통한 기본 설정

---

## 목차

1. [기본 이메일 발송](#기본-이메일-발송)
2. [Gmail 계정 사용](#gmail-계정-사용)
3. [네이버 메일 사용](#네이버-메일-사용)
4. [AWS SES 사용](#aws-ses-사용)
5. [파일 첨부](#파일-첨부)
6. [HTML 이메일](#html-이메일)
7. [여러 수신자](#여러-수신자)
   - [여러 명에게 발송](#1-여러-명에게-발송)
   - [DB에서 조회하여 발송](#2-db에서-조회하여-발송)
   - [개별 발송 (개인화)](#3-개별-발송-개인화)
   - [대량 발송 (백그라운드 스레드)](#4-대량-발송-백그라운드-스레드)
8. [환경설정](#환경설정)

---

## 기본 이메일 발송

### 1. Malgn 클래스 사용 (가장 간단)

Config에 설정된 기본 메일 서버를 사용합니다:

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 가장 간단한 방법
m.mail("user@example.com", "제목", "내용");

%>
```

### 2. Gmail 클래스 사용 (기본 설정)

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

Gmail mail = new Gmail();
mail.send("user@example.com", "안녕하세요", "테스트 메일입니다.");

%>
```

---

## Gmail 계정 사용

Gmail SMTP를 통해 이메일을 발송할 수 있습니다.

### 1. Gmail 앱 비밀번호 발급

1. Google 계정 설정 → 보안
2. 2단계 인증 활성화
3. 앱 비밀번호 생성 (16자리)

### 2. Gmail로 발송

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// Gmail 계정 정보 설정
Gmail mail = new Gmail("your-email@gmail.com", "앱비밀번호");

// 이메일 발송
mail.send("recipient@example.com", "제목입니다", "본문 내용입니다.");

%>
```

### 3. 전체 예제

**send_email.jsp**:
```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

f.addElement("email", null, "required:'Y', type:'email'");
f.addElement("subject", null, "required:'Y'");
f.addElement("message", null, "required:'Y'");

if(m.isPost() && f.validate()) {

    String toEmail = f.get("email");
    String subject = f.get("subject");
    String message = f.get("message");

    try {
        Gmail mail = new Gmail("your-email@gmail.com", "앱비밀번호");
        mail.send(toEmail, subject, message);

        m.jsAlert("이메일이 발송되었습니다.");
        m.jsReplace("list.jsp");
    } catch(Exception e) {
        m.jsError("이메일 발송 실패: " + e.getMessage());
    }
    return;
}

p.setBody("mail.send_form");
p.setVar("form_script", f.getScript());
p.display();

%>
```

**html/mail/send_form.html**:
```html
<form name="form1" method="post">
    <p>받는 사람 : <input type="text" name="email"></p>
    <p>제목 : <input type="text" name="subject"></p>
    <p>내용 : <textarea name="message" rows="10"></textarea></p>
    <p><input type="submit" value="발송"></p>
</form>
{{form_script}}
```

---

## 네이버 메일 사용

네이버 메일 SMTP를 통해 발송할 수 있습니다.

### 1. 네이버 메일 설정

```jsp
Gmail mail = new Gmail("smtp.naver.com", "네이버아이디", "비밀번호");
mail.setMailFrom("네이버아이디@naver.com");
mail.send("recipient@example.com", "제목", "내용");
```

### 2. 전체 예제

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

String naverId = "your_naver_id";
String naverPw = "your_password";

Gmail mail = new Gmail("smtp.naver.com", naverId, naverPw);
mail.setMailFrom(naverId + "@naver.com");

mail.send("recipient@example.com", "네이버 메일 테스트", "안녕하세요. 네이버 메일로 발송합니다.");

m.p("이메일이 발송되었습니다.");

%>
```

---

## AWS SES 사용

Amazon Simple Email Service를 통해 대량 이메일을 안정적으로 발송할 수 있습니다.

### 1. AWS SES 설정

1. AWS SES 콘솔에서 이메일 주소 인증
2. SMTP 자격 증명 생성
3. 리전별 SMTP 엔드포인트 확인

### 2. AWS SES로 발송

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

String sesHost = "email-smtp.ap-northeast-2.amazonaws.com";  // 서울 리전
String sesUsername = "AKIAXXXXXXXXXXXXXXXX";  // SMTP 사용자 이름
String sesPassword = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx";  // SMTP 비밀번호

Gmail mail = new Gmail(sesHost, sesUsername, sesPassword);
mail.setMailFrom("verified@yourdomain.com");  // SES에서 인증된 이메일

mail.send("recipient@example.com", "AWS SES 테스트", "AWS SES를 통해 발송합니다.");

m.p("이메일이 발송되었습니다.");

%>
```

### 3. 주요 리전별 엔드포인트

```jsp
// 서울 (ap-northeast-2)
String sesHost = "email-smtp.ap-northeast-2.amazonaws.com";

// 미국 동부 (us-east-1)
String sesHost = "email-smtp.us-east-1.amazonaws.com";

// 미국 서부 (us-west-2)
String sesHost = "email-smtp.us-west-2.amazonaws.com";

// 유럽 (eu-west-1)
String sesHost = "email-smtp.eu-west-1.amazonaws.com";
```

---

## 파일 첨부

### 1. 단일 파일 첨부

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

String filePath = Config.getDataDir() + "/file/document.pdf";

Gmail mail = new Gmail("your-email@gmail.com", "앱비밀번호");
mail.send(
    "recipient@example.com",
    "파일 첨부 메일",
    "첨부 파일을 확인해주세요.",
    new String[] { filePath }
);

m.p("첨부 파일과 함께 발송되었습니다.");

%>
```

### 2. 여러 파일 첨부

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

String[] files = {
    Config.getDataDir() + "/file/document1.pdf",
    Config.getDataDir() + "/file/document2.pdf",
    Config.getDataDir() + "/file/image.jpg"
};

Gmail mail = new Gmail("your-email@gmail.com", "앱비밀번호");
mail.send("recipient@example.com", "여러 파일 첨부", "파일들을 확인해주세요.", files);

m.p("여러 파일과 함께 발송되었습니다.");

%>
```

### 3. 업로드된 파일 첨부

**send_email_with_file.jsp**:
```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

f.addElement("email", null, "required:'Y', type:'email'");
f.addElement("subject", null, "required:'Y'");
f.addElement("message", null, "required:'Y'");
f.addElement("file", null, "");

if(m.isPost() && f.validate()) {

    String toEmail = f.get("email");
    String subject = f.get("subject");
    String message = f.get("message");

    // 파일 업로드
    File uploadedFile = f.saveFile("file");
    String[] files = null;

    if(uploadedFile != null) {
        files = new String[] { uploadedFile.getAbsolutePath() };
    }

    // 이메일 발송
    Gmail mail = new Gmail("your-email@gmail.com", "앱비밀번호");
    mail.send(toEmail, subject, message, files);

    m.jsAlert("이메일이 발송되었습니다.");
    m.jsReplace("list.jsp");
    return;
}

p.setBody("mail.send_form_with_file");
p.setVar("form_script", f.getScript());
p.display();

%>
```

**html/mail/send_form_with_file.html**:
```html
<form name="form1" method="post" enctype="multipart/form-data">
    <p>받는 사람 : <input type="text" name="email"></p>
    <p>제목 : <input type="text" name="subject"></p>
    <p>내용 : <textarea name="message" rows="10"></textarea></p>
    <p>첨부 파일 : <input type="file" name="file"></p>
    <p><input type="submit" value="발송"></p>
</form>
{{form_script}}
```

---

## HTML 이메일

HTML 형식의 이메일을 발송할 수 있습니다.

### 1. HTML 본문

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

String htmlBody = "<html>" +
    "<body>" +
    "<h1 style='color: #333;'>안녕하세요!</h1>" +
    "<p>이것은 <strong>HTML</strong> 이메일입니다.</p>" +
    "<p style='color: #666;'>HTML 태그를 사용할 수 있습니다.</p>" +
    "<hr>" +
    "<p><a href='https://example.com' style='color: #007bff;'>링크 보기</a></p>" +
    "</body>" +
    "</html>";

Gmail mail = new Gmail("your-email@gmail.com", "앱비밀번호");
mail.send("recipient@example.com", "HTML 이메일", htmlBody);

m.p("HTML 이메일이 발송되었습니다.");

%>
```

### 2. 템플릿을 사용한 HTML 이메일

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 사용자 정보 조회
int userId = m.ri("user_id");
UserDao dao = new UserDao();
DataSet user = dao.find("id = ?", userId);

if(user.next()) {

    // init.jsp의 Page 객체(p) 활용
    p.setBody("mail.welcome");
    p.setVar("name", user.s("name"));
    p.setVar("email", user.s("email"));
    p.setVar("site_url", Config.getSiteUrl());

    String htmlBody = p.fetch();

    // 이메일 발송
    Gmail mail = new Gmail();
    mail.send(user.s("email"), "회원가입을 환영합니다", htmlBody);

    m.p("회원가입 환영 이메일이 발송되었습니다.");
}

%>
```

**html/mail/welcome.html**:
```html
<html>
<head>
    <style>
        body { font-family: Arial, sans-serif; line-height: 1.6; color: #333; }
        .container { max-width: 600px; margin: 0 auto; padding: 20px; }
        .header { background-color: #007bff; color: white; padding: 20px; text-align: center; }
        .content { padding: 20px; background-color: #f9f9f9; }
        .button { display: inline-block; padding: 10px 20px; background-color: #007bff; color: white; text-decoration: none; border-radius: 5px; }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>회원가입을 환영합니다!</h1>
        </div>
        <div class="content">
            <p><strong>{{name}}</strong>님, 안녕하세요!</p>
            <p>회원가입이 완료되었습니다.</p>
            <p>이메일: {{email}}</p>
            <p style="text-align: center; margin-top: 30px;">
                <a href="{{site_url}}" class="button">사이트 방문하기</a>
            </p>
        </div>
    </div>
</body>
</html>
```

---

## 여러 수신자

### 1. 여러 명에게 발송

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

String[] recipients = {
    "user1@example.com",
    "user2@example.com",
    "user3@example.com"
};

Gmail mail = new Gmail("your-email@gmail.com", "앱비밀번호");
mail.send(recipients, "공지사항", "중요한 공지사항입니다.");

m.p("여러 명에게 발송되었습니다.");

%>
```

### 2. DB에서 조회하여 발송

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 이메일 수신 동의한 회원 조회
UserDao dao = new UserDao();
DataSet users = dao.find("email_agree = 'Y'");

ArrayList<String> emailList = new ArrayList<>();
while(users.next()) {
    emailList.add(users.s("email"));
}

if(emailList.size() > 0) {
    String[] recipients = emailList.toArray(new String[0]);

    String subject = "이벤트 안내";
    String message = "안녕하세요. 특별 이벤트를 안내드립니다...";

    Gmail mail = new Gmail();
    mail.send(recipients, subject, message);

    m.p(emailList.size() + "명에게 이메일을 발송했습니다.");
} else {
    m.p("발송할 이메일이 없습니다.");
}

%>
```

### 3. 개별 발송 (개인화)

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

UserDao dao = new UserDao();
DataSet users = dao.find("email_agree = 'Y'");

Gmail mail = new Gmail("your-email@gmail.com", "앱비밀번호");

int sentCount = 0;
while(users.next()) {
    String email = users.s("email");
    String name = users.s("name");

    String subject = name + "님께 특별 제안";
    String message = "<h1>" + name + "님, 안녕하세요!</h1>" +
                     "<p>" + name + "님을 위한 특별한 혜택을 준비했습니다.</p>";

    try {
        mail.send(email, subject, message);
        sentCount++;
    } catch(Exception e) {
        m.p("발송 실패: " + email + " - " + e.getMessage());
    }
}

m.p("총 " + sentCount + "명에게 개별 이메일을 발송했습니다.");

%>
```

### 4. 대량 발송 (백그라운드 스레드)

대량의 이메일을 발송할 때는 `m.mailer()` 메소드를 사용하여 백그라운드 스레드로 처리할 수 있습니다.

#### 기본 사용법

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 백그라운드 스레드로 이메일 발송 (논블로킹)
m.mailer("user@example.com", "제목", "내용");

// 즉시 다음 코드 실행 (이메일 발송 완료를 기다리지 않음)
m.p("이메일이 발송 대기열에 추가되었습니다.");

%>
```

#### 파일 첨부와 함께 사용

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

String filePath = Config.getDataDir() + "/file/document.pdf";

// 백그라운드로 파일 첨부 이메일 발송
m.mailer("user@example.com", "파일 첨부", "첨부 파일을 확인하세요.", filePath);

m.p("파일 첨부 이메일이 발송 대기열에 추가되었습니다.");

%>
```

#### 템플릿과 함께 대량 발송

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

UserDao dao = new UserDao();
DataSet users = dao.find("email_agree = 'Y'");

int queuedCount = 0;

while(users.next()) {
    String email = users.s("email");
    String name = users.s("name");

    // 템플릿으로 개인화된 HTML 생성
    p.setBody("mail.promotion");
    p.setVar("name", name);
    p.setVar("user_id", users.s("id"));
    String htmlBody = p.fetch();

    // 백그라운드로 발송 (즉시 반환)
    m.mailer(email, name + "님께 특별 할인", htmlBody);
    queuedCount++;
}

m.p(queuedCount + "건의 이메일이 발송 대기열에 추가되었습니다.");

%>
```

#### m.mail() vs m.mailer() 비교

| 메소드 | 처리 방식 | 속도 | 사용 시기 |
|--------|----------|------|-----------|
| `m.mail()` | 동기 (블로킹) | 느림 (발송 완료까지 대기) | 1~10건 정도의 소량 발송<br>발송 성공 여부를 즉시 확인해야 할 때 |
| `m.mailer()` | 비동기 (논블로킹) | 빠름 (즉시 반환) | 수십~수백 건의 대량 발송<br>발송 완료를 기다릴 필요 없을 때<br>사용자 대기 시간을 줄여야 할 때 |

**예시:**

```jsp
// 동기 발송 - 발송이 완료될 때까지 대기 (약 2~3초)
m.mail("user@example.com", "제목", "내용");
m.p("이메일이 발송되었습니다.");  // 실제 발송 완료 후 출력

// 비동기 발송 - 즉시 반환 (약 0.01초)
m.mailer("user@example.com", "제목", "내용");
m.p("이메일이 발송 대기열에 추가되었습니다.");  // 즉시 출력
```

#### MailThread 동작 원리

`m.mailer()`는 내부적으로 `MailThread` 클래스를 사용하여 백그라운드에서 이메일을 발송합니다:

1. **스레드 풀 관리**: 최대 50개의 동시 발송 스레드 지원
2. **자동 큐잉**: 50개 스레드가 모두 사용 중이면 대기 (100ms씩 최대 50회 재시도)
3. **폴백 처리**: 50회 재시도 후에도 스레드를 확보하지 못하면 자동으로 `m.mail()`로 동기 발송
4. **자동 정리**: 발송 완료 후 스레드 카운트 자동 감소

```jsp
// 내부 동작 예시
Malgn.mailThreadNum = 0;  // 현재 실행 중인 메일 스레드 수

// 첫 번째 발송
m.mailer("user1@example.com", "제목1", "내용1");  // mailThreadNum = 1

// 두 번째 발송
m.mailer("user2@example.com", "제목2", "내용2");  // mailThreadNum = 2

// ... (계속 발송)

// 51번째 발송 시 (스레드 풀이 가득 찬 경우)
m.mailer("user51@example.com", "제목51", "내용51");  // 100ms 대기 후 재시도
```

#### 주의사항

1. **SMTP 서버 제한**: Gmail 등은 일일 발송 제한이 있습니다 (Gmail: 500건/일)
2. **발송 결과 확인 불가**: 비동기 발송이므로 발송 성공/실패를 즉시 확인할 수 없습니다
3. **스레드 제한**: 동시에 50개 이상의 발송 요청 시 대기 또는 동기 발송으로 폴백됩니다
4. **서버 재시작**: 서버 재시작 시 대기열의 이메일은 손실될 수 있습니다

#### 대량 발송 권장 방법

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

UserDao dao = new UserDao();
DataSet users = dao.find("email_agree = 'Y'");

int totalCount = 0;
int batchSize = 100;  // 100건씩 배치 처리

while(users.next()) {
    String email = users.s("email");
    String name = users.s("name");

    p.setBody("mail.newsletter");
    p.setVar("name", name);
    String htmlBody = p.fetch();

    // 백그라운드로 발송
    m.mailer(email, "뉴스레터", htmlBody);
    totalCount++;

    // 100건마다 5초 대기 (SMTP 서버 부하 방지)
    if(totalCount % batchSize == 0) {
        Thread.sleep(5000);
        m.p(totalCount + "건 발송 대기열 추가 완료... 5초 대기");
    }
}

m.p("총 " + totalCount + "건의 이메일이 발송 대기열에 추가되었습니다.");

%>
```

#### config.xml 설정

대량 발송 시 타임아웃 설정을 조정할 수 있습니다:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<config>
    <!-- 메일 서버 설정 -->
    <mailHost>smtp.gmail.com</mailHost>
    <mailPort>587</mailPort>
    <mailFrom>your-email@gmail.com</mailFrom>
    <mailUser>your-email@gmail.com</mailUser>
    <mailPass>your-app-password</mailPass>

    <!-- 메일 스레드 수 (선택 사항, 기본값: 0) -->
    <mailThreadNum>0</mailThreadNum>
</config>
```

---

## 환경설정

### 1. config.xml 설정

**WEB-INF/config/config.xml**:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<config>
    <!-- 메일 서버 설정 -->
    <mailHost>smtp.gmail.com</mailHost>
    <mailPort>587</mailPort>
    <mailFrom>your-email@gmail.com</mailFrom>
    <mailUser>your-email@gmail.com</mailUser>
    <mailPass>your-app-password</mailPass>
</config>
```

### 2. 기본 설정 사용

```jsp
// config.xml에 설정된 정보로 자동 초기화
Gmail mail = new Gmail();
mail.send("recipient@example.com", "제목", "내용");
```

### 3. 수동 설정

```jsp
Gmail mail = new Gmail();

// SMTP 서버 설정
mail.setHost("smtp.gmail.com");
mail.setPort(587);
mail.setSSL(true);

// 인증 정보
mail.setAuth("your-email@gmail.com", "your-app-password");

// 발신자 설정
mail.setMailFrom("your-email@gmail.com");

// 인코딩 설정 (기본: utf-8)
mail.setEncoding("utf-8");

// 이메일 발송
mail.send("recipient@example.com", "제목", "내용");
```

### 4. 포트 번호

```jsp
// 587 포트 (TLS/STARTTLS) - 권장
mail.setPort(587);
mail.setSSL(true);

// 465 포트 (SSL)
mail.setPort(465);

// 25 포트 (일반 SMTP)
mail.setPort(25);
```

---

## 전체 예제

### 문의하기 폼

**contact.jsp**:
```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

f.addElement("name", null, "required:'Y', maxlength:50");
f.addElement("email", null, "required:'Y', type:'email'");
f.addElement("subject", null, "required:'Y', maxlength:200");
f.addElement("message", null, "required:'Y'");

if(m.isPost() && f.validate()) {

    String name = f.get("name");
    String email = f.get("email");
    String subject = f.get("subject");
    String message = f.get("message");

    // DB에 저장
    ContactDao dao = new ContactDao();
    DataMap data = new DataMap();
    data.put("name", name);
    data.put("email", email);
    data.put("subject", subject);
    data.put("message", message);
    data.put("reg_date", m.time());
    int newId = dao.insert(data);

    // 관리자에게 이메일 발송
    String adminEmail = "admin@example.com";
    String emailSubject = "[문의] " + subject;
    String emailBody = "<h2>새로운 문의가 등록되었습니다</h2>" +
                       "<p><strong>이름:</strong> " + name + "</p>" +
                       "<p><strong>이메일:</strong> " + email + "</p>" +
                       "<p><strong>제목:</strong> " + subject + "</p>" +
                       "<p><strong>내용:</strong></p>" +
                       "<div style='padding: 10px; background: #f5f5f5;'>" + message + "</div>" +
                       "<hr>" +
                       "<p><a href='" + Config.getSiteUrl() + "/admin/contact/view.jsp?id=" + newId + "'>문의 보기</a></p>";

    try {
        Gmail mail = new Gmail();
        mail.send(adminEmail, emailSubject, emailBody);

        // 고객에게 자동 응답 발송
        String replySubject = "[자동응답] 문의가 접수되었습니다";
        String replyBody = "<h2>" + name + "님, 안녕하세요</h2>" +
                          "<p>문의가 정상적으로 접수되었습니다.</p>" +
                          "<p>빠른 시일 내에 답변 드리겠습니다.</p>" +
                          "<p>감사합니다.</p>";
        mail.send(email, replySubject, replyBody);

        m.jsAlert("문의가 접수되었습니다.");
        m.jsReplace("index.jsp");

    } catch(Exception e) {
        m.jsError("이메일 발송에 실패했습니다: " + e.getMessage());
    }
    return;
}

p.setBody("mail.contact_form");
p.setVar("form_script", f.getScript());
p.display();

%>
```

**html/mail/contact_form.html**:
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>문의하기</title>
    <style>
        .form-group { margin-bottom: 15px; }
        label { display: block; margin-bottom: 5px; font-weight: bold; }
        input[type="text"], input[type="email"], textarea {
            width: 100%;
            padding: 8px;
            border: 1px solid #ddd;
            border-radius: 4px;
        }
        textarea { min-height: 150px; }
        button { padding: 10px 20px; background: #007bff; color: white; border: none; border-radius: 4px; cursor: pointer; }
    </style>
</head>
<body>
    <h1>문의하기</h1>
    <form name="form1" method="post">
        <div class="form-group">
            <label>이름</label>
            <input type="text" name="name">
        </div>
        <div class="form-group">
            <label>이메일</label>
            <input type="email" name="email">
        </div>
        <div class="form-group">
            <label>제목</label>
            <input type="text" name="subject">
        </div>
        <div class="form-group">
            <label>내용</label>
            <textarea name="message"></textarea>
        </div>
        <button type="submit">문의하기</button>
    </form>
    {{form_script}}
</body>
</html>
```

---

## 베스트 프랙티스

### 1. 예외 처리

```jsp
try {
    Gmail mail = new Gmail();
    mail.send("user@example.com", "제목", "내용");
    m.p("이메일 발송 성공");
} catch(Exception e) {
    m.p("이메일 발송 실패: " + e.getMessage());
    Malgn.errorLog("Email send error", e);
}
```

### 2. 이메일 주소 검증

```jsp
String email = f.get("email");

// Form 클래스로 검증
f.addElement("email", null, "type:'email', required:'Y'");

// 또는 정규표현식으로 검증
if(!email.matches("^[A-Za-z0-9+_.-]+@(.+)$")) {
    m.jsError("올바른 이메일 주소를 입력하세요.");
    return;
}
```

### 3. 대량 발송 시 지연

```jsp
UserDao dao = new UserDao();
DataSet users = dao.find("email_agree = 'Y'");

Gmail mail = new Gmail();
int count = 0;

while(users.next()) {
    String email = users.s("email");

    try {
        mail.send(email, "제목", "내용");
        count++;

        // 100건마다 5초 대기 (SMTP 서버 부하 방지)
        if(count % 100 == 0) {
            Thread.sleep(5000);
        }

    } catch(Exception e) {
        Malgn.errorLog("Email send error: " + email, e);
    }
}

m.p("총 " + count + "건 발송 완료");
```

### 4. 비동기 발송

중요하지 않은 이메일은 별도 스레드로 발송:

```jsp
// 회원가입 처리
UserDao dao = new UserDao();
dao.insert(userData);

// 비동기로 환영 이메일 발송
new Thread(() -> {
    try {
        Gmail mail = new Gmail();
        mail.send(userEmail, "회원가입 환영", welcomeMessage);
    } catch(Exception e) {
        Malgn.errorLog("Welcome email error", e);
    }
}).start();

// 즉시 다음 페이지로 이동
m.jsReplace("login.jsp");
```

### 5. 로깅

```jsp
Gmail mail = new Gmail();

try {
    mail.send(email, subject, body);

    // 발송 로그 저장
    EmailLogDao logDao = new EmailLogDao();
    DataMap log = new DataMap();
    log.put("to_email", email);
    log.put("subject", subject);
    log.put("status", "success");
    log.put("sent_date", m.time());
    logDao.insert(log);

} catch(Exception e) {
    // 실패 로그 저장
    EmailLogDao logDao = new EmailLogDao();
    DataMap log = new DataMap();
    log.put("to_email", email);
    log.put("subject", subject);
    log.put("status", "failed");
    log.put("error_message", e.getMessage());
    log.put("sent_date", m.time());
    logDao.insert(log);
}
```

---

## 참고 문서

- [환경설정 및 캐시](configuration.md)
- [파일 업로드 및 다운로드](file-upload-download.md)
- [템플릿 - 이메일 템플릿](template.md)

---

[← 목차로 돌아가기](README.md)
