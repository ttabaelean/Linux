### 1. 리눅스 시스템 로그 확인 실습

- 시스템에서 발생하는 다양한 활동 기록(로그)을 실시간으로 점검하고 분석합니다.
- 주요 로그 파일 내용 확인
    
    ```bash
    # 시스템 전체 메시지 확인: 커널 및 시스템 일반 메시지가 기록되는 로그를 확인합니다.
    sudo -i
    tail -f /var/log/messages
    
    # 인증 및 보안 로그 확인: ssh, sudo 등 사용자 로그인 및 인증 관련 기록을 점검합니다.
    tail -3 /var/log/secure
    
    # 크론탭(예약 작업) 로그 확인: 주기적으로 실행되는 자동화 작업의 성공 여부를 확인합니다.
    tail -3 /var/log/cron
    ```
    
- **바이너리 로그 및 로그인 정보 확인**
    
    ```bash
    # 로그인/재부팅 기록 확인: wtmp 파일을 참조하여 최근 로그인한 사용자 목록과 시스템 재부팅 기록을 확인합니다.
    last | head -2
    
    # 로그인 실패 기록 확인: btmp 파일을 참조하여 접속에 실패한 비정상적인 시도를 점검합니다.
    lastb | head -2
    ```
    

<br>


### 2. logrotate를 이용한 로그 관리 실습

- 로그 파일이 무한정 커져서 디스크 용량을 차지하지 않도록 순환(Rotate) 및 압축 설정을 점검합니다.
- **설정 파일 구조 및 정책 확인**•
    
    ```bash
    # 메인 설정 파일 확인: 로그 순환 주기, 보관 개수 등 전역 설정을 확인합니다.
    cat /etc/logrotate.conf
    
    # 개별 서비스 설정 확인: 웹 서버(httpd)나 시스템 로그(syslog) 등 서비스별로 분리된 상세 설정 파일을 확인합니다.
    ls -l /etc/logrotate.d/
    
    ```
    
- logrotate를 이용한 개별 서비스 로그 관리 실습
    - `/etc/logrotate.d/` 디렉토리에 새로운 설정 파일을 생성하여, 특정 로그 파일의 순환 주기, 보관 개수, 압축 여부를 직접 제어합니다.
    - Step 1. 실습용 가상 로그 파일 생성
        - 실습을 위해 용량이 큰 가상의 로그 파일을 만듭니다.
            
            ```bash
            # 10MB 크기의 가상 로그 파일 생성
            sudo dd if=/dev/zero of=/var/log/myapp.log bs=1M count=10
            ls -lh /var/log/myapp.log
            ```
            
    - Step 2. 개별 서비스용 logrotate 설정 파일 작성
        - `/etc/logrotate.d/` 경로에 `myapp`이라는 이름의 설정 파일을 생성합니다.
            
            ```bash
            # 아래 내용을 작성합니다.
            cat << EOF > /etc/logrotate.d/myapp
            /var/log/myapp.log {
                daily
                rotate 7
                compress
                delaycompress
                missingok
                notifempty
                create 0644 root root
            }
            EOF
            ```
            
    - Step 3. 설정 테스트 및 강제 실행
        - 로그 순환 주기가 되지 않았더라도, 명령어를 통해 설정이 올바른지 강제로 실행해 봅니다.
            
            ```bash
            # 1. 문법 오류가 없는지 테스트 (실제 실행은 안 함)
            logrotate -d /etc/logrotate.d/myapp
            
            # 2. 강제로 로그 순환 실행 (-f: force)
            logrotate -f /etc/logrotate.d/myapp
            ```
            
    - Step 4. 결과 확인
        - 로그 파일이 순환되어 새로운 파일이 생성되었는지 확인합니다.
            
            ```bash
            # 기존 myapp.log는 myapp.log.1로 바뀌고 새로운 myapp.log가 생성되었는지 확인
            ls -l /var/log/myapp.log*
            ```
            

<br>

    

### 3. rsyslog를 이용한 로그 관리 실습

- `rsyslog` 설정 파일을 수정하여 특정 수준(err) 이상의 로그만 별도의 파일(`/var/log/error.log`)에 기록하도록 설정하고, `logger` 명령어로 이를 검증합니다.
- Step 1. rsyslog 설정 파일 수정
    - 모든 서비스에서 발생하는 에러(`err`) 급 이상의 로그를 별도 파일에 저장하도록 규칙을 추가합니다.
        
        ```bash
        # rsyslog 설정 파일에 에러 로그 기록 규칙 추가
        # *.err 의미: 모든 서비스(Facility)의 에러(Priority) 수준 로그
        sudo -i
        echo "*.err    /var/log/error.log" >> /etc/rsyslog.conf
        
        # 설정이 잘 추가되었는지 확인
        grep ".err" /etc/rsyslog.conf
        ```
        
- Step 2. 서비스 재시작 및 설정 적용
    - 변경된 설정 내용을 시스템에 반영하기 위해 `rsyslog` 데몬을 재시작합니다.
        
        ```bash
        # rsyslog 서비스 재시작
        systemctl restart rsyslog.service
        ```
        
- Step 3. 테스트 로그 발생 (logger)
    - 시스템에 인위적으로 로그 메시지를 던지는 `logger` 명령어를 사용하여 설정이 작동하는지 테스트합니다.
        
        ```bash
        # 1. 에러 수준(err)의 테스트 로그 발생
        logger -p err -t ERR-TEST "test message : err level"
        
        # 2. 정보 수준(info)의 테스트 로그 발생 (설정상 기록되지 않아야 함)
        logger -p info -t INFO-TEST "test message : info level"
        ```
        
- Step 4. 결과 확인
    - 지정한 로그 파일에 에러 메시지만 정상적으로 기록되었는지 확인합니다.
        
        ```bash
        # 에러 로그 파일의 마지막 내용 확인
        tail /var/log/error.log
        ```

<br>

        

### 4. Systemd Journal(journalctl) 활용 실습

- **커널 및 시스템 로그 생성과 확인**
    - 인위적으로 로그를 발생시켜 Journal과 Syslog에 어떻게 기록되는지 비교합니다.
    - **커널 로그 확인**: `/dev/kmsg`에 테스트 메시지를 던진 후, `dmesg`와 `journalctl`로 확인합니다.
        
        ```bash
        # 커널 로그 생성 및 dmesg 확인
        echo "kernel message test" > /dev/kmsg
        dmesg | tail -n 1
        
        # journalctl을 이용한 커널 로그 확인 (-k: kernel, -r: 역순, -n 1: 1개만)
        journalctl -krn 1 --no-pager
        ```
        
    - **사용자 로그 확인**: `logger` 명령어로 로그를 생성하고 기록을 검증합니다.
        
        ```bash
        logger -t logger "Rsyslog message test"
        journalctl | grep logger
        grep Rsyslog /var/log/messages
        ```
        
    
- 로그 저장소 관리 및 영구 보존 설정
    - Journal 로그는 기본적으로 메모리(`run/log`)에 저장되어 재부팅 시 사라지지만, 설정을 통해 영구 보관할 수 있습니다.
    - **저장 위치 및 용량 확인**: 현재 로그가 차지하는 용량을 확인합니다.
        
        ```bash
        journalctl --disk-usage
        ls -lh /run/log/journal/*
        ```
        
    - **로그 영구 보관 설정**: `/var/log/journal` 디렉토리를 생성하여 재부팅 후에도 로그가 남도록 설정합니다.
        
        ```bash
        sudo mkdir /var/log/journal
        sudo systemctl restart systemd-journald
        ls -lh /var/log/journal/*
        ```
        
    
- 저널 로그 필터링 및 상세 검색
    - 특정 조건(시간, 서비스, 사용자 ID)을 부여하여 방대한 로그 속에서 필요한 정보만 빠르게 찾습니다.
    - **기본 필터링**: 최신순 정렬이나 상세 모드(verbose)로 출력합니다.
        
        ```bash
        # 최신 로그 10개 출력
        journalctl -r --no-pager -n 10
        # 로그의 모든 필드를 상세히 출력
        journalctl -o verbose -n 2
        ```
        
    - **조건별 검색**: 프로세스명, 특정 UID, 혹은 시간 범위를 지정합니다.
        
        ```bash
        # 특정 서비스(sshd) 및 UID(0: root) 로그 검색
        journalctl _COMM=sshd --no-pager
        journalctl _UID=0 --no-pager
        
        # 시간 범위 지정 검색
        journalctl --since=today
        journalctl --since "2025-04-30 00:00:00" --until "2025-05-01 10:30:00"
        journalctl --since "1 hour ago"
        ```
        

<br>

    

### **Hands-on LAB: 시스템 로그 통합 관리 및 트러블슈팅**

<aside>
📢 시나리오
최근 서버의 디스크 사용량이 급증하고 정체불명의 접속 시도가 보고되었습니다. 시스템 관리자인 당신은 1) 현재 발생 중인 시스템 로그를 실시간 분석하고, ) 로그 비대화를 막기 위한 자동 순환 정책을 수립해야 합니다. 또한, 3) 특정 에러만 별도로 수집하도록 시스템을 개편하고 4) 휘발성 저널 로그를 영구 보존하여 추후 보안 감사에 대비하십시오.

</aside>

---

<br>
 
**Mission 1. 실시간 로그 분석 및 보안 점검**

- 현재 시스템에서 발생하는 일반 메시지와 보안 위협 요소를 즉시 확인합니다.
    - **실시간 모니터링**: 커널 및 시스템의 일반적인 활동 기록을 실시간으로 추적하세요.
    - **보안 점검**: 로그인 실패 기록(`lastb`)을 확인하여 외부로부터의 무단 접속 시도가 있었는지 파악하세요.
    
    ```bash
    # 1. 시스템 전체 메시지 실시간 확인
    sudo -i
    tail -f /var/log/messages
    
    # 2. 로그인 실패 기록 확인 (상위 2개)
    lastb | head -2
    ```
    

<br>


**Mission 2. 개별 서비스 로그 순환 정책(logrotate) 적용**

- 특정 애플리케이션(`testapp`)의 로그 파일이 디스크를 점유하지 않도록 관리 정책을 생성합니다.
    - **로그 생성**: 테스트를 위한 10MB 크기의 가상 로그 파일(`testapp.log`)을 생성하세요.
    - **정책 수립**: 해당 로그를 매일 순환하고, 7개까지 보관하며 압축(`compress`)하도록 설정하세요.
    - **강제 실행**: 설정을 즉시 반영하여 파일이 분리되는지 확인하세요.
    
    ```bash
    # 1. 10MB 가상 로그 파일 생성
    dd if=/dev/zero of=/var/log/testapp.log bs=1M count=10
    
    # 2. logrotate 설정 파일 생성
    cat << EOF > /etc/logrotate.d/testapp
    /var/log/testapp.log {
        daily
        rotate 7
        compress
        delaycompress
        missingok
        notifempty
        create 0644 root root
    }
    EOF
    
    # 3. 강제 순환 실행 및 결과 확인
    logrotate -f /etc/logrotate.d/testapp
    ls -l /var/log/testapp.log*
    ```
    

<br>


**Mission3. rsyslog를 활용한 중요 에러 로그 별도 수집**

- 로그 필터링 규칙을 수정하여 심각한 오류(`err`) 상황을 별도로 관리합니다.•
    - **규칙 추가**: 모든 서비스의 에러 등급 로그만 `/var/log/error.log`에 따로 기록되도록 수정하세요.
    - **로그 테스트**: `logger` 명령어로 `err` 레벨과 `info` 레벨 로그를 각각 발생시켜 필터링이 작동하는지 검증하세요.
    
    ```bash
    # 1. rsyslog 에러 수집 규칙 추가 및 서비스 재시작
    echo "*.err    /var/log/error.log" >> /etc/rsyslog.conf
    systemctl restart rsyslog.service
    
    # 2. 에러 및 정보 로그 발생 테스트
    logger -p err -t ERR-TEST "test message : err level"
    logger -p info -t INFO-TEST "test message : info level"
    
    # 3. 결과 확인 (error.log에는 err 레벨만 있어야 함)
    tail /var/log/error.log
    ```
    

<br>


**Mission 4. Systemd 저널 로그 영구 보존 및 정밀 검색**

- 재부팅 시 사라지는 저널 로그를 디스크에 저장하고, 특정 조건으로 검색합니다.
    - **영구 보존**: 로그 저장 디렉토리(`journal`)를 수동 생성하여 데이터가 영구히 남도록 하세요.
    - **정밀 검색**: 오늘 발생한 로그 중 `sshd` 서비스와 관련된 기록만 상세 모드(`verbose`)로 출력하세요.
    
    ```bash
    # 1. 저널 로그 영구 보관 설정
    mkdir /var/log/journal
    systemctl restart systemd-journald
    
    # 2. 오늘 발생한 sshd 관련 상세 로그 검색
    journalctl -u sshd --since=today -o verbose
    ```
