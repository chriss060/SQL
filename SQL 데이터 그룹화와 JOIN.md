# SQL 데이터 그룹화

## 1. Group BY 절

Group by : 특정 칼럼 값을 기준으로 테이블의 전체 행을 그룹별로 나눔
					Group by 절에 명시되지 않은 칼럼은 그룹함수와 함께 사용 불가

#### Group by 사용 규칙

 1. 그룹핑 전에 where 절을 사용해 그룹 대상 집합 선정
 2. Group by 절에 select 절에서 나열 된 칼럼 이름 반드시 명시
    (Group by절에 명시한 칼럼명은 select 절에 필수적을 명시할 필요 없음)

```sql
 SELECT deptno, AVG(sal), MIN(sal), MAX(sal)
 FROM professor
 GROUP BY deptno;
```

## 2. 다중 칼럼을 이용한 그룹핑

​	: 하나 이상의 칼럼을 사용해 그룹을 나누고 그룹별로 다시 서브 그룹 나눔

```sql
select deptno, grade, count(*), round(avg(weight))
from student
group by deptno, grade;
```

## 3. Rollup, Cube 연산자

​	(1) Rollup( ) :  group by 그룹 조건에 따라 전체 행을 그룹화 하고 각 그룹에 대해 부분합 구함 (rollup 그룹 조합 : n+1)

​	(2) Cube( ) :  rollup 에 의한 그룹 결과와 group by 절에 기술된 조건에 따라 그룹 조합 만드는 연산자 (cube 그룹핑 조합 : 2n)

```sql
SELECT deptno, position, count(*)
FROM professor
GROUP BY ROLLUP(deptno, position);

SELECT deptno, position, count(*)
FROM professor
GROUP BY CUBE(deptno, position);
```

## 4. Grouping 함수

​	: 인수로 지정된 칼럼이 rollup 이나 cube 연산자로 생성된 그룹 조합에서 사용되었는지 여부

​      0(사용됨) , 1(사용안됨) 으로 반환

```sql
select deptno, grade, count(*), grouping(deptno) grp_dno, grouping(grade) grp_grade
from student
group by rollup(deptno, grade);
```



#### Grouping Sets 함수

:  group by 절에서 그룹 조건을 여러 개 지정할 수 있는 함수

  각 그룹 조건에 대해 별도로 Group by 한 결과를 UNION ALL 한 결과와 동일

```sql
#Group by 결과 UNION ALL로 결합
select deptno, grade, null, count(*)
from student
UNION ALL
select deptno, null, to_char(birthdate, 'YYYY'), count(*)
from student
group by deptno, to_char(birthdate, 'YYYY');

#Grouping Sets 함수
select deptno, grade, to_char(birthdate, 'YYYY'), count(*)
from student
group by grouping sets((deptno, grade), (deptno,to_char(birthdate, 'YYYY')));
```



# HAVING                  

- HAVING 절 : group by 절에 의해 생성된 그룹을 대상으로 조건 적용 (where절에 의해 조건 만족하는 행 집합 선택 후)
- where절에는 group 함수 사용 불가(ex: count(*))                                                                                                

```sql
-- Having 절을 사용하여 4명 초과 학년만 출력 --
SELECT grade, count(*), round(AVG(height)) avg_height
FROM student
GROUP BY grade
HAVING count(*) > 4
ORDER BY avg_height desc;

-- 평균 급여가 1900 이상인 부서별 평균 급여 출력 --
select deptno, avg(sal) avg_sal
from emp
group by deptno
Having avg(sal) >= 1900
order by avg(sal);
```

	#### Having vs Where

1. Having : 내부 정렬 과정에 의해 그룹화 된 집합에 대해 검색 조건 실시

2. Where :  그룹화 하기 전에 먼저 검색 조건 실행

```sql
select  deptno, round(avg(sal),1) avg_sal
from emp
where sal>1000
group by deptno
having avg(sal) >= 1900;
```



# JOIN

​	: 하나의 SQL 명령문에 의해 여러 테이블에 저장된 데이터를 한번에 조회할 수 있는 기능(결합)

 * 칼럼 이름의 모호성 해결방법

   : 동일한 칼럼 이름 연결시 칼럼 이름 앞에 테이블 이름을 접두사로 사용

 * 테이블 이름이 너무 긴 경우, 테이블 별명 사용 가능

   - sql 명령문에서 테이블 이름 과 별명 혼용 불가

```sql
# department table 과 student table 조인 (deptno 공유)
select student.studno, student.name, student.deptno, department.dname
from student, department
where student.deptno = department.deptno;
```

```sql
# 테이블 별명 사용
select p.name, p.sal, p.deptno, d.dname, d.loc
from professor p, department d
where p.deptno = d.deptno
and s.weight >= 70;

-- DALLAS에 근무하는 직원 정보 출력 --
select e.empno, e.ename, e.deptno, d.dname, d.loc
from emp e, dept d
where e.deptno = d.deptno
and d.loc = 'DALLAS'

-- 학생 이름, 학번, 지도교수, 학과이름, 학과위치 정보 출력 --
select s.name, s.studno, s.profno, p.name 지도교수, d.dname, d.loc
from student s, professor p, department d
where s.profno = p.profno
and s.deptno = d.deptno;
```

 #### 카티션 곱 (Cross JOIN)

​	: 두 개 이상의 테이블에 대해 연결 가능한 행을 모두 결합

​      대용량 테이블에서 발생할 경우 명령문 처리속도 저하

​     (FROM 절에서 CROSS JOIN 키워드 사용)

```sql
select studno, name, student.deptno, dname
from student cross join department ;
```



#### EQUI JOIN

: 조인 대상 테이블에서 공통 칼럼을 equal 비교를 통해 같은 값을 가지는 행 연결

######  where 절을 이용한 EQUI JOIN

```sql
select table.col, table2.col
from table, table2
where table.col1 = table2.col2;
```

```sql
# 조인 대상 테이블에 별명 사용
select s.name, s.studno, s.profno, p.name 지도교수, d.dname, d.loc
from student s, professor p, department d
where s.profno = p.profno
and s.deptno = d.deptno
and s.weight >= 70;
```



#### EQUI JOIN - *Natural JOIN*

: where 절을 사용하지 않고 Natural Join 키워드 사용

  오라클에서 테이블 모든 칼럼을 대상으로 공통 칼럼 조사 후 내부적으로 조인문 생성

  *조인 attribute에 테이블 별명 사용시 오류 발생*

```sql
select s.name, s.grade, deptno, d.dname
from student s
    Natural join department d
where s.grade = '4';
```

#### EQUI JOIN - *JOIN ~ Using*

: Using 절에 조인 대상 칼럼 지정 (조인 애트리뷰트에 테이블 별명 사용 금지)

  *조인 attribute에 테이블 별명 사용시 오류 발생*

```SQL
# using 절에 join 대상 칼럼 지정
select studno, name, deptno, dname, loc
student JOIN department 
Using (deptno);
```



* Inner JOIN ~ On

  ```sql
  select name, dname, loc
  from student s inner join department d
  on s.deptno = d.deptno;
  ```

  

#### NON-EQUI JOIN

: ! = 이나 between a And b 등 '=' 조건이 아닌 연산자 사용

*조인하는 테이블간 조인 attribute 없어도 성립가능*

```sql
select p.profno, p.name, p.sal, s.grade
from professor p, salgrade s
where p.sal between s.losal and s.hisal;
```



#### Outer Join

: EQUI JOIN 에서 양 측 칼럼 값중 하나가 NULL 이지만 조인 결과로 출력할 필요가 있는 경우 사용

  *IN 연산자 사용 불가, 다른조건과 OR 연산자로 결합 불가*

```sql
# NULL 값이 존재하는 칼럼 쪽에 (+) 표시
select table1.col, table2.col
from table1, table2
where table1.col(+) = table2.col; (table1.col = table2.col(+))
```

```sql
select s.name, s.grade, p.name, p.position
from student s inner join professor p
on s.profno = p.profno(+)
order by p.profno;
```

- Left Outer Join

  : From 절의 왼쪽에 위치한 테이블이 NULL을 가질 경우 사용

  ```sql
  select s.name, s.grade, p.name, p.position
  from student s left outer join professor p
  on s.profno = p.profno;
  ```

  

- Right Outer Join

  : From 절의 오른쪽에 위치한 테이블이 NULL을 가질 경우 사용

  ```sql
  select s.name, s.grade, p.name, p.position
  from student s right outer join professor p
  on s.profno = p.profno;
  ```

  

- Full Outer Join

  : LEFT Outer Join , Right Outer Join 동시에 실행한 결과 출력

  ```sql
  select s.name, s.grade, p.name, p.position
  from student s full outer join professor p
  on s.profno = p.profno
  where s.profno is null;
  ```

  

#### Self Join

:  하나의 테이블 내에 있는 칼럼끼리 연결하는 조인이 필요한 경우

- Where 절을 사용한 SELF JOIN

  - 한 테이블에서 두 개의 칼럼 연결해 EQUI JOIN

  ```sql
  select dep.dname || '의 소속은 ' || org.dname
  from department dep, department org
  where dep.college = org.deptno;
  ```

```sql
select dep.dname || '의 소속은 ' || org.dname
from department dep join department org
on dep.college = org.deptno;
```

