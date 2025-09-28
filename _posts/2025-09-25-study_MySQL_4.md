---
layout: single
title: "[MySQL] SQL 기본문법 : UNION, EXCLUSIVE, Distinct"
categories: MySQL_study
tags: [MySQL_study, study]
toc: true
author_profile: false
---

# SQL 기본문법 : UNION, EXCLUSIVE, Distinct

[sql-기본-문법](https://hongong.hanbit.co.kr/sql-%EA%B8%B0%EB%B3%B8-%EB%AC%B8%EB%B2%95-joininner-outer-cross-self-join/)

[JOIN](https://velog.io/@wijoonwu/JOIN)

[MYSQL-📚-JOIN-조인-그림으로-알기쉽게-정리](https://inpa.tistory.com/entry/MYSQL-%F0%9F%93%9A-JOIN-%EC%A1%B0%EC%9D%B8-%EA%B7%B8%EB%A6%BC%EC%9C%BC%EB%A1%9C-%EC%95%8C%EA%B8%B0%EC%89%BD%EA%B2%8C-%EC%A0%95%EB%A6%AC)

위 블로그를 보고 작성한 글입니다.

## UNION

- 여러개의 SELECT 문의 결과를 하나의 테이블이나 결과 집합으로 표현할 때 사용
- 각각의 SELECT문으로 선택된 필드의 개수와 타입은 모두 같아야하며, 필드의 순서도 같아야한다.
- 기본 집합 쿼리에는 (DISTINCT) 중복제거가 자동 포함되어 있다.

```sql
SELECT 필드이름 FROM 테이블이름
UNION
SELECT 필드이름 FROM 테이블이름
```

### UNION ALL

- 중복되는 레코드까지 모두 출력하고 싶을때 사용

```sql
SELECT 필드이름 FROM 테이블이름
UNION ALL
SELECT 필드이름 FROM 테이블이름
```

## EXCLUSIVE LEFT JOIN

- 어느 특정 테이블에 있는 레코드만 가져오는 것
- 만약 테이블 두개를 JOIN 한다면 둘중 하나의 테이블에만 있는 데이터를 가져온다.
- 별도의 EXCLUSIVE JOIN 함수가 있는 건 아니고 기존 LEFT JOIN과 Where절의 조건을 함께 사용하여 만든다.

```sql
SELECT *
FROM tableA A LEFT JOIN tableB B
ON A.ID_SEQ = B.ID_SEQ
WHERE B.ID_SEQ IS NULL --- 조인한 tableB의 값으로 null만 출력해라.
```

## Distinct 사용

- mysql에서 지원하는 distinct 문법을 사용
- 중복 되는 데이터가 있을 때 하나만 카운트 하도록 함
- 매우 쉬운 방법, 하지만 레코드 수가 많은 경우 성능이 느림

```sql
SELECT DISTINCT person.id, person.name, job.job_name
FROM person INNER JOIN job
ON person.name = job.person_name;
```

### JOIN 중복제거

성능을 위해서는 JOIN 전에 중복을 제거하는 작업을 해줘야 함.

- JOIN 전에 중복 제거는 조인할 테이블에 서브쿼리의 inline view를 사용해 distinct하고 조인하면 된다.

```sql
SELECT A.name, A.countryCode
FROM city A
LEFT JOIN ( select distinct name, Code form country ) as B
-- 조인할 테이블에 먼저 distinct로 중복제거 후 select 문을 서브쿼리로 불러와
-- 임시 테이블로 만든뒤 조인
on A.countryCode = B.Code
```
