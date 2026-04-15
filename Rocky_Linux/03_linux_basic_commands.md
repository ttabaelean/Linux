# 03. 서버 관리를 위한 기본 명령어

### 리눅스 사용자 권한 얻기

- sudoers 설정
    
    ```bash
    su -
    암호: <root 패스워드 입력:password>
    
    cat << EOF > /etc/sudoers.d/admin
    admin  ALL=(ALL)   NOPASSWD: ALL
    EOF
    
    cat /etc/sudoers.d/admin
    admin  ALL=(ALL)   NOPASSWD: ALL
    
    # sudo 명령어 사용
    
    ```
    
- sudo 명령 사용
    
    ```bash
    sudo --help
    sudo - 다른 사용자 권한으로 명령을 실행합니다
    
    # 권한 거부(Permission Denied) 확인
    ls -l /root
    
    sudo ls -l /root
    
    ```
    

<br>
  
### 시스템 정보 보기

- 시스템 기본 정보 (Identity & Kernel)
    
    ```bash
    # 리눅스 커널 버전과 아키텍처, 빌드 날짜를 확인
    uname -a
    
    # 호스트이름 확인 및 변경
    # 호스트이름을 server1 -> webserver로 변경
    hostname
    sudo hostnamectl hostname webserver
    hostname
    
    # 로그인 사용자의id id 확인
    # user: UID가 1000입니다. 일반 사용자는 보통 1000번부터 할당됩니다.
    id
    whoami
    
    # root 사용자의 id 확인
    # 리눅스에서 UID 0은 무소불위의 권력을 가진 관리자를 상징합니다.
    id root
    ```
    
- 하드웨어/리소스 정보 (Hardware & Resources)
    
    ```bash
    # CPU 정보 확인
    # 가상머신에 할당된 CPU 코어 개수와 모델명을 확인합니다.
    lscpu | grep -i "CPU(s):"
    lscpu | grep -i "Model name"
    
    # 메모리 사용량 확인 (-h: Human-readable, 사람이 읽기 편한 단위)
    # 전체 메모리(total) 대비 사용 중인 메모리(used)를 확인합니다.
    free -h
    
    # 디스크 사용량 확인
    # 현재 연결된 저장 장치들의 전체 용량과 남은 공간을 확인합니다.
    # / (루트) 파티션의 Use%가 100%에 가까워지면 시스템이 멈출 수 있습니다.
    df -h
    
    # 특정 디렉토리 사용량 확인 (s: summary, h: human-readable)
    # 현재 위치 또는 특정 경로가 차지하는 실제 디스크 크기를 계산합니다.
    sudo du -sh /var/log
    
    # 실시간 시스템 리소스 모니터링
    # 윈도우의 '작업 관리자'처럼 CPU/메모리 점유율을 실시간으로 확인합니다.
    # (종료하려면 키보드의 'q'를 누르세요)
    top
    ```
    

<br>
  
### 파일 및 디렉토리 관리

- 작업 공간 만들기 및 탐색
    
    ```bash
    # 현재 위치를 확인하고 홈 디렉토리(~)로 이동
    pwd
    cd ~
    
    # lab_dir이라는 이름의 디렉토리를 만들고 이동
    
    mkdir lab_dir
    cd lab_dir
    
    # 현재 디렉토리에 아무 파일도 없는지 상세 목록 확인
    ls -alh
    ```
    
- 파일 생성 및 편집(복사/이동/삭제)
    
    ```bash
    # test1.txt와 test2.txt라는 빈 파일 2개 만들기
    touch test1.txt test2.txt
    
    # test1.txt를 backup.txt라는 이름으로 복사
    cp test1.txt backup.txt
    ls
    
    # test2.txt의 이름을 final.txt로 변경
    mv test2.txt final.txt
    ls
    
    # backup.txt 파일을 삭제
    rm backup.txt
    ls
    ```
    
- 시스템 로그 실시간 관찰 (관리자 권한 필요)
    
    ```bash
    # 시스템의 최근 로그 20줄을 출력
    sudo tail -n 20 /var/log/messages
    
    # 시스템 로그를 실시간 모니터링 상태로 띄워두고, Ctrl+c를 눌러 종료
    sudo tail -f /var/log/messages
    ```
    
- 텍스트 검색 및 파일 찾기 (find & grep)
    
    ```bash
    # /etc 디렉토리 내에서 확장자가 .conf로 끝나는 모든 파일 찾기
    find /etc -name "*.conf"
    
    # /etc/passwd 파일에서 로그인 쉘이 /bin/bash인 사용자 라인만 골라서 출력
    grep "/bin/bash" /etc/passwd
    
    # 로그 파일에서 "error"가 포함된 모든 줄 출력
    grep "error" /var/log/messages
    sudo grep "error" /var/log/messages
    ```
    
- 파일 권한 및 소유권 변경
    
    ```bash
    # root 홈 디렉토리에 web_backup.sh라는 빈 파일 생성
    sudo -i
    cd /tmp
    pwd
    touch web_backup.sh
    ls -l web_backup.sh
    
    # 이 파일의 권한을 확인한 뒤, 소유자는 모든 권한(rwx)을 갖고 그룹과 나머지는 읽기 및 실행(r-x)만 가능하도록 변경
    chmod 755 web_backup.sh
    ls -l web_backup.sh
    
    # 파일의 소유권을 변경
    chown user:user web_backup.sh
    ls -l web_backup.sh
    ```

    
<br>
  

### [Hands-On LAB] 신규 프로젝트 개발자(developer) 환경 구축 실습

<aside>
📢시나리오
여러분은 인프라 관리자입니다. 새로운 프로젝트에 투입된 개발자 `developer`를 위해 서버 계정을 생성하고, 개발자가 스스로 시스템 상태를 확인하거나 백업 스크립트를 관리할 수 있도록 **권한 설정 및 초기 작업 공간**을 마련해 주어야 합니다.
    
</aside>


---

<br>
  

- Mission 1. 개발자 계정 생성 및 권한 부여 (sudoers)
    - `root` 계정으로 전환하고 `developer` 계정을 생성하고 비밀번호를 설정합니다.
        
        ```bash
        # root 계정으로 전환
        sudo -i
        whoami
        
        # developer 계정을 생성하고 비밀번호를 설정
        useradd developer
        echo "password123" | passwd --stdin developer
        ```
        
    - 개발자가 `sudo`를 사용할 수 있도록 설정 파일을 만듭니다.
        
        ```bash
        cat << EOF > /etc/sudoers.d/developer
        developer  ALL=(ALL)   NOPASSWD: ALL
        EOF
        
        exit
        ```
        
- Mission 2. 개발 환경 프로필 확인 (Identity & Hardware)
    - 개발자가 할당받은 서버의 사양을 직접 파악하여 애플리케이션 사양을 결정할 수 있도록 합니다.
    - **`developer`** 계정으로 전환합니다.
        
        ```bash
        su - developer
        password: password123
        ```
        
    - 현재 서버의 메모리 여유 공간(free)과 디스크 사용량(df)을 확인.
        
        ```bash
        free -h
        df -h
        ```
        
- Mission 3. 프로젝트 디렉토리 및 권한 관리 (chmod & chown)
    - 관리자가 미리 만들어둔 파일을 개발자가 소유권을 넘겨받아 직접 운영하는 실습입니다.
    - 관리자 권한으로 프로젝트 폴더를 만들고, 소유권을 개발자에게 넘깁니다.
        
        ```bash
        sudo mkdir /app
        sudo chown developer:developer /app
        ```
        
    - `/app` 디렉토리로 이동하여 개발용 백업 스크립트 `build.sh`를 만듭니다.
        
        ```bash
        cd /app
        touch build.sh
        ```
        
    - `build.sh` 파일이 실행 가능하도록 권한을 수정하세요. (소유자 전권 부여)
        
        ```bash
        chmod 755 build.sh
        ```
        
    - `ls -l`로 파일의 권한(`-rwxr-xr-x`)과 소유자(`developer`)를 최종 확인하세요.
        
        ```bash
        ls -l
        ```
        
- **Mission 4. 텍스트 필터링 및 실시간 로그 분석**
    - 개발자가 자신이 띄운 서비스의 에러를 찾기 위해 로그를 검색하고 관찰하는 연습입니다.
    - 시스템 설정 파일(`/etc/passwd`)에서 내 계정 정보(`developer`)가 포함된 라인만 찾아보세요.
        
        ```bash
        grep "developer" /etc/passwd
        ```
        
    - 최근 시스템 로그 중 "loggin" 메시지가 포함된 기록이 있는지 확인하여 로그인을 시도한 이력을 찾습니다.
        
        ```bash
        sudo grep "login" /var/log/messages
        ```
        
    - 실습종료
        
        ```bash
        exit
        ```
