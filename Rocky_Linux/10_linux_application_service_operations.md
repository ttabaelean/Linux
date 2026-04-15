### 1.  **Database(MySQL) 서버 구성**

- MySQL 데이터베이스 설치
    
    ```bash
    sudo -i
    
    # MySQL 설치(@: 그룹설치)
    dnf install -y @mysql
    
    ```
    
- 서비스 시작 및 자동시작 설정
    
    ```bash
    # 서비스 시작
    systemctl enable --now mysqld
    
    # 자동 시작 설정
    systemctl status mysqld
    netstat -napt | grep 3306
    
    # 방화벽 비활성화
    systemctl stop firewalld
    systemctl disable firewalld
    ```
    
- MySQL 초기 보안 설정
    - MySQL 관리자 root의 패스워드 설정 : Test123!
    
    ```bash
    mysql_secure_installation
    ```
    
    ```bash
    VALIDATE PASSWORD component? **N**
    New password:**Test1234!**
    Re-enter new password: **Test1234!**
    Remove anonymous users? Y
    Disallow root login remotely? N
    Remove test DB? Y
    Reload privilege tables now? Y
    ```
    
    - VALIDATE PASSWORD component? (N)   : 복잡한 비밀번호 규칙(특수문자, 대문자 필수 등)을 강제하는 컴포넌트를 사용 유무
    - Remove anonymous users? (Y)  : 이름이 없는 '익명 사용자' 계정을 삭제
    - Disallow root login remotely? (N) :  root 계정의 **원격 접속**을 차단
    - Remove test DB? (Y) : 기본으로 생성되는 `test` 데이터베이스를 삭제
    - Reload privilege tables now? (Y) : 변경한 설정 내용(권한 정보)을 즉시 적용
    - 설정 사항은 /var/lib/mysql/ 데이터베이스와, /etc/my.cnf, /etc/my.cnf.d/*  설정파일에 기록됨
- localhost 에서 MySQL 데이터베이스 콘솔 접속
    
    ```bash
    # 관리자 계정으로 접속
    mysql -u root -p
    Enter password: Test1234!
    ```
    
    - Petclinic 애플리케이션을 위한 데이터베이스 생성
        - petclinic 계정에게 모든 데이터베이스(*.*) 생성 및 관리 권한 부여
        - 
            
            ```bash
            # 1. petclinic 계정 생성
            CREATE USER 'petclinic'@'192.168.100.%'  IDENTIFIED WITH caching_sha2_password BY 'petclinic#!';
            
            # 2. petclinic 계정에게 모든 데이터베이스(*.*) 생성 및 관리 권한 부여
            GRANT ALL PRIVILEGES ON petclinic.* TO 'petclinic'@'192.168.100.%';
            GRANT ALL PRIVILEGES ON petclinic.* TO 'petclinic'@'192.168.100.%';
            # 3. 권한 즉시 적용
            FLUSH PRIVILEGES;
            
            # 4. petclinic 데이터베이스 생성
            CREATE DATABASE petclinic DEFAULT CHARACTER SET utf8mb4;
            SHOW DATABASES;
            
            exit;
            ```
            
        - petclinic 계정으로 petclinic db 연결 TEST
            
            ```bash
            mysql -h 192.168.100.30 -u petclinic -p
            ```
            
            ```bash
            Enter password:petclinic#!
            Welcome to the MySQL monitor.  Commands end with ; or \g.
            Your MySQL connection id is 13
            Server version: 8.4.8 Source distribution
            
            Copyright (c) 2000, 2026, Oracle and/or its affiliates.
            
            Oracle is a registered trademark of Oracle Corporation and/or its
            affiliates. Other names may be trademarks of their respective
            owners.
            
            Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
            
            mysql> show databases;
            +--------------------+
            | Database           |
            +--------------------+
            | information_schema |
            | performance_schema |
            | petclinic          |
            +--------------------+
            3 rows in set (0.00 sec)
            
            mysql> use petclinic;
            Database changed
            mysql> show tables;
            Empty set (0.01 sec)
            
            mysql> exit
            Bye
            ```
            

<br>

### 2. Application Server 구성 - Spring Petclinic

- 런타임 환경 구축 (Java 설치)
    - 애플리케이션이 구동될 수 있는 엔진을 설치하는 단계
    - **OpenJDK 25**: 최신 Java 환경을 구축
    
    ```bash
    sudo -i
    dnf install epel-release -y
    dnf -y install java-25-openjdk java-25-openjdk-devel
    java -version
    ```
    
- 소스 코드 확보 및 구조 확인
- GitHub에서 실제 서비스 코드를 가져와 서버에 배치
    
    ```bash
    git clone https://github.com/spring-projects/spring-petclinic.git
    cd spring-petclinic
    tree 
    ```
    
- 데이터베이스 연결 설정
    - 애플리케이션이 어떤 DB에 접속할지 가르쳐주는 단계
    - **설정 파일**: `src/main/resources/application-mysql.properties`
    - **URL**: `jdbc:mysql://192.168.100.30:3306/petclinic` (DB 서버 IP와 포트, DB명 지정)
    - **계정/비번**: 앞서 생성한 `petclinic` / `petclinic#!`를 입력합니다.
    - **init.mode**: `always`로 설정하면 앱 시작 시 필요한 테이블을 자동으로 생성합니다.
    
    ```bash
    vi src/main/resources/application-mysql.properties
    ```
    
    - 미리 준비한 database 서버의 정보를 기록해서 petclinc 빌드
    
    ```bash
    # database init, supports mysql too
    database=mysql
    spring.datasource.url=${MYSQL_URL:jdbc:mysql://192.168.100.30:3306/petclinic}
    spring.datasource.username=${MYSQL_USER:petclinic}
    spring.datasource.password=${MYSQL_PASS:petclinic#!}
    # SQL is written to be idempotent so this is safe
    spring.sql.init.mode=always
    
    **
    ```
    
- PetClinic 빌드
    - 소스 코드를 실행 가능한 파일(JAR)로 만들고 서비스를 시작
        
        ```bash
        # Maven Wrapper (./mvnw): 별도의 Maven 설치 없이도 프로젝트를 빌드할 수 있게 해주는 도구입니다. 
        # package 명령을 통해 소스를 target/ 폴더 안의 .jar 파일로 압축합니다.
        ./mvnw package
        ```
        
        ```bash
        /3.14.0/commons-lang3-3.14.0.jar (658 kB at 13 MB/s)
        [INFO] Building jar: /root/spring-petclinic/target/spring-petclinic-4.0.0-SNAPSHOT.jar
        [INFO]
        [INFO] --- spring-boot:4.0.3:repackage (repackage) @ spring-petclinic ---
        [INFO] Replacing main artifact /root/spring-petclinic/target/spring-petclinic-4.0.0-SNAPSHOT.jar with repackaged archive, adding nested dependencies in BOOT-INF/.
        [INFO] The original artifact has been renamed to /root/spring-petclinic/target/spring-petclinic-4.0.0-SNAPSHOT.jar.original
        [INFO] ------------------------------------------------------------------------
        [INFO] BUILD SUCCESS
        [INFO] ------------------------------------------------------------------------
        [INFO] Total time:  01:10 min
        [INFO] Finished at: 2026-04-15T10:41:50+09:00
        [INFO] ------------------------------------------------------------------------
        
        ```
        
    - target 폴더 안에 JAR 파일이 생성되었는지 확인
        
        ```bash
        ls target/*.jar
        ```
        
    
- PetClinic 실행
    - `--spring.profiles.active=mysql`을 붙여야 기본 메모리 DB가 아닌 **MySQL** 설정을 사용합니다.
    
    ```bash
    java -jar target/*.jar --spring.profiles.active=mysql &
    ```
    
    ```bash
    ....
    2026-04-15T10:44:42.852+09:00  INFO 74941 --- [           main] j.LocalContainerEntityManagerFactoryBean : Initialized JPA EntityManagerFactory for persistence unit 'default'
    2026-04-15T10:44:42.958+09:00  INFO 74941 --- [           main] o.s.d.j.r.query.QueryEnhancerFactories   : Hibernate is in classpath; If applicable, HQL parser will be used.
    2026-04-15T10:44:44.624+09:00  INFO 74941 --- [           main] o.s.b.a.e.web.EndpointLinksResolver      : Exposing 13 endpoints beneath base path '/actuator'
    2026-04-15T10:44:44.726+09:00  INFO 74941 --- [           main] o.s.boot.tomcat.TomcatWebServer          : Tomcat started on port 8080 (http) with context path '/'
    2026-04-15T10:44:44.739+09:00  INFO 74941 --- [           main] o.s.s.petclinic.PetClinicApplication     : Started PetClinicApplication in 8.427 seconds (process running for 9.072)
    ```
    
    - 서비스 동작 중인지 확인
    
    ```bash
    netstat -napt | grep 8080
    tcp6       0      0 :::8080                 :::*                    LISTEN      74941/java    
    ```
    
- 방화벽 해제
    
    ```bash
    systemctl stop firewalld 
    systemctl disable firewalld
    ```
    
- 웹 브라우저를 통해 연결되는지 확인
    
    [http://192.168.100.20:8080](http://192.168.100.20:8080/)
    
    <img width="526" height="446" alt="Image" src="https://github.com/user-attachments/assets/43807391-cbcd-4333-a951-58256894721c" />
    
    간단히 고객 정보 추가후 DB통해 확인 . application 서버에서 db 접속 성공하는지 확인
    
    ```bash
    **mysql -h 192.168.100.30 -u petclinic -p**
    
    Enter password:petclinic#!
    
    mysql> show databases;
    +--------------------+
    | Database           |
    +--------------------+
    | information_schema |
    | performance_schema |
    | petclinic          |
    +--------------------+
    3 rows in set (0.00 sec)
    
    mysql> use petclinic
    mysql> select * from owners;
    +----+------------+-----------+-----------------------+-------------+------------+
    | id | first_name | last_name | address               | city        | telephone  |
    +----+------------+-----------+-----------------------+-------------+------------+
    |  1 | George     | Franklin  | 110 W. Liberty St.    | Madison     | 6085551023 |
    |  2 | Betty      | Davis     | 638 Cardinal Ave.     | Sun Prairie | 6085551749 |
    ..
    | 10 | Carlos     | Estaban   | 2335 Independence La. | Waunakee    | 6085555487 |
    | 11 | test       | user      | add                   | add         | 0011112222 |
    +----+------------+-----------+-----------------------+-------------+------------+
    11 rows in set (0.00 sec)
    
    mysql> exit
    
    ```
    

<br>


### 3. Websever 구성: Apache+Reverse Proxy

- **Apache 리버스 프록시**
    
    리버스 프록시는 사용자의 요청을 대신 받아 앱 서버에 전달하고, 앱 서버의 응답을 다시 사용자에게 보내주는 **중계자 역할**을 합니다.
    
    - **보안**: 앱 서버의 IP와 포트(8080)를 외부에 노출하지 않고 웹 서버만 공개합니다.
    - **사용자 편의성**: 사용자가 주소창에 포트 번호(`:8080`)를 붙이지 않고 기본 80번 포트로 편하게 접속하게 해줍니다
- 웹 서버 설치
    
    ```bash
    sudo -i
    dnf -y install httpd
    ```
    
- Proxy 모듈 설치/활성화: 클라이언트 접속을 application server 로 전달
    
    ```bash
    dnf -y install mod_proxy_html
    ```
    
- Reverse Proxy 설정
    
    ```bash
    cat << EOF > /etc/httpd/conf.d/petclinic.conf
    <VirtualHost *:80>
        ServerName webserver
        ProxyPreserveHost On
        ProxyPass / http://192.168.100.20:8080/
        ProxyPassReverse / http://192.168.100.20:8080/
        ErrorLog /var/log/httpd/petclinic_error.log
        CustomLog /var/log/httpd/petclinic_access.log combined
    </VirtualHost>
    EOF
    ```
    
- 웹서버 데몬 실행 및 방화벽 해제
    
    ```bash
    systemctl enable --now httpd
    
    systemctl stop firewalld
    systemctl disable firewalld
    ```
    

<br>


### **참고: Spring Petclinic 부트 스크립트 생성하기**   (appliaction server)

- 실행 스크립트 만들기
    
    ```bash
    sudo mkdir -p /opt/petclinic
    sudo cp spring-petclinic/target/*.jar /opt/petclinic/petclinic.jar
    sudo vi /opt/petclinic/start.sh
    #!/bin/bash
    /usr/bin/java -jar /opt/petclinic/petclinic.jar   --spring.profiles.active=mysql
    sudo chmod +x /opt/petclinic/start.sh
    
    ```
    
- systemd 서비스 파일 생성
    
    ```bash
    cat << EOF > /etc/systemd/system/petclinic.service
    [Unit]
    Description=Spring Petclinic Application
    After=network.target
    
    [Service]
    User=root
    WorkingDirectory=/opt/petclinic
    ExecStart=/opt/petclinic/start.sh
    Restart=always
    RestartSec=5
    
    [Install]
    WantedBy=multi-user.target
    EOF
    ```
    
- 서비스 등록 및 시작
    
    ```bash
    sudo systemctl daemon-reload
    sudo systemctl enable petclinic
    sudo systemctl start petclinic
    sudo systemctl status petclinic
    ```
    
- 로그 확인
    
    ```bash
    journalctl -u petclinic -f
    ```
