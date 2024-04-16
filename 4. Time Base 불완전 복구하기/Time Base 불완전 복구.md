# 백업과 복구 이수자 평가 제출물 4번

소유자: 쏘니

> scott유저를 drop하고 , scott유저가 drop되기 전 시점을 불완전 복구해보기
> 

---

- user drop 명령어 - soctt이 가지고 있는 모든 테이블들도 같이 사라짐
    
    ```sql
    SYS > drop user scott cascade;
    ```
    

---

1. **혹시 모를 상황에 대비하여 cold backup을 수행합니다.**
    
    1-1. /home/oracle 아래에 cold_20240305 라는 디렉토리를 생성합니다.
    
    ```sql
    [shm:~]$ mkdir cold_20240305
    
    [shm:~]$ ls -ld cold_20240305
    drwxr-xr-x 2 oracle oinstall 4096  3월  5 15:56 cold_20240305
    ```

    
    1-2. datafile과 control file과 redo logfiile의 위치를 확인합니다.
    
    ```sql
    #1. datafile 위치 확인 => 모두 shm 디렉토리 아래에 있어야한다. 
    
    shm SYS > @datafile
    
    FILE_NAME
    --------------------------------------------------------------------------------
    /u01/app/oracle/oradata/shm/cuppang03.dbf
    /u01/app/oracle/oradata/shm/cuppang02.dbf
    /u01/app/oracle/oradata/shm/cuppang01.dbf
    /u01/app/oracle/oradata/shm/ts03.dbf
    /u01/app/oracle/oradata/shm/ts02.dbf
    /u01/app/oracle/oradata/shm/ts01.dbf
    /u01/app/oracle/oradata/shm/users02.dbf
    /u01/app/oracle/oradata/shm/example01.dbf
    /u01/app/oracle/oradata/shm/users01.dbf
    /u01/app/oracle/oradata/shm/undotbs01.dbf
    /u01/app/oracle/oradata/shm/sysaux01.dbf
    
    FILE_NAME
    --------------------------------------------------------------------------------
    /u01/app/oracle/oradata/shm/system01.dbf
    
    #2. control file의 위치 확인 => 모두 shm 디렉토리 아래에 있어야한다.
    
    shm SYS > @controlfile
    
    NAME
    --------------------------------------------------------------------------------
    /u01/app/oracle/oradata/shm/control01.ctl
    /u01/app/oracle/oradata/shm/control02.ctl
    /u01/app/oracle/oradata/shm/control03.ctl
    
    #3. control file의 위치 확인 => 모두 shm 디렉토리 아래에 있어야한다.
    
    shm SYS > @logfile
    
        GROUP# MEMBER
    ---------- --------------------------------------------------
             1 /u01/app/oracle/oradata/shm/redo01.log
             3 /u01/app/oracle/oradata/shm/redo03.log
             2 /u01/app/oracle/oradata/shm/redo02.log
             1 /u01/app/oracle/oradata/shm/redo01b.log
             2 /u01/app/oracle/oradata/shm/redo02b.log
             3 /u01/app/oracle/oradata/shm/redo03b.log
    
    6 rows selected.
    ```
    
    
    1-3. shutdown immediate 로 내립니다. (반드시!!!)
    
    ```sql
    shm SYS > @si
    ```
    
    1-4. 원본 data file, control file, redo log file을 모두 /home/oracle/coldbackup3 디렉토리에 copy합니다.
    
    ```sql
    [shm:~]$ cd cold_20240305
    [shm:cold_20240305]$
    [shm:cold_20240305]$ cp /u01/app/oracle/oradata/shm/* .
    [shm:cold_20240305]$
    [shm:cold_20240305]$ ls
    control01.ctl  cuppang02.dbf  redo01b.log  redo03b.log   ts01.dbf       users01.dbf
    control02.ctl  cuppang03.dbf  redo02.log   sysaux01.dbf  ts02.dbf       users02.dbf
    control03.ctl  example01.dbf  redo02b.log  system01.dbf  ts03.dbf
    cuppang01.dbf  redo01.log     redo03.log   temp01.dbf    undotbs01.dbf
    
    -------------------------------------------------------
    cp /u01/app/oracle/oradata/shm/* .
    shm아래의 모든 데이터를 현재 디렉토리(.) 에 복사하라는 의미
    따라서 한칸 띄고 . 을 써줘야함
    ```
   
    
2. **DB를 startup으로 올리고 log switch를 3번 일으킵니다.**
    1. 월요일에 백업하고 시간이 좀 지났다는 것을 구현하기 위함
    
    ```sql
    shm SYS > startup
    ORACLE instance started.
    
    Total System Global Area  636100608 bytes
    Fixed Size                  1338392 bytes
    Variable Size             184550376 bytes
    Database Buffers          444596224 bytes
    Redo Buffers                5615616 bytes
    Database mounted.
    Database opened.
    shm SYS >
    shm SYS > @logsw -- 3번 수행
    
    System altered.
    
    shm SYS > /
    
    System altered.
    
    shm SYS > /
    
    System altered.
    
    shm SYS > alter system checkpoint;
    
    System altered.
    ```
    
    
3. **현재 시간을 확인합니다.**
    
    ```sql
    #1. 시간 format 설정해주기 
    shm SYS > alter session set nls_date_format='RRRR/MM/DD:HH24:MI:SS';
    
    Session altered.
    
    #2. 현재 시간 확인하기
    shm SYS > select sysdate from dual;
    
    SYSDATE
    -------------------
    2024/03/05:16:03:51  <== 이 시간 소중하게 간직하고 있기 
    ```
    
    
4. **scott 유저를 drop 합니다.**
    1. drop하기 전에 한 번 더 log switch 와 check point 일으키고 drop 하기
    (시간이 지났다는 것을 가정하기 위함)
        
        ```sql
        shm SYS > @logsw
        
        System altered.
        
        shm SYS > alter system checkpoint;
        
        System altered.
        ```
        
        
    2. **scott 유저를 drop 합니다.**
        
        ```sql
        shm SYS > 
        drop user scott cascade;
        
        -- purge : 휴지통에 넣지 않도록 하는 옵션
        -- purge recyclebin; : 휴지통 지우는 명령어
        ```
        
        
        
    

**— scott유저가 drop 되기 전으로 db를 불완전 복구 합니다—-**

1. **[복구 작업 시작] sys유저로 가서 db를 shutdown immediate로 내린 후 mount상태로 db를 올립니다.**
    
    ```sql
    
    shm SYS > @si
    Database closed.
    Database dismounted.
    ORACLE instance shut down.
    shm SYS >
    shm SYS > startup mount
    ORACLE instance started.
    
    Total System Global Area  636100608 bytes
    Fixed Size                  1338392 bytes
    Variable Size             184550376 bytes
    Database Buffers          444596224 bytes
    Redo Buffers                5615616 bytes
    Database mounted.
    
    ```
    
    
2. **OS에서 기존의 data files를 모두 삭제합니다. (control file과 redo log file은 그대로 둠)**
    1. 확장자가 .dbf인 파일들 모두 지우면 됨
    
    ```sql
    # 아래의 .dbf 파일들 모두 삭제할 것
    
    [shm:~]$ oradata
    [shm:shm]$
    [shm:shm]$ ls *.dbf
    cuppang01.dbf  cuppang03.dbf  sysaux01.dbf  temp01.dbf  ts02.dbf  undotbs01.dbf  users02.dbf
    cuppang02.dbf  example01.dbf  system01.dbf  ts01.dbf    ts03.dbf  users01.dbf
    [shm:shm]$
    [shm:shm]$ rm *.dbf
    [shm:shm]$
    [shm:shm]$ ls *.dbf
    ls: *.dbf: 그런 파일이나 디렉토리가 없음
    ```
    
   
3. **backup 받은 모든 data files 를 복원합니다.**
    
    ```sql
    [shm:~]$ oradata
    [shm:shm]$
    
    [shm:shm]$ cp /home/oracle/cold_20240305/*.dbf .
    [shm:shm]$
    [shm:shm]$ ls *.dbf
    cuppang01.dbf  cuppang03.dbf  sysaux01.dbf  temp01.dbf  ts02.dbf  undotbs01.dbf  users02.dbf
    cuppang02.dbf  example01.dbf  system01.dbf  ts01.dbf    ts03.dbf  users01.dbf
    ```
    
4. **user가 drop 되기 전으로 불완전 복구를 수행합니다.**
    - 아카이브 로그 파일 자동 적용 시키는 옵션 켜놓기
        
        ```sql
        shm SYS >
        
        set autorecovery on
        
        -----------------------------
        복구할 때 아카이브 로그 파일을 적용하는지를 물어보지 않고
        자동으로 아카이브 로그 파일을 적용하도록 자동으로 수행해주는 명령
        ```
        
    - 불완전 복구 수행
        
        ```sql
        #1. 시간 format 설정해주기 
        shm SYS > 
        alter session set nls_date_format='RRRR/MM/DD:HH24:MI:SS';
        
        Session altered.
        
        #2. 복구 수행해주기(앞에서 확인 했던 시간을 적어주면 됨)
        shm SYS >
        recover database until time '2024/03/05:16:03:51';
        
        ORA-00279: change 98178299 generated at 03/05/2024 16:00:54 needed for thread 1
        ORA-00289: suggestion :
        /u01/app/oracle/flash_recovery_area/SHM/archivelog/2024_03_05/o1_mf_1_3_lyfjwjtm
        _.arc
        ORA-00280: change 98178299 for thread 1 is in sequence #3
        
        ORA-00279: change 98178530 generated at 03/05/2024 16:02:40 needed for thread 1
        ORA-00289: suggestion :
        /u01/app/oracle/flash_recovery_area/SHM/archivelog/2024_03_05/o1_mf_1_4_lyfjwlcj
        _.arc
        ORA-00280: change 98178530 for thread 1 is in sequence #4
        
        Log applied.
        Media recovery complete.
        ```
        
        
        
    
5. **resetlogs 옵션을 사용하여 db를 올립니다.**
    
    ```sql
    alter database open resetlogs;
    ```
    
    
    

1. scott 유저의 테이블들이 보이는지 확인하기.
    
    ```sql
    shm SYS > connect scott/tiger
    Connected.
    shm SCOTT >
    shm SCOTT > select table_name
      2     from user_tables;
    
    TABLE_NAME
    ------------------------------
    EMP400
    DEPT_CUPPANG
    EMP_CUPPANG
    EMP022
    DEPT03
    EMP05
    SALGRADE
    BONUS
    EMP
    DEPT
    
    10 rows selected.
    
    shm SCOTT > select count(*) from emp400;
    
      COUNT(*)
    ----------
            14
    ```
    