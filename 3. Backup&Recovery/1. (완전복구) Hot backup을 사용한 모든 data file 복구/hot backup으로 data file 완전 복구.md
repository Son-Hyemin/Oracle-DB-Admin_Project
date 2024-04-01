# (완전복구)Hot backup을 사용하여 모든 data file 복구

소유자: 쏘니

<aside>
💡 순서

1. hot_20240304_2 디렉토리 생성
2. data file전체 hot backup을 수행하고 
3. scott 유저에서 dept_cuppang 테이블을 ts01 테이블 스페이스에 생성
4. dept_cuppang 테이블에 데이터를 입력하고, log switch를 일으키는 작업 3번 수행
5. shutdown abort로 DB 내리기
6. 모든 data file들을 전부 삭제한 후 startup  한 이후에 복원, 복구
</aside>

---

1. **shm DB로 접속합니다.**
    
    ```sql
    $
    [orcl:~]$ . oraenv
    ORACLE_SID = [orcl] ? shm
    The Oracle base for ORACLE_HOME=/u01/app/oracle/product/11.2.0/dbhome_1 is /u01/app/oracle
    [shm:~]$
    
    ```
    
    ![Untitled]((%E1%84%8B%E1%85%AA%E1%86%AB%E1%84%8C%E1%85%A5%E1%86%AB%E1%84%87%E1%85%A9%E1%86%A8%E1%84%80%E1%85%AE)Hot%20backup%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%84%8B%E1%85%A7%20%E1%84%86%E1%85%A9%E1%84%83%E1%85%B3%E1%86%AB%20data%20fi%20991415a37b6c49e89f1d8532fe97d594/Untitled.png)
    
2. **hot backup을 수행합니다. - 쉘로 만들어서 수행**
    1. hot backup을 수행하기 위해 begin backup script 추출하기
    2. TEMPORARY TABLESPACE는 begin backup할 필요가 없음. 
    왜? 혹시라도 장애가나서 삭제되면, startup할 때 알아서 복구됨. 때문에 우리가 백업해줄 필요가 없음 
        
        ```sql
        select 'alter tablespace ' || tablespace_name || ' begin backup;'
        	from dba_tablespaces
        	where tablespace_name != 'TEMP';
        ```
        
        ```sql
        'ALTERTABLESPACE'||TABLESPACE_NAME||'BEGINBACKUP;'
        -------------------------------------------------------------
        alter tablespace SYSTEM begin backup;
        alter tablespace SYSAUX begin backup;
        alter tablespace UNDOTBS1 begin backup;
        alter tablespace USERS begin backup;
        alter tablespace EXAMPLE begin backup;
        alter tablespace TS01 begin backup;
        alter tablespace TS02 begin backup;
        alter tablespace TS03 begin backup;
        alter tablespace CUPPANG01 begin backup;
        alter tablespace CUPPANG02 begin backup;
        alter tablespace CUPPANG03 begin backup;
        
        11 rows selected.
        ```
        
        ```sql
        # 스크립트 수행하기
        
        alter tablespace SYSTEM begin backup;
        alter tablespace SYSAUX begin backup;
        alter tablespace UNDOTBS1 begin backup;
        alter tablespace USERS begin backup;
        alter tablespace EXAMPLE begin backup;
        alter tablespace TS01 begin backup;
        alter tablespace TS02 begin backup;
        alter tablespace TS03 begin backup;
        alter tablespace CUPPANG01 begin backup;
        alter tablespace CUPPANG02 begin backup;
        alter tablespace CUPPANG03 begin backup;
        
        ```
        
        ![Untitled]((%E1%84%8B%E1%85%AA%E1%86%AB%E1%84%8C%E1%85%A5%E1%86%AB%E1%84%87%E1%85%A9%E1%86%A8%E1%84%80%E1%85%AE)Hot%20backup%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%84%8B%E1%85%A7%20%E1%84%86%E1%85%A9%E1%84%83%E1%85%B3%E1%86%AB%20data%20fi%20991415a37b6c49e89f1d8532fe97d594/Untitled%201.png)
        
    3. backup mode에 있는지 확인하기
        
        ```sql
        shm SYS > 
        
        select * from v$backup;
        
             FILE# STATUS                CHANGE# TIME
        ---------- ------------------ ---------- ---------
                 1 ACTIVE               98016007 04-MAR-24
                 2 ACTIVE               98016013 04-MAR-24
                 3 ACTIVE               98016019 04-MAR-24
                 4 ACTIVE               98016025 04-MAR-24
                 5 ACTIVE               98016031 04-MAR-24
                 6 ACTIVE               98016025 04-MAR-24
                 7 ACTIVE               98016038 04-MAR-24
                 8 ACTIVE               98016044 04-MAR-24
                 9 ACTIVE               98016051 04-MAR-24
                10 ACTIVE               98016057 04-MAR-24
                11 ACTIVE               98016064 04-MAR-24
        
             FILE# STATUS                CHANGE# TIME
        ---------- ------------------ ---------- ---------
                12 ACTIVE               98016070 04-MAR-24
        
        12 rows selected.
        
        ```
        
        ![Untitled]((%E1%84%8B%E1%85%AA%E1%86%AB%E1%84%8C%E1%85%A5%E1%86%AB%E1%84%87%E1%85%A9%E1%86%A8%E1%84%80%E1%85%AE)Hot%20backup%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%84%8B%E1%85%A7%20%E1%84%86%E1%85%A9%E1%84%83%E1%85%B3%E1%86%AB%20data%20fi%20991415a37b6c49e89f1d8532fe97d594/Untitled%202.png)
        
        하나라도 not active가 있으면 안됨
        
    4. OS에서 `/home/oracle/hot_20240304_2` 디렉토리를 생성한 후 cp 스크립트 추출하여, data file을 copy 한다.
        
        ```sql
        # 1. 디렉토리 생성하기
        [shm:~]$ mkdir /home/oracle/hot_20240304_2
        
        [shm:~]$ ls -ld hot_20240304_2
        drwxr-xr-x 2 oracle oinstall 4096  3월  4 10:59 hot_20240304_2
        
        -------------------------------------------------------------
        
        # 2. cp 스크립트 생성하기 ( /home 앞에 반드시 한 칸 띄어야함)
        select 'cp ' || file_name || ' /home/oracle/hot_20240304_2/'
        	from dba_data_files;
        
        'CP'||FILE_NAME||'/HOME/ORACLE/HOT_20240304_2/'
        --------------------------------------------------------------------------------
        cp /u01/app/oracle/oradata/shm/ts01.dbf /home/oracle/hot_20240304_2/
        cp /u01/app/oracle/oradata/shm/users02.dbf /home/oracle/hot_20240304_2/
        cp /u01/app/oracle/oradata/shm/example01.dbf /home/oracle/hot_20240304_2/
        cp /u01/app/oracle/oradata/shm/users01.dbf /home/oracle/hot_20240304_2/
        cp /u01/app/oracle/oradata/shm/undotbs01.dbf /home/oracle/hot_20240304_2/
        cp /u01/app/oracle/oradata/shm/sysaux01.dbf /home/oracle/hot_20240304_2/
        cp /u01/app/oracle/oradata/shm/system01.dbf /home/oracle/hot_20240304_2/
        cp /u01/app/oracle/oradata/shm/ts02.dbf /home/oracle/hot_20240304_2/
        cp /u01/app/oracle/oradata/shm/ts03.dbf /home/oracle/hot_20240304_2/
        cp /u01/app/oracle/oradata/shm/cuppang01.dbf /home/oracle/hot_20240304_2/
        cp /u01/app/oracle/oradata/shm/cuppang02.dbf /home/oracle/hot_20240304_2/
        cp /u01/app/oracle/oradata/shm/cuppang03.dbf /home/oracle/hot_20240304_2/
        
        12 rows selected.
        
        # 3. 스크립트 돌리기 
        
        cp /u01/app/oracle/oradata/shm/ts01.dbf /home/oracle/hot_20240304_2/
        cp /u01/app/oracle/oradata/shm/users02.dbf /home/oracle/hot_20240304_2/
        cp /u01/app/oracle/oradata/shm/example01.dbf /home/oracle/hot_20240304_2/
        cp /u01/app/oracle/oradata/shm/users01.dbf /home/oracle/hot_20240304_2/
        cp /u01/app/oracle/oradata/shm/undotbs01.dbf /home/oracle/hot_20240304_2/
        cp /u01/app/oracle/oradata/shm/sysaux01.dbf /home/oracle/hot_20240304_2/
        cp /u01/app/oracle/oradata/shm/system01.dbf /home/oracle/hot_20240304_2/
        cp /u01/app/oracle/oradata/shm/ts02.dbf /home/oracle/hot_20240304_2/
        cp /u01/app/oracle/oradata/shm/ts03.dbf /home/oracle/hot_20240304_2/
        cp /u01/app/oracle/oradata/shm/cuppang01.dbf /home/oracle/hot_20240304_2/
        cp /u01/app/oracle/oradata/shm/cuppang02.dbf /home/oracle/hot_20240304_2/
        cp /u01/app/oracle/oradata/shm/cuppang03.dbf /home/oracle/hot_20240304_2/
        
        # 4. 잘 카피 되었는지 확인하기
        
        [shm:~]$ cd hot_20240304_2
        [shm:hot_20240304_2]$
        [shm:hot_20240304_2]$ ls -l
        합계 1763168
        -rw-r----- 1 oracle oinstall   5251072  3월  4 11:00 cuppang01.dbf
        -rw-r----- 1 oracle oinstall   5251072  3월  4 11:00 cuppang02.dbf
        -rw-r----- 1 oracle oinstall   5251072  3월  4 11:00 cuppang03.dbf
        -rw-r----- 1 oracle oinstall 104865792  3월  4 11:00 example01.dbf
        -rw-r----- 1 oracle oinstall 534781952  3월  4 11:00 sysaux01.dbf
        -rw-r----- 1 oracle oinstall 713039872  3월  4 11:00 system01.dbf
        -rw-r----- 1 oracle oinstall  10493952  3월  4 11:00 ts01.dbf
        -rw-r----- 1 oracle oinstall  10493952  3월  4 11:00 ts02.dbf
        -rw-r----- 1 oracle oinstall   5251072  3월  4 11:00 ts03.dbf
        -rw-r----- 1 oracle oinstall  89137152  3월  4 11:00 undotbs01.dbf
        -rw-r----- 1 oracle oinstall   5251072  3월  4 11:00 users01.dbf
        -rw-r----- 1 oracle oinstall 314580992  3월  4 11:00 users02.dbf
        
        shm SYS > select count(*)
          2     from dba_data_files;
        
          COUNT(*)
        ----------
                12
        
        ```
        
        ![Untitled]((%E1%84%8B%E1%85%AA%E1%86%AB%E1%84%8C%E1%85%A5%E1%86%AB%E1%84%87%E1%85%A9%E1%86%A8%E1%84%80%E1%85%AE)Hot%20backup%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%84%8B%E1%85%A7%20%E1%84%86%E1%85%A9%E1%84%83%E1%85%B3%E1%86%AB%20data%20fi%20991415a37b6c49e89f1d8532fe97d594/Untitled%203.png)
        
        ![Untitled]((%E1%84%8B%E1%85%AA%E1%86%AB%E1%84%8C%E1%85%A5%E1%86%AB%E1%84%87%E1%85%A9%E1%86%A8%E1%84%80%E1%85%AE)Hot%20backup%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%84%8B%E1%85%A7%20%E1%84%86%E1%85%A9%E1%84%83%E1%85%B3%E1%86%AB%20data%20fi%20991415a37b6c49e89f1d8532fe97d594/Untitled%204.png)
        
        공간 없으면 dbca로 지워버리기 (orcl도 지워버리면 됨.,무서움)
        
    5. end_backup을 수행하는 스크립트 추출하기
        
        ```sql
        # 1. script 추출하기 
        select 'alter tablespace ' || tablespace_name || ' end backup;'
        	from dba_tablespaces
        	where tablespace_name != 'TEMP';
        
        'ALTERTABLESPACE'||TABLESPACE_NAME||'ENDBACKUP;'
        -----------------------------------------------------------
        alter tablespace SYSTEM end backup;
        alter tablespace SYSAUX end backup;
        alter tablespace UNDOTBS1 end backup;
        alter tablespace USERS end backup;
        alter tablespace EXAMPLE end backup;
        alter tablespace TS01 end backup;
        alter tablespace TS02 end backup;
        alter tablespace TS03 end backup;
        alter tablespace CUPPANG01 end backup;
        alter tablespace CUPPANG02 end backup;
        alter tablespace CUPPANG03 end backup;
        
        11 rows selected.
        
        # 2. 스크립트 수행하여 end_backup 하기
        
        ```
        
        ![Untitled]((%E1%84%8B%E1%85%AA%E1%86%AB%E1%84%8C%E1%85%A5%E1%86%AB%E1%84%87%E1%85%A9%E1%86%A8%E1%84%80%E1%85%AE)Hot%20backup%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%84%8B%E1%85%A7%20%E1%84%86%E1%85%A9%E1%84%83%E1%85%B3%E1%86%AB%20data%20fi%20991415a37b6c49e89f1d8532fe97d594/Untitled%205.png)
        
        ![Untitled]((%E1%84%8B%E1%85%AA%E1%86%AB%E1%84%8C%E1%85%A5%E1%86%AB%E1%84%87%E1%85%A9%E1%86%A8%E1%84%80%E1%85%AE)Hot%20backup%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%84%8B%E1%85%A7%20%E1%84%86%E1%85%A9%E1%84%83%E1%85%B3%E1%86%AB%20data%20fi%20991415a37b6c49e89f1d8532fe97d594/Untitled%206.png)
        
    6. backup mode → 일반 모드로 변경되었는지 확인하기
        
        ```sql
        select * from v$backup;
        ```
        
        ![Untitled]((%E1%84%8B%E1%85%AA%E1%86%AB%E1%84%8C%E1%85%A5%E1%86%AB%E1%84%87%E1%85%A9%E1%86%A8%E1%84%80%E1%85%AE)Hot%20backup%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%84%8B%E1%85%A7%20%E1%84%86%E1%85%A9%E1%84%83%E1%85%B3%E1%86%AB%20data%20fi%20991415a37b6c49e89f1d8532fe97d594/Untitled%207.png)
        
    
3. **scott 유저에서 테이블을 생성합니다.**
    
    ```sql
    shm SYS >
    
    connect scott/tiger
    
    shm SCOTT >
    
    create table dept_cuppang
    tablespace ts01
    as
    	select * from dept;
    
    shm SCOTT > select count(*) from dept_cuppang;
    
      COUNT(*)
    ----------
    	       4
    
    ```
    
    ![Untitled]((%E1%84%8B%E1%85%AA%E1%86%AB%E1%84%8C%E1%85%A5%E1%86%AB%E1%84%87%E1%85%A9%E1%86%A8%E1%84%80%E1%85%AE)Hot%20backup%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%84%8B%E1%85%A7%20%E1%84%86%E1%85%A9%E1%84%83%E1%85%B3%E1%86%AB%20data%20fi%20991415a37b6c49e89f1d8532fe97d594/Untitled%208.png)
    
4. **데이터를 입력하고 log switch를 여러번( 3번 ) 일으킵니다.**
    1. scott에 권한 없으면 sys에서 dba 권한 주고 수행하기 
    2. 현업 처럼 수행해보기 위해서 백업 이후에 여러 행동을 했다는 것을 인식시켜주기 위함
    
    ```sql
    shm SCOTT > 
    
    insert into dept_cuppang
    	select * from dept_cuppang;
    
    alter system swith logfile; -- @logsw
    ```
    
    ![Untitled]((%E1%84%8B%E1%85%AA%E1%86%AB%E1%84%8C%E1%85%A5%E1%86%AB%E1%84%87%E1%85%A9%E1%86%A8%E1%84%80%E1%85%AE)Hot%20backup%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%84%8B%E1%85%A7%20%E1%84%86%E1%85%A9%E1%84%83%E1%85%B3%E1%86%AB%20data%20fi%20991415a37b6c49e89f1d8532fe97d594/Untitled%209.png)
    
5. **sys유저에서 DB를 내리고, 모든 data file들을 전부 삭제합니다.**
    1. 데이터이 파일이 있는 곳으로 이동해야함. 단 하나라도 다른 곳에 있으면 안됨! 하나라도 /home/oracle/ 아래에 있으면 그것도 백업 시켜줘야함. 모두 이니셜 밑에 있어야함.
        
        ```sql
        shm SCOTT > 
        connect / as sysdba
        
        # data file위치 확인하기
        shm SYS >
         select file_name from dba_data_files; --  @datafile
        
        FILE_NAME
        --------------------------------------------------------------------------------
        /u01/app/oracle/oradata/shm/ts01.dbf
        /u01/app/oracle/oradata/shm/users02.dbf
        /u01/app/oracle/oradata/shm/example01.dbf
        /u01/app/oracle/oradata/shm/users01.dbf
        /u01/app/oracle/oradata/shm/undotbs01.dbf
        /u01/app/oracle/oradata/shm/sysaux01.dbf
        /u01/app/oracle/oradata/shm/system01.dbf
        /u01/app/oracle/oradata/shm/ts02.dbf
        /u01/app/oracle/oradata/shm/ts03.dbf
        /u01/app/oracle/oradata/shm/cuppang01.dbf
        /u01/app/oracle/oradata/shm/cuppang02.dbf
        
        FILE_NAME
        --------------------------------------------------------------------------------
        /u01/app/oracle/oradata/shm/cuppang03.dbf
        
        12 rows selected.
        
        ```
        
        ![Untitled]((%E1%84%8B%E1%85%AA%E1%86%AB%E1%84%8C%E1%85%A5%E1%86%AB%E1%84%87%E1%85%A9%E1%86%A8%E1%84%80%E1%85%AE)Hot%20backup%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%84%8B%E1%85%A7%20%E1%84%86%E1%85%A9%E1%84%83%E1%85%B3%E1%86%AB%20data%20fi%20991415a37b6c49e89f1d8532fe97d594/Untitled%2010.png)
        
    2. 해당 위치로 이동해서 *.dbf 로 끝나는 data file들 삭제하기 
        1. 단, log file과 ctl file은 삭제되면 안됨! 삭제 후 남아있는지 확인하기 
        
        ```sql
        #1. db를 내리기 -- 삭제 후 내려도 되긴 함
        shm SYS > shutdown abort
        
        #2. datafile이 있는 위치로 이동 후 rm으로 *.dbf로 끝나는 datafile들 삭제하기
        
        [shm:~]$ cd /u01/app/oracle/oradata/shm/
        [shm:shm]$
        [shm:shm]$ ls
        control01.ctl  cuppang03.dbf  redo02.log    system01.dbf  ts02.dbf   users01.dbf
        cuppang01.dbf  example01.dbf  redo03.log    temp01.dbf    ts03.dbf   users02.dbf
        cuppang02.dbf  redo01.log     sysaux01.dbf  ts01.dbf      undotbs01.dbf
        
        [shm:shm]$ rm *.dbf
        [shm:shm]$
        [shm:shm]$ ls
        control01.ctl  redo01.log  redo02.log  redo03.log
        ```
        
        ![Untitled]((%E1%84%8B%E1%85%AA%E1%86%AB%E1%84%8C%E1%85%A5%E1%86%AB%E1%84%87%E1%85%A9%E1%86%A8%E1%84%80%E1%85%AE)Hot%20backup%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%84%8B%E1%85%A7%20%E1%84%86%E1%85%A9%E1%84%83%E1%85%B3%E1%86%AB%20data%20fi%20991415a37b6c49e89f1d8532fe97d594/Untitled%2011.png)
        
    
6. **DB를 STARTUP 합니다.**
    1. 데이터 파일이 없기 때문에 mount까지만 올라가고 올라가지 않음
    
    ```sql
    shm SYS > startup
    ORACLE instance started.
    
    Total System Global Area  636100608 bytes
    Fixed Size                  1338392 bytes
    Variable Size             184550376 bytes
    Database Buffers          444596224 bytes
    Redo Buffers                5615616 bytes
    Database mounted.
    ORA-01157: cannot identify/lock data file 1 - see DBWR trace file
    ORA-01110: data file 1: '/u01/app/oracle/oradata/shm/system01.dbf'
    ```
    
    ![Untitled]((%E1%84%8B%E1%85%AA%E1%86%AB%E1%84%8C%E1%85%A5%E1%86%AB%E1%84%87%E1%85%A9%E1%86%A8%E1%84%80%E1%85%AE)Hot%20backup%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%84%8B%E1%85%A7%20%E1%84%86%E1%85%A9%E1%84%83%E1%85%B3%E1%86%AB%20data%20fi%20991415a37b6c49e89f1d8532fe97d594/Untitled%2012.png)
    
    ⇒ data file을 전부 삭제했는데  system01만 뜸! 그래서 복구해야할 파일 list를 보여달라고 오라클에게 아래와 같이 요청해야함
    
    ```sql
    # 방법 1 : 번호만 나와서 상세하게 알아보기 어렵다.
    shm SYS >
    select * from v$recover_file;
    
         FILE# ONLINE  ONLINE_ ERROR                   CHANGE# TIME
    ---------- ------- ------- -------------------- ---------- ---------
             1 ONLINE  ONLINE  FILE NOT FOUND                0
             2 ONLINE  ONLINE  FILE NOT FOUND                0
             3 ONLINE  ONLINE  FILE NOT FOUND                0
             4 ONLINE  ONLINE  FILE NOT FOUND                0
             5 ONLINE  ONLINE  FILE NOT FOUND                0
             6 ONLINE  ONLINE  FILE NOT FOUND                0
             7 ONLINE  ONLINE  FILE NOT FOUND                0
             8 ONLINE  ONLINE  FILE NOT FOUND                0
             9 ONLINE  ONLINE  FILE NOT FOUND                0
            10 ONLINE  ONLINE  FILE NOT FOUND                0
            11 ONLINE  ONLINE  FILE NOT FOUND                0
    
         FILE# ONLINE  ONLINE_ ERROR                   CHANGE# TIME
    ---------- ------- ------- -------------------- ---------- ---------
            12 ONLINE  ONLINE  FILE NOT FOUND                0
    
    # 방법 2 : @need_recovery 로 저장하기 -- 앞으로 백업 복구 시 많이 쓸 것 
    shm SYS > 
    
    col name for a50
    
    select r.file#, d.name
    	from v$recover_file r, v$datafile d
    	where r.file# = d.file#
    	and error='FILE NOT FOUND';
    
         FILE# NAME
    ---------- --------------------------------------------------
             1 /u01/app/oracle/oradata/shm/system01.dbf
             2 /u01/app/oracle/oradata/shm/sysaux01.dbf
             3 /u01/app/oracle/oradata/shm/undotbs01.dbf
             4 /u01/app/oracle/oradata/shm/users01.dbf
             5 /u01/app/oracle/oradata/shm/example01.dbf
             6 /u01/app/oracle/oradata/shm/users02.dbf
             7 /u01/app/oracle/oradata/shm/ts01.dbf
             8 /u01/app/oracle/oradata/shm/ts02.dbf
             9 /u01/app/oracle/oradata/shm/ts03.dbf
            10 /u01/app/oracle/oradata/shm/cuppang01.dbf
            11 /u01/app/oracle/oradata/shm/cuppang02.dbf
    
         FILE# NAME
    ---------- --------------------------------------------------
            12 /u01/app/oracle/oradata/shm/cuppang03.dbf
    
    ```
    
    ![Untitled]((%E1%84%8B%E1%85%AA%E1%86%AB%E1%84%8C%E1%85%A5%E1%86%AB%E1%84%87%E1%85%A9%E1%86%A8%E1%84%80%E1%85%AE)Hot%20backup%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%84%8B%E1%85%A7%20%E1%84%86%E1%85%A9%E1%84%83%E1%85%B3%E1%86%AB%20data%20fi%20991415a37b6c49e89f1d8532fe97d594/Untitled%2013.png)
    
    ![Untitled]((%E1%84%8B%E1%85%AA%E1%86%AB%E1%84%8C%E1%85%A5%E1%86%AB%E1%84%87%E1%85%A9%E1%86%A8%E1%84%80%E1%85%AE)Hot%20backup%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%84%8B%E1%85%A7%20%E1%84%86%E1%85%A9%E1%84%83%E1%85%B3%E1%86%AB%20data%20fi%20991415a37b6c49e89f1d8532fe97d594/Untitled%2014.png)
    
    ![Untitled]((%E1%84%8B%E1%85%AA%E1%86%AB%E1%84%8C%E1%85%A5%E1%86%AB%E1%84%87%E1%85%A9%E1%86%A8%E1%84%80%E1%85%AE)Hot%20backup%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%84%8B%E1%85%A7%20%E1%84%86%E1%85%A9%E1%84%83%E1%85%B3%E1%86%AB%20data%20fi%20991415a37b6c49e89f1d8532fe97d594/Untitled%2015.png)
    
7. **백업 받은 파일들을 복원합니다.**
    1. 백업 디렉토리로 가서 파일 복원하기 
    
    ```sql
    [shm:~]$ cd hot_20240304_2
    [shm:hot_20240304_2]$
    [shm:hot_20240304_2]$ ls
    cuppang01.dbf  cuppang03.dbf  sysaux01.dbf  ts01.dbf  ts03.dbf       users01.dbf
    cuppang02.dbf  example01.dbf  system01.dbf  ts02.dbf  undotbs01.dbf  users02.dbf
    [shm:hot_20240304_2]$
    [shm:hot_20240304_2]$ cp *.dbf /u01/app/oracle/oradata/shm/
    [shm:hot_20240304_2]$
    [shm:hot_20240304_2]$ cd /u01/app/oracle/oradata/shm/
    [shm:shm]$
    [shm:shm]$ ls *.dbf
    cuppang01.dbf  cuppang03.dbf  sysaux01.dbf  ts01.dbf  ts03.dbf       users01.dbf
    cuppang02.dbf  example01.dbf  system01.dbf  ts02.dbf  undotbs01.dbf  users02.dbf
    ```
    
    ![Untitled]((%E1%84%8B%E1%85%AA%E1%86%AB%E1%84%8C%E1%85%A5%E1%86%AB%E1%84%87%E1%85%A9%E1%86%A8%E1%84%80%E1%85%AE)Hot%20backup%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%84%8B%E1%85%A7%20%E1%84%86%E1%85%A9%E1%84%83%E1%85%B3%E1%86%AB%20data%20fi%20991415a37b6c49e89f1d8532fe97d594/Untitled%2016.png)
    
8. **복원한 파일에 로그 파일을 적용해서 완전 복구를 합니다.**
    1. specify log 를 어떻게 할지 물어보는데, auto라고 적으면 알아서 아카이브 로그 파일을 복원한 파일에 적용하여 최신 파일로 만들어줌 
    
    ```sql
    shm SYS >
    recover database;
    
    Specify log: {<RET>=suggested | filename | AUTO | CANCEL}
    _____   
    <--- auto라고 적기! 이렇게 적으면 알아서 아카이브 로그 파일을 복원한 파일에 적용합니다.
    
    Log applied.
    Media recovery complete.
    ```
    
    ![Untitled]((%E1%84%8B%E1%85%AA%E1%86%AB%E1%84%8C%E1%85%A5%E1%86%AB%E1%84%87%E1%85%A9%E1%86%A8%E1%84%80%E1%85%AE)Hot%20backup%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%84%8B%E1%85%A7%20%E1%84%86%E1%85%A9%E1%84%83%E1%85%B3%E1%86%AB%20data%20fi%20991415a37b6c49e89f1d8532fe97d594/Untitled%2017.png)
    
9. db를 open합니다.
    
    ```sql
    shm SYS >
    alter database open;
    ```
    
    ![Untitled]((%E1%84%8B%E1%85%AA%E1%86%AB%E1%84%8C%E1%85%A5%E1%86%AB%E1%84%87%E1%85%A9%E1%86%A8%E1%84%80%E1%85%AE)Hot%20backup%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%84%8B%E1%85%A7%20%E1%84%86%E1%85%A9%E1%84%83%E1%85%B3%E1%86%AB%20data%20fi%20991415a37b6c49e89f1d8532fe97d594/Untitled%2018.png)
    
10. **scott 유저에서 생성한 테이블을 select 합니다.**
    
    ```sql
    shm SYS > connect scott/tiger
    Connected.
    
    shm SCOTT > select count(*) from dept_cuppang;
    
      COUNT(*)
    ----------
            32
    ```
    
    ![Untitled]((%E1%84%8B%E1%85%AA%E1%86%AB%E1%84%8C%E1%85%A5%E1%86%AB%E1%84%87%E1%85%A9%E1%86%A8%E1%84%80%E1%85%AE)Hot%20backup%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%84%8B%E1%85%A7%20%E1%84%86%E1%85%A9%E1%84%83%E1%85%B3%E1%86%AB%20data%20fi%20991415a37b6c49e89f1d8532fe97d594/Untitled%2019.png)
    

### 최종 결과물

![Untitled]((%E1%84%8B%E1%85%AA%E1%86%AB%E1%84%8C%E1%85%A5%E1%86%AB%E1%84%87%E1%85%A9%E1%86%A8%E1%84%80%E1%85%AE)Hot%20backup%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%84%8B%E1%85%A7%20%E1%84%86%E1%85%A9%E1%84%83%E1%85%B3%E1%86%AB%20data%20fi%20991415a37b6c49e89f1d8532fe97d594/Untitled%2020.png)