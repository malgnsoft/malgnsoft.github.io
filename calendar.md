# 달력 및 날짜 선택

[← 목차로 돌아가기](README.md)

---

## 개요

MCal 클래스는 달력 UI와 날짜 선택 기능을 제공합니다. 년/월/일/시/분 선택 UI를 쉽게 생성할 수 있습니다.

### 주요 기능

- **Select Box 생성**: 년/월/일/시/분 선택 옵션
- **달력 데이터**: 월별 달력 데이터 생성
- **주차 계산**: 요일 번호, 주의 첫날/마지막날
- **유연한 범위**: 년도 범위 설정 가능

---

## 목차

1. [년/월/일 선택](#년월일-선택)
2. [시/분 선택](#시분-선택)
3. [달력 데이터 생성](#달력-데이터-생성)
4. [날짜 범위 선택](#날짜-범위-선택)
5. [전체 예제](#전체-예제)

---

## 년/월/일 선택

### 1. 년도 선택 (Select Box)

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

MCal cal = new MCal();

// 현재 년도 기준 앞뒤 5년 (기본값)
DataSet years = cal.getYears();

p.setBody("main.date_form");
p.setLoop("years", years);
p.display();

%>
```

**html/main/date_form.html**:
```html
<select name="year">
    <!--@loop(years)-->
    <option value="{{years.id}}">{{years.name}}년</option>
    <!--/loop(years)-->
</select>
```

**결과**:
```html
<select name="year">
    <option value="2020">2020년</option>
    <option value="2021">2021년</option>
    <option value="2022">2022년</option>
    <option value="2023">2023년</option>
    <option value="2024">2024년</option>
    <option value="2025">2025년</option>
    <option value="2026">2026년</option>
    <option value="2027">2027년</option>
    <option value="2028">2028년</option>
    <option value="2029">2029년</option>
</select>
```

### 2. 년도 범위 설정

```jsp
// 앞뒤 10년
MCal cal = new MCal(10);
DataSet years = cal.getYears();

// 또는
MCal cal = new MCal();
cal.setYearRange(10);
DataSet years = cal.getYears();

// 특정 년도 기준
DataSet years = cal.getYears(2020);  // 2020 기준 앞뒤 5년
DataSet years = cal.getYears("2020");
```

### 3. 월 선택

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

MCal cal = new MCal();
DataSet months = cal.getMonths();

p.setBody("main.month_form");
p.setLoop("months", months);
p.display();

%>
```

**html/main/month_form.html**:
```html
<select name="month">
    <!--@loop(months)-->
    <option value="{{months.id}}">{{months.name}}월</option>
    <!--/loop(months)-->
</select>
```

**DataSet 구조**:
- `id`: "01", "02", ... "12" (2자리 문자열)
- `name`: "01", "02", ... "12" (2자리 문자열)
- `name2`: 1, 2, ... 12 (정수)

### 4. 일 선택

```jsp
MCal cal = new MCal();
DataSet days = cal.getDays();

// 1일 ~ 31일까지
```

**DataSet 구조**:
- `id`: "01", "02", ... "31"
- `name`: "01", "02", ... "31"
- `name2`: 1, 2, ... 31

### 5. 전체 날짜 선택 폼

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

MCal cal = new MCal();

DataSet years = cal.getYears();
DataSet months = cal.getMonths();
DataSet days = cal.getDays();

p.setBody("main.full_date_form");
p.setLoop("years", years);
p.setLoop("months", months);
p.setLoop("days", days);
p.display();

%>
```

**html/main/full_date_form.html**:
```html
<form>
    <select name="year">
        <!--@loop(years)-->
        <option value="{{years.id}}">{{years.name}}년</option>
        <!--/loop(years)-->
    </select>

    <select name="month">
        <!--@loop(months)-->
        <option value="{{months.id}}">{{months.name2}}월</option>
        <!--/loop(months)-->
    </select>

    <select name="day">
        <!--@loop(days)-->
        <option value="{{days.id}}">{{days.name2}}일</option>
        <!--/loop(days)-->
    </select>
</form>
```

---

## 시/분 선택

### 1. 시간 선택 (0~23시)

```jsp
MCal cal = new MCal();
DataSet hours = cal.getHours();

// 00, 01, 02, ... 23
```

**DataSet 구조**:
- `id`: "00", "01", ... "23"
- `name`: "00", "01", ... "23"
- `name2`: 0, 1, ... 23

### 2. 분 선택

```jsp
MCal cal = new MCal();

// 1분 간격 (00 ~ 59)
DataSet minutes = cal.getMinutes();

// 5분 간격 (00, 05, 10, ... 55)
DataSet minutes5 = cal.getMinutes(5);

// 10분 간격 (00, 10, 20, ... 50)
DataSet minutes10 = cal.getMinutes(10);

// 15분 간격 (00, 15, 30, 45)
DataSet minutes15 = cal.getMinutes(15);
```

### 3. 시간/분 선택 폼

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

MCal cal = new MCal();
DataSet hours = cal.getHours();
DataSet minutes = cal.getMinutes(10);  // 10분 간격

p.setBody("main.time_form");
p.setLoop("hours", hours);
p.setLoop("minutes", minutes);
p.display();

%>
```

**html/main/time_form.html**:
```html
<form>
    <select name="hour">
        <!--@loop(hours)-->
        <option value="{{hours.id}}">{{hours.name2}}시</option>
        <!--/loop(hours)-->
    </select>
    :
    <select name="minute">
        <!--@loop(minutes)-->
        <option value="{{minutes.id}}">{{minutes.name2}}분</option>
        <!--/loop(minutes)-->
    </select>
</form>
```

---

## 달력 데이터 생성

### 1. 월별 달력 데이터

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

MCal cal = new MCal();

// 현재 월 달력 데이터
String currentDate = m.time("yyyy-MM-dd");
DataSet calendar = cal.getMonthDays(currentDate);

// 특정 월 달력 데이터
DataSet calendar2 = cal.getMonthDays("2025-06-01");

p.setBody("main.calendar");
p.setLoop("days", calendar);
p.display();

%>
```

**DataSet 구조**:
- `date`: 날짜 (yyyy-MM-dd 형식)
- `type`: "1" (이전달), "2" (현재달), "3" (다음달)
- `weeknum`: 요일 번호 (1=일요일, 2=월요일, ..., 7=토요일)
- `__last`: 마지막 행 여부 (Boolean)

### 2. 달력 HTML 생성

**html/main/calendar.html**:
```html
<style>
    .calendar { width: 100%; border-collapse: collapse; }
    .calendar th, .calendar td { padding: 10px; text-align: center; border: 1px solid #ddd; }
    .calendar th { background: #007bff; color: white; }
    .calendar .prev-month, .calendar .next-month { color: #ccc; }
    .calendar .sunday { color: red; }
    .calendar .saturday { color: blue; }
    .calendar .today { background: #fffacd; font-weight: bold; }
</style>

<table class="calendar">
    <thead>
        <tr>
            <th>일</th>
            <th>월</th>
            <th>화</th>
            <th>수</th>
            <th>목</th>
            <th>금</th>
            <th>토</th>
        </tr>
    </thead>
    <tbody>
        <!--@loop(days)-->
        <!--@if(days.weeknum:1)--><tr><!--/if(days.weeknum:1)-->

        <td class="
            <!--@if(days.type:1)-->prev-month<!--/if(days.type:1)-->
            <!--@if(days.type:3)-->next-month<!--/if(days.type:3)-->
            <!--@if(days.weeknum:1)-->sunday<!--/if(days.weeknum:1)-->
            <!--@if(days.weeknum:7)-->saturday<!--/if(days.weeknum:7)-->
            <!--@if(days.date:{{today}})-->today<!--/if(days.date:{{today}})-->
        ">
            {{days.date}}
        </td>

        <!--@if(days.__last)--></tr><!--/if(days.__last)-->
        <!--/loop(days)-->
    </tbody>
</table>
```

### 3. 일정이 있는 달력

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

String month = m.rs("month", m.time("yyyy-MM"));

MCal cal = new MCal();
DataSet calendar = cal.getMonthDays(month + "-01");

// 해당 월의 일정 조회
EventDao event = new EventDao();
DataSet list = event.find("event_date >= ? AND event_date < ?",
    new Object[] { month + "-01", Malgn.addMonth(1, month + "-01") });

// 날짜별로 일정 카운트
Hashtable<String, Integer> eventCounts = new Hashtable<>();
while(list.next()) {
    String date = list.s("event_date");
    eventCounts.put(date, eventCounts.getOrDefault(date, 0) + 1);
}

// 달력 데이터에 일정 카운트 추가
while(calendar.next()) {
    String date = calendar.s("date");
    int count = eventCounts.getOrDefault(date, 0);
    calendar.put("event_count", count);
}

p.setBody("main.event_calendar");
p.setLoop("days", calendar);
p.setVar("month", month);
p.setVar("prev_month", Malgn.addMonth(-1, month + "-01").substring(0, 7));
p.setVar("next_month", Malgn.addMonth(1, month + "-01").substring(0, 7));
p.display();

%>
```

**html/main/event_calendar.html**:
```html
<div style="text-align: center; margin-bottom: 20px;">
    <a href="?month={{prev_month}}">&laquo; 이전</a>
    <strong>{{month}}</strong>
    <a href="?month={{next_month}}">다음 &raquo;</a>
</div>

<table class="calendar">
    <!-- ... 달력 헤더 ... -->
    <tbody>
        <!--@loop(days)-->
        <!--@if(days.weeknum:1)--><tr><!--/if(days.weeknum:1)-->

        <td class="<!--@if(days.type:1)-->prev-month<!--/if(days.type:1)--> <!--@if(days.type:3)-->next-month<!--/if(days.type:3)-->">
            <div>{{days.date}}</div>
            <!--@if(days.event_count)-->
            <div style="color: red; font-size: 11px;">{{days.event_count}}건</div>
            <!--/if(days.event_count)-->
        </td>

        <!--@if(days.__last)--></tr><!--/if(days.__last)-->
        <!--/loop(days)-->
    </tbody>
</table>
```

---

## 날짜 범위 선택

### 1. 시작일/종료일 선택

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

MCal cal = new MCal();

DataSet years = cal.getYears();
DataSet months = cal.getMonths();
DataSet days = cal.getDays();

p.setBody("main.date_range_form");
p.setLoop("years", years);
p.setLoop("months", months);
p.setLoop("days", days);
p.display();

%>
```

**html/main/date_range_form.html**:
```html
<form method="get">
    <div>
        <label>시작일</label>
        <select name="start_year">
            <!--@loop(years)-->
            <option value="{{years.id}}">{{years.name}}년</option>
            <!--/loop(years)-->
        </select>
        <select name="start_month">
            <!--@loop(months)-->
            <option value="{{months.id}}">{{months.name2}}월</option>
            <!--/loop(months)-->
        </select>
        <select name="start_day">
            <!--@loop(days)-->
            <option value="{{days.id}}">{{days.name2}}일</option>
            <!--/loop(days)-->
        </select>
    </div>

    <div>
        <label>종료일</label>
        <select name="end_year">
            <!--@loop(years)-->
            <option value="{{years.id}}">{{years.name}}년</option>
            <!--/loop(years)-->
        </select>
        <select name="end_month">
            <!--@loop(months)-->
            <option value="{{months.id}}">{{months.name2}}월</option>
            <!--/loop(months)-->
        </select>
        <select name="end_day">
            <!--@loop(days)-->
            <option value="{{days.id}}">{{days.name2}}일</option>
            <!--/loop(days)-->
        </select>
    </div>

    <button type="submit">검색</button>
</form>
```

### 2. 날짜 범위로 데이터 조회

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

if(m.isPost()) {

    String startYear = f.get("start_year");
    String startMonth = f.get("start_month");
    String startDay = f.get("start_day");

    String endYear = f.get("end_year");
    String endMonth = f.get("end_month");
    String endDay = f.get("end_day");

    String startDate = startYear + "-" + startMonth + "-" + startDay;
    String endDate = endYear + "-" + endMonth + "-" + endDay;

    // 날짜 검증
    if(Malgn.strToDate("yyyy-MM-dd", startDate).after(Malgn.strToDate("yyyy-MM-dd", endDate))) {
        m.jsError("시작일이 종료일보다 늦을 수 없습니다.");
        return;
    }

    // 데이터 조회
    OrderDao order = new OrderDao();
    DataSet orderList = order.find("order_date >= ? AND order_date <= ?",
        new Object[] { startDate, endDate });

    p.setBody("main.order_list");
    p.setLoop("orders", orderList);
    p.setVar("start_date", startDate);
    p.setVar("end_date", endDate);
    p.display();
    return;
}

// 폼 표시
MCal cal = new MCal();
DataSet years = cal.getYears();
DataSet months = cal.getMonths();
DataSet days = cal.getDays();

p.setBody("main.date_range_form");
p.setLoop("years", years);
p.setLoop("months", months);
p.setLoop("days", days);
p.display();

%>
```

---

## 전체 예제

### 예약 시스템

**reservation_form.jsp**:
```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

f.addElement("name", null, "required:'Y'");
f.addElement("phone", null, "required:'Y', type:'phone'");
f.addElement("year", null, "required:'Y'");
f.addElement("month", null, "required:'Y'");
f.addElement("day", null, "required:'Y'");
f.addElement("hour", null, "required:'Y'");
f.addElement("minute", null, "required:'Y'");

if(m.isPost() && f.validate()) {

    String name = f.get("name");
    String phone = f.get("phone");

    String year = f.get("year");
    String month = f.get("month");
    String day = f.get("day");
    String hour = f.get("hour");
    String minute = f.get("minute");

    String reservationDate = year + "-" + month + "-" + day;
    String reservationTime = hour + ":" + minute;
    String reservationDatetime = reservationDate + " " + reservationTime + ":00";

    // 과거 날짜 체크
    if(Malgn.strToDate("yyyy-MM-dd HH:mm:ss", reservationDatetime)
        .before(new Date())) {
        m.jsError("과거 날짜는 예약할 수 없습니다.");
        return;
    }

    // 중복 예약 체크
    ReservationDao reservation = new ReservationDao();
    DataSet info = reservation.find("reservation_datetime = ?", new Object[]{reservationDatetime});
    if(info.next()) {
        m.jsError("해당 시간은 이미 예약되어 있습니다.");
        return;
    }

    // 예약 저장
    reservation.item("name", name);
    reservation.item("phone", phone);
    reservation.item("reservation_datetime", reservationDatetime);
    reservation.item("status", "pending");
    reservation.item("reg_date", m.time());
    int newId = reservation.insert();

    m.jsAlert("예약이 완료되었습니다.");
    m.jsReplace("reservation_view.jsp?id=" + newId);
    return;
}

// 폼 표시
MCal cal = new MCal();

// 오늘부터 1년치
cal.setYearRange(1);
DataSet years = cal.getYears();
DataSet months = cal.getMonths();
DataSet days = cal.getDays();

// 09:00 ~ 18:00, 30분 간격
DataSet hours = new DataSet();
for(int i = 9; i <= 18; i++) {
    hours.addRow();
    hours.put("id", (i < 10 ? "0" : "") + i);
    hours.put("name", i + "시");
}

DataSet minutes = new DataSet();
minutes.addRow();
minutes.put("id", "00");
minutes.put("name", "00분");
minutes.addRow();
minutes.put("id", "30");
minutes.put("name", "30분");

p.setBody("main.reservation_form");
p.setLoop("years", years);
p.setLoop("months", months);
p.setLoop("days", days);
p.setLoop("hours", hours);
p.setLoop("minutes", minutes);
p.setVar("form_script", f.getScript());
p.display();

%>
```

**html/main/reservation_form.html**:
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>예약하기</title>
    <style>
        .form-group { margin-bottom: 15px; }
        label { display: block; margin-bottom: 5px; font-weight: bold; }
        input, select { padding: 8px; border: 1px solid #ddd; border-radius: 4px; }
        button { padding: 10px 20px; background: #007bff; color: white; border: none; border-radius: 4px; cursor: pointer; }
    </style>
</head>
<body>
    <h1>예약하기</h1>
    <form name="form1" method="post">
        <div class="form-group">
            <label>이름</label>
            <input type="text" name="name">
        </div>

        <div class="form-group">
            <label>전화번호</label>
            <input type="text" name="phone">
        </div>

        <div class="form-group">
            <label>예약 날짜</label>
            <select name="year">
                <!--@loop(years)-->
                <option value="{{years.id}}">{{years.name}}년</option>
                <!--/loop(years)-->
            </select>
            <select name="month">
                <!--@loop(months)-->
                <option value="{{months.id}}">{{months.name2}}월</option>
                <!--/loop(months)-->
            </select>
            <select name="day">
                <!--@loop(days)-->
                <option value="{{days.id}}">{{days.name2}}일</option>
                <!--/loop(days)-->
            </select>
        </div>

        <div class="form-group">
            <label>예약 시간</label>
            <select name="hour">
                <!--@loop(hours)-->
                <option value="{{hours.id}}">{{hours.name}}</option>
                <!--/loop(hours)-->
            </select>
            <select name="minute">
                <!--@loop(minutes)-->
                <option value="{{minutes.id}}">{{minutes.name}}</option>
                <!--/loop(minutes)-->
            </select>
        </div>

        <button type="submit">예약하기</button>
    </form>
    {{form_script}}
</body>
</html>
```

---

## 요일 및 주차 계산

### 1. 요일 번호 구하기

```jsp
MCal cal = new MCal();

// 요일 번호 (1=일요일, 2=월요일, ..., 7=토요일)
int weekNum = cal.getWeekNum("20250601");
int weekNum2 = cal.getWeekNum("2025-06-01", "yyyy-MM-dd");

m.p("요일 번호: " + weekNum);
```

### 2. 주의 첫날/마지막날 구하기

```jsp
MCal cal = new MCal();

// 해당 주의 첫날 (일요일)
Date weekFirst = cal.getWeekFirstDate("20250601");
m.p("주의 첫날: " + Malgn.getTimeString("yyyy-MM-dd", weekFirst));

// 해당 주의 마지막날 (토요일)
Date weekLast = cal.getWeekLastDate("20250601");
m.p("주의 마지막날: " + Malgn.getTimeString("yyyy-MM-dd", weekLast));

// 월의 마지막날
Date monthLast = cal.getMonthLastDate("20250601");
m.p("월의 마지막날: " + Malgn.getTimeString("yyyy-MM-dd", monthLast));
```

---

## 베스트 프랙티스

### 1. 현재 날짜 선택 상태로 표시

```html
<select name="year">
    <!--@loop(years)-->
    <option value="{{years.id}}" <!--@if(years.id:{{current_year}})-->selected<!--/if(years.id:{{current_year}})-->>{{years.name}}년</option>
    <!--/loop(years)-->
</select>
```

```jsp
p.setVar("current_year", m.time("yyyy"));
p.setVar("current_month", m.time("MM"));
p.setVar("current_day", m.time("dd"));
```

### 2. JavaScript로 동적 처리

```html
<script>
// 월에 따라 일자 선택 동적 변경
document.querySelector('select[name="month"]').addEventListener('change', function() {
    var year = document.querySelector('select[name="year"]').value;
    var month = this.value;
    var daySelect = document.querySelector('select[name="day"]');

    // 해당 월의 마지막 날 계산
    var lastDay = new Date(year, month, 0).getDate();

    // 옵션 재생성
    daySelect.innerHTML = '';
    for(var i = 1; i <= lastDay; i++) {
        var option = document.createElement('option');
        option.value = (i < 10 ? '0' : '') + i;
        option.text = i + '일';
        daySelect.appendChild(option);
    }
});
</script>
```

### 3. 오늘 날짜 하이라이트

```jsp
p.setVar("today", m.time("yyyy-MM-dd"));
```

```html
<td class="<!--@if(days.date:{{today}})-->today<!--/if(days.date:{{today}})-->">
    {{days.date}}
</td>
```

---

## 참고 문서

- [템플릿](template.md)
- [DataSet 활용](dataset.md)
- [유틸리티 메소드 - 날짜/시간](utility-methods.md#날짜시간-처리)

---

[← 목차로 돌아가기](README.md)
