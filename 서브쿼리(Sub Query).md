# 서브쿼리(Sub Query)

: 하나의 명령문의 결과를 다른 SQL 명령문에 전달하기 위해 두 개 이상의  SQL 명령문을 하나로 연결해 처리

## 단일 행 서브 쿼리

: 서브 쿼리에서 단 하나의 행만 검색, 메인쿼리의 where절에서 서브쿼리의 결과와 비교 할 시 단일 행 비교연산자만 사용가능

*서브쿼리의 결과로 하나의 행만 출력되야함*

```sql
select studno, name, grade
from student
where grade = (select grade
				from student
				where userid= 'jun123');
```



## 다중 행 서브 쿼리 

:서브 쿼리에서 반환되는 결과 행이 하나 이상 일때 사용하는 서브쿼리

다중행 비교 연산자 IN, ANY, SOME, ALL, EXIST 사용

| IN        | 메인 쿼리의 비교 조건이 서브 쿼리 결과 중 하나라도 일치하면 참 ('=' 비교만 가능) |
| --------- | ------------------------------------------------------------ |
| ANY, SOME | 메인 쿼리의 비교 조건이 서브 쿼리의 결과 중 하나라도 일치하면 참 ('=' 비교만 가능) |
| ALL       | 메인 쿼리의 비교 조건이 서브 쿼리의 결과 중에서 모든 값이 일치하면 참 |
| EXISTS    | 메인 쿼리의 비교 조건이 서브쿼리의 결과 중에서 만족하는 값이 하나라도 존재하면 참 |



####  IN 연산자를 이용한 다중 행 서브 쿼리

```sql
select name, grade, deptno
from student 
where deptno in ( select deptno
                  from department
                  where college = 100);
```

#### ANY 연산자를 이용한 다중 행 서브 쿼리

: '<' 나 '>' 와 같은 범위 비교도 가능

```sql
select studno, name, height
from student
where height > ANY (select height
                    from student
                    where grade = '4');
                    
select studno, name, height
from student
where height >  (select min(height)
                    from student
                    where grade = '4');
```

#### All 연산자를 이용한 다중 행 서브 쿼리

- Any 연산자와 차이

  '>ANY' : 서브 쿼리 결과 중 최소 값보다 크면 메인쿼리의 비교 조건이 참

  '>ALL' : 서브 쿼리 결과 중 최대 값보다 크면 메인쿼리의 비교 조건이 참

  ```sql
  select studno,grade, name, height
  from student
  where height > ALL (select height
                      from student
                      where grade = '4');
                      
  select studno,grade, name, height
  from student
  where height > (select max(height)
                      from student
                      where grade = '4');
  ```

#### EXISTS 연산자를 이용한 다중 행 서브 쿼리

: 서브 쿼리에서 검색된 결과가 존재하지 않으면 메인 쿼리의 조건절은 거짓

(NOT EXISTS : 서브 쿼리에서 검색된 결과가 하나도 존재 하지 않으면 메인 쿼리 조건절이 참)

```sql
--교수 중에 커미션을 받는 교수의 교수번호, 이름, 급여, 총급여 출력--
select profno, name, sal, comm, sal+nvl(comm,0)
from professor
where exists (select profno
              from professor
              where comm is not null);

--학생 중에서 'goodstudent'라는 사용자 아이디가 없으면 1 출력--
select 1 userid_exist
from dual
where not exists(select userid
                 from student
                 where userid = 'goodstudent');
```



## 다중 컬럼 서브쿼리

: 서브 쿼리에서 여러 개 칼럼 값을 검색해 메인 쿼리의 조건절과 비교

(메인 쿼리의 조건절에서도 서브쿼리의 칼럼 수 만큼 지정해야 함) 

 

### PAIRWISE 다중 컬럼 쿼리

:메인 쿼리와 서브쿼리의 비교 대상 칼럼을 쌍으로 묶어 행별로 비교

*메인 쿼리와 서브쿼리에서 비교하는 칼럼 수는 동일*

```sql
-- 학년 별로 몸무게가 최소인 학생의 이름, 학년, 몸무게 비교--
select name, grade, weight
from student
where (grade,weight) in (select grade, min(weight)
                         from student
                         group by grade);

-- 각 부서별 최대 급여를 받는 사원들의 부서번호, 이름, 급여 출력--
select deptno,ename, sal
from emp
where (deptno, sal) in (select deptno, max(sal)
                        from emp
                        group by deptno)
order by sal desc;
```

### UNPAIRWISE 다중 컬럼 쿼리

: 메인 쿼리와 서브쿼리의 비교 대상 칼럼을 분리하여 개별적으로 비교한 후 AND 연산에 의해 최종 결과 출력

*각 칼럼이 동시에 만족하지 않더라도 개별적으로 만족 시 결과 출력 가능*

```sql
select name, grade, weight
from student
where grade in (select grade
                from student
                group by grade)
and weight in (select min(weight)
                from student
                group by grade);
```



### 상호 연관 서브쿼리

: 메인쿼리 절과 서브쿼리 간에 검색 결과를 교환 하는 서브 쿼리

*행 비교시 결과를 메인으로 반환하는 관계로 처리 성능이 저하됨 (비선호)*

```sql
select column list
from table1
where [col | expr] operator
		(select [col | expr]
		 from table2
		 where table2.column operator table1.column);
```

```sql
select name,deptno, height
from student s1
where height > (select avg(height)
				from student s2
				where s2.deptno = s1.deptno)
order by deptno;
```



### 서브 쿼리 주의 사항

1. 단일 행 서브쿼리에서 오류

   - 복수 행 값을 반환하는 서브 쿼리와 단일행 비교 연산자가 함께 사용하는 경우

   - 반환되는 칼럼의 수와 메인쿼리에서 비교되는 칼럼 수가 일치하지 않는 경우

   - 복수행을 출력하는 서브쿼리와 '=' 단일행 연산자로 비교하는 경우

     ```sql
     # grade의 종류가 4개 이므로 서브쿼리의 결과 행은 4개 - 오류
     select name, grade, weight
     from student
     where weight = (select Min(weight)
     				from student
     				group by grade);
     ```

2. 메인쿼리와 서브쿼리 칼럼 수가 맞지 않는 경우
3. 서브쿼리 내에서 order by 절 사용하면 오류 발생
4. 서브 쿼리의 결과가 null 인 경우