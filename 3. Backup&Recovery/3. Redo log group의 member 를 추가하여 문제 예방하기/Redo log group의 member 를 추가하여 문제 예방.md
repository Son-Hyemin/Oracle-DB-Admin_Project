# Redo log group의 member 를 추가하여 문제 예방하기

소유자: 쏘니

<aside>
<img src="https://www.notion.so/icons/compose_gray.svg" alt="https://www.notion.so/icons/compose_gray.svg" width="40px" /> Redo log group별로 member 의 개수가 2개씩 되도록 생성

</aside>

---

1. **redo log group이 몇 개인지 확인합니다. - `@log_status.sql`**
    
    ```sql
    select group#, status, sequence#, archived
    	from v$log;
    ```
    
    - archived ⇒ archiving 됐는지 여부를 알려줌. 즉 archive log file로 써졌는지 여부!
    
    ![Untitled](Redo%20log%20group%E1%84%8B%E1%85%B4%20member%20%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%8E%E1%85%AE%E1%84%80%E1%85%A1%E1%84%92%E1%85%A1%E1%84%8B%E1%85%A7%20%E1%84%86%E1%85%AE%E1%86%AB%E1%84%8C%E1%85%A6%20%E1%84%8B%E1%85%A8%E1%84%87%E1%85%A1%E1%86%BC%E1%84%92%E1%85%A1%20f76166c95a2246cba0f6c594326d2658/Untitled.png)
    
    ![Untitled](Redo%20log%20group%E1%84%8B%E1%85%B4%20member%20%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%8E%E1%85%AE%E1%84%80%E1%85%A1%E1%84%92%E1%85%A1%E1%84%8B%E1%85%A7%20%E1%84%86%E1%85%AE%E1%86%AB%E1%84%8C%E1%85%A6%20%E1%84%8B%E1%85%A8%E1%84%87%E1%85%A1%E1%86%BC%E1%84%92%E1%85%A1%20f76166c95a2246cba0f6c594326d2658/Untitled%201.png)
    
    ![Untitled](Redo%20log%20group%E1%84%8B%E1%85%B4%20member%20%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%8E%E1%85%AE%E1%84%80%E1%85%A1%E1%84%92%E1%85%A1%E1%84%8B%E1%85%A7%20%E1%84%86%E1%85%AE%E1%86%AB%E1%84%8C%E1%85%A6%20%E1%84%8B%E1%85%A8%E1%84%87%E1%85%A1%E1%86%BC%E1%84%92%E1%85%A1%20f76166c95a2246cba0f6c594326d2658/Untitled%202.png)
    
2. **redo log group의 member가 몇 개인지 확인합니다**
    
    ```sql
    shm SYS >
    
    select group#, members
    	from v$log;
    
        GROUP#    MEMBERS
    ---------- ----------
             1          2
             2          1
             3          1
    ```
    
    ![Untitled](Redo%20log%20group%E1%84%8B%E1%85%B4%20member%20%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%8E%E1%85%AE%E1%84%80%E1%85%A1%E1%84%92%E1%85%A1%E1%84%8B%E1%85%A7%20%E1%84%86%E1%85%AE%E1%86%AB%E1%84%8C%E1%85%A6%20%E1%84%8B%E1%85%A8%E1%84%87%E1%85%A1%E1%86%BC%E1%84%92%E1%85%A1%20f76166c95a2246cba0f6c594326d2658/Untitled%203.png)
    
    **※ group 당 member가 1개 뿐임..! 이렇게 되면 member 하나가 깨지면, DB가 다운되어 올라오지 않아서 다시 깔아야하거나 다른 여러 복잡한 작업이 필요함. ★★★**
    
3. **redo log group의 member가 무엇인지 확인합니다.**
    
    ```sql
    shm SYS > 
    
    col member for a50
    select group#, member
       from v$logfile;
    
        GROUP# MEMBER
    ---------- --------------------------------------------------
             1 /u01/app/oracle/oradata/shm/redo01.log
             3 /u01/app/oracle/oradata/shm/redo03.log
             2 /u01/app/oracle/oradata/shm/redo02.log
             1 /u01/app/oracle/oradata/shm/redo01b.log
    
    ```
    
    ⇒ 각 그룹의 번호와 member의 번호가 잘 호응되는지 확인하기 
    
    ![Untitled](Redo%20log%20group%E1%84%8B%E1%85%B4%20member%20%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%8E%E1%85%AE%E1%84%80%E1%85%A1%E1%84%92%E1%85%A1%E1%84%8B%E1%85%A7%20%E1%84%86%E1%85%AE%E1%86%AB%E1%84%8C%E1%85%A6%20%E1%84%8B%E1%85%A8%E1%84%87%E1%85%A1%E1%86%BC%E1%84%92%E1%85%A1%20f76166c95a2246cba0f6c594326d2658/Untitled%204.png)
    
4. **redo log group의 member를 추가합니다.**
    
    ⚠️ 주의 : unique한 이름으로 설정해줘야함. 
    
    ```sql
    # 2번 group에 member 1개 추가하기 
    shm SYS > 
    alter database add logfile member
    	'/u01/app/oracle/oradata/shm/redo02b.log' to group 2;
    
    # 3번 group에 member 1개 추가하기
    alter database add logfile member
    	'/u01/app/oracle/oradata/shm/redo03b.log' to group 3;
    ```
    
    ![Untitled](Redo%20log%20group%E1%84%8B%E1%85%B4%20member%20%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%8E%E1%85%AE%E1%84%80%E1%85%A1%E1%84%92%E1%85%A1%E1%84%8B%E1%85%A7%20%E1%84%86%E1%85%AE%E1%86%AB%E1%84%8C%E1%85%A6%20%E1%84%8B%E1%85%A8%E1%84%87%E1%85%A1%E1%86%BC%E1%84%92%E1%85%A1%20f76166c95a2246cba0f6c594326d2658/Untitled%205.png)
    
5. **redo log group의 member가 추가되었는지 확인합니다.**
    
    ```sql
    shm SYS > select group#, members
      2     from v$log;
    
        GROUP#    MEMBERS
    ---------- ----------
             1          2
             2          2
             3          2
    
    shm SYS > 
    
    col member for a45
    
    select group#, member
      from v$logfile;
    
        GROUP# MEMBER
    ---------- ---------------------------------------------
             1 /u01/app/oracle/oradata/shm/redo01.log
             3 /u01/app/oracle/oradata/shm/redo03.log
             2 /u01/app/oracle/oradata/shm/redo02.log
             1 /u01/app/oracle/oradata/shm/redo01b.log
             2 /u01/app/oracle/oradata/shm/redo02b.log
             3 /u01/app/oracle/oradata/shm/redo03b.log
    
    6 rows selected.
    
    ```
    
    ![Untitled](Redo%20log%20group%E1%84%8B%E1%85%B4%20member%20%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%8E%E1%85%AE%E1%84%80%E1%85%A1%E1%84%92%E1%85%A1%E1%84%8B%E1%85%A7%20%E1%84%86%E1%85%AE%E1%86%AB%E1%84%8C%E1%85%A6%20%E1%84%8B%E1%85%A8%E1%84%87%E1%85%A1%E1%86%BC%E1%84%92%E1%85%A1%20f76166c95a2246cba0f6c594326d2658/Untitled%206.png)