---
title: "SQL 200 - Part I"
permalink: /docs/study/sql200/part1/
last_modified_at: 2022-06-25T08:48:05
layout: single
#classes: wide
toc: true
toc_sticky: true
sidebar:
  nav: "sql200"
---

## Book Study 를 시작하며

**sql200** 시리즈는 사내 스터디 모임에서 사용하는 교재의 **PL/SQL** 실습 쿼리를 **BigQuery** 로 작성할 수 있도록 차이점들을 설명하고 있습니다.

- 초보자를 위한 SQL 200제 (PL/SQL)
  - http://www.yes24.com/Product/Goods/90226600
  - 참고 파일 - http://www.infopub.co.kr/new/down1/5674-850.zip

### Sample Tables

교재에서 사용하는 예시 테이블들 BigQuery의 **임시 테이블(Temp Table)** 로 만들기 위한 쿼리입니다.

스터디 참여하시는 분들은 임시 테이블을 반복적으로 실습 쿼리에 포함시키지 말고 **영구 테이블(Permanent Table)** 로 만들어 놓은 아래를 참조해서 실습 진행하시면 됩니다.

- `bigdata-adhoc.open_mart_ds.emp`
- `bigdata-adhoc.open_mart_ds.dept`
- `bigdata-adhoc.open_mart_ds.salgrade`

#### 예시 (임시) 테이블 생성 쿼리

>```sql
-------------------------------------------------------------------------------
-- Employee Table
-------------------------------------------------------------------------------
CREATE TEMP TABLE emp (
  EMPNO       INT64 NOT NULL,
  ENAME       STRING,
  JOB         STRING,
  MGR         INT64 ,
  HIREDATE    DATE,
  SAL         INT64,
  COMM        INT64,
  DEPTNO      INT64
);
--
INSERT INTO emp VALUES
(7839,'KING','PRESIDENT',NULL,'1981-11-17',5000,NULL,10),
(7698,'BLAKE','MANAGER',7839,'1981-05-01',2850,NULL,30),
(7782,'CLARK','MANAGER',7839,'1981-05-09',2450,NULL,10),
(7566,'JONES','MANAGER',7839,'1981-04-01',2975,NULL,20),
(7654,'MARTIN','SALESMAN',7698,'1981-09-10',1250,1400,30),
(7499,'ALLEN','SALESMAN',7698,'1981-02-11',1600,300,30),
(7844,'TURNER','SALESMAN',7698,'1981-08-21',1500,0,30),
(7900,'JAMES','CLERK',7698,'1981-12-11',950,NULL,30),
(7521,'WARD','SALESMAN',7698,'1981-02-23',1250,500,30),
(7902,'FORD','ANALYST',7566,'1981-12-11',3000,NULL,20),
(7369,'SMITH','CLERK',7902,'1980-12-11',800,NULL,20),
(7788,'SCOTT','ANALYST',7566,'1982-12-22',3000,NULL,20),
(7876,'ADAMS','CLERK',7788,'1983-01-15',1100,NULL,20),
(7934,'MILLER','CLERK',7782,'1982-01-11',1300,NULL,10)
;
-------------------------------------------------------------------------------
-- Departement Table
-------------------------------------------------------------------------------
CREATE TEMP TABLE dept (
  DEPTNO INT64,
  DNAME STRING,
  LOC STRING
);
--
INSERT INTO dept VALUES
(10, 'ACCOUNTING', 'NEW YORK'),
(20, 'RESEARCH',   'DALLAS'),
(30, 'SALES',      'CHICAGO'),
(40, 'OPERATIONS', 'BOSTON')
;
-------------------------------------------------------------------------------
--Salary Grade Table
-------------------------------------------------------------------------------
CREATE TEMP TABLE salgrade (
  grade   INT64,
  losal   INT64,
  hisal   INT64
);
--
INSERT INTO salgrade  VALUES
(1,700,1200),
(2,1201,1400),
(3,1401,2000),
(4,2001,3000),
(5,3001,9999)
;
```

## Part 1. SQL 첫발 내딛기

#### 001. 테이블에서 특정 열(Column) 선택하기
>```sql
SELECT empno, ename, sal
  FROM `bigdata-adhoc.open_mart_ds.emp`;
```

#### 002. 테이블에서 모든 열(Column) 가져오기
```sql
SELECT *
  FROM `bigdata-adhoc.open_mart_ds.emp`;

>**[BigQuery]** 이후에 특정 컬럼을 한 번 더 출력하는 경우, `테이블명.*` 이 아닌 `*` 이후에 컬럼명을 작성하시면 됩니다.
`테이블명`의 사용은 quoted identifier


>```sql
SELECT *, depno
  FROM `bigdata-adhoc.open_mart_ds.dept`;
```

#### 003. 컬럼 별칭을 사용하여 출력되는 컬럼명 변경하기

>```sql
SELECT empno AS employee_number, ename AS employee_name, sal AS salary
  FROM `bigdata-adhoc.open_mart_ds.emp`;
```

- 테이블을 조회하는 것은 기존 테이블에서 새로운 테이블을 만들어 내는 것과 동일합니다.
- 네모를 가로로 붙여가며 가로줄을 만들어 갈 때 재료로 쓰이는 네모는,
  - 기존 테이블에서 가져온 그대로 사용할 수 있습니다. (이름만 바꿀 수도 있습니다.)
  - 기존 테이블에서 가져온 네모들을 가공해서 새로운 네모를 만들 수도 있습니다.
    - Calculated Column (계산 컬럼)
  - 즉석에서 만들어 낼 수도 있습니다.
    - Constant Column (상수 컬럼)
- 한글 컬럼명은 지원하지 않습니다.


>```sql
SELECT ename, sal * (12 + 3000) AS monthly_pay
  FROM `bigdata-adhoc.open_mart_ds.emp`
 ORDER BY monthly_pay DESC;
```

여기서 `sal * (12 + 3000)` 은 원자재에 해당하는 테이블 (base table) 에서 네모를 가져와 새롭게 가공해 낸 네모입니다.  새롭게 만들어진 네모이기 때문에 이름도 새로 지어 줍니다.

#### 004. 연결 연산자 사용하기

연결 연산자는 문자열과 문자열을 연결하여 새로운 문자열을 만들어 냅니다.  (연산자)기호로는 `||` 를 사용하고 함수로는 `CONCAT(문자열1, 문자열2, ...)`을 사용합니다.

>```sql
SELECT ename || sal, CONCAT(ename, sal)
  FROM `bigdata-adhoc.open_mart_ds.emp`;
```

>```sql
SELECT ename || '의 월급은 ' || sal || '입니다.' AS salary_info,
       CONCAT(ename, '의 월급은 ', sal, '입니다.') AS salary_info2
  FROM `bigdata-adhoc.open_mart_ds.emp`;
```

#### 005. 중복된 데이터를 제거하여 출력하기 (DISTINCT)

- *DISTINCT col1, col2*   --> 나열된 컬럼의 Unique한 조합들 출력
- `UNIQUE`는 오라클에서만 지원

>```sql
SELECT job FROM `bigdata-adhoc.open_mart_ds.emp`;
+-----+-----------+
| Row |    job    |
+-----+-----------+
|   1 | CLERK     |
|   2 | PRESIDENT |
|   3 | MANAGER   |
|   4 | ANALYST   |
|   5 | ANALYST   |
|   6 | MANAGER   |
|   7 | CLERK     |
|   8 | CLERK     |
|   9 | MANAGER   |
|  10 | CLERK     |
|  11 | SALESMAN  |
|  12 | SALESMAN  |
|  13 | SALESMAN  |
|  14 | SALESMAN  |
+-----+-----------+
--
SELECT DISTINCT job FROM `bigdata-adhoc.open_mart_ds.emp`;
+-----+-----------+
| Row |    job    |
+-----+-----------+
|   1 | CLERK     |
|   2 | PRESIDENT |
|   3 | MANAGER   |
|   4 | ANALYST   |
|   5 | SALESMAN  |
+-----+-----------+
```

#### 006. 데이터를 정렬하여 출력하기 (ORDER BY)

>```sql
SELECT ename, sal
  FROM `bigdata-adhoc.open_mart_ds.emp`
 ORDER BY sal ASC;
--
-- 실행 순서 : `FROM` --> `SELECT` --> `ORDER BY`
```

정렬 조건으로 여러 컬럼을 차례로 지정할 수 있습니다.

>```sql
SELECT ename, deptno, sal
  FROM `bigdata-adhoc.open_mart_ds.emp`
 ORDER BY deptno ASC, sal DESC;
```

#### 007. WHERE 절 배우기 ① (숫자 데이터 검색)

>```sql
SELECT ename, sal, job
  FROM `bigdata-adhoc.open_mart_ds.emp`
 WHERE sal = 3000;
--
-- 비교연산자
+----------+--------------------------+
| Operator |         Meaning          |
+----------+--------------------------+
| >=       | greater than or equal to |
| <=       | less than or equal to    |
| =        | equal                    |
| <>       | not equal                |
| !=       | not equal                |
+----------+--------------------------+
```

#### 008. WHERE 절 배우기 ② (문자와 날짜 검색)

>```sql
SELECT ename, sal, job, hiredate, deptno
  FROM `bigdata-adhoc.open_mart_ds.emp`
 WHERE ename = 'SCOTT';
```

- 문자열을 표현할 때 따옴표는 홑따옴표('), 쌍따옴표(") 모두 사용가능합니다.
- 홑따옴표(')는 테이블 이름 앞뒤에 있는 역따옴표(`, Backtick) 과 혼동하시면 안됩니다.

>```sql
SELECT ename, sal, job, hiredate, deptno
  FROM `bigdata-adhoc.open_mart_ds.emp`
 WHERE hiredate = '1981-11-17';
```

#### 009. 산술 연산자 배우기 (*, /, +, -)

>```sql
SELECT ename, sal * 12 AS annual_salary
  FROM `bigdata-adhoc.open_mart_ds.emp`
 WHERE sal * 12 >= 36000;
```


- 계산기 처럼 쿼리를 사용할 수 있다.

>```sql
-- SELECT 300 + 200 * 2 FROM DUAL;  -- 오라클 SQL
SELECT 300 + 200 * 2;
```
- 오라클 SQL은 `FROM` 절이 없으면 실행이 되지 않기 때문에 `FROM DUAL` 이라는 Dummy 테이블을 사용한다.
- BigQuery는 `FROM` 절 없이 `SELECT` 절 만으로도 실행이 가능하다.

>```sql
SELECT ename, sal, comm, sal + comm AS sum
  FROM `bigdata-adhoc.open_mart_ds.emp`
 WHERE deptno = 10;
--
+-----+--------+------+------+------+
| Row | ename  | sal  | comm | sum  |
+-----+--------+------+------+------+
|   1 | MILLER | 1300 | null | null |
|   2 | KING   | 5000 | null | null |
|   3 | CLARK  | 2450 | null | null |
+-----+--------+------+------+------+
```

- `sum`이 null인 이유는 ?

>```sql
SELECT ename, sal, comm, sal + IFNULL(comm, 0) AS sum
  FROM `bigdata-adhoc.open_mart_ds.emp`
 WHERE deptno = 10;
--
+-----+--------+------+------+------+
| Row | ename  | sal  | comm | sum  |
+-----+--------+------+------+------+
|   1 | MILLER | 1300 | null | 1300 |
|   2 | KING   | 5000 | null | 5000 |
|   3 | CLARK  | 2450 | null | 2450 |
+-----+--------+------+------+------+
```
- NVL() 은 오라클 함수 --> BigQuery에서는 IFNULL() 또는 COALESCE() 사용

#### NULL ???

- WHERE


#### 010. 비교 연산자 배우기 (>, <, >=, <=, =, !=, <>) ①

#### 011. 비교 연산자 배우기 (BETWEEN ~ AND ~) ②


>```sql
SELECT ename, sal
  FROM `bigdata-adhoc.open_mart_ds.emp`
 WHERE sal BETWEEN 1000 AND 3000;
--
- `WHERE sal >= 1000 AND sal <= 3000;` 와 동일


#### 012. 비교 연산자 배우기 (LIKE) ③

>```sql
SELECT ename, sal
  FROM `bigdata-adhoc.open_mart_ds.emp`
 WHERE ename LIKE 'S%';
```
- `WHERE ename LIKE '_M%';`

#### 013. 비교 연산자 배우기 (IS NULL) ④

>```sql
SELECT ename, comm
  FROM `bigdata-adhoc.open_mart_ds.emp`
 WHERE comm IS NULL;
```

#### 014. 비교 연산자 배우기 (IN) ⑤

>```sql
SELECT ename, sal, job
  FROM `bigdata-adhoc.open_mart_ds.emp`
 WHERE job IN ('SALESMAN', 'ANALYST', 'MANAGER');
```


>```sql
SELECT ename, sal, job
  FROM `bigdata-adhoc.open_mart_ds.emp`
 WHERE job NOT IN ('SALESMAN', 'ANALYST', 'MANAGER');
```

#### 015. 논리 연산자 배우기 ⑤

- `AND`, `OR` AND `NOT`

>```sql
SELECT ename, sal, job
  FROM `bigdata-adhoc.open_mart_ds.emp`
 WHERE job = 'ABCDEFG' AND sal >= 1200;
```