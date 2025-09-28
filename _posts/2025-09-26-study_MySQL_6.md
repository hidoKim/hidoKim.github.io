---
layout: single
title: "[MySQL] SQL 기본문법 : 페이지네이션"
categories: MySQL_study
tags: [MySQL_study, study]
toc: true
author_profile: false
---

# SQL 기본문법 : 페이지네이션

[커서-기반-페이지네이션-Cursor-based-Pagination-구현하기](https://velog.io/@minsangk/%EC%BB%A4%EC%84%9C-%EA%B8%B0%EB%B0%98-%ED%8E%98%EC%9D%B4%EC%A7%80%EB%84%A4%EC%9D%B4%EC%85%98-Cursor-based-Pagination-%EA%B5%AC%ED%98%84%ED%95%98%EA%B8%B0)

위 블로그를 보고 작성한 글입니다.

# ✅ 페이지네이션이란?

- 서버에서 데이터를 가져올 때 모든 데이터를 한번에 가져올 수 없기 때문에 **특정한 정렬 기준에 따라 + 지정된 갯수**의 데이터를 가져오는 것이 필요하다. 이를 _페이지네이션_ 이라고 한다.

## 1. 오프셋 기반 페이지네이션

- DB의 offset쿼리를 사용하여 ‘페이지’ 단위로 구분하여 요청/응답하게 구현

## 2. 커서 기반 페이지네이션

- 클라이언트가 가져간 마지막 row(행)의 순서상 다음 row들을 n개 요청/응답하게 구현

### 🔴 오프셋 기반 페이지네이션 단점

1. 각각의 페이지를 요청하는 사이에 데이터의 변화가 있을 경우 중복 데이터 노출

2. 대부분의 RDBMS에서 OFFSET 쿼리의 퍼포먼스 이슈

- DB는 모든 정렬 기준(ORDER BY)에 대해 해당 row가 몇번째 순서를 갖는지 모른다. 따라서 offset 값을 지정하여 쿼리를 한다고 했을 때 임시로 해당 쿼리의 모든 값들을 전부 만들어놓은 후 지정된 갯수만 순회해 자르는 방식을 사용한다.

- offset이 작으면 문제 없지만 row가 아주 많은 경우 offset이 커질 수록 쿼리의 퍼포먼스는 비례해 떨어진다.

> offset기반 페이지네이션은 원하는 데이터가 ‘몇 번째’에 있다는 데에 집중하고 있다.

> cursor기반 페이지네이션은 원하는 데이터가 ‘어떤 데이터의 다음’에 있다는 데에 집중하고 있다.

# ✅ 커서 기반 페이지네이션

## 1️⃣ 케이스 1: id DESC 정렬 시

### 첫번째 리스트 출력

```sql
SELECT id FROM 'products' ORDER BY id DESC LIMIT 5
```

### 결과

| id   | title     |
| ---- | --------- |
| 1000 | 상품#1000 |
| 999  | 상품#999  |
| 998  | 상품#998  |
| 997  | 상품#997  |
| 996  | 상품#996  |

### 여기서 cursor는 products 테이블의 id (996)

### 다음 리스트 출력

```sql
SELECT id, title
		FROM 'products'
		WHERE id < 996
		ORDER BY id DESC
		LIMIT 5
```

## 2️⃣ 케이스 2: price ASC 정렬시

### 첫번째 리스트 출력

```sql
SELECT id, title, price
		FROM 'products'
		ORDER BY price ASC
		LIMIT 5
```

### 결과

| id  | title    | price |
| --- | -------- | ----- |
| 242 | 상품#242 | 5800  |
| 335 | 상품#335 | 5900  |
| 798 | 상품#798 | 9500  |
| 957 | 상품#957 | 13200 |
| 446 | 상품#446 | 14100 |

### 여기서 cursor는 products 테이블의 price

### 다음 리스트 출력 (ASC니까 부등호 방향 반대)

```sql
SELECT id, title, price
		FROM 'products'
		WHERE price > 14100
		ORDER BY price ASC
		LIMIT 5
```

### 🔴 문제점

id는 고유값이라 겹칠 일이 없지만 가격은 겹칠 수 있음

→ 때문에 커서 기반 페이지네이션을 위해서는 반드시 정렬 기준이 되는 필드 중 (적어도 하나는) 고유값이어야 함.

→ ORDER BY절에 id 필드를 두번째 정렬 기준으로 추가해봄

## 3️⃣ 케이스 3: price ASC, id ASC 정렬 시

### 첫번째 리스트 출력

```sql
SELECT id, FROM 'products' ORDER BY price ASC, id ASC LIMIT 5
```

### 다음 리스트 출력

```sql
SELECT id, title, price
		FROM 'products'
		WHERE (price > 14100 OR (price = 14100 AND id > 446))
		ORDER BY price ASC, id ASC
		LIMIT 5
```

### 🔴 OR절 사용의 문제점

1. 대부분의 RDBMS는 WHERE절에 OR-clause를 사용하면 인덱싱을 못함.
2. 클라이언트가 ORDER BY에 있는 모든 필드를 알아야함 + 매 요청시마다 이 값들을 전부 보내야 함

## 4️⃣ 케이스 4: price DESC, id DESC (커스텀 cursor 생성)

## CONCAT

- MySQL 내장함수
- 문자열을 합친다. (숫자인 경우 문자열로 자동 변환)

## LPAD

- MySQL 내장함수
- 문자열/숫자를 지정된 길이의 문자열로 채움(왼쪽)

### - 첫번째 리스트 출력

```sql
SELECT id, title, price,
			CONCAT(LPAD(price, 10, '0'), LPAD(id, 10 , '0'))
			as 'cursor'
			FROM 'products'
			ORDER BY price DESC, id DESC
			LIMIT 5;
```

### - 결과

| id  | title    | price | cursor                       |
| --- | -------- | ----- | ---------------------------- |
| 446 | 상품#446 | 14100 | 00000**14100**0000000**446** |
| 957 | 상품#957 | 13200 | 00000**13200**0000000**957** |
| 798 | 상품#798 | 9500  | 000000**9500**0000000**798** |
| 335 | 상품#335 | 5900  | 000000**5900**0000000**335** |
| 242 | 상품#242 | 5800  | 000000**5800**0000000**242** |

1. 숫자값인 데이터를 LPAD를 이용해 10자 길이의 고정 문자열로 만들고
2. 이를 CONCAT으로 붙인다.
3. 10자인 이유: price, id 필드가 10자를 넘어가지 않을 거라는 확신 때문 (늘려도 무방)

### - 다음 리스트 출력

```sql
SELECT id, title, price,
			CONCAT(LPAD(price, 10, '0'), LPAD(id, 10, '0'))
			as 'cursor'
			FROM 'products'
			HAVING 'cursor' < '0000005800000000242'
			ORDER BY price DESC, id DESC
			LIMIT 5
```

🔴 HAVING절을 사용하면 인덱싱에 문제가 생김.

## 5️⃣ 케이스 5: price ASC, id ASC (커스텀 cursor 생성)

ASC가 섞여있을 경우, 두번째 리스트 쿼리에서 부등호 방향만 바꾸면 됨.

하지만 생성하는 cursor값을 일관되게 만드는 것이 더 좋음.

→ ASC/DESC가 섞여 있을때 대응도 쉽고 쿼리 생성하는 코드를 자동화 할 수 있기 때문

## POW

- MySQL 내장함수
- 제곱연산

### - 첫번째 리스트 출력

```sql
SELECT id, title, price
			CONCAT(LPAD(POW(10, 10) - price, 10, '0'), LPAD(POW(10, 10) - id, 10, '0'))
			as 'cursor'
			FROM 'products'
			ORDER BY price ASC, id ASC
			LIMIT 5;
```

### - 결과

| id  | title    | price | cursor               |
| --- | -------- | ----- | -------------------- |
| 242 | 상품#242 | 5800  | 99999942009999999758 |
| 335 | 상품#335 | 5900  | 99999941009999999665 |
| 798 | 상품#798 | 9500  | 99999905009999999202 |
| 957 | 상품#957 | 13200 | 99999868009999999043 |
| 446 | 상품#446 | 14100 | 99999859009999999554 |

price → POW(10, 10) - price

id → POW(10, 10) - id

10\*\*10 = 10,000,000,000

가장 낮은 price 값을 가진 상품이 가장 높음 cursor를 가지게 됨.

### - 다음 리스트 출력(ASC/DESC와 무관하게 동일함)

```sql
SELECT id, title, price,
			CONCAT(LPAD(POW(10, 10) - price, 10 , '0',
			LPAD(POW(10,10) - id, 10, '0')
			) as 'cursor'
			FROM 'products'
			HAVING 'cursor' < '99999859009999999554'
			ORDER BY price ASC, id ASC
			LIMIT 5;
```

이 쿼리는 alias를 HAVING 절에서 조건으로 쓰는 경우다.

이는 후처리 단계에서 동작하므로 인덱스 활용도 어렵고, MySQL은 이때 filesort 같은 임시 정렬 작업을 많이 하게 되어 퍼포먼스가 저하된다.

- 쿼리의 정렬 조건(**`ORDER BY price ASC, id ASC`**)에 사용된 컬럼들이 함수(**`CONCAT`**, **`LPAD`**, **`POW`**) 연산 결과로 만들어진 복합 표현식(**`cursor`**)과 연관되어 있어 인덱스 직접 사용이 어렵다.
- 특히 **`CONCAT(LPAD(POW(10, 10) - price, 10 , '0'), LPAD(POW(10,10) - id, 10, '0'))`**는 컬럼 자체가 아니라 계산된 결과값이기 때문에 인덱스 컬럼이 아니다.

## 🔴 Using Filesort

- 인덱스를 이용한 정렬이 불가능할 때 MySQL이 임시 공간을 사용해 데이터를 정렬하는 방식
- 성능에 부정적인 영향을 미친다.
- filesort는 메모리 버퍼에 데이터가 적재되지 않으면 임시 디스크 파일을 만들고 외부 정렬을 수행하여 디스크 I/O가 증가하고 CPU와 메모리 사용량 또한 늘어나 쿼리 실행 속도가 느려진다.

## 🧩 HAVING 대신 WHERE 쓰기

```sql
SELECT * FROM (
    SELECT id, title, price,
        CONCAT(LPAD(POW(10, 10) - price, 10 , '0'),
        LPAD(POW(10,10) - id, 10, '0')
        ) as 'cursor'
    FROM 'products'
) AS sub
WHERE 'cursor' < '99999859009999999554'
ORDER BY price ASC, id ASC
LIMIT 5;
```

WHERE 절로 바꾸면 alias를 직접 쓸 수 없으니 서브쿼리로 감싸서 alias를 생성한다.

외부 쿼리에서 WHERE로 필터링하는 방법을 쓰는 것이 일반적이며, 이 방법이 인덱스 활용과 쿼리 성능 개선에 도움된다.
