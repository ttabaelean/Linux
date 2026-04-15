### 1. 프로세스 정보보기

- 프로세스 계층 구조 시각화 : pstree
    
    ```bash
    # 시스템 전체 프로세스의 계층 구조와 PID를 확인합니다.
    pstree -p
    
    # developer 사용자가 실행한 프로세스 계층만 확인합니다.
    pstree developer -p
    ```
    
- 프로세스 목록 보기 : ps
    - user 사용자 계정으로 다음의 ps 명령을 실습합니다.
        
        ```bash
        # 현재 터미널에서 동작중인 프로세스 보기
        ps
        
        # 현재 터미널에서 동작중인 프로세스 정보 자세히 보기
        ps -f
        
        # 시스템 내부에서 동작중인 모든 프로세스 보기
        ps -ef
        
        # 전체 프로세스 중 상위 10개만 출력
        ps -ef | head
        
        # 현재 시스템에서 실행 중인 sshd(원격 접속 서비스) 관련 프로세스만 찾기
        ps -ef | grep sshd
        ```
        
    - 두 번째 터미널에 developer / password123 로그인하고 sleep 명령을 실행합니다.
        
        ```bash
        whoami
        developer
        
        sleep 100 &
        ```
        
    - 다시 user 계정 터미널로 이동
        
        ```bash
        # developer사용자가 실행 중인 프로세스를 상세히 출력
        ps -u developer -f
        
        # CPU 사용율(%CPU)이 높은 순서대로 프로세스를 정렬하여 상위 5개를 확인
        ps aux --sort=-%cpu | head -n 6
        ```
        

  <br>
  
### 2. 프로세스 관리 명령어

- htop 사용 예
    - EPEL 저장소 활성화 후 htop 설치
    
    ```bash
    # 관리자 권한으로 EPEL 저장소 설치
    sudo dnf install -y epel-release
    
    # htop 패키지 설치
    sudo dnf install -y htop
    ```
    
    - htop 실행
    
    ```bash
    # htop 실행
    htop
    ```
    
    - **화면 구성 변경:** `htop` 실행 중 `F2` (Setup)를 눌러 상단 바의 항목을 추가하거나 테마(Colors)를 변경해 보세요.
    - **프로세스 필터링:** `F3`을 누르고 `sshd`를 입력하여 원격 접속 프로세스만 화면에 남겨보세요.
    - **정렬 기준 변경:** `F6`을 누른 뒤 `PERCENT_MEM`을 선택하여 메모리 사용량이 높은 순으로 정렬하세요.
    - **종료:** **`F10`** 또는 `q`를 눌러 프로그램을 빠져나옵니다.

- lsof
    
    ```bash
    # 현재 내 쉘(bash)이 어떤 파일들을 열고 있는지 확인
    lsof -p $$
    
    # 현재 시스템에서 네트워크 연결(Internet) 상태인 프로세스만 출력
    sudo lsof -i
    
    # 22번 포트(sshd)를 점유 중인 프로세스 찾기
    sudo lsof -i :22
    
    # developer 계정이 실행 중인 모든 파일 보기
    lsof -u developer
    ```
    

  <br>
  
### **Hands-on LAB: 프로세스 트러블슈팅 및 최적화**

<aside>
📢 시나리오
`developer` 사용자가 개발 중 실수(또는 테스트)로 CPU 자원을 과다하게 사용하는 무한 루프 프로그램을 실행했습니다. 관리자는 시스템 성능 저하 문제를 해결하기 위해 이 프로세스를 찾아내고 안전하게 종료해야 합니다..

</aside>

  <br>
  
사전 설정 : developer 터미널 (부하 발생)

- 먼저 `developer` 계정으로 로그인된 터미널에서 다음 명령을 실행하여 고의로 부하를 만듭니다.
    
    ```bash
    # CPU 부하를 일으키는 무한 루프 프로세스를 백그라운드에서 실행합니다.
    # /dev/null은 출력 결과물들을 버리는 쓰레기통 역할을 합니다.
    yes > /dev/null &
    yes > /dev/null &
    
    # yes는 특정 문자열을 무한히 출력하는 명령어로, 백그라운드(&)에서 실행하면 CPU 코어 하나를 가득 점유하게 됩니다.
    ```
    
  <br>
  

**Mission 1. 시스템 부하 원인 파악 (top & htop)**

- 관리자 터미널에서 시스템 상태를 모니터링하여 범인을 찾습니다.
    1.  `htop` 명령을 실행하여 CPU 사용률이 100%에 근접한 프로세스를 찾습니다.   
    2. 해당 프로세스를 실행한 사용자가 `developer`가 맞는지 확인하고 **PID**를 메모합니다. 100087ㅂ
    3. `F5` (Tree)를 눌러 이 프로세스가 어떤 부모(`bash`)로부터 시작되었는지 확인하세요. 


  <br>
  
**Mission 2. 프로세스 상세 분석 및 파일 추적 (ps & lsof)**

- 선택한 프로세스가 정말 종료해도 되는 프로세스인지 더 자세히 검사합니다.
    
    ```bash
    # 메모한 PID를 사용하여 해당 프로세스의 시작 시간과 상세 명령어를 확인하세요.
    ps -fp [PID]
    
    # 해당 프로세스가 어떤 자원을 건드리고 있는지 확인하여 시스템에 치명적인 작업인지 판단합니다.
    sudo lsof -p [PID]
    ```

  <br>
  

**Mission 3. 프로세스 강제 종료 및 결과 검증 (kill)**

- 부하의 원인인 `yes` 프로세스를 종료하여 시스템을 정상화합니다.
    
    ```bash
    # 먼저 SIGTERM (15)을 보내 프로세스가 스스로 종료될 기회를 줍니다.
    sudo kill -15 [PID]
    
    # 만약 종료되지 않는다면 SIGKILL(9)을 사용하여 즉시 강제 종료하세요.
    sudo kill -9 [PID]
    
    # 다시 htop이나 top을 실행하여 CPU 사용률이 정상(0~수 %)으로 돌아왔는지 확인합니다.
    
    ```
