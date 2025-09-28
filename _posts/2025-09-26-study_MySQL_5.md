---
layout: single
title: "[MySQL] SQL 기본문법 : 서브쿼리"
categories: MySQL_study
tags: [MySQL_study, study]
toc: true
author_profile: false
---

# SQL 기본문법 : 서브쿼리

https://snowple.tistory.com/360

[https://inpa.tistory.com/entry/MYSQL-📚-서브쿼리-정리](https://inpa.tistory.com/entry/MYSQL-%F0%9F%93%9A-%EC%84%9C%EB%B8%8C%EC%BF%BC%EB%A6%AC-%EC%A0%95%EB%A6%AC)

위 블로그를 보고 작성한 글입니다.

# [서브쿼리란? (Subquery)]

하나의 SQL문 안에 포함되어있는 또 다른 SQL문

> 다른 쿼리 내부에 포함되어 있는 SELECT 문

- 서브쿼리는 메인쿼리가 서브쿼리를 포함하는 종속적인 관계
- Join은 조인에 참여하는 모든 테이블이 대등한 관계에 있기 때문에 조인에 참여하는 모든 테이블의 컬럼을 어느 위치에서라도 자유롭게 사용할 수 있다.
- 서브쿼리는 메인쿼리의 컬럼을 모두 사용할 수 있지만, 메인쿼리는 서브쿼리의 칼럼을 사용할 수 없다.
- 서브쿼리는 서브쿼리 레벨과는 상관없이 항상 메인쿼리 레벨로 결과 집합이 생성된다.
- 서브쿼리를 포함하고 있는 쿼리를 외부쿼리(outer query)라고 부르며, 서브쿼리는 내부쿼리(inner query)라고 부른다.
- java 객체지향의 상속과 똑같은 개념.
- 상속당한 자식 객체는 부모 객체의 인스턴스를 사용할 수 있고, 부모는 자식객체의 인스턴스를 사용할 수 없다.

## 서브쿼리 장점

1. 서브쿼리는 쿼리를 구조화시키므로, 쿼리의 각 부분을 명확히 구분할 수 있게 해준다.
2. 서브쿼리는 복잡한 JOIN이나 UNION과 같은 동작을 수행할 수 있는 또 다른 방법을 제공한다.
3. 서브쿼리는 가독성을 높혀준다.

## 서브쿼리의 위치에 따른 명칭

```sql
SELECT col1, (SELECT ...) -- 스칼라 서브쿼리
FROM (SELECT ...) -- 인라인 뷰
WHERE col = (SELECT ..) -- 일반 서브쿼리
```

### - 스칼라 서브쿼리

-> 하나의 컬럼처럼 사용 (표현 용도)

### - 인라인 뷰

-> 하나의 테이블처럼 사용 (테이블 대체 용도)

### - 일반 서브쿼리

-> 하나의 변수(상수)처럼 사용 (서브쿼리의 결과에 따라 달라지는 조건절)

## 서브쿼리 실행 순서

1. 서브쿼리 실행
2. 메인(부모)쿼리 실행

## 사용방법

- 주의할 점
  - 서브쿼리를 괄호로 감싸서 사용한다.
  - 서브쿼리는 단일 행 또는 복수 행 비교 연산자와 함께 사용 가능
  - 서부쿼리에서는 ORDER BY를 사용하지 못한다.
  - SELECT 문으로만 작성가능.
  - 괄호가 끝나고 끝에 ;(세미콜론)을 쓰지 않는다.
- 사용 가능한 곳
  - SELECT
  - FROM
  - WHERE
  - HAVING
  - ORDER BY
  - INSERT 문의 VALUES
  - UPDATE 문의 SET

```sql
SELECT *
FROM db_table
WHERE table_fk IN ( SELECT table_fk FROM db_table_other WHERE ... )
```

# [서브쿼리의 종류]

## 단일행 서브쿼리 (Single Row Subquery)

```sql
SELECT *
FROM Player
WHERE Team_ID = (
		SELECT Team_ID
		FROM Player
		WHERE Player_name = "yonglae"
		)
ORDER BY Team_name;
```

## 다중행 서브쿼리 (Multiple Row Subquery)

```sql
SELECT *
FROM Team
WHERE Team_ID IN (
		SELECT Team_ID
		FROM Player
		WHERE Player_name = "yonglae"
		)
ORDER BY Team_name;
```

## 다중컬럼 서브쿼리 (Multi Column Subquery)

```sql
SELECT *
FROM Player
WHERE ( Team_ID, Height) IN (
		SELECT Team_ID, MIN(Height)
		FROM Player
		GROUP BY Team_ID
		)
ORDER BY Team_ID, Player_name;
```

# [위치에 따라 사용되는 서브쿼리]

## SELECT 절에서 사용 (스칼라 서브쿼리)

- 다른 테이블에서 어떠한 값을 가져올때 쓰임
- 하나의 레코드만 리턴가능, 두개 이상의 레코드는 리턴할 수 없다.
- 일치하는 데이터가 없더라도 NULL 값을 리턴할 수 있다.

```sql
SELECT Player_name, Height, (
	SELECT AVG(Height)
	FROM Player p
	WHERE p.Team_ID = x.Team_ID as AVG_Height
	)
FROM Player x
```

## FROM 절에서 사용 (인라인 뷰)

- FROM 절에서 사용되는 서브쿼리를 인라인 뷰(inline View)라고 한다.
- FROM 절에는 테이블 명이 오도록 되어있다. 그런데 서브쿼리가 FROM절에 사용되면 뷰(View)처럼 결과가 동적으로 생성된 테이블로 사용할 수 있다. 임시적인 뷰이기 때문에 데이터베이스에 저장되지는 않는다.
- 인라인 뷰로 동적으로 생성된 테이블이여서 인라인 뷰의 칼럼은 자유롭게 참조가 가능하다.
- 서브쿼리가 FROM 절에 사용되는 경우 무조건 AS별칭을 지정해 주어야함.

```sql
SELECT t.Team_name, p.Player_name, p.Height
FROM Player t,(
		SELECT Team_ID, Player_name, Back_no
		FROM Player
		WHERE Position = 'MF'
		) p
WHERE p.Team_ID = t.Team_ID;
```

## WHERE 절에서 사용 (중첩 서브쿼리)

```sql
SELECT Player_name
FROM Player
WHERE Team_ID IN (
		SELECT Team_ID
		FROM Team
		WHERE Team_country = 'KOR'
		)
```
