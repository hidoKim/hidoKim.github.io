---
layout: single
title: "[MySQL] SQL 기본문법 : JOIN"
categories: MySQL_study
tags: [MySQL_study, study]
toc: true
author_profile: false
---

# SQL 기본문법 : JOIN

[sql-기본-문법](https://hongong.hanbit.co.kr/sql-%EA%B8%B0%EB%B3%B8-%EB%AC%B8%EB%B2%95-joininner-outer-cross-self-join/)

[JOIN](https://velog.io/@wijoonwu/JOIN)

[MYSQL-📚-JOIN-조인-그림으로-알기쉽게-정리](https://inpa.tistory.com/entry/MYSQL-%F0%9F%93%9A-JOIN-%EC%A1%B0%EC%9D%B8-%EA%B7%B8%EB%A6%BC%EC%9C%BC%EB%A1%9C-%EC%95%8C%EA%B8%B0%EC%89%BD%EA%B2%8C-%EC%A0%95%EB%A6%AC)

위 블로그를 보고 작성한 글입니다.

첨부한 이미지는 PPT로 직접 만들거나 Gemini를 이용해 만든 이미지 입니다.

# JOIN 이란?

- 두개의 테이블은 서로 묶어서 하나의 결과를 만들어 내는 것
- JOIN을 이용하면 두개의 테이블을 엮어서 원하는 데이터를 추출할 수 있음
- 두 테이블의 조인을 위해서는 기본키와 외래키 관계로 맺어져야함
  → 일대다 관계

### 사용 예시

인스타그램 댓글창처럼 유저의 아이디와 댓글의 내용을 동시에 보여줄때

- 서로다른 각각의 테이블(유저 아이디, 유저 댓글) 속 데이터를 동시에 보여주려 할때 사용하는 SQL

→ INNER JOIN으로 가능

## JOIN의 종류

### INNER JOIN(내부 조인)

- 두 테이블을 조인할 때, 두 테이블에 모두 지정한 열의 데이터가 있어야 함.
- 그냥 join 이라고 부르면 내부 조인임.
- “,” 로 두 테이블은 이어주면 join임.

- 조인하는 테이블의 ON 절의 조건이 일치하는 결과만 출력
- 표준 SQL과는 달리 MySQL에서는 `JOIN, INNER JOIN, CROSS JOIN`이 모두 같은 의미로 사용된다.
- inner join은 어느 테이블에서 기준으로 조인하든 조인 관계에 부합되는 레코드를 모두 가지게 된다. 그리고 조인에 부합되지 않는 레코드는 모두 삭제된다.

### OUTER JOIN(외부 조인)

- 두 테이블을 조인할 때, 1개의 테이블에만 데이터가 있어도 결과가 나온다.

### CROSS JOIN(상호 조인)

- 한쪽 테이블의 모든행과 다른쪽 테이블의 모든 행을 조인하는 기능
- 카티션 곱(CARTESIAN PRODUCT)라고도 함

### SELF JOIN(자체 조인)

- 자신이 자신과 조인, 1개의 테이블을 사용함.
- 별도의 문법이 있는 것은 아니고 1개로 조인하면 자체 조인이 됨.
- 예: 모든 사원에 대해 사원의 이름과 직속 상사의 이름을 검색
  - EMPNAME테이블에서 MANAGER의 번호와 EMPNO의 번호가 같으면 직속 상관 관계

## INNER JOIN(내부조인)

```sql
SELECT <열 목록>
FROM <첫 번째 테이블>
		INNER JOIN <두 번째 테이블>
		ON <조인 조건>
[WHERE 검색 조건]
```

```sql
SELECT <열 목록>
FROM <첫 번째 테이블>
		JOIN <두 번째 테이블>
		ON <조인 조건>
[WHERE 검색 조건]
```

```sql
SELECT <열 목록>
FROM <첫 번째 테이블>, <두 번째 테이블>
		ON <조인 조건>
[WHERE 검색 조건]
```

<img src="/assets/images/innerJoin.png" style="width:400px; height:400px;" />

| users |
| ----- | -------- | ------------------- |
| id    | nickname | email               |
| 1     | mingming | ming01@naver.com    |
| 2     | kim      | kim02@gmail.com     |
| 3     | cutelee  | leesong03@naver.com |

| comments |
| -------- | ---- | ------- | -------- |
| id       | body | user_id | photo_id |
| 1        | neow | 1       | 1        |
| 2        | 냐옹 | 2       | 1        |
| 3        | 멍   | 3       | 2        |

위와 같은 테이블이 있을때의 `INNER JOIN`

```sql
SELECT
  comments.body,
  users.nickname
FROM
  comments
JOIN users ON
  users.id = comments.user_id
WHERE
  comments.photo_id = 1
;
```

## OUTER JOIN(외부조인)

```sql
SELECT <열 목록>
FROM <첫 번째 테이블(LEFT 테이블)>
		<LEFT | RIGHT | FULL> OUTER JOIN <두 번째 테이블(RIGHT 테이블)>
		ON <조인 조건>
[WHERE 검색 조건]
```

### OUTER JOIN의 종류

- LEFT OUTER JOIN : 왼쪽 테이블의 모든 값이 출력되는 조인 (가장 많이 사용됨)
  - 두 테이블이 있을 경우 첫번째 테이블을 기준으로 두번째 테이블을 조합하는 join
  - inner join과 다르게 조인하는 테이블의 순서가 중요함 (조인 순서에 따라 결과 테이블에 조회되는 행의 개수며 구성 등이 달라질 수 있음)
  - 따라서 가장 첫번째 테이블로 SELECT문에 가장 많은 열을 가져와야하는 테이블을 우선으로 적는다.
  - 조인을 여러번할때, LEFT JOIN으로 시작했으면 나머지 조인도 LEFT JOIN을 이어나간다. (갑자기 INNER JOIN등 다른 join을 쓰지 않는다.)
- RIGHT OUTER JOIN : 오른쪽 테이블의 모든 값이 출력되는 조인
- FULL OUTER JOIN : 왼쪽 외부 조인과 오른쪽 외부 조인이 합쳐진 것 (성능상 거의 사용하지 않음)

### LEFT OUTER JOIN

<img src="/assets/images/leftOuterJoin.png" style="width:400px; height:400px;" />
```sql
SELECT * FROM TableA a LEFT OUTER JOIN TableB b ON a.KEY = b.KEY
```

```sql
SELECT * FROM TableA a LEFT OUTER JOIN TableB b ON a.KEY = b.KEY WHERE b.KEY IS NULL
```

### FULL OUTER JOIN

<img src="/assets/images/fullOuterJoin.png" style="width:400px; height:400px;" />

```sql
SELECT * FROM TableA a FULL OUTER JOIN TableB b ON a.KEY = b.KEY
```

```sql
SELECT * FROM TableA a FULL OUTER JOIN TableB b ON a.KEY = b.KEY WHERE a.KEY IS NULL OR b.KEY IS NULL
```

### RIGHT OUTER JOIN

<img src="/assets/images/rightOuterJoin.png" style="width:400px; height:400px;" />

```sql
SELECT * FROM TableA a RIGHT OUTER JOIN TableB b ON a.KEY = b.KEY
```

```sql
SELECT * FROM TableA a RIGHT OUTER JOIN TableB b ON a.KEY = b.KEY WHERE a.KEY IS NULL
```

## CROSS JOIN(상호조인)

<img src="/assets/images/crossJoin.png" style="width:400px;" />

```sql
SELECT *
FROM <첫 번째 테이블>
		CROSS JOIN <두 번째 테이블>
```

## SELF JOIN(자체조인)

<img src="/assets/images/selfJoin.png" style="width:400px; height:400px;" />

```sql
SELECT <열 목록>
FROM <테이블> 별칭A
		INNER JOIN <테이블> 별칭B
[WHERE 검색 조건]
```

---
