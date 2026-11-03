# 멀티테넌시

하나의 웹앱(WebApp 하나)이 **도메인별로 서로 다른 DB·계정**을 사용해 여러 사이트를 서비스하는 기능입니다(`malgnsoft.tenant` 패키지, 1.19.0~). 앱 코드(JSP, 클래스)를 고치지 않고 쓸 수 있고, **설정하지 않으면 기존과 100% 동일하게 동작**합니다(옵트인).

## 켜는 방법 (1.20.6~)

Servlet 3.0 이상 컨테이너(Resin 4, Tomcat)에서는 `WEB-INF/tenants.xml`을 만들고 `config.xml`에 한 줄만 두면 됩니다. `web.xml`·`resin-web.xml`은 고치지 않습니다.

```xml
<multitenancy>Y</multitenancy>
```

- `TenantFilter`가 다른 필터보다 앞에 자동 등록됩니다. 시스템 속성 `-Dmalgn.tenant.auto=Y|N`이 `config.xml`보다 우선합니다(`N`이면 끔).
- 키가 없으면 아무 일도 하지 않습니다(기존과 동일). `web.xml`에 이미 `TenantFilter`를 등록했으면 중복 등록하지 않습니다.
- Servlet 2.5 컨테이너(Resin 3 등)이거나 필터 `init-param`을 `web.xml`에서 바꾸고 싶으면 예전처럼 `web.xml`에 직접 등록합니다(예시는 `SETUP.md`의 "멀티테넌시" 절).
- **DataSource `<resource>` 등록도 생략할 수 있습니다.** `DB`가 `config.xml`의 `jndi`(기본 `jdbc/malgn`)·`rojndi` 이름을 JNDI에서 찾지 못하면 내장 `TenantDataSource`를 씁니다. 컨테이너에 같은 이름의 리소스가 있으면 그쪽이 먼저입니다. 직접 등록하려면 `resin-web.xml`에 `<resource jndi-name="jdbc/malgn" type="malgnsoft.tenant.TenantDataSource"/>`처럼 `config.xml`의 `jndi`와 같은 이름으로 씁니다. 다른 이름(`new DB("jdbc/other")` 등)은 내장 DataSource가 처리하지 않습니다.

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
| `WEB-INF/tenants.xml` | 사이트 id, DB 접속 정보, 풀·한도, 점검 메시지, 도메인(`<domain>`/`<alias>`: 사이트 매핑, 별칭, 리다이렉트, `https="force"`) | 있음(권한 600). 비밀번호는 `ENC:` 암호문 권장 |

1.20.4부터 도메인을 `tenants.xml`의 `<tenant>` 안에 `<domain>`/`<alias>`로 적습니다(별도 `domains.xml`은 쓰지 않으며, 1.20.6에서 샘플도 없앴습니다). 이전 버전의 `domains.xml`이 있으면 계속 읽어 합치고, 도메인·별칭 이름은 둘을 통틀어 유일해야 합니다. 안쪽 `<alias>`는 같은 `<tenant>`의 `<domain>`만 가리킵니다(`<domain>`을 `<alias>`보다 먼저 적습니다).

```xml
<tenant id="site001">
  <db url="..." user="..." password="ENC:..."/>
  <domain name="a.example.com" primary="Y" https="force"/>
  <alias name="www.a.example.com" to="a.example.com" redirect="301"/>
</tenant>
```

샘플: 저장소의 `tenants.xml.sample`. 파일을 고치면 수 초 안에 자동으로 다시 읽고, 오류가 있으면 새 설정을 적용하지 않습니다.

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

## 읽기 전용 DB (`<rodb>`, 1.20.5~, 옵트인)

사이트마다 읽기 복제본이나 SELECT 전용 계정을 따로 둘 수 있습니다. `<tenant>` 안에 `<rodb>`(사이트당 1개, 속성은 `<db>`와 같음)를 적습니다.

- `config.xml`의 `rojndi` 이름은 읽기 전용 `TenantDataSource`가 맡습니다(`rojndi`를 JNDI에서 찾지 못하면 내장 읽기 전용 DataSource, 1.20.6~). 직접 등록하려면 리소스에 `readOnly=true`를 줍니다(`SETUP.md` 참고).
- 읽기 연결은 세션이 읽기 전용이고, 앱이 `setReadOnly(false)`를 불러도 거부됩니다. 단 SQL로 세션을 다시 쓰기로 바꾸는 것까지 막지는 않으므로 SELECT 전용 계정을 권장합니다.
- `<rodb>`가 없는 사이트는 읽기 전용 DataSource도 쓰기 DB 풀을 씁니다.
- 읽기 풀은 `<global maxConnections>`와 별개로 셉니다. DB 서버의 연결 한도를 계산할 때 둘을 더하세요. `driver`를 생략하면 `<db>`의 것을 이어받습니다.
- MySQL 5.6.5 미만처럼 `SET SESSION TRANSACTION READ ONLY`를 모르는 서버에서는 경고 후 JDBC의 읽기 전용 표시에만 의존합니다.
- 통계(`/__tenant/stats`)의 `readInUse`와 `pools_detail`의 `"ro":true`로 읽기 풀을 따로 볼 수 있습니다.

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
- **로그아웃 폐기 목록**: 사이트별로 나뉩니다(1.20.4~). 한 사이트의 로그아웃이 다른 사이트의 같은 로그인 시각 항목에 영향을 주지 않습니다.
- **개인정보 암호화(PII)**: 고객사마다 다른 데이터 키(DEK)를 씁니다([암호화](encryption.md)).

## 요청 범위 DB 회수와 재진입 (1.20.4~)

- `TenantFilter`가 요청마다 `DB.openRequestScope()`/`closeRequestScope()`를 호출해, `begin()`/`startTrans()` 뒤 `close()`를 빠뜨린 커넥션을 요청 끝에 롤백·반환하고 경고를 남깁니다. 범위가 없는 스레드(배치 등)는 영향이 없습니다.
- 같은 요청이 필터를 다시 지날 때(`forward`/`include`, 오류 페이지)는 동시 요청 한도를 다시 걸지 않고, 안쪽이 끝나도 바깥 통과의 사이트 지정을 지우지 않습니다.

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
- 사이트 추가: DB·계정 생성 → `tenants.xml` 수정 → 수 초 후 반영(재시작 없음).
- Resin 풀(`resin-web.xml`)을 쓰는 단일 사이트는 그대로 유지됩니다.
- 설정 키는 저장소 `public_html/WEB-INF/SETUP.md`의 "멀티테넌시" 절을 참고하세요.
