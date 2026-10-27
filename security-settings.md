# 보안 설정 모음

맑은프레임워크 1.18~1.20에서 추가되거나 **기본값이 바뀐** 보안 설정을 한곳에 모은 문서입니다. 설정은 `/WEB-INF/config.xml`의 `<env>` 아래에 적고 웹앱을 재시작해야 반영됩니다. 모든 설정 키의 정본은 저장소의 `public_html/WEB-INF/config.xml.sample`이며, 설치·운영 절차는 `SETUP.md`에 있습니다.

> 끄는 값은 키마다 같은 규칙을 씁니다: `N`, `no`, `false`, `off`, `0` (대소문자 무시).
> 설정이 맞게 되어 있는지는 `bin/malgn doctor`로 점검할 수 있습니다.

## 1. 인증 (Auth)

| 설정 | 기본 | 설명 |
|------|------|------|
| `WEB-INF/auth.keys` | 필요 | 로그인 쿠키·API 토큰의 서명 키 목록. `bin/malgn auth-keys new`로 만들고 `auth-keys check`로 확인. 경로는 `authKeyFile`로 바꿀 수 있음. **키를 바꾸면 모든 사용자가 다시 로그인합니다** |
| `authSameSite` | Lax | 인증 쿠키 `SameSite`. 결제(PG)·본인인증처럼 다른 사이트가 POST로 돌려보내는 흐름이 있으면 `off` |
| `authHttpOnly` | Y | 인증 쿠키를 JS(`document.cookie`)에서 읽을 수 없음. 읽어야 하는 코드가 있으면 `N` |
| `authSecure` | Y | HTTPS에서만 전송. **HTTP 개발 환경은 `N`** |
| `authValidTime` | 7200 (초) | 무활동 만료(슬라이딩). `setValidTime()`을 부르는 사이트는 그 값이 우선. `0`이면 끔 |
| `authAbsoluteTime` | 꺼짐 | 처음 로그인부터의 절대 만료(초) |
| `authLogoutRevoke` | Y | 로그아웃한 쿠키를 같은 서버에서 더는 인증하지 않음 |

## 2. 입력·출력

| 설정 | 기본 | 설명 |
|------|------|------|
| `requestFilter` | `safe` | `m.request()` 계열이 NUL 등 제어문자를 제거(탭·줄바꿈은 유지). 이전 동작은 `legacy` |
| `templateEscape` | N | 템플릿 `{{변수}}` HTML 이스케이프. `log`(영향받는 변수만 로그에 기록) → HTML 변수에 `{{변수\|raw}}` 적용 → `Y` 순서로 전환 |
| `Form.get()` XSS 필터 | 항상 | 화이트리스트 sanitizer. 신뢰하는 관리자 필드만 `addElement(..., "allowscript:'Y'")` |
| `redirect()` | 항상 | `javascript:`·`data:`·`//호스트` 같은 위험 URL은 사이트 루트로 이동. 허용할 외부 호스트는 `redirectAllowHosts`(쉼표 구분, `*.corp.co.kr` 형식 가능) |

## 3. 데이터베이스

| 설정 | 기본 | 설명 |
|------|------|------|
| `queryTimeout` | **30 (초)** | 모든 쿼리 기본 제한 시간(1.20.2~, 키가 없어도 적용). 넘으면 쿼리가 중단되고 에러 로그에 남음. 끄려면 `0`/`off`. **30초를 넘는 엑셀·통계·배치 쿼리는 해당 쿼리만 `setTimeout(초)`로 늘리세요** |
| `slowQueryMs` | 10000 | 이 시간(ms) 이상 걸린 쿼리를 에러 로그에 `{DB.slowQuery}`로 기록(동작 불변). 끄려면 `0`/`off` |
| `maxRows`, `maxListNum` | 없음 | `RecordSet`이 읽는 최대 행 수, `ListManager` 한 페이지 최대 크기 |
| `strictColumns` | Y | `DataObject.item(Map)` 키가 컬럼 이름 형식이 아니면 거부 |
| `strictOper` | Y | `addSearch`의 연산자를 허용 목록(`= <> != < > <= >= LIKE %LIKE% LIKE% %LIKE`)으로 제한 |
| `strictFields` | Y | `addSearch`·`addFilter`·`addSearchLike`·`setOrderBy(String)`의 필드를 문법으로 제한. `AES_DEC(col)`, `LOWER/UPPER/TRIM(col)`, `IS [NOT] NULL`, `col <op> 정수`, `ORDER BY 1`, `-col`은 허용 |
| `sqlFilter` | `log` | 위험한 SQL(주석, `INTO OUTFILE`, `SLEEP(` 등)을 `{DB.sqlFilter}` 로그로 기록. `block`이면 실행을 차단(주석·여러 문장을 보내는 코드가 있으면 실패하므로 로그로 확인한 뒤 전환). 근본 대책은 항상 `?` 바인딩 |

쿼리 제한 시간을 개별로 바꾸는 방법:

```java
DataObject dao = new DataObject("tb_report");
dao.setTimeout(120);          // 이 객체의 쿼리는 120초까지 (0이면 제한 없음)

ListManager lm = new ListManager();
lm.setTimeout(120);

DB db = new DB();
db.setTimeout(120);
```

코드의 `setTimeout()` 값이 `config.xml`의 `queryTimeout`보다 우선합니다.

## 4. 업로드·압축·엑셀

| 설정 | 기본 | 설명 |
|------|------|------|
| `denyExtExtra` | `svg,svgz,xht,xhtml` | 기본 거부 확장자에 더하는 목록. svg가 필요하면 `off` |
| `uploadVerifyContent` | Y | 저장 전에 파일 내용을 확장자와 대조(`.jpg`에 HTML이면 거부). `N`으로 끔 |
| `zipDenyExecutable` | Y | 서버 실행 확장자(jsp·php·asp·shtml 등)가 든 zip은 풀지 않고 거부 |
| `excelMaxCells` | 힙÷1800 (10만~500만) | 엑셀을 읽을 때 최대 칸 수. 열기 전에 시트를 훑어 초과하면 거부 |
| `excelMaxRows` / `excelMaxCols` | 1048576 / 256 | |
| `excelMaxEntrySize` | 힙÷20 (8~512MB) | 압축을 푼 엑셀 내부 파일 하나의 최대 크기 |
| `maxPostSize` | | 업로드 최대 크기(MB) |

## 5. 프록시(ALB·CDN·Apache) 뒤에서

| 설정 | 설명 |
|------|------|
| `trustedProxies` | 프록시 서버 IP(쉼표 구분, CIDR 가능). 설정하면 연결 IP가 목록에 있을 때만 `X-Forwarded-For` 등을 신뢰. **프록시 뒤 사이트는 반드시 설정** |
| `forwardedProto` | 기본 켜짐. `X-Forwarded-Proto: https`가 오면 `redirect()`의 상대 경로 이동과 `getWebUrl()`이 `https://` URL을 만듦(1.20.3~, ALB·CDN이 TLS를 끝내고 웹서버는 HTTP만 받는 구성). 끄려면 `N` |
| `redirectPort` | 이동 URL의 포트를 고정(설정하면 `forwardedProto`보다 우선) |

CSRF 쿠키 `Secure`·HSTS 같은 **보안 판단은 계속 `trustedProxies`에 등록된 프록시가 보낸 헤더만** 믿습니다.

## 6. 요청 제한·로그인 시도 제한

```java
// 같은 키로 60초에 5번까지만 허용
if(!Malgn.rateLimit("sms:" + phone, 5, 60)) { m.jsError("잠시 후 다시 시도하세요."); return; }

// 로그인 실패 기록과 차단 확인
if(Malgn.loginBlocked(loginId)) { m.jsError("로그인 시도가 너무 많습니다."); return; }
if(!ok) Malgn.loginFailed(loginId);
```

멀티테넌시에서는 카운터가 사이트마다 따로입니다(한 사이트의 실패가 다른 사이트를 막지 않음).

## 7. 점검 도구

`bin/malgn`(저장소의 `bin/malgn`, GitHub Release에도 첨부)이 운영 도구를 감싸 줍니다.

```
bin/malgn doctor [웹앱폴더]       설정·키·라이브러리 점검 (종료 코드 0 정상, 1 실패, 2 경고)
bin/malgn auth-keys new|rotate|check
bin/malgn vault gen-key|wrap <용도>
bin/malgn jsp-compile <웹앱>      JSP 병렬 사전 컴파일
```

자세한 내용은 [암호화](encryption.md), [설치 및 환경설정](installation.md)을 참고하세요.
