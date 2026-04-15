### 1. RPM 패키지 관리 실습

- 패키지 파일 준비 및 정보 확인
    - 공식 저장소에서 설치 파일을 다운로드하고, 설치 전 패키지에 담긴 정보를 미리 확인합니다.
    - **패키지 다운로드**: `dnf download`를 사용하여 설치용 `.rpm` 파일을 가져옵니다.
    - **설치 전 정보 조회**: `-qpi` 옵션을 사용하면 파일 형태의 패키지 상세 정보(이름, 버전, 릴리즈 등)를 확인할 수 있습니다.
    
    ```bash
    # zsh 패키지 파일을 현재 디렉토리에 다운로드합니다.
    sudo -i
    dnf download zsh
    ls -l zsh*
    
    # 다운로드한 파일의 상세 정보를 설치 전에 확인합니다.
    rpm -qpi zsh-*.rpm
    ```
    
- 패키지 설치 및 결과 검증
    - 준비된 `.rpm` 파일을 시스템에 설치하고 정상적으로 반영되었는지 체크합니다.
    - **패키지 설치**: `-ivh` 옵션을 사용하여 설치 과정을 시각적으로 확인하며 진행합니다.
    - **설치 여부 확인**: `-q` 옵션으로 특정 패키지가 데이터베이스에 등록되었는지 조회합니다.
    
    ```bash
    # 다운로드한 rpm 파일을 관리자 권한으로 설치합니다.
    sudo rpm -ivh zsh-*.rpm
    
    # 설치가 완료된 패키지의 버전 정보를 확인합니다.
    rpm -q zsh
    ```
    
- 상세 정보 조회 및 목록 관리
    - 설치된 패키지의 상세 명세와 시스템 전체 설치 목록을 관리합니다.
    - **상세 내역 출력**: `-qi` 옵션으로 설치된 패키지의 아키텍처, 빌드 날짜, 설명 등을 확인합니다.
    - **전체 목록 검색**: `-qa` 옵션과 `grep`을 조합하여 특정 패키지 포함 여부를 검색합니다.
    
    ```bash
    # 설치된 zsh 패키지의 상세 정보를 출력합니다.
    rpm -qi zsh
    
    # 시스템 전체 설치 목록에서 zsh 관련 항목을 검색합니다.
    rpm -qa | grep zsh
    ```
    
- 패키지 제거 (삭제)
    - 더 이상 필요하지 않은 패키지를 시스템에서 안전하게 삭제합니다.
    - **패키지 삭제**: `-e` 옵션을 사용하여 설치된 패키지를 제거합니다.
    
    ```bash
    # 설치된 zsh 패키지를 관리자 권한으로 삭제합니다.
    sudo rpm -e zsh
    
    # 삭제가 잘 되었는지 다시 확인합니다.
    rpm -q zsh
    ```
    

<br>


### 2. dnf 패키지 관리 실습

- 패키지 탐색 및 정보 확인
    - 설치 전 저장소에 있는 패키지를 검색하고 상세 정보를 확인합니다.
    - **패키지 검색**: `search` 명령어를 사용하여 키워드와 관련된 패키지를 찾습니다.
    - **상세 정보 조회**: `info` 명령어로 패키지의 버전, 크기, 설명 등을 확인합니다.
    
    ```bash
    # nano 에디터 관련 패키지가 있는지 검색합니다.
    dnf search nano
    
    # 검색된 nano 패키지의 상세 정보를 확인합니다.
    dnf info nano
    ```
    
- 패키지 설치 및 목록 관리
    - 원하는 패키지를 시스템에 설치하고 설치된 내역을 확인합니다.
    - **패키지 설치**: `install` 명령어를 사용하여 패키지를 설치하며, `sudo` 권한이 필요합니다.
    - **설치 목록 확인**: `list installed`를 통해 시스템에 실제 설치된 패키지만 골라 확인합니다.
    
    ```bash
    # nano 패키지를 관리자 권한으로 설치합니다.
    sudo dnf install nano -y
    
    # 설치된 전체 패키지 목록 중 nano가 포함되어 있는지 확인합니다.
    dnf list installed | grep nano
    ```
    
- 패키지 업데이트 및 시스템 유지보수
    - 설치된 패키지를 최신 버전으로 유지하거나 전체 패키지 리스트를 확인합니다.
    - **패키지 업데이트**: `update` 명령어로 최신 보안 패치나 기능이 적용된 버전으로 갱신합니다.
    - **전체 목록 보기**: `list` 명령어로 저장소에서 제공하는 모든 패키지 상태를 확인합니다.
        
        ```bash
        # 설치된 nano 패키지를 최신 버전으로 업데이트합니다.
        sudo dnf update nano
        
        # 저장소의 전체 패키지 목록을 출력합니다.
        dnf list
        ```
        
- 패키지 제거 (삭제)
    - 더 이상 사용하지 않는 패키지를 안전하게 삭제합니다.
    - **패키지 삭제**: `remove` 명령어를 사용하여 패키지를 제거합니다.
        
        ```bash
        # 설치된 nano 패키지를 관리자 권한으로 삭제합니다.
        sudo dnf remove nano -y
        
        # 삭제 후 패키지 정보가 사라졌는지 다시 확인합니다.
        dnf info nano
        ```
        

<br>


### 3.  dnf 저장소(Repository) 관리 실습

- **현재 활성화된 저장소 목록 확인**
    - 시스템이 패키지를 다운로드할 수 있도록 연결된 저장소 리스트를 확인합니다.
    - **저장소 리스트 확인**: `dnf repolist` 명령어를 사용하여 현재 사용 가능한 저장소 ID와 이름을 출력합니다.
        
        ```bash
        # 현재 시스템에 등록되어 활성화된 저장소 목록을 확인합니다.
        dnf repolist
        ```
        
- **특정 저장소 비활성화 및 활성화 (config-manager)**
    - 필요에 따라 특정 저장소를 일시적으로 끄거나 켤 수 있습니다.
    - **저장소 비활성화**: `config-manager --set-disabled` 옵션으로 특정 저장소(예: appstream)를 리스트에서 제외합니다.
    - **저장소 활성화**: `config-manager --set-enabled` 옵션으로 다시 저장소를 사용 가능한 상태로 복구합니다.
        
        ```bash
        # appstream 저장소를 비활성화합니다.
        sudo dnf config-manager --set-disabled appstream
        
        # 비활성화 여부를 다시 확인합니다. (목록에서 사라짐)
        dnf repolist
        
        # appstream 저장소를 다시 활성화합니다.
        sudo dnf config-manager --set-enabled appstream
        ```
        
- **외부 저장소 추가 및 확장 (EPEL)**
    - 기본 저장소에서 제공하지 않는 다양한 패키지(예: htop, nodejs 관련 도구 등)를 사용하기 위해 외부 저장소를 추가합니다.
    - **EPEL 저장소 설치**: `epel-release` 패키지를 설치하여 엔터프라이즈 리눅스용 추가 패키지 저장소를 등록합니다.
        
        ```bash
        # EPEL 저장소 패키지를 설치합니다.
        sudo dnf install epel-release -y
        
        # 새롭게 추가된 epel 저장소가 리스트에 나타나는지 확인합니다.
        dnf repolist
        ```
        
- **확장된 저장소를 이용한 패키지 검색**
    - 새로 추가된 저장소를 통해 이전에는 찾을 수 없었던 패키지들을 검색해 봅니다.
    - **패키지 검색**: `yum search` (또는 `dnf search`) 명령어로 새 저장소에 포함된 패키지를 찾아봅니다.
        
        ```bash
        # epel 저장소 추가 후 node 관련 패키지를 검색해 봅니다.
        yum search nodejs
        ```
        

<br>


### 4. systemctl 명령어 활용 실습

- 서비스 상태 확인 및 시작/중지
    - 설치된 서비스(데몬)의 현재 상태를 점검하고 제어합니다.
    - **상태 확인**: `status` 명령을 통해 서비스의 가동 여부(Active), PID, 최근 로그 등을 확인합니다.
    - **시작 및 중지**: `start`와 `stop` 명령으로 서비스의 동작을 수동으로 제어합니다.
        
        ```bash
        # 아파티 패키지 설치
        sudo dnf install -y httpd
        sudo dnf list httpd
        
        # httpd(웹 서버) 서비스의 현재 상태를 상세히 확인합니다.
        sudo systemctl status httpd
        
        # 중지되어 있는 httpd 서비스를 시작합니다.
        sudo systemctl start httpd
        netstat -napt | grep 80
        curl localhost
        
        # 동작 중인 httpd 서비스를 중지합니다.
        sudo systemctl stop httpd
        ```
        
- 서비스 재시작 및 설정 반영
    - 서비스가 오동작하거나 설정 파일을 수정했을 때 사용합니다.
    - **재시작**: `restart`는 서비스를 완전히 껐다가 다시 켭니다.
    - **설정 반영**: `reload`는 서비스를 중지하지 않고 수정된 설정 값만 다시 불러옵니다.
        
        ```bash
        # 서비스를 중지 후 다시 시작하여 깨끗한 상태로 만듭니다.
        sudo systemctl restart httpd
        
        # 연결을 끊지 않고 수정된 설정 파일 내용만 적용합니다.
        sudo systemctl reload httpd
        ```
        
- 부팅 시 자동 실행 설정
    - 시스템이 재부팅되어도 서비스가 자동으로 실행되도록 관리합니다.
    - **자동 실행 등록**: `enable`을 사용하여 부팅 시 서비스가 자동 시작되도록 설정합니다.
    - **자동 실행 해제**: `disable`을 사용하여 더 이상 자동 실행되지 않도록 설정합니다.
        
        ```bash
        # 시스템 재부팅 후에도 httpd가 자동으로 실행되도록 등록합니다.
        sudo systemctl enable httpd
        
        # 등록된 자동 실행 설정을 해제합니다.
        sudo systemctl disable httpd
        
        # [꿀팁] 현재 실행과 자동 실행 등록을 한 번에 처리합니다.
        sudo systemctl enable --now httpd
        ```
        
- 전체 유닛 목록 확인
    - 시스템에서 관리 중인 전체 서비스 유닛들의 목록을 조회합니다.
    - **유닛 목록 조회**: `list-units` 명령으로 현재 로드된 모든 유닛 상태를 확인합니다.
        
        ```bash
        # 현재 시스템에서 활성화되어 있는 유닛 목록을 출력합니다.
        systemctl list-units
        ```
        

<br>


### **Hands-on LAB: 시스템 패키지 최적화 및 신규 서비스 배포**

<aside>
📢 시나리오
개발자(`developer`)로부터 특정 프로젝트를 위해 **Python 3.9** 설치와 실행 환경 구축을 요청받았습니다. 관리자는 `dnf`를 통해 안정적인 패키지를 설치하고, 개발자가 만든 가상의 Python 스크립트가 시스템 부팅 시마다 자동으로 실행될 수 있도록 **시스템 데몬(Service)**로 등록하여 관리하십시오.

</aside>

---

<br>


**Mission 1. Python 패키지 탐색 및 설치 (dnf)**

- 저장소에서 제공하는 Python 버전을 확인하고 개발자가 요청한 환경을 조성합니다.
    - **패키지 검색**: 저장소에 설치 가능한 `python3` 관련 패키지 리스트를 확인하세요.
    - **상세 정보 조회**: 설치할 `python3` 패키지의 버전과 용량 정보를 미리 확인하세요.
    - **패키지 설치**: `dnf`를 사용하여 Python 3를 설치하세요.
        
        ```bash
        # 1. Python3 관련 패키지 검색
        sudo -i
        dnf search python3
        
        # 2. 설치할 패키지의 상세 정보 확인
        dnf info python3
        
        # 3. Python3 설치 (확인 질문 생략 옵션 -y 사용)
        dnf install python3 -y
        ```

<br>

        

**Mission 2. 개발자(developer) 환경 확인 및 스크립트 준비**

- 개발자 계정으로 전환하여 Python이 정상 작동하는지 확인하고, 실습용 가상 스크립트를 만듭니다.
    - **버전 확인**: 설치된 Python의 버전을 확인합니다.
    - **테스트 스크립트 생성**: 무한 루프로 실행되는 간단한 Python 파일을 생성합니다.
        
        ```bash
        # developer 계정으로 전환 (없을 경우 sudo useradd developer로 생성 후 진행)
        su - developer
        
        # Python 설치 버전 확인
        python3 --version
        
        # 테스트용 스크립트 생성 (10초마다 로그를 남기는 무한 루프)
        echo "import time
        while True:
            print('Python Service is running...')
            time.sleep(10)" > ~/dev_app.py
        
        exit
        ```
        

<br>


**Mission 3. Python 애플리케이션의 서비스화 (systemctl)**

- 관리자 권한으로 위 스크립트를 시스템 서비스 유닛으로 등록하여 관리합니다.
    - **유닛 파일 생성**: `/etc/systemd/system/` 경로에 `pyapp.service` 파일을 생성합니다.
    - **서비스 등록 및 가동**: 작성한 서비스를 로드하고 시작하세요.
    - **자동 실행 설정**: 서버 재시작 시에도 해당 Python 앱이 자동 실행되도록 설정하세요.
        
        ```bash
        # 1. 서비스 설정 파일 작성 (관리자 권한)
        cat > /etc/systemd/system/pyapp.service << EOF
        [Unit]
        Description=Python Development App
        After=network.target
        
        [Service]
        ExecStart=/usr/bin/python3 /home/developer/dev_app.py
        User=developer
        Restart=always
        
        [Install]
        WantedBy=multi-user.target
        EOF
        
        # 2. 시스템 데몬 설정 리로드 및 서비스 시작
        systemctl daemon-reload
        systemctl enable --now pyapp.service
        ```
        
    

<br>


**Mission 4. 서비스 상태 모니터링 및 검증**

- 등록된 Python 서비스가 리소스를 적절히 사용하며 잘 돌아가는지 최종 점검합니다.
    - **동작 상태 확인**: 서비스가 `active (running)` 상태인지 확인하세요.
    - **프로세스 추적**: `ps` 또는 `htop`을 사용하여 `developer` 사용자가 실행 중인 Python 프로세스를 찾으세요.
    - **로그 확인**: `journalctl`을 사용하여 서비스가 남기는 메시지를 실시간으로 모니터링하세요.
        
        ```bash
        # 1. 서비스 가동 상태 확인
        systemctl status pyapp.service
        
        # 2. 프로세스 리스트에서 확인
        ps -ef | grep dev_app.py
        
        # 3. 실시간 서비스 로그 모니터링
        journalctl -u pyapp.service -f
        ```
        
    - Ctrl + c 를 눌러 로그 모니터링을 종료합니다.
