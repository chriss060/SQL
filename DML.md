# DML

: 테이블에 새로운 데이터를 입력하거나 기존 데이터를 수정, 삭제 하기 위한 명령어

* INSERT, DELETE, UPDATE, MERGE

  

  ### 데이터 입력

  INSERT 명령문

  : 테이블에 데이터를 입력하기 위한 명령어

  1) 단일행 입력: 한번에 하나의 행을 테이블에 입력

     INTO 절에 명시한 칼럼에 VALUES 절에서 지정한 칼럼 값 입력 (데이터값 기존 칼럼과 동일해야함)

     ```sql
     insert into student
     values(10110, '홍길동', 'hong','1','8501011143098','85/01/01','041)540-34444',170, 70,101,9903);
     commit;
     ```

     * NULL 입력 : 데이터 입력하는 시점에 해당 칼럼 값을 모르거나, 미확정일때 해당 칼럼 이름, 값 생략 (or VALUES 절에 NULL 사용)

       ```sql
       # 나머지 칼럼 값은 NULL로 자동 설정
       insert into department(deptno, dname)
       values(300, '생명공학부');
       commit;
       ```

       

  2) 다중 행 입력: 서브쿼리를 이용해 한번에 여러 행 동시에 입력

     - 단일 테이블에 다중 행 입력

       : insert 명령문에서 서브쿼리 절을 이용해 자신이나 다른 테이블에 데이터를 복사하여 여러 행 동시 입력

     - INSERT ALL / FIRST

       : 서브 쿼리의 결과 집합을 조건 없이 여러 테이블에 동시에 입력 

        ALL :서브쿼리 결과 집합을 해당 하는 insert절에 모두 입력

       FIRST : 서브 쿼리의 결과 집합을 해당하는 첫번째 insert 절에 입력

       ```sql
       create table height_info(
       studno number(5),
       name varchar2(10),
       height number(5,2));
       
       create table weight_info(
       studno number(5),
       name varchar2(10),
       weight number(5,2));
       
       insert all
       into height_info values(studno, name, height)
       into weight_info values(studno, name ,height)
       select studno, name, height, weight
       from student
       where grade >= '2';
       ```

       ```sql
       insert first
       when height>170 then
           into height_info values(studno,name,height)
       when weight>70 then
           into weight_info values(studno,name,weight)
           (select studno, name, height, weight
           from student
           where grade>='2');
       ```

       

     - CONDITIONAL INSERT ALL / FIRST

       ALL : 서브 쿼리의 결과 집합 중에서 조건절 1을 만족하는 결과 행은 table1에 입력, 조건절 2을 만족하는 결과 행은 table2에 입력

        어느 조건도 만족하지 않은 행은 table3에 입력

       ```sql
       insert all
       [when 조건절 1 then
       into table1 values [(col1, col2 ...)]
       when 조건절 2 then
       into table2 values [(col1, col2 ...)]
       else
       into table3 values [(col1, col2 ...)]
       subquery;
       ```

       ```sql
       insert first
       when height>170 then
           into height_info values(studno,name,height)
       when weight>70 then
           into weight_info values(studno,name,weight)
           (select studno, name, height, weight
           from student
           where grade>='2');
       ```

       FIRST : 서브 쿼리의 결과 집합에 대해 when 조건절에서 지정한 조건을 만족하는 첫번째 테이블에 우선적으로 입력하고 그 외의 결과집합에서

       ​			나머지 when 절 조건이 만족하면 첫번째 insert한 행을 제외하고 insert.

       ```sql
       insert first
       [when 조건절 1 then
       into table1 values [(col1, col2 ...)]
       when 조건절 2 then
       into table2 values [(col1, col2 ...)]
       else
       into table3 values [(col1, col2 ...)]
       subquery;
       ```

       

     - PIVOTING INSERT ALL

​					 : 하나의 행을 여러 개의 행으로 나누어서 입력하는 기능

​					 UNCONDITIONAL INSERT ALL 명령문과 거의 동일

 					INTO 절에서 하나의 테이블만 지정

```sql
create table sales(
sales_no number(4),
week_no number(2),
sales_mon number(7,2),
sales_tue number(7,2),
sales_wed number(7,2),
sales_thu number(7,2),
sales_fri number(7,2));

insert into sales values(1101, 4, 100, 150, 80, 60, 120);
insert into sales values(1102, 5, 300, 300, 230, 120, 150);

create table sales_data(
sales_no number(4),
week_no number(4),
day_no number(2),
sales number(7,2));

#Pivoting insert 명령문을 사용해 sales 테이블의 요일별 data를 통합하여 sales_data 테이블 하나의 행으로 입력
insert all
into sales_data values(sales_no, week_no, '1',sales_mon)
into sales_data values(sales_no, week_no, '1',sales_tue)
into sales_data values(sales_no, week_no, '1',sales_wed)
into sales_data values(sales_no, week_no, '1',sales_thu)
into sales_data values(sales_no, week_no, '1',sales_fri)
    select sales_no, week_no, sales_mon, sales_tue, sales_wed, sales_thu, sales_fri
    from sales;

```



### 데이터 수정

: UPDATE 명령문을 사용해서 저장된 데이터 수정 

 (where절 생략시 테이블 모든 행 수정 가능)

```sql
update table
set column = value [, column =value, ...]
[where condition];
```

```sql
create table professor_temp as
select *
from professor
where position = '교수';

update professor_temp
set position='명예교수'
where position ='교수';

```

#### 서브 쿼리를 이용한 데이터 수정

:  update 명령문의 set절에서 서브쿼리 이용

  다른 테이블에 저장된 데이터를 검색하여 한꺼번에 여러 칼럼 수정

 

```sql
update table1
set (col1, col2 ,...) = (select s_col1, s_col2, ...)
						from table2
						[where condition]
where condition 2;
```

```sql
# 학번이 10201인 학생 정보 10102번 학생 정보로 덮어쓰기
update student
set (grade, deptno) = (select grade, deptno
                        from student
                        where studno = 10102)
where studno = 10201;
```



### 데이터 삭제

:delete 명령문은 테이블에 저장된 데이터 삭제를 위한 조작어 (테이블 자체 삭제는 아님)

```sql
delete [from] table
[where condition];
```



#### 서브쿼리를 이용한 데이터 삭제

:where 절에서 서브 쿼리 사용, 다른 테이블에 저장된 데이터를 검색해 한꺼번에 여러행의 내용 삭제

(where 절의 칼럼이름과 서브쿼리의 칼럼 이름은 달라도 됨)

```sql
delete from table
where (col1, col2, ..)=(select s_col1, s_col2, ...
						from table2
						[where condition2]);
```

```sql
delete from student
where deptno= (select deptno
                from department
                where dname ='컴퓨터공학과');
```



### MERGE

: 구조가 같은 두개의 테이블을 비교하여 하나의 테이블로 합치기 위한 데이터 조작어

 when 절의 조건절에서 결과 테이블에 해당 행이 존재하지 않으면 

 update 명령문에 의해 새로운 값으로 수정, 그렇지 않으면 insert 명령문으로 새로운 행 삽입

```sql
merge into table
using [table | view | subquery] alias
on [join conditon]
#when절에서는 테이블이나 뷰 이름 대신 using절에서 지정한 별명 사용
when matched then
	update set ...
when not matched then
	insert into
	values ...,
```

```sql
merge into professor p
using professor_temp f
on (p.profno = f.profno)
when matched then
update set p.position = f.position
when not matched then
insert values(f.profno, f.name, f.userid, f.position, f.sal,f.hiredate, f.comm, f.deptno);
```



### 트랜잭션 관리

: 관계형 데이터 베이스에서 실행 되는 여러 개의 SQL 명령문을 하나의 논리적인 작업 단위로 처리하는 개념

COMMIT : 트랜잭션의 정상적인 종료, 처리 결과를 디스크에 영구적으로 저장

​				commit 명령문 실행하기 전에 하나의 트랜잭션 변경한 결과를 다른 트랜잭션에서 접근 불가

ROLLBACK : 트랜잭션의 전체 취소, 해당 트랜잭션에 할당된 자원을 해제, 강제종료



### 시퀀스

: 유일한 식별자, 기본키 값을 자동으로 생성하기 위하여 일련번호 생성하는 객체 (여러 테이블에서 공유 가능)

```sql
create SEQUENCE sequence
increment by n # 시퀀스 번호 증가치, 기본은 1
start with n #시퀀스 시작번호
maxvalue n | Nomaxvalue 
minvalue n | Nominvalue
cycle | nocycle #maxvalue 또는 minvalue에 도달한 후 시퀀스의 순환적인 시퀀스 번호 생성 여부 지정
cache n | nocache; # 시퀀스 생성 속도 개선을 위해 메모리에 캐쉬하는 시퀀스 개수, 기본은 20
```

```sql
create sequence s_seq
increment by 1
start with 1
maxvalue 100;
```

#### CURRVAL , NEXTVAL 함수

:insert, update 문에서 사용

서브쿼리, groupby, having, order by, distinct와 함께 사용 불가

currval : 시퀀스에서 생성된 현재 번호를 확인

nextval : 시퀀스에서 다음 번호 생성

```sql
select S_SEQ.CURRVAl
from dual;

select S_SEQ.NEXTVAL
from dual;
```

#### 시퀀스를 이용한 기본 키 생성

: 기본키로 사용할 수  있는 적절한 칼럼이 없거나 다수의 칼럼을 결합해야 식별 가능한 경우, 시퀀스 이용

NEXTVAL 함수 이용

```sql
insert into emp(empno,ename)
values(S_SEQ.nextval, 'TEST2');
```



#### 시퀀스 정의 변경, 삭제

1) 시퀀스 정의 변경:  ALTER SEQUENCE 명령문 사용

2) 시퀀스 삭제 : DROP SEQUENCE 명령문 사용

   ```sql
   #s_seq 의 최대값읜 200으로 변경
   alter sequence s_seq MAXVALUE 200;
   
   drop SEQUENCE s_seq;
   ```

   