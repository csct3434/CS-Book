# 참고

MySQL의 내부 구조 및 동작 방식에 초점을 두고 정리했습니다.

# 01. 소개

### MySQL

- 오픈소스
- 커뮤니티 에디션 : 무료, 오픈 소스
- 엔터프라이즈 에디션 : 유료, 소스 코드 비공개

### 왜 MySQL인가?

- MySQL이 오라클에 비해 가지는 경쟁력은 가격(비용) 측면
- 최근 10여년 간 처리하는 데이터의 크기가 급격히 커짐 → 이를 오라클 RDBMS에 저장하기에는 너무 비쌈
- ‘어떤 DBMS가 좋은가요?’ 에 대한 저자의 답 : 자기가 가장 잘 활용할 수 있는 DBMS가 가장 좋은 DBMS다
- 그래도 고민된다면 다음의 순서로 고려 : 안정성 → 성능과 기능 → 커뮤니티나 인지도
    - 성능과 기능은 돈이나 노력으로 해결할 수 있지만 안정성은 그렇지 않다
    - 커뮤니티나 인지도가 적은 DBMS는 필요한 경험이나 지식을 구하기 어렵다

# 02. 설치와 설정

### 버전 선택

- 특별한 제약사항이 없다면 가능한 한 최신 버전을 설치하는 것이 좋다
- 새로운 메이저 버전으로 업그레이드 하는 경우, 최소 패치 버전이 15~20번 이상 릴리즈된 버전을 선택하는 것이 안정성 측면에서 좋다 (ex: MySQL 8.0 버전이라면 8.0.15 ~ 20 사이의 버전부터 시작하는 것을 권장)
- 메이저 버전은 많은 변화를 거친 버전이므로 갓 출시된 상태에서는 치명적이거나 픽스하는데 오래걸리는 버그가 발생할 가능성이 있음

### 에디션 선택

- MySQL의 상용화 전략 : 오픈 코어 모델
    - 핵심 내용은 엔터프라이즈 에디션과 커뮤니티 에디션 모두 동일하며, 특정 부가 기능들만 상용 버전인 엔터프라이즈 에디션에 포함하는 방식
- 엔터프라이즈 에디션과 커뮤니티 에디션의 기본 성능이 다르다거나 한 것은 아니므로, 엔터프라이즈 에디션에서 지원하는 것들이 꼭 필요한지 검토하여 선택
- 저자 : 경험상 Percona의 도구를 활용해서 MySQL 커뮤니티 에디션의 부족한 부분을 메꿀 수 있었기 때문에 엔터프라이즈 에디션의 필요성이 그다지 크지 않았다

### 서버 설정

- mysqld : 서버 프로그램
- mysql : 클라이언트 프로그램
- `mysqld —default-file=/etc/my.cnf --initialize-insecure`
    - `--initialize-insecure` : 초기 데이터 파일(시스템 테이블이 저장되는 데이터 파일) 및 로그파일 생성, 비밀번호가 없는 root 유저 생성
    - `--initialize` : 초기 데이터파일 및 로그파일 생성, 임시 비밀번호가 설정된 root 사용자 생성, 임시 비밀번호는 에러 로그 파일 /var/log/mysqld.log에서 확인
- 클린 셧다운(Clean Shutdown) 설정 : `mysql> SET GLOBAL innodb_fast_shutdown=0;`
    - 서버 종료 시 모든 커밋 내용을 데이터 파일에 적용
    - 클린 셧다운으로 종료되면 MySQL 서버가 다시 기동할 때 별도의 트랜잭션 복구 과정을 진행하지 않기 때문에 빠른 시작이 가능

### 서버 접속

- `mysql -uroot -p --host=localhost --socket=/tmp/mysql.sock`
- `mysql -uroot -p --host=127.0.0.1 --port=3306`
- `mysql -uroot -p` : localhost로 접속, 소켓 파일 사용
- 호스트 주소 localhost와 127.0.0.1의 차이
    - localhost : 소켓 파일을 통해 서버에 접속, Unix domain socket을 이용하는 방식, TCP/IP를 통한 통신이 아닌 IPC의 일종
    - 127.0.0.1 : TCP/IP 방식으로 접속

### 서버 업그레이드 방식

- In-Place Upgrade
    - 서버의 데이터 파일을 두고 업그레이드 하는 방법
    - 제약사항
        - 마이너 버전 간 업그레이드는 대부분 허용된다
        - 메이저 버전 간 업그레이드는 직전 버전에서만 업그레이드가 허용된다
- Logical Upgrade
    - mysqldump 도구 등을 이용해 MySQL 서버의 데이터를 SQL 문장이나 텍스트 파일로 덤프한 후, 새로 업그레이드된 버전의 MySQL 서버에서 덤프된 데이터를 적재하는 방법

# 사용자 및 권한

### 참고

레퍼런스성 내용이 많아 핵심만 간략히 정리

### 요약

- MySQL은 다른 DBMS와 다르게 사용자의 접속 지점도 계정의 일부다.
    - ‘svc_id’@’127.0.0.1’
- 사용자 계정은 SYSTEM_USER 권한을 가지고 있느냐에 따라 시스템 계정(DBA)과 일반 계정으로 구분된다.
- 계정 생성 예시
    
    ```sql
    CREATE USER 'user'@'%'
    	IDENTIFIED WITH 'mysql_native_password' BY 'password123'
    	REQUIRE NONE
    	PASSWORD EXPIRE INTERVAL 30 DAY
    	ACCOUNT UNLOCK
    	PASSWORD HISTORY DEFAULT
    	PASSWORD REUSE INTERVAL DEFAULT
    	PASSWORD REQUIRE CURRENT DEFAULT
    ```
    
- validate_password 컴포넌트를 이용하여 유효성 체크를 적용할 수 있다
    - 이력 관리를 통한 재사용 금지 기능
    - 금칙어 사전 기능
- 한 계정에 비밀번호 두 개를 설정하는 이중 비밀번호(Dual Password) 기능을 지원한다
    - 비밀번호 두 개를 동시에 운용 : PRIMARY, SECONDARY
    - 여러 응용 프로그램에서 공유되는 계정의 비밀번호를 변경할 때 사용
    - 예시
        
        ```sql
        ALTER USER 'root'@'localhost' IDENTIFIED BY 'old_password'
        ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password' RETAIN CURRENT PASSWORD;
        ```
        
        - 두 번째 ALTER USER 명령이 실행되면 이전 비밀번호(old_password)는 세컨더리 비밀번호가 되고, 새로운 비밀번호인 ‘new_password’가 프라이머리 비밀번호가 됨
        
        ```sql
        ALTER USER 'root'@'localhost' DISCARD OLD PASSWORD;
        ```
        
        - 세컨더리 비밀번호 삭제
- 권한은 정적 권한과 동적 권한으로 구분되고, 정적 권한에는 글로벌 권한과 객체 권한이 있다
- MySQL 서버는 역할(Role)과 계정을 구분하지 않는다
    - 역할을 생성하면 account_locked가 설정되어 로그인 할 수 없다
    - 예시
        
        ```sql
        CREATE ROLE role_emp_local_read@localhost;
        
        CREATE USER reader@'127.0.0.1' IDENTIFIED BY 'qwerty'
        
        GRANT SELECT ON employees.* TO role_emp_local_read@'localhost';
        
        GRANT role_emp_local_read@'localhost' TO reader@'127.0.0.1';
        ```
