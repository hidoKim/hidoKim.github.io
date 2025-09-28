---
layout: single
title: "[MySQL] SQL 기본 문법: SELECT, WHERE, ORDER BY 등"
categories: MySQL_study
tags: [MySQL_study, study]
toc: true
author_profile: false
---

# SQL 기본 문법: SELECT, WHERE, ORDER BY 등

[SQL-기본-문법-정리-SELECT-절](https://rachel0115.tistory.com/entry/SQL-%EA%B8%B0%EB%B3%B8-%EB%AC%B8%EB%B2%95-%EC%A0%95%EB%A6%AC-SELECT-%EC%A0%88)

위 블로그를 보고 작성한 글입니다.

# 1. SELECT 문의 기본 형식

```jsx
SELECT 열_이름
		FROM 테이블_이름
		WHERE 조건식
		GROUP BY 열_이름
		HAVING 조건식
		ORDER BY 열_이름
		LIMIT 숫자
```

# 2. SQL 기본 문법

## USE - 스키마(DB) 선택

```jsx
USE user_db;
```

- SQL을 사용할 스키마(DB)를 지정

## SELECT - 조회할 데이터(컬럼) 지정

```jsx
-- SELECT '컬럼명' FROM '테이블명'
SELECT user_id, name FROM user;

-- SELECT 와 FROM 사이에 * 을 적으면 테이블의 모든 컬럼을 조회한다.
SELECT * FROM user;

-- 아래 두 SQL은 동일한 기능을 한다.
SELECT * FROM user_db.user;
SELECT * FROM user;
```

- 기본적으로 테이블이름은 `스키마명.테이블명`으로 표현한다.
- 하지만 `USE`문으로 스키마지정을 했다면 테이블명만 명시해도 된다.

## WHERE - 특정 조건만 조회하기

```jsx
-- user 테이블에서 user_num 컬럼 값이 5 이상인 데이터 조회
SELECT * FROM user
	WHERE user_num >= 5;
```

- `WHERE`절을 사용해 특정 **조건**에 해당하는 데이터만 조회할 수 있다.
- 관계연산자/논리연산자 사용가능

## 논리연산자 AND, OR 사용가능

```jsx
SELECT TRUE OR FALSE AND FALSE; -- 1

SELECT (TRUE OR FALSE) AND FALSE; -- 0
```

- 여러 조건이 필요한 경우 논리 연산자를 사용한다.
- `AND`가 `OR`보다 우선순위를 가진다.
- MySQL에서는 `&&`이나 `||`도 사용 가능하다.

## BETWEEN - 범위표현식

```jsx
-- user 테이블에서 height 컬럼 값이 160이상 165이하인 데이터 조회
SELECT * FROM user
		WHERE height between 160 and 165;
```

- `between` 연산자를 이용하여 특정 범위에 해당하는 데이터를 조회할 수 있다.
- 하지만 인덱스를 사용할 수 없으므로 주의

## IN() - 여러 값 매칭

```jsx
-- addr 컬럼값이 경기, 전남, 경남인 데이터 조회
SELECT * FROM user
		WHERE addr IN('경기', '전남', '경남');

-- IN()을 사용하지 않으면 아래처럼 조회.
SELECT * FROM user
		WHERE addr = '경기' OR addr = '전남' OR addr = '경남';
```

- `IN()` 연산자를 이용하여 특정 값이 포함된 데이터를 조회할 수 있다.
- `IN()` 연산자는 동등비교 ‘=’를 여러번 수행하는 효과를 가진다. 따라서 인덱스를 최적으로 활용할 수 있다.

## LIKE - 문자열의 일부 글자 검색

```jsx
-- user_name 컬럼 값이 '블'로 시작하는 4글자 데이터 조회
SELECT * FROM user WHERE user_name LIKE '블___';

-- user_name 컬럼 값이 '블'로 시작하는 모든 데이터 조회
SELECT * FROM user WHERE user_name LIKE '블%';

-- user_name 컬럼 값에 '블'이 들어가는 모든 데이터 조회
SELECT * FROM user WHERE user_name LIKE '%블%';
```

- 문자열의 일부 글자 검색
  - `_`: 한글자만 매치
  - `%`: 몇글자든 매치

## ORDER BY - 조회된 데이터를 정렬

```jsx
-- debut_date 값을 기준으로 정렬 (기본 ASC)
SELECT * FROM user
		ORDER BY debut_date
```

- `ASC(ascending order)`: 오름차순 - 생략시 기본값
- `DESC(descending order)`: 내림차순
- `ORDER BY`절은 데이터를 정렬한다.
- `WHERE`절 다음에 나와야 한다.
- `,`로 여러 정렬 조건 지정 가능

```jsx
-- height 컬럼 값이 164 이상인 데이터 조회
-- height 값 기준 내림차순 정렬, 동일 값이면 debut_data 기준 오름차순
SELECT * FROM USER
		WHRER height >= 164
		ORDER BY height DESC, debut_data ASC;
```

## LIMIT - 출력 개수 제한

```jsx
SELECT * FROM user
		LIMIT 3; -- 상위 3건만 조회

SELECT * FROM user
		LIMIT 3,2; -- 3번째 데이터부터 2건만 조회
		LIMIT 2 OFFSET 3; -- 3번째 데이터부터 2건만 조회
```

- `LIMIT` 시작, 개수
- `LIMIT` 뒤에 하나의 숫자만 입력시 처음부터 N까지
- `LIMIT`와 `OFFSET`조합으로도 출력 개수 제한가능

## DISTINCT - 중복 데이터 제거

```jsx
-- addr의 모든 컬럼 값을 중복을 제거하여 조회
SELECT DISTINCT addr
		FROM user;
```

- `DISTINCT`를 열 이름 앞에 붙이면 중복된 값은 1개만 출력됨.

## GROUP BY - 그룹화

```jsx
-- user_id가 같은 데이터를 그룹으로 묶음
-- 그룹핑된 데이터에서 user_id와 amount의 합계를 구함
SELECT user_id, SUM(amount) AS "합계"
		FROM buy
		GROUP BY user_id
		ORDER BY user_id;
```

- 컬럼이 같은 데이터를 그룹화 해주는 기능
- 보통 집계 함수와 같이 쓰임

```jsx
SELECT genre, AVG(price) AS "평균"
		FROM library
		GROUP BY genre;
```

- `GROUP BY`로 지정한 컬럼이 같은 데이터끼리 그룹화
- 여러 컬럼 지정 가능

## 집계함수

- `SUM()`: 컬럼의 합계 반환
- `AVG()`: 컬럼의 평균 반환
- `MIN()`: 컬럼의 최소값 반환
- `MAX()`: 컬럼의 최대값 반환
- `COUNT()`: 행의 개수를 셈
- `COUNT(DISTINCT)`: 고유 행의 개수를 셈 (중복, null 제외)

```jsx
-- 집계함수 안에서 연산도 가능
SELECT user_id, SUM(amount*price) AS "총 금액"
		FROM buy
		GROUP BY user_id
		ORDER BY user_id;
```

## COUNT()

```jsx
-- user 테이블의 모든 데이터 개수를 셈
SELECT COUNT(*)
		FROM user;

-- user 테이블의 phone_num 컬럼이 NULL인 것을 제외한 모든 데이터 개수를 셈
SELECT COUNT(phone_num)
		FROM user;
```

- `COUNT(\*)` 연산은 모든 행을 대상으로 이루어지기 때문에 `NULL` 값이 포함되어있어도 카운트됨
- `COUNT(phone_num)` 연산은 `phone_num`값에 `NULL`이 있을 경우 카운트하지 않음

## HAVING - 그룹 조건

```jsx
-- user_id를 기준으로 그룹화
-- 그룹화 된 데이터를 기준으로 amount * price 합계가 1000이상인 그룹만 남김
-- 조건에 걸러진 그룹에서 amount * price 합계를 조회
SELECT SUM(amount*price) AS "총 금액"
		FROM buy
		GROUP BY user_id
		HAVING SUM(amount*price) >= 1000;
```

- 그룹화된 데이터에 대해 조건을 제한
- 때문에 `GROUP BY` 뒤에 와야함
