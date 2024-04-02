# Inactive 상태의 redo log file이 삭제 되었을 때 복구

소유자: 쏘니

<aside>
<img src="https://www.notion.so/icons/list-indent_gray.svg" alt="https://www.notion.so/icons/list-indent_gray.svg" width="40px" /> 순서

1. redo log group의 상태를 확인합니다.
2. log switch를 한 번 일으킵니다. 
3. check point를 한 번 일으킵니다.
4. redo log group의 상태를 확인합니다.
5. redo log group 중 inactive 상태인 redo log group의 member를 확인합니다. 
6. DB를 shutdown abort로 내립니다. 
7. OS 에서 inactive 상태의 log group member를 rm으로 지웁니다.
    
    ※ 이때 active나 current를 지우지 않도록 주의해야한다.
    
8. DB를 startup 합니다.
    1. redo log file이 없기 때문에 mount → open으로 올라가지 않을 것
9. 따라서 rm 으로 날린 inactive 상태의 redo log group을 control file에서 지웁니다.
10. `alter database open`으로 DB를 올립니다.
11. 지워진 redo log group을 새로 추가합니다. 
</aside>

---

1. **redo log group의 상태를 확인합니다.**
    1. 현재 사용 중인 redo log group에 대해서 알 수 있다.
    
    ```sql
    shm SYS > select group#, status, sequence# from v$log;
    
        GROUP# STATUS            SEQUENCE#
    ---------- ---------------- ----------
             1 INACTIVE                  1
             2 INACTIVE                  2
             3 CURRENT                   3
    ```
    

    
2. **log switch를 한 번 일으킵니다.** 
    
    ```sql
    @logsw
    ```
    
3. **check point를 한 번 일으킵니다.**
    
    ```sql
    alter system checkpoint; 
    ```
    

    
4. **redo log group의 상태를 확인합니다.**
    
    ```sql
    shm SYS > 
    select group#, status, sequence# from v$log;
    
        GROUP# STATUS            SEQUENCE#
    ---------- ---------------- ----------
             1 CURRENT                   4
             2 INACTIVE                  2
             3 INACTIVE                  3
    ```
    

    
    ⇒ 3번 group지울 것
    
5. **redo log group 중 inactive 상태인 redo log group의 member를 확인합니다.** 
    - 그룹 : 논리적인 단위
    - 멤버 : OS에 눈에 보이는 파일
    
    ```sql
    shm SYS > col member for a40
    shm SYS > select group#, member
            from v$logfile;  2
    
        GROUP# MEMBER
    ---------- ----------------------------------------
             1 /u01/app/oracle/oradata/shm/redo01.log
             3 /u01/app/oracle/oradata/shm/redo03.log
             2 /u01/app/oracle/oradata/shm/redo02.log
    
    ```
    

    
6. **DB를 shutdown abort로 내립니다.** 
    
    ```sql
    shm SYS > shutdown abort
    ORACLE instance shut down.
    ```
    

    
7. **OS 에서 inactive 상태의 log group member를 rm으로 지웁니다.**
    
    **※ 이때 active나 current를 지우지 않도록 주의해야한다.**
    
    ⇒ 그 사이 current가 다음 순서로 넘어올 수도 있기 때문에 current와 최대한 멀리 있는 group member 날리기
    
    ```sql
    [shm:~]$ oradata
    [shm:shm]$
    [shm:shm]$ pwd
    /u01/app/oracle/oradata/shm
    [shm:shm]$
    
    [shm:shm]$ ls *.log
    redo01.log  redo02.log  redo03.log
    
    [shm:shm]$ rm redo03.log
    [shm:shm]$
    [shm:shm]$ ls *.log
    redo01.log  redo02.log
    ```
    

    
8. **DB를 startup 합니다.**
    1. **redo log file이 없기 때문에 mount → open으로 올라가지 않을 것**
    
    ```sql
    
    shm SYS > startup
    ORACLE instance started.
    
    Total System Global Area  636100608 bytes
    Fixed Size                  1338392 bytes
    Variable Size             184550376 bytes
    Database Buffers          444596224 bytes
    Redo Buffers                5615616 bytes
    Database mounted.
    ORA-03113: end-of-file on communication channel
    Process ID: 13971
    Session ID: 191 Serial number: 3
    ```
    

    
9. **따라서 rm 으로 날린 inactive 상태의 redo log group을 control file에서 지웁니다.**
    1. startup mount로 올리기
        
        ```sql
        shm SYS > exit
        Disconnected from Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - Production
        With the Partitioning, OLAP, Data Mining and Real Application Testing options
        [shm:~]$
        [shm:~]$ ss
        
        SQL*Plus: Release 11.2.0.1.0 Production on Mon Mar 4 16:39:43 2024
        
        Copyright (c) 1982, 2009, Oracle.  All rights reserved.
        
        Connected to an idle instance.
        
        shm SYS > startup mount
        ORACLE instance started.
        
        Total System Global Area  636100608 bytes
        Fixed Size                  1338392 bytes
        Variable Size             184550376 bytes
        Database Buffers          444596224 bytes
        Redo Buffers                5615616 bytes
        Database mounted.
        ```
        
    2. rm으로 날린 inactive 상태의 redo log group을 controlfile에서 지우기
        
        ```sql
        alter database drop logfile group 3;
        ```
        

        
    
10. `**alter database open`으로 DB를 올립니다.**
    
    ```sql
    alter database open;
    ```
    

    
11. **지워진 redo log group을 새로 추가합니다.** 
    1. 추가하기 전에 redo log group 의 상태 확인하기
        
        ```sql
        
        shm SYS > select group#, status, sequence# from v$log;
        
            GROUP# STATUS            SEQUENCE#
        ---------- ---------------- ----------
                 1 INACTIVE                  4
                 2 CURRENT                   5
        
        ```
        

        
    2. redo log group 새로 추가하기
        
        ```sql
        alter database add logfile group 3
        	'/u01/app/oracle/oradata/shm/redo03.log' size 5m;
        ```
        
        

1. **redo log group의 상태를 확인합니다.**
    
    ```sql
    shm SYS > 
    select group#, status, sequence# from v$log;
    
        GROUP# STATUS            SEQUENCE#
    ---------- ---------------- ----------
             1 INACTIVE                  4
             2 CURRENT                   5
             3 UNUSED                    0
    ```
    

    
