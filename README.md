# Oracle-DB-Project
**DB 생성, 데이터 이행, 백업과 복구, RAC 환경 구성 등 Oracle DB Admin 관련 프로젝트**
</br>
</br>
</br>
</br>
</br>

## 1. 수동으로 DB 생성
**수동으로 File system 스토리지를 사용한 나의 이니셜 shm2 DB 생성하는 미니 프로젝트** 📁[바로가기](https://github.com/Son-Hyemin/Oracle-DB-Admin_Project/tree/789fcc18560fef695f6b25932086b705e7ccf4a7/1.%20%EC%88%98%EB%8F%99%EC%9C%BC%EB%A1%9C%20DB%20%EC%83%9D%EC%84%B1)
</br>
</br>
</br>

##  2. 데이터 이관 
**최종적으로 orcl DB의 모든 데이터를 shm DB로 이관하는 미니 프로젝트**
📁[바로가기](https://github.com/Son-Hyemin/Oracle-DB-Admin_Project/tree/789fcc18560fef695f6b25932086b705e7ccf4a7/2.%20%EB%8D%B0%EC%9D%B4%ED%84%B0%20%EC%9D%B4%EA%B4%80)

</br>

1) 사용자 레벨로 데이터 이관 : 👉🥰[보러가기](https://github.com/Son-Hyemin/Oracle-DB-Project/blob/958af76ee082352ec3d4f517f0f5267e1a1bd29c/2.%20%EB%8D%B0%EC%9D%B4%ED%84%B0%20%EC%9D%B4%EA%B4%80/1.%20%EC%9C%A0%EC%A0%80%EB%A0%88%EB%B2%A8%EB%A1%9C%20%EB%8D%B0%EC%9D%B4%ED%84%B0%20%EC%9D%B4%EA%B4%80/%EC%9C%A0%EC%A0%80%EB%A0%88%EB%B2%A8%EB%A1%9C%20%EB%8D%B0%EC%9D%B4%ED%84%B0%20%EC%9D%B4%EA%B4%80.md)
2) 테이블스페이스 레벨로 데이터 이관 : 👉🥰[보러가기](https://github.com/Son-Hyemin/Oracle-DB-Project/blob/958af76ee082352ec3d4f517f0f5267e1a1bd29c/2.%20%EB%8D%B0%EC%9D%B4%ED%84%B0%20%EC%9D%B4%EA%B4%80/2.%20%ED%85%8C%EC%9D%B4%EB%B8%94%EC%8A%A4%ED%8E%98%EC%9D%B4%EC%8A%A4%20%EB%A0%88%EB%B2%A8%EB%A1%9C%20%EB%8D%B0%EC%9D%B4%ED%84%B0%20%EC%9D%B4%EA%B4%80/%ED%85%8C%EC%9D%B4%EB%B8%94%EC%8A%A4%ED%8E%98%EC%9D%B4%EC%8A%A4%20%EB%A0%88%EB%B2%A8%EB%A1%9C%20%EB%8D%B0%EC%9D%B4%ED%84%B0%20%EC%9D%B4%EA%B4%80.md)
3) 테이블스페이스 레벨로 export pump / import : 👉🥰[보러가기](https://github.com/Son-Hyemin/Oracle-DB-Project/blob/958af76ee082352ec3d4f517f0f5267e1a1bd29c/2.%20%EB%8D%B0%EC%9D%B4%ED%84%B0%20%EC%9D%B4%EA%B4%80/3.%20%ED%85%8C%EC%9D%B4%EB%B8%94%EC%8A%A4%ED%8E%98%EC%9D%B4%EC%8A%A4%20%EB%A0%88%EB%B2%A8%EB%A1%9C%20export%20pump_import/%ED%85%8C%EC%9D%B4%EB%B8%94%EC%8A%A4%ED%8E%98%EC%9D%B4%EC%8A%A4%20%EB%A0%88%EB%B2%A8%EB%A1%9C%20export%20pump_import.md)
4) DB reorg : 👉🥰[보러가기](https://github.com/Son-Hyemin/Oracle-DB-Project/blob/958af76ee082352ec3d4f517f0f5267e1a1bd29c/2.%20%EB%8D%B0%EC%9D%B4%ED%84%B0%20%EC%9D%B4%EA%B4%80/4.%20DB%20reorg/DB%20reorg.md)

</br>
</br>

## 3. Backup & Recovery (백업과 복구) 
**완전복구, 불완전 복구, RMAN을 사용한 복원과 복구를 수행하는 미니 프로젝트** 📁[바로가기](https://github.com/Son-Hyemin/Oracle-DB-Project/tree/3be4633d4c3ecb2e8874fed0f00840478c13b727/3.%20Backup%26Recovery)

</br>

1) Hot backup을 사용하여 모든 datafile 완전 복구,복원 하기 : 👉🥰[보러가기](https://github.com/Son-Hyemin/Oracle-DB-Project/blob/958af76ee082352ec3d4f517f0f5267e1a1bd29c/3.%20Backup%26Recovery/1.%20(%EC%99%84%EC%A0%84%EB%B3%B5%EA%B5%AC)%20Hot%20backup%EC%9D%84%20%EC%82%AC%EC%9A%A9%ED%95%9C%20%EB%AA%A8%EB%93%A0%20data%20file%20%EB%B3%B5%EA%B5%AC/hot%20backup%EC%9C%BC%EB%A1%9C%20data%20file%20%EC%99%84%EC%A0%84%20%EB%B3%B5%EA%B5%AC.md)
2) inactive 상태의 redo log file이 삭제 되었을 때 복구 : 👉🥰[보러가기](https://github.com/Son-Hyemin/Oracle-DB-Project/blob/958af76ee082352ec3d4f517f0f5267e1a1bd29c/3.%20Backup%26Recovery/2.%20inactive%20%EC%83%81%ED%83%9C%EC%9D%98%20redo%20log%20file%EC%9D%B4%20%EC%82%AD%EC%A0%9C%20%EB%90%98%EC%97%88%EC%9D%84%20%EB%95%8C%20%EB%B3%B5%EA%B5%AC/Inactive%20%EC%83%81%ED%83%9C%EC%9D%98%20redo%20log%20file%EC%9D%B4%20%EC%82%AD%EC%A0%9C%20%EB%90%98%EC%97%88%EC%9D%84%20%EB%95%8C.md)
3) Redo log group의 member 를 추가하여 문제 예방 : 👉🥰[보러가기](https://github.com/Son-Hyemin/Oracle-DB-Project/blob/958af76ee082352ec3d4f517f0f5267e1a1bd29c/3.%20Backup%26Recovery/3.%20Redo%20log%20group%EC%9D%98%20member%20%EB%A5%BC%20%EC%B6%94%EA%B0%80%ED%95%98%EC%97%AC%20%EB%AC%B8%EC%A0%9C%20%EC%98%88%EB%B0%A9%ED%95%98%EA%B8%B0/Redo%20log%20group%EC%9D%98%20member%20%EB%A5%BC%20%EC%B6%94%EA%B0%80%ED%95%98%EC%97%AC%20%EB%AC%B8%EC%A0%9C%20%EC%98%88%EB%B0%A9.md)
4) Time base 불완전 복구 수행하기 : 👉🥰[보러가기](https://github.com/Son-Hyemin/Oracle-DB-Project/blob/958af76ee082352ec3d4f517f0f5267e1a1bd29c/3.%20Backup%26Recovery/4.%20Time%20Base%20%EB%B6%88%EC%99%84%EC%A0%84%20%EB%B3%B5%EA%B5%AC%ED%95%98%EA%B8%B0/Time%20Base%20%EB%B6%88%EC%99%84%EC%A0%84%20%EB%B3%B5%EA%B5%AC.md)
5) RMAN을 사용해서 모든 Control file 삭제하고 복구하기 : 👉🥰[보러가기](https://github.com/Son-Hyemin/Oracle-DB-Project/blob/4ebadc0cb2e6924a8242a0dfec39bc63483234de/3.%20Backup%26Recovery/5.%20RMAN%EC%9D%84%20%EC%82%AC%EC%9A%A9%ED%95%B4%EC%84%9C%20%EB%AA%A8%EB%93%A0%20control%20file%20%EC%82%AD%EC%A0%9C%ED%95%98%EA%B3%A0%20%EB%B3%B5%EA%B5%AC/RMAN%EC%9D%84%20%EC%82%AC%EC%9A%A9%ED%95%B4%EC%84%9C%20%EB%AA%A8%EB%93%A0%20control%20file%20%EC%82%AD%EC%A0%9C%ED%95%98%EA%B3%A0%20%EB%B3%B5%EA%B5%AC.md)
6) 데이터 가드 구현 : 👉🥰[보러가기](https://github.com/Son-Hyemin/Oracle-DB-Project/blob/28e006c45f96349e9a7806624c599a47d1be969b/3.%20Backup%26Recovery/6.%20%EB%8D%B0%EC%9D%B4%ED%84%B0%20%EA%B0%80%EB%93%9C%20%EA%B5%AC%ED%98%84/%EB%8D%B0%EC%9D%B4%ED%84%B0%20%EA%B0%80%EB%93%9C%20%EA%B5%AC%ED%98%84.md)
</br>
</br>

## 4. 11g RAC 설치
리눅스 기반의 2 NODE RAC 구성
👉🥰[보러가기](https://github.com/Son-Hyemin/Oracle-DB-Project/tree/cf318be810b74307040dcfa3ea6f6b5cebe8c2e5/4.%2011g%20RAC%20%EC%84%A4%EC%B9%98)
