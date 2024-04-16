# RMAN을 사용해서 모든 control file 삭제하고 복구

소유자: 쏘니

> control file 3개 중 2개를 삭제하고 RMAN으로 복구
> 

---

1. **먼저 현재 database 의 DB ID를 조회합니다.**
    1. parameter file을 복구할 때는 무조건 DB  id가 필요하지만 
    2. control file은 반드시 필요한 것은 아님. 
    3. 하지만 필요할 때도 있어서 조회해 두는 것을 권장함 
    
    ```sql
    shm SYS > select dbid from v$database;
    
          DBID
    ----------
    1022900107
    ```
    
2. **Control file을 생성하는 script를 생성합니다. (혹시 백업 실패할 때 사용하기 위함)**
    
    ```sql
    alter database backup controlfile to trace
    	as '/home/oracle/create_controlfile_20240307.sql';
    ```
    
3. **RMAN으로 접속해서 control file이 자동 백업 되도록 설정합니다.**
    
    ```sql
    [shm:~]$ rnc
    
    Recovery Manager: Release 11.2.0.1.0 - Production on Thu Mar 7 10:59:42 2024
    
    Copyright (c) 1982, 2009, Oracle and/or its affiliates.  All rights reserved.
    
    connected to target database: SHM (DBID=1022900107) <--- 여기서도 DB ID 확인 가능
    using target database control file instead of recovery catalog
    
    **#2. control file이 자동 백업 되도록 설정**
    RMAN> configure controlfile autobackup on;
    
    new RMAN configuration parameters:
    CONFIGURE CONTROLFILE AUTOBACKUP ON;
    new RMAN configuration parameters are successfully stored
    ```
    
    - `configure controlfile autobackup on;`
        
        **데이터베이스 구조가 자꾸 바뀌기 때문에 CONTROL FILE이 백업 받기 어려움.. 그래서 RMAN으로 앞으로 어떤 파일을 백업 받던 백업 받을 때 마다 control file이 자동으로 백업되어지게 설정하는 것. 
        이렇게 해두면 앞으로 DB 구조가 변경될 때 마다 RMAN이 controlfile을 자동으로 백업합니다. (ex. tablespace 삭제, tablespace 추가, tablespace 생성 , data file 위치 이동 등 …)
        이렇게 안하면 예시의 상황 때 마다 2번을 매번 해야함.** 
        
    
4. **RMAN 에서 data file 중에 아무거나 하나를 백업 받습니다.**
    1. 그래야 autobackup으로 current controlfile을 백업 받을 수 있기 때문! 
    
    ```sql
    RMAN> backup datafile 13;
    
    Starting backup at 07-MAR-24
    allocated channel: ORA_DISK_1
    channel ORA_DISK_1: SID=63 device type=DISK
    channel ORA_DISK_1: starting full datafile backup set
    channel ORA_DISK_1: specifying datafile(s) in backup set
    input datafile file number=00013 name=/u01/app/oracle/oradata/shm/ts5000.dbf
    channel ORA_DISK_1: starting piece 1 at 07-MAR-24
    channel ORA_DISK_1: finished piece 1 at 07-MAR-24
    piece handle=/u01/app/oracle/flash_recovery_area/SHM/backupset/2024_03_07/o1_mf_nnndf_TAG20240307T112711_lyl9hzh4_.bkp tag=TAG20240307T112711 comment=NONE
    channel ORA_DISK_1: backup set complete, elapsed time: 00:00:01
    Finished backup at 07-MAR-24
    
    Starting Control File and SPFILE Autobackup at 07-MAR-24
    piece handle=/u01/app/oracle/flash_recovery_area/SHM/autobackup/2024_03_07/o1_mf_s_1162985232_lyl9j0my_.bkp comment=NONE
    Finished Control File and SPFILE Autobackup at 07-MAR-24
    
    RMAN>
    ```
    
    ![스크린샷 2024-03-07 112747.png](RMAN%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A2%E1%84%89%E1%85%A5%20%E1%84%86%E1%85%A9%E1%84%83%E1%85%B3%E1%86%AB%20control%20file%20%E1%84%89%E1%85%A1%E1%86%A8%E1%84%8C%E1%85%A6%E1%84%92%E1%85%A1%E1%84%80%E1%85%A9%20%E1%84%87%E1%85%A9%E1%86%A8%20494850cda17a472e8f31b7f281684819/%25EC%258A%25A4%25ED%2581%25AC%25EB%25A6%25B0%25EC%2583%25B7_2024-03-07_112747.png)
    
    마지막에 보면, control file과 spfile이 자동으로 백업 된 것을 확인할 수 있다.
    
5. **control file의 위치를 확인합니다.**
    
    ```sql
    shm SYS > @controlfile
    
    NAME
    --------------------------------------------------------------------------------
    /u01/app/oracle/oradata/shm/control01.ctl
    /u01/app/oracle/oradata/shm/control02.ctl
    /u01/app/oracle/oradata/shm/control03.ctl
    ```
    
    ![Untitled](RMAN%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A2%E1%84%89%E1%85%A5%20%E1%84%86%E1%85%A9%E1%84%83%E1%85%B3%E1%86%AB%20control%20file%20%E1%84%89%E1%85%A1%E1%86%A8%E1%84%8C%E1%85%A6%E1%84%92%E1%85%A1%E1%84%80%E1%85%A9%20%E1%84%87%E1%85%A9%E1%86%A8%20494850cda17a472e8f31b7f281684819/Untitled.png)
    
6. **shutdown abort로 DB를 내립니다.** 
    
    ```sql
    shm SYS > shutdown abort
    ORACLE instance shut down.
    ```
    
    ![Untitled](RMAN%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A2%E1%84%89%E1%85%A5%20%E1%84%86%E1%85%A9%E1%84%83%E1%85%B3%E1%86%AB%20control%20file%20%E1%84%89%E1%85%A1%E1%86%A8%E1%84%8C%E1%85%A6%E1%84%92%E1%85%A1%E1%84%80%E1%85%A9%20%E1%84%87%E1%85%A9%E1%86%A8%20494850cda17a472e8f31b7f281684819/Untitled%201.png)
    
7. **모든 control file을 2개만 삭제합니다.**
    
    ```sql
    [shm:~]$ oradata
    [shm:shm]$
    [shm:shm]$ ls *.ctl
    control01.ctl  control02.ctl  control03.ctl
    [shm:shm]$
    [shm:shm]$ rm control01.ctl
    [shm:shm]$ rm control02.ctl
    [shm:shm]$
    [shm:shm]$ ls *.ctl
    control03.ctl
    ```
    
    ![Untitled](RMAN%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A2%E1%84%89%E1%85%A5%20%E1%84%86%E1%85%A9%E1%84%83%E1%85%B3%E1%86%AB%20control%20file%20%E1%84%89%E1%85%A1%E1%86%A8%E1%84%8C%E1%85%A6%E1%84%92%E1%85%A1%E1%84%80%E1%85%A9%20%E1%84%87%E1%85%A9%E1%86%A8%20494850cda17a472e8f31b7f281684819/Untitled%202.png)
    
8. **DB startup을 시도합니다. ←—NOMOUNT 단계 까지만 올라감**
    
    ```sql
    shm SYS > startup
    ORACLE instance started.
    
    Total System Global Area  636100608 bytes
    Fixed Size                  1338392 bytes
    Variable Size             184550376 bytes
    Database Buffers          444596224 bytes
    Redo Buffers                5615616 bytes
    ORA-00205: error in identifying control file, check alert log for more info
    
    ** alter log file **
    
    ALTER DATABASE   MOUNT
    ORA-00210: cannot open the specified control file
    ORA-00202: control file: '/u01/app/oracle/oradata/shm/control02.ctl'
    ORA-27037: unable to obtain file status
    Linux Error: 2: No such file or directory
    Additional information: 3
    ORA-00210: cannot open the specified control file
    ORA-00202: control file: '/u01/app/oracle/oradata/shm/control01.ctl'
    ORA-27037: unable to obtain file status
    Linux Error: 2: No such file or directory
    Additional information: 3
    ORA-205 signalled during: ALTER DATABASE   MOUNT...
    
    ```
    
    ![Untitled](RMAN%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A2%E1%84%89%E1%85%A5%20%E1%84%86%E1%85%A9%E1%84%83%E1%85%B3%E1%86%AB%20control%20file%20%E1%84%89%E1%85%A1%E1%86%A8%E1%84%8C%E1%85%A6%E1%84%92%E1%85%A1%E1%84%80%E1%85%A9%20%E1%84%87%E1%85%A9%E1%86%A8%20494850cda17a472e8f31b7f281684819/Untitled%203.png)
    
    ![Untitled](RMAN%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A2%E1%84%89%E1%85%A5%20%E1%84%86%E1%85%A9%E1%84%83%E1%85%B3%E1%86%AB%20control%20file%20%E1%84%89%E1%85%A1%E1%86%A8%E1%84%8C%E1%85%A6%E1%84%92%E1%85%A1%E1%84%80%E1%85%A9%20%E1%84%87%E1%85%A9%E1%86%A8%20494850cda17a472e8f31b7f281684819/Untitled%204.png)
    
9. **다시 DB를 shutdown abort로 내린 후 startup nomount 단계로 다시 올림**
    
    ```sql
    shm SYS > shutdown abort
    ORACLE instance shut down.
    shm SYS >
    shm SYS > startup nomount
    ORACLE instance started.
    
    Total System Global Area  636100608 bytes
    Fixed Size                  1338392 bytes
    Variable Size             184550376 bytes
    Database Buffers          444596224 bytes
    Redo Buffers                5615616 bytes
    ```
    
    ![Untitled](RMAN%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A2%E1%84%89%E1%85%A5%20%E1%84%86%E1%85%A9%E1%84%83%E1%85%B3%E1%86%AB%20control%20file%20%E1%84%89%E1%85%A1%E1%86%A8%E1%84%8C%E1%85%A6%E1%84%92%E1%85%A1%E1%84%80%E1%85%A9%20%E1%84%87%E1%85%A9%E1%86%A8%20494850cda17a472e8f31b7f281684819/Untitled%205.png)
    
10. **RMAN으로 접속합니다.**
    
    ```sql
    [shm:~]$ rnc
    
    Recovery Manager: Release 11.2.0.1.0 - Production on Thu Mar 7 11:11:39 2024
    
    Copyright (c) 1982, 2009, Oracle and/or its affiliates.  All rights reserved.
    
    connected to target database: SHM (not mounted)
    using target database control file instead of recovery catalog
    
    RMAN>
    
    RMAN>
    ```
    
    ![Untitled](RMAN%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A2%E1%84%89%E1%85%A5%20%E1%84%86%E1%85%A9%E1%84%83%E1%85%B3%E1%86%AB%20control%20file%20%E1%84%89%E1%85%A1%E1%86%A8%E1%84%8C%E1%85%A6%E1%84%92%E1%85%A1%E1%84%80%E1%85%A9%20%E1%84%87%E1%85%A9%E1%86%A8%20494850cda17a472e8f31b7f281684819/Untitled%206.png)
    
    **지금까지 mount 상태에서 RMAN으로 접속 했었음. 
    왜냐하면, 복구는 항상 MOUNT 상태에서 진행해야 하기 때문
    하지만 위와 같이 NOMOUNT 상태에서도 RMAN으로 접속할 수 있음.**
    
11. **RMAN으로 control file을 복원합니다.**
    
    ```sql
    RMAN> restore controlfile from autobackup;
    
    Starting restore at 07-MAR-24
    allocated channel: ORA_DISK_1
    channel ORA_DISK_1: SID=63 device type=DISK
    
    recovery area destination: /u01/app/oracle/flash_recovery_area
    database name (or database unique name) used for search: SHM
    channel ORA_DISK_1: AUTOBACKUP /u01/app/oracle/flash_recovery_area/SHM/autobackup/2024_03_07/o1_mf_s_1162985232_lyl9j0my_.bkp found in the recovery area
    AUTOBACKUP search with format "%F" not attempted because DBID was not set
    channel ORA_DISK_1: restoring control file from AUTOBACKUP /u01/app/oracle/flash_recovery_area/SHM/autobackup/2024_03_07/o1_mf_s_1162985232_lyl9j0my_.bkp
    channel ORA_DISK_1: control file restore from AUTOBACKUP complete
    output file name=/u01/app/oracle/oradata/shm/control01.ctl
    output file name=/u01/app/oracle/oradata/shm/control02.ctl
    output file name=/u01/app/oracle/oradata/shm/control03.ctl
    Finished restore at 07-MAR-24
    
    ```
    
    ![Untitled](RMAN%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A2%E1%84%89%E1%85%A5%20%E1%84%86%E1%85%A9%E1%84%83%E1%85%B3%E1%86%AB%20control%20file%20%E1%84%89%E1%85%A1%E1%86%A8%E1%84%8C%E1%85%A6%E1%84%92%E1%85%A1%E1%84%80%E1%85%A9%20%E1%84%87%E1%85%A9%E1%86%A8%20494850cda17a472e8f31b7f281684819/Untitled%207.png)
    
    알아서 auto backup 본에서 control file을 가져옴. 
    그래서 마지막 메시지를 보면, control file 3개 모두 복원했다는 것을 확인할 수 있음 
    
12. **DB를 alter database mount로 올립니다.** 
    
    ```sql
    RMAN> alter database mount;
    
    database mounted
    released channel: ORA_DISK_1
    ```
    
    ![Untitled](RMAN%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A2%E1%84%89%E1%85%A5%20%E1%84%86%E1%85%A9%E1%84%83%E1%85%B3%E1%86%AB%20control%20file%20%E1%84%89%E1%85%A1%E1%86%A8%E1%84%8C%E1%85%A6%E1%84%92%E1%85%A1%E1%84%80%E1%85%A9%20%E1%84%87%E1%85%A9%E1%86%A8%20494850cda17a472e8f31b7f281684819/Untitled%208.png)
    
    **※ control file을 복원 했다고 자동으로 mount 되는 것이 아니기 때문에 직접 mount 단계로 올려줘야함.**
    
13. **RMAN으로 recover database를 수행하여 복구합니다.**
    
    ```sql
    RMAN> recover database;
    
    Starting recover at 07-MAR-24
    Starting implicit crosscheck backup at 07-MAR-24
    allocated channel: ORA_DISK_1
    channel ORA_DISK_1: SID=63 device type=DISK
    Crosschecked 15 objects
    Finished implicit crosscheck backup at 07-MAR-24
    
    Starting implicit crosscheck copy at 07-MAR-24
    using channel ORA_DISK_1
    Finished implicit crosscheck copy at 07-MAR-24
    
    searching for all files in the recovery area
    cataloging files...
    cataloging done
    
    List of Cataloged Files
    =======================
    File Name: /u01/app/oracle/flash_recovery_area/SHM/autobackup/2024_03_07/o1_mf_s_1162983941_lyl87oof_.bkp
    File Name: /u01/app/oracle/flash_recovery_area/SHM/autobackup/2024_03_07/o1_mf_s_1162985232_lyl9j0my_.bkp
    
    using channel ORA_DISK_1
    
    starting media recovery
    
    archived log for thread 1 with sequence 1 is already on disk as file /u01/app/oracle/oradata/shm/redo01.log
    archived log file name=/u01/app/oracle/oradata/shm/redo01.log thread=1 sequence=1
    media recovery complete, elapsed time: 00:00:01
    Finished recover at 07-MAR-24
    
    ```
    
    ![Untitled](RMAN%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A2%E1%84%89%E1%85%A5%20%E1%84%86%E1%85%A9%E1%84%83%E1%85%B3%E1%86%AB%20control%20file%20%E1%84%89%E1%85%A1%E1%86%A8%E1%84%8C%E1%85%A6%E1%84%92%E1%85%A1%E1%84%80%E1%85%A9%20%E1%84%87%E1%85%A9%E1%86%A8%20494850cda17a472e8f31b7f281684819/Untitled%209.png)
    
    **※ 복원한 control file이 옛날 파일이기 때문에 recover(복구)를 해서 최신 파일로 만들어줘야 하기 때문에 복구를 수행한다.** 
    
14. **DB를 resetlogs 옵션을 사용하여 OPEN 단계로 올립니다.**
    
    ```sql
    RMAN> alter database open resetlogs;
    
    database opened
    ```
    
    ![Untitled](RMAN%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A2%E1%84%89%E1%85%A5%20%E1%84%86%E1%85%A9%E1%84%83%E1%85%B3%E1%86%AB%20control%20file%20%E1%84%89%E1%85%A1%E1%86%A8%E1%84%8C%E1%85%A6%E1%84%92%E1%85%A1%E1%84%80%E1%85%A9%20%E1%84%87%E1%85%A9%E1%86%A8%20494850cda17a472e8f31b7f281684819/Untitled%2010.png)
    
    control file은 모든 DB에 대한 정보를 알고 있는 선생님 같은 존재임. 
    그래서 control file을 복원해 왔다는 의미는. 새로운 선생님이 부임했다는 것. 
    따라서 다시 reset을 해야하기 때문에 resetlogs를 해주는 것. 
    
    하지만 학생과 비슷한 data file 은 사라지는게 아님.
    
    선생님이 바뀐다고 해서 학생들도 초기화되는게 아니기 때문이다. 
    
15. resetlogs로 DB를 열었으면 반드시 FULL BACKUP 을 수행해야 합니다.
    
    ```sql
    RMAN> backup database;
    
    Starting backup at 07-MAR-24
    using channel ORA_DISK_1
    channel ORA_DISK_1: starting full datafile backup set
    channel ORA_DISK_1: specifying datafile(s) in backup set
    input datafile file number=00001 name=/u01/app/oracle/oradata/shm/system01.dbf
    input datafile file number=00002 name=/u01/app/oracle/oradata/shm/sysaux01.dbf
    input datafile file number=00006 name=/u01/app/oracle/oradata/shm/users02.dbf
    input datafile file number=00005 name=/u01/app/oracle/oradata/shm/example01.dbf
    input datafile file number=00003 name=/u01/app/oracle/oradata/shm/undotbs01.dbf
    input datafile file number=00007 name=/u01/app/oracle/oradata/shm/ts01.dbf
    input datafile file number=00008 name=/u01/app/oracle/oradata/shm/ts02.dbf
    input datafile file number=00013 name=/u01/app/oracle/oradata/shm/ts5000.dbf
    input datafile file number=00014 name=/u01/app/oracle/oradata/shm/ts6000.dbf
    input datafile file number=00004 name=/u01/app/oracle/oradata/shm/users01.dbf
    input datafile file number=00009 name=/u01/app/oracle/oradata/shm/ts03.dbf
    input datafile file number=00010 name=/u01/app/oracle/oradata/shm/cuppang01.dbf
    input datafile file number=00011 name=/u01/app/oracle/oradata/shm/cuppang02.dbf
    input datafile file number=00012 name=/u01/app/oracle/oradata/shm/cuppang03.dbf
    channel ORA_DISK_1: starting piece 1 at 07-MAR-24
    channel ORA_DISK_1: finished piece 1 at 07-MAR-24
    piece handle=/u01/app/oracle/flash_recovery_area/SHM/backupset/2024_03_07/o1_mf_nnndf_TAG20240307T111924_lyl91dr7_.bkp tag=TAG20240307T111924 comment=NONE
    channel ORA_DISK_1: backup set complete, elapsed time: 00:00:15
    Finished backup at 07-MAR-24
    
    Starting Control File and SPFILE Autobackup at 07-MAR-24
    piece handle=/u01/app/oracle/flash_recovery_area/SHM/autobackup/2024_03_07/o1_mf_s_1162984779_lyl91vw6_.bkp comment=NONE
    Finished Control File and SPFILE Autobackup at 07-MAR-24
    ```
    
    ![Untitled](RMAN%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A2%E1%84%89%E1%85%A5%20%E1%84%86%E1%85%A9%E1%84%83%E1%85%B3%E1%86%AB%20control%20file%20%E1%84%89%E1%85%A1%E1%86%A8%E1%84%8C%E1%85%A6%E1%84%92%E1%85%A1%E1%84%80%E1%85%A9%20%E1%84%87%E1%85%A9%E1%86%A8%20494850cda17a472e8f31b7f281684819/Untitled%2011.png)
    

1. 제출 결과
    
    ```sql
    
    shm SYS > select name from v$controlfile;
    
    NAME
    --------------------------------------------------------------------------------
    /u01/app/oracle/oradata/shm/control01.ctl
    /u01/app/oracle/oradata/shm/control02.ctl
    /u01/app/oracle/oradata/shm/control03.ctl
    ```
    
    ![Untitled](RMAN%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A2%E1%84%89%E1%85%A5%20%E1%84%86%E1%85%A9%E1%84%83%E1%85%B3%E1%86%AB%20control%20file%20%E1%84%89%E1%85%A1%E1%86%A8%E1%84%8C%E1%85%A6%E1%84%92%E1%85%A1%E1%84%80%E1%85%A9%20%E1%84%87%E1%85%A9%E1%86%A8%20494850cda17a472e8f31b7f281684819/Untitled%2012.png)
    
    ```sql
    RMAN> list backup of controlfile;
    
    List of Backup Sets
    ===================
    
    ...
    
    BS Key  Type LV Size       Device Type Elapsed Time Completion Time
    ------- ---- -- ---------- ----------- ------------ ---------------
    14      Full    20.23M     DISK        00:00:00     07-MAR-24
            BP Key: 14   Status: AVAILABLE  Compressed: NO  Tag: TAG20240307T111939
            Piece Name: /u01/app/oracle/flash_recovery_area/SHM/autobackup/2024_03_07/o1_mf_s_1162984779_lyl91vw6_.bkp
      Control File Included: Ckp SCN: 98395963     Ckp time: 07-MAR-24
    
    BS Key  Type LV Size       Device Type Elapsed Time Completion Time
    ------- ---- -- ---------- ----------- ------------ ---------------
    16      Full    20.23M     DISK        00:00:00     07-MAR-24
            BP Key: 16   Status: AVAILABLE  Compressed: NO  Tag: TAG20240307T113359
            Piece Name: /u01/app/oracle/flash_recovery_area/SHM/autobackup/2024_03_07/o1_mf_s_1162985639_lyl9wqg7_.bkp
      Control File Included: Ckp SCN: 98396418     Ckp time: 07-MAR-24
    
    BS Key  Type LV Size       Device Type Elapsed Time Completion Time
    ------- ---- -- ---------- ----------- ------------ ---------------
    18      Full    20.23M     DISK        00:00:00     07-MAR-24
            BP Key: 18   Status: AVAILABLE  Compressed: NO  Tag: TAG20240307T113421
            Piece Name: /u01/app/oracle/flash_recovery_area/SHM/autobackup/2024_03_07/o1_mf_s_1162985661_lyl9xfcp_.bkp
      Control File Included: Ckp SCN: 98396452     Ckp time: 07-MAR-24
    
    ```
    
    ![Untitled](RMAN%E1%84%8B%E1%85%B3%E1%86%AF%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A2%E1%84%89%E1%85%A5%20%E1%84%86%E1%85%A9%E1%84%83%E1%85%B3%E1%86%AB%20control%20file%20%E1%84%89%E1%85%A1%E1%86%A8%E1%84%8C%E1%85%A6%E1%84%92%E1%85%A1%E1%84%80%E1%85%A9%20%E1%84%87%E1%85%A9%E1%86%A8%20494850cda17a472e8f31b7f281684819/Untitled%2013.png)