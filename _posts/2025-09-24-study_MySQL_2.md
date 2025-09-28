---
layout: single
title: "[MySQL] node.js + MySQL  자리 표시자 (Placeholder)"
categories: MySQL_study
tags: [MySQL_study, study]
toc: true
author_profile: false
---

# 자리 표시자 (Placeholder)

## 자리 표시자란?

- 바인드 변수라고도 불리며, `?`로 표기한다.
- 자리 표시자는 실제 값이 실행 시점에 동적으로 바인딩된다. (쿼리 내에서 나중에 입력될 값을 대체)
  - 쿼리가 처음 만들어질 때는 고정된 값 없이 구조만 정의하고, 실행 시 실제 값은 별도로 전달되어 안전하고 동적인 쿼리 실행이 가능하도록 한다.
- 보통 Node.js, Python 등에서 DB 라이브러리 함수 호출 시 **`query(sql, [param1, param2, ...])`** 같은 방식으로 값을 배열 형태로 넘긴다.
- 자리 표시자는 "빈 자리"나 "값 들어갈 위치"를 뜻하고,
- 바인드 변수는 그 자리에 실제로 들어가는 "변수" 또는 "값"을 의미

> 실제로 SQL과 DBMS에서는 구문 최적화와 성능향상을 위해 자리 표시자를 사용해 쿼리를 만든 후, 바인드 변수로 실제 값을 넣는 방식으로 동작한다.

## 사용이유

- 쿼리와 값이 분리되어 SQL 인젝션 공격 위험을 크게 줄이고 실행 효율도 높일 수 있다.
- SQL 실행 계획을 재활용할 수 있어 DB의 파싱 비용과 부하를 줄일 수 있다.

## 사용예시

- **`WHERE m.id = ?`** 구문은 실행할 때 **`?`** 자리에 특정 회원 ID가 들어가서 해당 회원만 조회하는 역할을 함.

```sql
SELECT
    m.nick_name,
    m.email,
    m.point,
    p.phone_number,
    p.is_certified
FROM member m
LEFT JOIN phone p ON p.member_id = m.id AND p.is_deleted = 0
WHERE m.id = ?;
```
