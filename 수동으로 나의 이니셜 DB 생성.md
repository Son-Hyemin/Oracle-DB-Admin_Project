# 수동으로 나의 이니셜 DB(shm2 DB) 생성하기

소유자: 쏘니

<aside>
💡 shm2 DB를 수동으로 스크립트로 생성하기 (본인의 영문 이니셜 DB를 생성해봄)

</aside>

수동으로 database 생성하는데, file system 에 수동으로 db를 생성하는 작업을 진행

### DB 수동 생성

1. **환경 구성을 합니다.**
    - `. oraenv` 를 입력하여 ORACLE_HOME 경로를 복사해둔다.
    
    ```bash
    [orcl:~]$ . oraenv
    ORACLE_SID = [orcl] ?
    The Oracle base for **ORACLE_HOME=/u01/app/oracle/product/11.2.0/dbhome_1** is /u01/app/oracle
    ```
    
    ```sql
    ORACLE_HOME=/u01/app/oracle/product/11.2.0/dbhome_1
    ```
    
    ![Untitled](%E1%84%89%E1%85%AE%E1%84%83%E1%85%A9%E1%86%BC%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20%E1%84%82%E1%85%A1%E1%84%8B%E1%85%B4%20%E1%84%8B%E1%85%B5%E1%84%82%E1%85%B5%E1%84%89%E1%85%A7%E1%86%AF%20DB(shm2%20DB)%20%E1%84%89%E1%85%A2%E1%86%BC%E1%84%89%E1%85%A5%E1%86%BC%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%2007aba996d39e425691cbfabffe2a34ac/Untitled.png)
    
- . oraenv를 입력 후 SID가 없기 때문에 생성할 DB 이름을 입력 > 복사해둔 경로 넣기
    
    ```bash
    [orcl:~]$ . oraenv
    ORACLE_SID = [orcl] ? shm2
    ORACLE_HOME = [/home/oracle] ? /u01/app/oracle/product/11.2.0/dbhome_1
    
    The Oracle base for ORACLE_HOME=/u01/app/oracle/product/11.2.0/dbhome_1 is /u01/app/oracle
    ```
    
    이 경로가 ORACLE BASE이고 이는 최상단의 ROOT 디렉토리이고, 
    ORACLE_HOME 디렉토리 내에 오라클 설치 파일이 있어야한다.(이 경로에 SQLPLUS, DBCA등의 파일이 있을 것)
    
    ![Untitled](%E1%84%89%E1%85%AE%E1%84%83%E1%85%A9%E1%86%BC%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20%E1%84%82%E1%85%A1%E1%84%8B%E1%85%B4%20%E1%84%8B%E1%85%B5%E1%84%82%E1%85%B5%E1%84%89%E1%85%A7%E1%86%AF%20DB(shm2%20DB)%20%E1%84%89%E1%85%A2%E1%86%BC%E1%84%89%E1%85%A5%E1%86%BC%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%2007aba996d39e425691cbfabffe2a34ac/Untitled%201.png)
    

1. **database 를 생성할 디렉토리를 생성한다.**
    
    ```bash
    # shm2 디렉토리 생성
    $ mkdir -p /u01/app/oracle/oradata/shm2
    
    # 생성 되었는지 경로 이동으로 가보기
    $ cd /u01/app/oracle/oradata/shm2
    
    # 5개의 디렉토리 한 번에 생성
    $ mkdir disk1 disk2 disk3 disk4 disk5 
    
    # 확인
    $ ls
    ```
    
    ```bash
    [shm2:~]$ mkdir -p /u01/app/oracle/oradata/shm2
    
    [shm2:~]$ cd /u01/app/oracle/oradata/shm2
    [shm2:shm2]$
    
    [shm2:shm2]$ pwd
    /u01/app/oracle/oradata/shm2
    
    [shm2:shm2]$ mkdir disk1 disk2 disk3 disk4 disk5
    
    [shm2:shm2]$ ls
    disk1  disk2  disk3  disk4  disk5
    ```
    
    ![Untitled](%E1%84%89%E1%85%AE%E1%84%83%E1%85%A9%E1%86%BC%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20%E1%84%82%E1%85%A1%E1%84%8B%E1%85%B4%20%E1%84%8B%E1%85%B5%E1%84%82%E1%85%B5%E1%84%89%E1%85%A7%E1%86%AF%20DB(shm2%20DB)%20%E1%84%89%E1%85%A2%E1%86%BC%E1%84%89%E1%85%A5%E1%86%BC%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%2007aba996d39e425691cbfabffe2a34ac/Untitled%202.png)
    
2. **parameter file 을 만든다.**
    
    오라클 메모리 즉 인스턴스의 구조 정보를 담고 있는 파일(그래서 오라클 인스턴스를 먼저 띄울 것)
    
    - 경로 이동하여 파라미터 파일 명을 SID이름을 넣어서 구성
    
    ```bash
    cd $ORACLE_HOME/dbs
    vi $ORACLE_HOME/dbs/initshm2.ora
    
    # initSID.ora로 구성하는 것
    ```
    
    ```bash
    [PROD:PROD]$ cd $ORACLE_HOME/dbs
    [PROD:dbs]$ ls
    hc_DBUA0.dat  init.ora      lkSHM      peshm_DBUA0_0  spfileshm.ora
    hc_orcl.dat   initorcl.ora  orapworcl  peshm_orcl_0
    hc_shm.dat    lkORCL        orapwshm   peshm_shm_0
    [PROD:dbs]$
    [PROD:dbs]$ vi $ORACLE_HOME/dbs/initPROD.ora
    ```
    
    ![Untitled](%E1%84%89%E1%85%AE%E1%84%83%E1%85%A9%E1%86%BC%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20%E1%84%82%E1%85%A1%E1%84%8B%E1%85%B4%20%E1%84%8B%E1%85%B5%E1%84%82%E1%85%B5%E1%84%89%E1%85%A7%E1%86%AF%20DB(shm2%20DB)%20%E1%84%89%E1%85%A2%E1%86%BC%E1%84%89%E1%85%A5%E1%86%BC%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%2007aba996d39e425691cbfabffe2a34ac/Untitled%203.png)
    
    - 생성한 파라미터 파일 안에 아래의 내용을 복사하여 파라미터 파일을 만든다.
    
    ```bash
    db_name = shm2
    compatible=11.2.0.1.0
    
    sga_target = 256M
    
    undo_management = AUTO
    undo_tablespace = UNDOTBS
    
    processes = 100
    
    remote_login_passwordfile = EXCLUSIVE
     
    control_files = (/u01/app/oracle/oradata/shm2/disk1/ctrl1.ctl ,
                     /u01/app/oracle/oradata/shm2/disk2/ctrl2.ctl ,
                     /u01/app/oracle/oradata/shm2/disk3/ctrl3.ctl)
    ```
    
    ![Untitled](%E1%84%89%E1%85%AE%E1%84%83%E1%85%A9%E1%86%BC%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20%E1%84%82%E1%85%A1%E1%84%8B%E1%85%B4%20%E1%84%8B%E1%85%B5%E1%84%82%E1%85%B5%E1%84%89%E1%85%A7%E1%86%AF%20DB(shm2%20DB)%20%E1%84%89%E1%85%A2%E1%86%BC%E1%84%89%E1%85%A5%E1%86%BC%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%2007aba996d39e425691cbfabffe2a34ac/Untitled%204.png)
    
    `ls` 로 생성 확인
    
    ![Untitled](%E1%84%89%E1%85%AE%E1%84%83%E1%85%A9%E1%86%BC%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20%E1%84%82%E1%85%A1%E1%84%8B%E1%85%B4%20%E1%84%8B%E1%85%B5%E1%84%82%E1%85%B5%E1%84%89%E1%85%A7%E1%86%AF%20DB(shm2%20DB)%20%E1%84%89%E1%85%A2%E1%86%BC%E1%84%89%E1%85%A5%E1%86%BC%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%2007aba996d39e425691cbfabffe2a34ac/Untitled%205.png)
    
    < PROD DB 생성과 다른 점 >
    
    ```sql
    control_files = (/u01/app/oracle/oradata/PROD/disk1/ctrl1.ctl ,
                     /u01/app/oracle/oradata/PROD/disk2/ctrl2.ctl ,
                     /u01/app/oracle/oradata/PROD/disk3/ctrl3.ctl,
                     ~~/u01/app/oracle/flash_recovery_area/ctl4.ctl  )~~
    
    control_files = (/u01/app/oracle/oradata/shm2/disk1/ctrl1.ctl ,
                     /u01/app/oracle/oradata/shm2/disk2/ctrl2.ctl ,
                     /u01/app/oracle/oradata/shm2/disk3/ctrl3.ctl ,
    							  ~~/u01/app/oracle/flash_recovery_area/ctl4.ctl  )~~
    
    생성시 오류 발생할 수 있기 때문에 만들지 말고 날리기! control file 3번까지만 생성하기
    왜 ? 이름이 똑같아서 prod 만들었던것과 중복되기 때문
    ```
    
    있기 때문에 생성하지 않고 넘어가겠음 
    

1. **instance 를 nomount 로 올린다.**
    
    ```bash
    $ sqlplus / as sysdba
    ```
    
    - SHUT DOWN → NOMOUNT → MOUNT → OPEN 단계 중 NOMOUNT로 !
    
    ```sql
    # nomount 뒤에 파라미터 파일(pfile)의 위치를 넣어주는 것
    
    [shm2:dbs]$ sqlplus / as sysdba
    
    shm2 SYS > startup nomount pfile=$ORACLE_HOME/dbs/initshm2.ora
    
    shm2 SYS > select instance_name, status
          from v$instance;
    
    INSTANCE_NAME    STATUS
    ---------------- ------------
    shm2             STARTED
    ```
    
    상태가 started 잘 진행되고 있는 것 
    
    ![Untitled](%E1%84%89%E1%85%AE%E1%84%83%E1%85%A9%E1%86%BC%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20%E1%84%82%E1%85%A1%E1%84%8B%E1%85%B4%20%E1%84%8B%E1%85%B5%E1%84%82%E1%85%B5%E1%84%89%E1%85%A7%E1%86%AF%20DB(shm2%20DB)%20%E1%84%89%E1%85%A2%E1%86%BC%E1%84%89%E1%85%A5%E1%86%BC%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%2007aba996d39e425691cbfabffe2a34ac/Untitled%206.png)
    

1. [ASM 인스턴스를 내리는 방법](https://www.notion.so/ASM-746f8978862749828d2b58629cdec7dc?pvs=21) (현장에서는 안내림!)
    
    단, 시간이 많이 소요되기 때문에 다른 DB 모두 내리고 하는게 좋음 
    
    asm 내리기 전에 orcl먼저 내려야함
    
    ```sql
    [orcl:dbs]$ . oraenv
    ORACLE_SID = [orcl] ? orcl
    The Oracle base for ORACLE_HOME=/u01/app/oracle/product/11.2.0/dbhome_1 is /u01/app/oracle
    
    [orcl:dbs]$ sqlplus / as sysdba
    
    orcl SYS > shutdown immediate
    orcl SYS > exit;
    
    [orcl:dbs]$
    [orcl:dbs]$ ps -ef | grep pmon | grep -v grep
    oracle    5145     1  0 10:55 ?        00:00:04 ora_pmon_shm
    oracle    8688     1  0 12:31 ?        00:00:03 ora_pmon_PROD
    oracle   11198     1  0 17:03 ?        00:00:00 ora_pmon_shm2
    oracle   14827     1  0 14:17 ?        00:00:02 asm_pmon_+ASM
    ```
    
    ![Untitled](%E1%84%89%E1%85%AE%E1%84%83%E1%85%A9%E1%86%BC%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20%E1%84%82%E1%85%A1%E1%84%8B%E1%85%B4%20%E1%84%8B%E1%85%B5%E1%84%82%E1%85%B5%E1%84%89%E1%85%A7%E1%86%AF%20DB(shm2%20DB)%20%E1%84%89%E1%85%A2%E1%86%BC%E1%84%89%E1%85%A5%E1%86%BC%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%2007aba996d39e425691cbfabffe2a34ac/Untitled%207.png)
    
    ```sql
    [orcl:dbs]$ . oraenv
    ORACLE_SID = [orcl] ? +ASM
    The Oracle base for ORACLE_HOME=/u01/app/oracle/product/11.2.0/grid is /u01/app/oracle
    
    [+ASM:dbs]$ echo $ORACLE_SID
    +ASM
    [+ASM:dbs]$ echo $ORACLE_HOME
    /u01/app/oracle/product/11.2.0/grid
    
    [+ASM:dbs]$ select instance_name, status
    -bash: syntax error near unexpected token `status'
    
    [+ASM:dbs]$ sqlplus / as sysasm
    
    SQL> select instance_name, status
          from v$instance;
    
    INSTANCE_NAME    STATUS
    ---------------- ------------
    +ASM             STARTED
    
    SQL> shutdown immediate
    SQL> exit
    
    [+ASM:dbs]$
    [+ASM:dbs]$  ps -ef | grep pmon | grep -v grep
    oracle    5145     1  0 10:55 ?        00:00:04 ora_pmon_shm
    oracle    8688     1  0 12:31 ?        00:00:03 ora_pmon_PROD
    oracle   11198     1  0 17:03 ?        00:00:00 ora_pmon_shm2
    ```
    
    ![Untitled](%E1%84%89%E1%85%AE%E1%84%83%E1%85%A9%E1%86%BC%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20%E1%84%82%E1%85%A1%E1%84%8B%E1%85%B4%20%E1%84%8B%E1%85%B5%E1%84%82%E1%85%B5%E1%84%89%E1%85%A7%E1%86%AF%20DB(shm2%20DB)%20%E1%84%89%E1%85%A2%E1%86%BC%E1%84%89%E1%85%A5%E1%86%BC%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%2007aba996d39e425691cbfabffe2a34ac/Untitled%208.png)
    
    ![Untitled](%E1%84%89%E1%85%AE%E1%84%83%E1%85%A9%E1%86%BC%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20%E1%84%82%E1%85%A1%E1%84%8B%E1%85%B4%20%E1%84%8B%E1%85%B5%E1%84%82%E1%85%B5%E1%84%89%E1%85%A7%E1%86%AF%20DB(shm2%20DB)%20%E1%84%89%E1%85%A2%E1%86%BC%E1%84%89%E1%85%A5%E1%86%BC%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%2007aba996d39e425691cbfabffe2a34ac/Untitled%209.png)
    
    ![Untitled](%E1%84%89%E1%85%AE%E1%84%83%E1%85%A9%E1%86%BC%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20%E1%84%82%E1%85%A1%E1%84%8B%E1%85%B4%20%E1%84%8B%E1%85%B5%E1%84%82%E1%85%B5%E1%84%89%E1%85%A7%E1%86%AF%20DB(shm2%20DB)%20%E1%84%89%E1%85%A2%E1%86%BC%E1%84%89%E1%85%A5%E1%86%BC%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%2007aba996d39e425691cbfabffe2a34ac/Untitled%2010.png)
    
2. **create database 스크립트를 수행한다 (조금 오래 돌 것)**
    
    실패할 경우 생성된 파일을 삭제 후 다시 수행해야함 
    
    - database 를 생성한다는것은 어떠한 파일들을 생성하면 db 가 생성되는
    것인가 ?
        
        ```bash
        # 아래 1,2,3은 DB 생성 시 생성됨
        # database 를 수동으로 생성하는 스크립트를 수행하면 파일중 1,2,3이 생성된다.
        
        1. data file : DATA가 저장되어져 있는 파일
        2. controlfile : DB의 구조정보를 담고 있는 파일
        3. redo log file : 복구를 하기 위해서 필요한 파일
        -----------------------
        4. parameter file : 방금 initPROD.ora 라는 이름으로 생성했음 
        5. archive log file
        6. password file
        ```
        
    - `. oraenv`로 shm2 로 이동
        
        ```bash
        /u01/app/oracle/product/11.2.0/dbhome_1
        ```
        
        ```sql
        [+ASM:dbs]$ . oraenv
        ORACLE_SID = [+ASM] ? shm2 
        ORACLE_HOME = [/home/oracle] ? /u01/app/oracle/product/11.2.0/dbhome_1
        The Oracle base for ORACLE_HOME=/u01/app/oracle/product/11.2.0/dbhome_1 is /u01/app/oracle
        ```
        
        ![Untitled](%E1%84%89%E1%85%AE%E1%84%83%E1%85%A9%E1%86%BC%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20%E1%84%82%E1%85%A1%E1%84%8B%E1%85%B4%20%E1%84%8B%E1%85%B5%E1%84%82%E1%85%B5%E1%84%89%E1%85%A7%E1%86%AF%20DB(shm2%20DB)%20%E1%84%89%E1%85%A2%E1%86%BC%E1%84%89%E1%85%A5%E1%86%BC%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%2007aba996d39e425691cbfabffe2a34ac/Untitled%2011.png)
        
    - SYS 접속 후 INSTANCE 이름과 상태 조회(status가 started면 정상)
        
        ```sql
        [shm2:dbs]$ sqlplus / as sysdba
        
        shm2 SYS > select instance_name, status
               from v$instance;
        
        INSTANCE_NAME    STATUS
        ---------------- ------------
        shm2             STARTED
        
        shm2 SYS >
        ```
        
        ![Untitled](%E1%84%89%E1%85%AE%E1%84%83%E1%85%A9%E1%86%BC%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20%E1%84%82%E1%85%A1%E1%84%8B%E1%85%B4%20%E1%84%8B%E1%85%B5%E1%84%82%E1%85%B5%E1%84%89%E1%85%A7%E1%86%AF%20DB(shm2%20DB)%20%E1%84%89%E1%85%A2%E1%86%BC%E1%84%89%E1%85%A5%E1%86%BC%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%2007aba996d39e425691cbfabffe2a34ac/Untitled%2012.png)
        
    - create database (데이터베이스 생성)
        
        ```sql
        create database shm2
        user sys identified by oracle
        user system identified by oracle
        datafile '/u01/app/oracle/oradata/shm2/disk1/system01.dbf' 
        size 100M autoextend on maxsize unlimited extent management local
        sysaux 
        datafile '/u01/app/oracle/oradata/shm2/disk2/sysaux01.dbf' 
        size 50M autoextend on maxsize unlimited
        default temporary tablespace temp
        tempfile '/u01/app/oracle/oradata/shm2/disk3/temp01.dbf' 
        size 50M autoextend on maxsize unlimited
        undo tablespace undotbs
        datafile '/u01/app/oracle/oradata/shm2/disk4/undotbs01.dbf' 
        size 50M autoextend on maxsize unlimited
        logfile 
        group 1 ('/u01/app/oracle/oradata/shm2/disk4/redoG1M1.rdo',
                '/u01/app/oracle/oradata/shm2/disk5/redoG1M2.rdo') size 100M,
        group 2 ('/u01/app/oracle/oradata/shm2/disk4/redoG2M1.rdo',
                '/u01/app/oracle/oradata/shm2/disk5/redoG2M2.rdo') size 100M,
        group 3 ('/u01/app/oracle/oradata/shm2/disk4/redoG3M1.rdo',
                '/u01/app/oracle/oradata/shm2/disk5/redoG3M2.rdo') size 100M,
        group 4 ('/u01/app/oracle/oradata/shm2/disk4/redoG4M1.rdo',
                '/u01/app/oracle/oradata/shm2/disk5/redoG4M2.rdo') size 100M,
        group 5 ('/u01/app/oracle/oradata/shm2/disk4/redoG5M1.rdo',
                 '/u01/app/oracle/oradata/shm2/disk5/redoG5M2.rdo') size 100M;
        ```
        
        ```sql
        create database PROD --prod라는 db를 만드는 것
        
        # 아래 두 줄이 수동 생성하는 스크립트임
        user sys identified by oracle --by뒤에는 패스워드
        user system identified by oracle
        
        #system datafile의 위치를 씀! 
        #만약 raw device라면, raw device의 위치를 넣어줘야함
        #여기선 raw device 말고 file system을 사용해줘서 아래와 같이 system datafile의 위치를 씀
        # .dbf 파일 자동으로 생성됨 
        # datfile 4개 생성되면서, .dbf 파일 자동으로 생성됨
        datafile '/u01/app/oracle/oradata/PROD/disk1/**system01.dbf'** 
        size 100M autoextend on maxsize unlimited extent management local
        sysaux 
        datafile '/u01/app/oracle/oradata/PROD/disk2/**sysaux01.dbf'** 
        size 50M autoextend on maxsize unlimited
        default temporary tablespace temp
        tempfile '/u01/app/oracle/oradata/PROD/disk3/**temp01.dbf'** 
        size 50M autoextend on maxsize unlimited
        undo tablespace undotbs
        datafile '/u01/app/oracle/oradata/PROD/disk4/**undotbs01.dbf'** 
        size 50M autoextend on maxsize unlimited
        
        # redo log 파일의 위치를 넣음 
        # redo log file 자동으로 생성됨 
        logfile 
        group 1 ('/u01/app/oracle/oradata/PROD/disk4/redoG1M1.rdo',
                '/u01/app/oracle/oradata/PROD/disk5/redoG1M2.rdo') size 100M,
        group 2 ('/u01/app/oracle/oradata/PROD/disk4/redoG2M1.rdo',
                '/u01/app/oracle/oradata/PROD/disk5/redoG2M2.rdo') size 100M,
        group 3 ('/u01/app/oracle/oradata/PROD/disk4/redoG3M1.rdo',
                '/u01/app/oracle/oradata/PROD/disk5/redoG3M2.rdo') size 100M,
        group 4 ('/u01/app/oracle/oradata/PROD/disk4/redoG4M1.rdo',
                '/u01/app/oracle/oradata/PROD/disk5/redoG4M2.rdo') size 100M,
        group 5 ('/u01/app/oracle/oradata/PROD/disk4/redoG5M1.rdo',
                 '/u01/app/oracle/oradata/PROD/disk5/redoG5M2.rdo') size 100M;
        ```
        
        ![Untitled](%E1%84%89%E1%85%AE%E1%84%83%E1%85%A9%E1%86%BC%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20%E1%84%82%E1%85%A1%E1%84%8B%E1%85%B4%20%E1%84%8B%E1%85%B5%E1%84%82%E1%85%B5%E1%84%89%E1%85%A7%E1%86%AF%20DB(shm2%20DB)%20%E1%84%89%E1%85%A2%E1%86%BC%E1%84%89%E1%85%A5%E1%86%BC%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%2007aba996d39e425691cbfabffe2a34ac/Untitled%2013.png)
        
    
    - shm2 instance 상태가 open까지 올라왔는지 확인하기
        
        ```sql
         SYS > select instance_name, status
              from v$instance;
        ```
        
        ```sql
        shm2 SYS > select instance_name, status
              from v$instance;  2
        
        INSTANCE_NAME    STATUS
        ---------------- ------------
        shm2             OPEN
        ```
        
        ![Untitled](%E1%84%89%E1%85%AE%E1%84%83%E1%85%A9%E1%86%BC%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20%E1%84%82%E1%85%A1%E1%84%8B%E1%85%B4%20%E1%84%8B%E1%85%B5%E1%84%82%E1%85%B5%E1%84%89%E1%85%A7%E1%86%AF%20DB(shm2%20DB)%20%E1%84%89%E1%85%A2%E1%86%BC%E1%84%89%E1%85%A5%E1%86%BC%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%2007aba996d39e425691cbfabffe2a34ac/Untitled%2014.png)
        

1. **data dictionary 를 생성하는 스크립트를 수행한다(3개 중 하나라도 빼먹으면 안됨)**
    - 아래 2개의 스크립트는 **SYS** 유저에서 수행( 오래 걸림 )
    
    ```sql
    SQL> 
    @$ORACLE_HOME/rdbms/admin/catalog.sql
    
    SQL> 
    @$ORACLE_HOME/rdbms/admin/catproc.sql
    ```
    
    중간중간 발생하는 에러는 DROP하려는데 없는거라 신경쓰지 않아도 됨
    
    `@$ORACLE_HOME/rdbms/admin/catalog.sql` : 수행 시 마지막에 뜨는 화면
    
    ![Untitled](%E1%84%89%E1%85%AE%E1%84%83%E1%85%A9%E1%86%BC%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20%E1%84%82%E1%85%A1%E1%84%8B%E1%85%B4%20%E1%84%8B%E1%85%B5%E1%84%82%E1%85%B5%E1%84%89%E1%85%A7%E1%86%AF%20DB(shm2%20DB)%20%E1%84%89%E1%85%A2%E1%86%BC%E1%84%89%E1%85%A5%E1%86%BC%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%2007aba996d39e425691cbfabffe2a34ac/Untitled%2015.png)
    
    `@$ORACLE_HOME/rdbms/admin/catproc.sql` : 수행 시 마지막에 뜨는 화면
    
    ![Untitled](%E1%84%89%E1%85%AE%E1%84%83%E1%85%A9%E1%86%BC%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20%E1%84%82%E1%85%A1%E1%84%8B%E1%85%B4%20%E1%84%8B%E1%85%B5%E1%84%82%E1%85%B5%E1%84%89%E1%85%A7%E1%86%AF%20DB(shm2%20DB)%20%E1%84%89%E1%85%A2%E1%86%BC%E1%84%89%E1%85%A5%E1%86%BC%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%2007aba996d39e425691cbfabffe2a34ac/Untitled%2016.png)
    
    - 아래 2개의 스크립트는 **SYSTEM**에서 수행
        
        ```sql
        SQL> connect system/oracle
        SQL> @$ORACLE_HOME/sqlplus/admin/pupbld.sql
        ```
        
        ![Untitled](%E1%84%89%E1%85%AE%E1%84%83%E1%85%A9%E1%86%BC%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20%E1%84%82%E1%85%A1%E1%84%8B%E1%85%B4%20%E1%84%8B%E1%85%B5%E1%84%82%E1%85%B5%E1%84%89%E1%85%A7%E1%86%AF%20DB(shm2%20DB)%20%E1%84%89%E1%85%A2%E1%86%BC%E1%84%89%E1%85%A5%E1%86%BC%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%2007aba996d39e425691cbfabffe2a34ac/Untitled%2017.png)
        
        ![Untitled](%E1%84%89%E1%85%AE%E1%84%83%E1%85%A9%E1%86%BC%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20%E1%84%82%E1%85%A1%E1%84%8B%E1%85%B4%20%E1%84%8B%E1%85%B5%E1%84%82%E1%85%B5%E1%84%89%E1%85%A7%E1%86%AF%20DB(shm2%20DB)%20%E1%84%89%E1%85%A2%E1%86%BC%E1%84%89%E1%85%A5%E1%86%BC%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%2007aba996d39e425691cbfabffe2a34ac/Untitled%2018.png)
        
    - shm DB가 OPEN 상태인지 확인하기
        
        ```sql
        shm2 SYSTEM > select instance_name, status from v$instance;
        
        INSTANCE_NAME    STATUS
        ---------------- ------------
        shm2             OPEN
        
        1 row selected.
        ```
        
        ![Untitled](%E1%84%89%E1%85%AE%E1%84%83%E1%85%A9%E1%86%BC%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20%E1%84%82%E1%85%A1%E1%84%8B%E1%85%B4%20%E1%84%8B%E1%85%B5%E1%84%82%E1%85%B5%E1%84%89%E1%85%A7%E1%86%AF%20DB(shm2%20DB)%20%E1%84%89%E1%85%A2%E1%86%BC%E1%84%89%E1%85%A5%E1%86%BC%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%2007aba996d39e425691cbfabffe2a34ac/Untitled%2019.png)
        

1. **shm2 DB 에 운영 DATA 생성**
    - scott 계정 생성하고 demobld 스크립트를 수행한다.
    
    ```sql
    create  user  scott
        identified  by  tiger;
    
    grant  dba  to  scott;
    
    connect scott/tiger
    
    ed demobld.sql
    
    @demobld.sql
    ```
    
    ```sql
    shm2 SYSTEM > create  user  scott
        identified  by  tiger;  2
    
    User created.
    
    shm2 SYSTEM >
    shm2 SYSTEM > grant  dba  to  scott;
    
    Grant succeeded.
    
    shm2 SYSTEM > connect scott/tiger
    Connected.
    shm2 SCOTT >
    shm2 SCOTT > ed demobld.sql
    
    shm2 SCOTT >
    shm2 SCOTT > @demobld
    ```
    
    —-@demobld.sql에 넣을 내용(scott에서 emp테이블 조회해보기 위함)
    
    ```sql
    alter session set nls_Date_format='RR/MM/DD';
    drop table emp;
    drop table dept;
    
    CREATE TABLE DEPT
           (DEPTNO number(10),
            DNAME VARCHAR2(14),
            LOC VARCHAR2(13) );
    
    INSERT INTO DEPT VALUES (10, 'ACCOUNTING', 'NEW YORK');
    INSERT INTO DEPT VALUES (20, 'RESEARCH',   'DALLAS');
    INSERT INTO DEPT VALUES (30, 'SALES',      'CHICAGO');
    INSERT INTO DEPT VALUES (40, 'OPERATIONS', 'BOSTON');
    
    CREATE TABLE EMP (
     EMPNO               NUMBER(4) NOT NULL,
     ENAME               VARCHAR2(10),
     JOB                 VARCHAR2(9),
     MGR                 NUMBER(4) ,
     HIREDATE            DATE,
     SAL                 NUMBER(7,2),
     COMM                NUMBER(7,2),
     DEPTNO              NUMBER(2) );
    
    INSERT INTO EMP VALUES (7839,'KING','PRESIDENT',NULL,'81-11-17',5000,NULL,10);
    INSERT INTO EMP VALUES (7698,'BLAKE','MANAGER',7839,'81-05-01',2850,NULL,30);
    INSERT INTO EMP VALUES (7782,'CLARK','MANAGER',7839,'81-05-09',2450,NULL,10);
    INSERT INTO EMP VALUES (7566,'JONES','MANAGER',7839,'81-04-01',2975,NULL,20);
    INSERT INTO EMP VALUES (7654,'MARTIN','SALESMAN',7698,'81-09-10',1250,1400,30);
    INSERT INTO EMP VALUES (7499,'ALLEN','SALESMAN',7698,'81-02-11',1600,300,30);
    INSERT INTO EMP VALUES (7844,'TURNER','SALESMAN',7698,'81-08-21',1500,0,30);
    INSERT INTO EMP VALUES (7900,'JAMES','CLERK',7698,'81-12-11',950,NULL,30);
    INSERT INTO EMP VALUES (7521,'WARD','SALESMAN',7698,'81-02-23',1250,500,30);
    INSERT INTO EMP VALUES (7902,'FORD','ANALYST',7566,'81-12-11',3000,NULL,20);
    INSERT INTO EMP VALUES (7369,'SMITH','CLERK',7902,'80-12-09',800,NULL,20);
    INSERT INTO EMP VALUES (7788,'SCOTT','ANALYST',7566,'82-12-22',3000,NULL,20);
    INSERT INTO EMP VALUES (7876,'ADAMS','CLERK',7788,'83-01-15',1100,NULL,20);
    INSERT INTO EMP VALUES (7934,'MILLER','CLERK',7782,'82-01-11',1300,NULL,10);
    
    commit;
    ```
    
    ![Untitled](%E1%84%89%E1%85%AE%E1%84%83%E1%85%A9%E1%86%BC%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20%E1%84%82%E1%85%A1%E1%84%8B%E1%85%B4%20%E1%84%8B%E1%85%B5%E1%84%82%E1%85%B5%E1%84%89%E1%85%A7%E1%86%AF%20DB(shm2%20DB)%20%E1%84%89%E1%85%A2%E1%86%BC%E1%84%89%E1%85%A5%E1%86%BC%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%2007aba996d39e425691cbfabffe2a34ac/Untitled%2020.png)
    
    ![Untitled](%E1%84%89%E1%85%AE%E1%84%83%E1%85%A9%E1%86%BC%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20%E1%84%82%E1%85%A1%E1%84%8B%E1%85%B4%20%E1%84%8B%E1%85%B5%E1%84%82%E1%85%B5%E1%84%89%E1%85%A7%E1%86%AF%20DB(shm2%20DB)%20%E1%84%89%E1%85%A2%E1%86%BC%E1%84%89%E1%85%A5%E1%86%BC%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%2007aba996d39e425691cbfabffe2a34ac/Untitled%2021.png)
    
    - sqlplus 에서 vi 편집기를 실행할 수 있도록 설정한다.
        
        ```sql
        $ cd $ORACLE_HOME/sqlplus/admin
        $ vi glogin.sql
        ------------------
        define_editor='vi'
        ------------------
        ```
        
        ```sql
        shm2 SCOTT > exit
        
        [shm2:dbs]$ cd
        
        [shm2:~]$ cd $ORACLE_HOME/sqlplus/admin
        [shm2:admin]$ ls
        asm.sql     glogin.sql      plustrce.sql  table.sql
        column.sql  help            pupbld.sql    tablespace.sql
        extent.sql  libsqlplus.def  segment.sql
        
        [shm2:admin]$ vi glogin.sql
        ```
        
        ![Untitled](%E1%84%89%E1%85%AE%E1%84%83%E1%85%A9%E1%86%BC%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20%E1%84%82%E1%85%A1%E1%84%8B%E1%85%B4%20%E1%84%8B%E1%85%B5%E1%84%82%E1%85%B5%E1%84%89%E1%85%A7%E1%86%AF%20DB(shm2%20DB)%20%E1%84%89%E1%85%A2%E1%86%BC%E1%84%89%E1%85%A5%E1%86%BC%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%2007aba996d39e425691cbfabffe2a34ac/Untitled%2022.png)
        
        ![Untitled](%E1%84%89%E1%85%AE%E1%84%83%E1%85%A9%E1%86%BC%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20%E1%84%82%E1%85%A1%E1%84%8B%E1%85%B4%20%E1%84%8B%E1%85%B5%E1%84%82%E1%85%B5%E1%84%89%E1%85%A7%E1%86%AF%20DB(shm2%20DB)%20%E1%84%89%E1%85%A2%E1%86%BC%E1%84%89%E1%85%A5%E1%86%BC%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%2007aba996d39e425691cbfabffe2a34ac/Untitled%2023.png)