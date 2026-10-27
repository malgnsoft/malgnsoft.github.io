# 멀티테넌시

하나의 웹앱(WebApp 하나)이 **도메인별로 서로 다른 DB·계정**을 사용해 여러 사이트를 서비스하는 기능입니다(`malgnsoft.tenant` 패키지, 1.19.0~). 앱 코드(JSP, 클래스)를 고치지 않고 쓸 수 있고, **설정하지 않으면 기존과 100% 동일하게 동작**합니다(옵트인).

## 구조

```
요청 Host → TenantFilter
   1) Host 정규화·검증, 도메인 → 사이트 (등록되지 않은 Host는 404)
   2) 사이트 상태 확인 (정지 403, 점검 503)
   3) 사이트별 동시 요청 한도 (초과 시 503 안내 페이지)
   4) 현재 사이트 지정(TenantContext) → 앱 실행 → 끝나면 반드시 해제
앱의 DB 접근 → 사이트별 커넥션 풀 → 그 사이트의 DB
```

## 설정 파일

| 파일 | 내용 | 비밀 |
|------|------|------|
| `WEB-INF/tenants.xml` | 사이트 id, DB 접속 정보, 풀·한도, 점검 메시지 | 있음(권한 600). 비밀번호는 `ENC:` 암호문 권장 |
| `WEB-INF/domains.xml` | 도메인 → 사이트 매핑, 별칭, 리다이렉트, `https="force"` | 없음 |

샘플: 저장소의 `tenants.xml.sample`, `domains.xml.sample`, `web.xml.sample`(필터 등록). 파일을 고치면 수 초 안에 자동으로 다시 읽고, 오류가 있으면 새 설정을 적용하지 않습니다.

## 커넥션 풀 (`<pool>`)

사이트마다 자체 풀을 가집니다(Resin 풀과 별개).

| 속성 | 기본 | 설명 |
|------|------|------|
| `max` | 3 | 사이트당 최대 연결 수 |
| `maxWaitMs` | 8000 | 풀이 가득 찼을 때 대기 시간. 넘으면 실패 |
| `leakWarnSec` | 60 | 연결을 이 시간 넘게 쥐고 있으면 경고 로그 |
| `leakReclaimSec` | 600 | 이 시간 넘게 쥔 연결을 **강제로 끊고 회수**(0이면 끔). 쥐고 있던 요청은 다음 사용 때 예외 |
| `queryTimeoutMs` | 300000 | MySQL `max_execution_time`(SELECT만)을 연결에 걸어 두는 서버 쪽 마지막 안전망. 0이면 안 함 |

일반 쿼리 제한은 `config.xml`의 `queryTimeout`(기본 30초)이 맡고, 코드의 `setTimeout()`으로 개별 연장합니다.

## 동시 요청 한도 (`<limits>`)

| 속성 | 기본 | 설명 |
|------|------|------|
| `requests` | 16 | 사이트별 동시 요청 상한 |
| `queueWaitMs` | 1000 | 상한이 찼을 때 기다리는 시간. 넘으면 503 |
| `heavy` / `heavyPerClient` | 4 / 1 | `heavyUrls`(기본 `excel,download`)에 해당하는 무거운 요청의 사이트별·IP별 상한 |

한도를 넘으면 **503 + `Retry-After`** 와 함께 방문자 안내 페이지가 나갑니다(브라우저는 HTML, Ajax·JSON 요청은 JSON). 필터 `init-param`으로 문구(`busyMessage`, `busyHeavyMessage`)나 HTML 파일(`busyPage`, `maintenancePage`, 안의 `{{message}}`가 문구로 바뀜)을 바꿀 수 있습니다.

## 사이트 간 분리

- **세션**: 다른 사이트 도메인으로 보낸 세션 ID는 무효화됩니다(`sessionBind`).
- **인증 쿠키·API 토큰**: 서명이 사이트 id로 묶여서 다른 사이트에서 재사용할 수 없습니다.
- **로그인 실패·요청 제한 카운터**: 사이트마다 따로입니다.
- **개인정보 암호화(PII)**: 고객사마다 다른 데이터 키(DEK)를 씁니다([암호화](encryption.md)).

## 요청 밖(배치·스케줄러)에서

요청을 거치지 않는 스레드에는 현재 사이트가 없습니다. 사이트를 지정해서 실행하세요.

```java
import malgnsoft.tenant.*;

Tenants.run("site001", () -> {
    // 이 안의 DB 접근은 site001
});

Tenants.forEach(tenant -> {
    // 모든 활성 사이트를 순회 (사이트별 오류 격리)
});
```

## 운영

- 통계·헬스: `/__tenant/stats` (로컬 접속만).
- 사이트 추가: DB·계정 생성 → `tenants.xml`·`domains.xml` 수정 → 수 초 후 반영(재시작 없음).
- Resin 풀(`resin-web.xml`)을 쓰는 단일 사이트는 그대로 유지됩니다.
- 설정 키는 저장소 `public_html/WEB-INF/SETUP.md`의 "멀티테넌시" 절을 참고하세요.
