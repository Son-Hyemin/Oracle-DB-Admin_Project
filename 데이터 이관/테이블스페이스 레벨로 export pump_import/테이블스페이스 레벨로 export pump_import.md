# 3. 테이블 스페이스 레벨로 EXPORT PUMP / IMPORT

소유자: 쏘니

<aside>
💡 순서

1. shm2 DB에 ts30000 테이블 스페이스를 생성
2. scott 유져에서 emp30000 테이블을 생성한
3. ts30000 테이블 스페이스를 export 하고 PROD DB에 import 
4. 잘 import 되었는지 확인하기 
</aside>

1. **PROD DB와 shm2 DB가 올라왔는지 확인(올라오지 않았을 경우 startup으로 올리기)**
    
    ```sql
    [shm2:~]$ ps -ef | grep pmon
    oracle   10833     1  0 10:07 ?        00:00:00 ora_pmon_PROD
    oracle   11277     1  0 10:08 ?        00:00:00 ora_pmon_shm2
    oracle   11500  8878  0 10:09 pts/2    00:00:00 grep pmon
    ```
    
    ![Untitled](3%20%E1%84%90%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%87%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%B3%E1%84%91%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%20%E1%84%85%E1%85%A6%E1%84%87%E1%85%A6%E1%86%AF%E1%84%85%E1%85%A9%20EXPORT%20PUMP%20IMPORT%20958c03fefdaa40d7860846f8aa947cbf/Untitled.png)
    
2. **shm2 DB에서 ts30000 이라는 테이블스페이스 생성**
    
    ```sql
    shm2 SYS >
    create tablespace ts30000
    	datafile '/u01/app/oracle/oradata/shm2/disk1/ts30000.dbf' size 10m;
    ```
    
    ![Untitled](3%20%E1%84%90%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%87%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%B3%E1%84%91%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%20%E1%84%85%E1%85%A6%E1%84%87%E1%85%A6%E1%86%AF%E1%84%85%E1%85%A9%20EXPORT%20PUMP%20IMPORT%20958c03fefdaa40d7860846f8aa947cbf/Untitled%201.png)
    
3.  **shm2 DB의 SOCTT유저로 접속해서 ts30000 테이블 스페이스에 emp30000테이블 생성**
    1. 생성하기
    
    ```sql
    shm2 SYS >
    connect scott/tiger
    
    shm2 SCOTT >
    create table emp30000
    ( empno number(10),
    	ename varchar2(10),
    	sal number(10) )
    tablespace ts30000;
    ```
    
    ![Untitled](3%20%E1%84%90%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%87%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%B3%E1%84%91%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%20%E1%84%85%E1%85%A6%E1%84%87%E1%85%A6%E1%86%AF%E1%84%85%E1%85%A9%20EXPORT%20PUMP%20IMPORT%20958c03fefdaa40d7860846f8aa947cbf/Untitled%202.png)
    
    b. 데이터 insert
    
    ```sql
    shm2 SCOTT >
    insert into emp30000
    	select empno, ename, sal
    		from emp;
    ```
    
    ![Untitled](3%20%E1%84%90%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%87%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%B3%E1%84%91%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%20%E1%84%85%E1%85%A6%E1%84%87%E1%85%A6%E1%86%AF%E1%84%85%E1%85%A9%20EXPORT%20PUMP%20IMPORT%20958c03fefdaa40d7860846f8aa947cbf/Untitled%203.png)
    
4. **ts30000 테이블 스페이스를 read only로 변경합니다.**
    1. read only로 변경하기
    
    ```sql
    shm2 SCOTT >
    connect / as sysdba
    
    shm2 SYS >
    alter tablespace ts30000 read only;
    ```
    
    ![Untitled](3%20%E1%84%90%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%87%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%B3%E1%84%91%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%20%E1%84%85%E1%85%A6%E1%84%87%E1%85%A6%E1%86%AF%E1%84%85%E1%85%A9%20EXPORT%20PUMP%20IMPORT%20958c03fefdaa40d7860846f8aa947cbf/Untitled%204.png)
    
    b. 잘 변경되었는지 확인하기
    
    ```sql
    shm2 SYS >
    
    select t.name, d.enabled
    	from v$tablespace t, v$datafile d
    	where t.ts# = d.ts#;
    ```
    
    ![Untitled](3%20%E1%84%90%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%87%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%B3%E1%84%91%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%20%E1%84%85%E1%85%A6%E1%84%87%E1%85%A6%E1%86%AF%E1%84%85%E1%85%A9%20EXPORT%20PUMP%20IMPORT%20958c03fefdaa40d7860846f8aa947cbf/Untitled%205.png)
    
5. **ts30000 테이블 스페이스를 export 합니다.**
    
    ```sql
    $ expdp directory=shm2_dir dumpfile=ts30000.dmp transport_tablespace=y tablespaces=ts30000
    
    ------------- 유저이름과 패스워드 입력하기 --------------
    
    Username : sys as sysdba
    Password : oracle
    -------------------------------------------------------
    
    ================================================================
    directory=datapump_dir : 덤프파일이 생성될 위치
    transport_tablespace=y : 이걸 입력해줘야 테이블스페이스로 이관 가능
    ```
    
    ```bash
    
    [shm2:~]$ expdp directory=shm2_dir dumpfile=ts30000.dmp transport_tablespace=y tablespaces=ts30000
    
    Export: Release 11.2.0.1.0 - Production on Mon Feb 26 10:59:00 2024
    
    Copyright (c) 1982, 2009, Oracle and/or its affiliates.  All rights reserved.
    
    Username: sys as sysdba
    Password:
    
    Connected to: Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production
    With the Partitioning, OLAP, Data Mining and Real Application Testing options
    Legacy Mode Active due to the following parameters:
    Legacy Mode Parameter: "transport_tablespace=TRUE" Location: Command Line, Replaced with: "transport_tablespaces=ts30000"
    Legacy Mode has set reuse_dumpfiles=true parameter.
    Starting "SYS"."SYS_EXPORT_TRANSPORTABLE_01":  sys/******** AS SYSDBA directory=shm2_dir dumpfile=ts30000.dmp tablespaces=ts30000 reuse_dumpfiles=true
    Processing object type TRANSPORTABLE_EXPORT/PLUGTS_BLK
    Processing object type TRANSPORTABLE_EXPORT/TABLE
    Processing object type TRANSPORTABLE_EXPORT/POST_INSTANCE/PLUGTS_BLK
    Master table "SYS"."SYS_EXPORT_TRANSPORTABLE_01" successfully loaded/unloaded
    ******************************************************************************
    Dump file set for SYS.SYS_EXPORT_TRANSPORTABLE_01 is:
      /home/oracle/pump_shm2/ts30000.dmp
    ******************************************************************************
    Datafiles required for transportable tablespace TS30000:
      /u01/app/oracle/oradata/shm2/disk1/ts30000.dbf
    Job "SYS"."SYS_EXPORT_TRANSPORTABLE_01" successfully completed at 10:59:18
    
    ```
    
    ![Untitled](3%20%E1%84%90%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%87%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%B3%E1%84%91%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%20%E1%84%85%E1%85%A6%E1%84%87%E1%85%A6%E1%86%AF%E1%84%85%E1%85%A9%20EXPORT%20PUMP%20IMPORT%20958c03fefdaa40d7860846f8aa947cbf/Untitled%206.png)
    
    b. 잘 생성되었는지 확인하기
    
    ```bash
    
    [shm2:~]$ cd pump_shm2
    [shm2:pump_shm2]$
    
    [shm2:pump_shm2]$ ls
    dept_pump.dmp  emp612_pump.dmp  emp_pump.dmp  export.log  import.log  scott.dmp  ts20000.dmp  ts30000.dmp
    
    [shm2:pump_shm2]$ pwd
    /home/oracle/pump_shm2
    
    ```
    
    ![Untitled](3%20%E1%84%90%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%87%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%B3%E1%84%91%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%20%E1%84%85%E1%85%A6%E1%84%87%E1%85%A6%E1%86%AF%E1%84%85%E1%85%A9%20EXPORT%20PUMP%20IMPORT%20958c03fefdaa40d7860846f8aa947cbf/Untitled%207.png)
    
    ### ⚠️ 만약에 아래와 같이 에러가 발생 했을 때 조치사항
    
    ```sql
    UDE-04031: operation generated ORACLE error 4031
    ORA-04031: unable to allocate 16 bytes of shared memory ("shared pool","select obj#,type#,ctime,mtim...","SQLA","tmp")
    ```
    
    `UDE-04031: operation generated ORACLE error 4031`
    `ORA-04031: unable to allocate 16 bytes of shared memory ("shared pool","select obj#,type#,ctime,mtim...","SQLA","tmp")`
    
    - 원인 : Shared pool이 작아서 메모리를 할당할 수 없다는 것! 그럼 memory_target 파라미터를 재설정 해줘야함
    - 해결방법 : PROD DB 쪽에 memory_max_target을 1024m 로 늘리고 memory_target을 1024m로 늘립니다.
        
        ```sql
        PROD SYS >
        
        alter system set memory_max_target=1024m scope=spfile;
        alter system set memory_target=1024m scope=spfile;
        shutdown immediate
        startup
        ```
        
    - export를 시도했었기 때문에 pump파일이 생성될 위치에 생성이 되었는지 미리 확인하고 다시 export하기
    
6. **shm2 DB에서 export하면서 생성된 데이터 파일인 ts30000.dbf를 PROD DB의 data file이 있는 위치에 copy 합니다.**
    1. pump 쓰지 않고 export 했을 때와 동일한 방식!
    
    ```sql
    [shm2:~]$
    cp /u01/app/oracle/oradata/shm2/disk1/ts30000.dbf /u01/app/oracle/oradata/PROD/disk1/ts30000.dbf
    ```
    
    ![Untitled](3%20%E1%84%90%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%87%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%B3%E1%84%91%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%20%E1%84%85%E1%85%A6%E1%84%87%E1%85%A6%E1%86%AF%E1%84%85%E1%85%A9%20EXPORT%20PUMP%20IMPORT%20958c03fefdaa40d7860846f8aa947cbf/Untitled%208.png)
    
7. **shm2 DB의 ts30000.dmp 파일을 PROD DB의 pump 디렉토리로 copy 합니다.**
    
    ```sql
    [shm2:~]$
    cp /home/oracle/pump_shm2/ts30000.dmp /home/oracle/pump_prod/ts30000.dmp
    ```
    
    ![Untitled](3%20%E1%84%90%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%87%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%B3%E1%84%91%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%20%E1%84%85%E1%85%A6%E1%84%87%E1%85%A6%E1%86%AF%E1%84%85%E1%85%A9%20EXPORT%20PUMP%20IMPORT%20958c03fefdaa40d7860846f8aa947cbf/Untitled%209.png)
    
8. **shm2 DB에 ts30000 테이블 스페이스를 import 합니다.**
    1. import 하기 전에 PROD DB의 SYS 계정 패스워드가 무엇인지 확인하기
    2. import 하기 전에 PROD DB의 scott 계정이 있는지 확인하고 dba 권한이 있는지 확인
    
    ```sql
    PROD SYS > connect scott/tiger
    Connected.
    
    PROD SCOTT >
    PROD SCOTT >
    PROD SCOTT > select * from session_roles;
    
    ROLE
    ------------------------------
    DBA
    SELECT_CATALOG_ROLE
    HS_ADMIN_SELECT_ROLE
    EXECUTE_CATALOG_ROLE
    HS_ADMIN_EXECUTE_ROLE
    DELETE_CATALOG_ROLE
    EXP_FULL_DATABASE
    IMP_FULL_DATABASE
    DATAPUMP_EXP_FULL_DATABASE
    DATAPUMP_IMP_FULL_DATABASE
    GATHER_SYSTEM_STATISTICS
    
    ROLE
    ------------------------------
    SCHEDULER_ADMIN
    ```
    
    ![Untitled](3%20%E1%84%90%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%87%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%B3%E1%84%91%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%20%E1%84%85%E1%85%A6%E1%84%87%E1%85%A6%E1%86%AF%E1%84%85%E1%85%A9%20EXPORT%20PUMP%20IMPORT%20958c03fefdaa40d7860846f8aa947cbf/Untitled%2010.png)
    
    c. ts30000 테이블 스페이스 import
    
    ```bash
    [PROD:~]$ impdp directory=datapump_dir dumpfile=ts30000.dmp transport_datafiles='/u01/app/oracle/oradata/PROD/disk1/ts30000.dbf'
    
    Import: Release 11.2.0.1.0 - Production on Mon Feb 26 11:08:19 2024
    
    Copyright (c) 1982, 2009, Oracle and/or its affiliates.  All rights reserved.
    
    Username: sys as sysdba
    Password: oracle  <--- **입력하는 것 보이지 않으므로 주의하기**
    
    Connected to: Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production
    With the Partitioning, OLAP, Data Mining and Real Application Testing options
    Master table "SYS"."SYS_IMPORT_TRANSPORTABLE_01" successfully loaded/unloaded
    Starting "SYS"."SYS_IMPORT_TRANSPORTABLE_01":  sys/******** AS SYSDBA directory=datapump_dir dumpfile=ts30000.dmp transport_datafiles=/u01/app/oracle/oradata/PROD/disk1/ts30000.dbf
    Processing object type TRANSPORTABLE_EXPORT/PLUGTS_BLK
    Processing object type TRANSPORTABLE_EXPORT/TABLE
    Processing object type TRANSPORTABLE_EXPORT/POST_INSTANCE/PLUGTS_BLK
    Job "SYS"."SYS_IMPORT_TRANSPORTABLE_01" successfully completed at 11:08:30
    
    ```
    
    ![Untitled](3%20%E1%84%90%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%87%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%B3%E1%84%91%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%20%E1%84%85%E1%85%A6%E1%84%87%E1%85%A6%E1%86%AF%E1%84%85%E1%85%A9%20EXPORT%20PUMP%20IMPORT%20958c03fefdaa40d7860846f8aa947cbf/Untitled%2011.png)
    
9. **PROD DB에서 ts30000 테이블 스페이스를 read write 로 변경합니다.**
    
    ```sql
    PROD SYS >
    alter tablespace ts30000 read write;
    ```
    
    ```sql
    PROD SYS > alter tablespace ts30000 read write;
    
    Tablespace altered.
    
    PROD SYS >
    PROD SYS > select t.name, d.enabled
            from v$tablespace t, v$datafile d
            where t.ts# = d.ts#;  2    3
    
    NAME                           ENABLED
    ------------------------------ ----------
    SYSTEM                         READ WRITE
    SYSAUX                         READ WRITE
    UNDOTBS                        READ WRITE
    TEST100                        READ WRITE
    EXAMPLE                        READ WRITE
    TEST100                        READ WRITE
    TEST100                        READ WRITE
    TS100                          READ WRITE
    SHM                            READ WRITE
    INSA01                         READ WRITE
    HR01                           READ WRITE
    
    NAME                           ENABLED
    ------------------------------ ----------
    TS501                          READ WRITE
    TS501                          READ WRITE
    TEST5000                       READ WRITE
    TS7000                         READ WRITE
    INSA02                         READ WRITE
    UNDOTBS7                       READ WRITE
    SMALL_UNDO                     READ WRITE
    UNDOTBS9                       READ WRITE
    SYSTEM                         READ WRITE
    SYSAUX                         READ WRITE
    TS50                           READ WRITE
    
    NAME                           ENABLED
    ------------------------------ ----------
    TS5000                         READ WRITE
    EXAMPLE                        READ WRITE
    EXAMPLE                        READ WRITE
    TS7100                         READ WRITE
    TS8100                         READ WRITE
    TS20000                        READ WRITE
    TS30000                        READ WRITE
    
    29 rows selected.
    ```
    
    ![Untitled](3%20%E1%84%90%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%87%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%B3%E1%84%91%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%20%E1%84%85%E1%85%A6%E1%84%87%E1%85%A6%E1%86%AF%E1%84%85%E1%85%A9%20EXPORT%20PUMP%20IMPORT%20958c03fefdaa40d7860846f8aa947cbf/Untitled%2012.png)
    
10. **PROD DB에서 emp30000테이블이 잘 조회되는지 확인합니다.**
    
    ```sql
    PROD SYS >
    connect scott/tiger
    
    PROD SCOTT >
    select count(*) from emp30000;
    ```
    
    ![Untitled](3%20%E1%84%90%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%87%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%B3%E1%84%91%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%20%E1%84%85%E1%85%A6%E1%84%87%E1%85%A6%E1%86%AF%E1%84%85%E1%85%A9%20EXPORT%20PUMP%20IMPORT%20958c03fefdaa40d7860846f8aa947cbf/Untitled%2013.png)
    
11. **PROD DB에서 ts30000 테이블 스페이스를 read write로 변경합니다.**
    
    ```sql
    PROD SYS >
    alter tablespace ts30000 read write;
    ```
    
    ![Untitled](3%20%E1%84%90%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%87%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%B3%E1%84%91%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%20%E1%84%85%E1%85%A6%E1%84%87%E1%85%A6%E1%86%AF%E1%84%85%E1%85%A9%20EXPORT%20PUMP%20IMPORT%20958c03fefdaa40d7860846f8aa947cbf/Untitled%2014.png)