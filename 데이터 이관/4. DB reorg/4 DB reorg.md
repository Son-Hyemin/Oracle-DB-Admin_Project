# 4. DB reorg

소유자: 쏘니

<aside>
💡 순서

1. emp 테이블 구성
2. 10번과 20번을 지운 후에  compact 과 shrink 로 HWM 가 내려가는지 테스트
3. 관련 인덱스가 VALID 한지 확인하시오 !

</aside>

---

0. 아래와 같이 테이블 구성

```sql
   @demobld.sql
   create  index  emp_empno  on  emp(empno);
   create  index  emp_ename  on  emp(ename);
   create  index  emp_sal    on  emp(sal);
   create  index  emp_job    on  emp(job);
   create  index  emp_detpno on  emp(deptno);

  1. scott 유져에서 emp 테이블의 HWM 를 높인다. 

  SQL> insert  into emp
         select  *
           from emp;
  
  SQL>   /  엔터  <---------  10번 수행 
  SQL>  commit;
  SQL>  delete  from  emp  where  deptno in (10,20);
  SQL>  commit; 

 오라클 이수자 평가 4번 문제 제출물로 결과 캡쳐해서 가지고 있으세요
 인덱스 상태가 valid 한지 확인하면 됩니다.

```


1. emp 테이블의 실제 사용하고 있는 block 의 개수
    
    ```sql
    
    orcl SCOTT > 
    
    select count(distinct dbms_rowid.rowid_block_number(rowid)) as blocks
          from emp;
    
        BLOCKS
    ----------
           169
    ```
    
    

1. High water mark 까지 할당된 block 의 개수 확인
    
    ```sql
    orcl SCOTT >
    
    select blocks
        from user_segments
        where segment_name='EMP';
    
        BLOCKS
    ----------
           256
    ```
        

1. 테이블 compact 작업 수행(COMPACT 작업 수행 명령어는 2문장)
    
    ```sql
    # emp테이블의 row를 이동이 가능하도록 변경하기
    alter table emp enable row movement;
    
    # emp테이블의 공간을 작게 줄이기
    alter table emp shrink space compact;
    ```
        

1. emp 테이블의 실제 사용하고 있는 block 의 수 (169 → 76로 줄어들었음)
    
    ```sql
    select count(distinct dbms_rowid.rowid_block_number(rowid)) 
            as blocks
        from  emp;
    
        BLOCKS
    ----------
            76
    ```
        

1.  High water mark 까지 할당된 block 의 개수 확인
    
    ```sql
    orcl SCOTT >
    
    select blocks
     from user_segments
     where segment_name='EMP';
    
        BLOCKS
    ----------
           256
    ```
    
    ⇒ BLOCK의 수는 줄었는데, HWM까지 할당된 BLOCK의 개수는 그대로인 것을 확인할 수 있다. 그래서 내려주는 작업이 필요한데, 그게 바로 SHRINK
    
    

1. High water mark 를 내려주는 작업 SHRINK 수행(256 → 88)
    
    ```sql
    alter table emp shrink space; 
    
    # 확인하기
    select  blocks
     from   user_segments
     where  segment_name='EMP';
    ```
    
    
2. 인덱스의 상태가 VALID한지 확인하기
    
    ```sql
    orcl SCOTT > select index_name, status
      2     from user_indexes
      3     where table_name='EMP';
    
    INDEX_NAME                     STATUS
    ------------------------------ --------
    EMP_DETPNO                     VALID
    EMP_JOB                        VALID
    EMP_SAL                        VALID
    EMP_ENAME                      VALID
    EMP_EMPNO                      VALID
    ```
    
