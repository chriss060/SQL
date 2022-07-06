# 계층적 질의문(HIERARCHICAL QUERY)

: 데이터간의 부모 관계를 표현할 수 있는 칼럼을 지정하여 계층적 관계 표현

```sql
select [level], column, expression
from table
start with conditions #시작하는 최상위 행(서브쿼리 사용가능)
connect by prior conditions(s); #계층관계의 데이터 지정하는 칼럼
```

START WITH, CONNECT BY 절을 이용해 계층적 출력 형식과 시작 위치 제어

NOCYLCLE : 무한 루프 방지를  위해 connect by nocycle 파라미터 활용



#### 계층적 질의문 TOP DOWN

:루트 노드부터 먼저 출력

CONNECT BY PRIOR 자식 칼럼= 부모 칼럼

```sql
select deptno, dname, college
from department
start with deptno = 10
connect by prior deptno = college;
```



#### 계층적 질의문 BOTTOM UP

:단말(leaf) 노드 부터 출력

CONNECT BY PRIOR 부모 칼럼= 자식 칼럼

```sql
select deptno, dname, college
from department
start with deptno = 102
connect by prior college = deptno;
```



##### 계층적 질의문 -1 : 레벨별 구분

```sql
select level, LPAD(' ', 2*(level-1)) || dname 조직도
from department
start with dname = '공과대학'
connect by prior deptno = college;
```



##### 계층적 질의문 -2 : 가지제거 방법

1.  where 절을 이용해 임의의 가지 삭제

   ```sql
   select deptno, college, dname, loc
   from department
   where dname != '정보미디어학부'
   start with college is null
   connect by prior deptno = college;
   ```

   

2. connect by 절에 AND 절을 이용해 임의의 가지와 자식 노드까지 동시 삭제

   ```sql
   select deptno, college, dname, loc
   from department
   start with college is null
   connect by prior deptno = college
   and dname != '정보미디어학부';
   ```

   

#### 계층적 질의문 응용

1. **connect_by_root** 

   :최상위 row(루트 노드)  정보를 얻어 올 수 있음

   ```sql
   select lpad(' ',4*(level-1)) || ename 사원명,
   empno 사번, connect_by_root empno 최상위사번,
   level
   from emp
   start with job = upper('president')
   connect by prior empno = mgr;
   ```

   

2. **connect_by_isleaf**

   :row의 최하위레벨(리프 노드) 여부를 반환 (리프노드 :1)

   ```sql
   select lpad(' ',4*(level-1)) || ename 사원명,
   empno 사번, connect_by_isleaf leaf_YN, level
   from emp
   start with job = upper('president')
   connect by nocycle prior empno = mgr;
   ```

   

3. **sys_connect_by_path**

   : 현재 row 까지의 path정보를 쉽게 얻어 올 수 있음.

   ```sql
   select lpad(' ',4*(level-1)) || ename 사원명,
   empno 사번, sys_connect_by_path(ename,'/') PATH
   from emp
   start with job = upper('president')
   connect by nocycle prior empno = mgr;
   ```

   ```sql
   #leaf노드만 전체 path가 나오도록 수정 - connect_by_isleaf
   select level, sys_connect_by_path(ename,'/') PATH
   from emp
   where connect_by_isleaf = 1
   start with mgr is null
   connect by prior empno = mgr;
   ```

   

4. **order siblings by**

   : order by 에 의해서 트리구조가 깨지는 것을 방지 하기 위해

    트리의 상관관계를 그대로 유지하면서 내부 요소 정렬

   ```sql
   select lpad(' ', 4*(level-1))|| ename 사원명,
   ename ename2, empno 사번, level
   from emp
   start with job = upper('president')
   connect by nocycle prior empno = mgr
   order siblings by ename;
   ```

   

