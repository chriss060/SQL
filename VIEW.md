# VIEW

: 하나 이상의 기본테이블이나 다른 뷰를 이용해 생성되는 가상 테이블

전체의 데이터 중에서 일부만 접근할 수 있도록 제한

뷰에 대한 수정결과를 뷰를 정의한 기본테이블에 적용

뷰를 정의한 기본 테이블에서 정의된 무결성 제약조건 상속

## VIEW 생성 

```sql
create [OR REPLACE][FORCE|NONFORCE] VIEW view
subquery;
#OR REPLACE : 기존 뷰와 동일한 이름으로 뷰를 재생성하는 경우
#FORCE: 기존 테이블의 존재 여부에 상관없이 뷰 생성
```

 뷰 생성시 칼럼 이름을 명시하지 않으면 기본 테이블의 칼럼 이름 상속

*함수나 표현식에 의해 정의된 칼럼은 별도로 이름을 명시(별명)*

```sql
create view view_professor as 
select profno, name, userid, position, hiredate, deptno
from professor;

select * from view_professor;
```



#### 단순 뷰

: 하나의 기본테이블에 의해 정의한 뷰

기본 테이블과 동일하게 DML 명령문 사용

뷰에 대한 무결성 제약조건도 기본테이블에 정의된 무결성 제약조건이 적용

```sql
create view v_stid_dept101 as
    select studno, name, deptno
    from student
    where deptno = 101;

select * from v_stid_dept101;
```



#### 복합 뷰

: 두 개 이상의 기본 테이블로 구성한 뷰

무결성 제약조건, 표현식, GROUP BY 절의 유무에 따라 DML 명령문의 제한적 사용

1) 뷰 정의에 포함되지 않는 기본 테이블의 칼럼이 not null 제약조건으로 지정된 경우
2) 뷰 정의시 표현식으로 정의된 칼럼에 대해서는 update, INSERT 명령문의 실행이 불가능
3) DISTINCT, 그룹함수, GROUPBY, START WITH CONNECT BY, ROWNUM 포함할 수 없음

```sql
create view v_stud_dept102
as select s.studno, s.name, s.grade, d.dname
from student s, department d
where s.deptno= d.deptno and s.deptno = 102;
```



#### 함수를 사용 한 뷰

```sql
#함수를 사용 한 뷰의 칼럼은 별명을 지어주어야함
create view v_prof_avg_sal
as select deptno, sum(sal) sum_sal, avg(sal) avg_sal
from professor
group by deptno;
```



#### 인라인 뷰

:FROM 절에서 참조하는 테이블의 크기가 큰 경우, 필요한 행과 컬럼만으로 구성된 집합을 재정의하여 질의문 효율적으로 구성

FROM절에서 서브쿼리 사용하여 생성한 임시 뷰

```sql
select dname, avg_height, avg_weight
from (select deptno, avg(height) avg_height, avg(weight) avg_weight
from student
group by deptno) s, department d
where s.deptno = d.deptno;

# 학년별 평균키를 구하고 평균키 보다 큰 학생 정보 출력
select d.grade, d.name, d.height, s.avg_height
from (select grade, avg(height) avg_height
     from student
     group by grade) s, student d
where s.grade = d.grade
and d.height > s.avg_height
order by 1;
```

```sql
select d.dname, s.m_height, t.name, s.m_height
from (select deptno, max(height) m_height
     from student
     group by deptno) s, department d, student t
where s.deptno = d.deptno
and s.m_height = t.height;
```



#### 뷰의 내부 처리 과정

1) USER_VIEW 데이터 딕셔너리에서 뷰에 대한 정의를 조회
2) 기본 테이블에 대한 뷰의 접근 권한을 확인
3) 뷰에 대한 질의를 기본 테이블에 대한 질의로 변환
4) 기본 테이블에 대한 질의를 통해 데이터 검색
5) 검색된 결과를 출력

```sql
#USERS_VIEW : 사용자가 생성한 모든 뷰에 대한 정의 저장

select view_name, text
from user_views;
```



## VIEW의 변경, 삭제

### VIEW의 변경

:CREATE 명령문에서 OR REPLACE 옵션을 이용하여 기존 뷰에 대한 정의를 삭제한 후 재생성

```sql
create or replace view v_stud_dept101
as select studno, name, deptno, grade
from student
where deptno = 101;
```



### VIEW의 삭제

:DROP을 사용해서 USERS_VIEW 데이터 딕셔너리에 저장된 뷰의 정의 삭제

```sql
drop view v_stud_dept101;
```

