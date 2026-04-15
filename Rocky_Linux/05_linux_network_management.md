### 1. 네트워크 관리 명령어

- 네트워크 인터페이스와 주소 보기 : **ip**
    
    ```bash
    # 현재 서버에 할당된 IP 주소와 인터페이스의 상태(UP/DOWN)를 확인합니다.
    ip addr
    
    # 외부망으로 나가는 길(Gateway) 확인하기
    ip route
    
    # 네트워크 연결확인
    ip link
    
    ```
    
- Hostname과 DNS 질의
    
    ```bash
    # 시스템의 현재 호스트 이름 확인하기
    hostname
    
    # 서버에 설정된 IP 주소를 호스트 명령어로 확인하기
    hostname -I
    
    # DNS 서버에 특정 도메인(예: google.com)의 IP 주소 물어보기
    nslookup google.com
    
    # nslookup 대화형 모드 진입 후 내부에서 server 입력하여 사용 중인 DNS 서버 확인
    nslookup
    > server
    > exit
    ```
    
- 네트워크 연결 확인 - ping 실습
    
    ```bash
    # 특정 IP(예: 구글 DNS)로 네트워크 연결이 잘 되는지 3번만 확인하기
    ping -c 3 8.8.8.8
    
    # 도메인 이름(www.google.com)을 사용하여 통신 상태 점검하기
    ping -c 3 www.google.com
    ```
    
- 네트워크 연결 확인 - netstat 실습
    
    ```bash
    # 현재 시스템의 라우팅 테이블(경로 정보) 확인하기
    netstat -nr
    
    # 현재 서버에서 대기 중(Listening)인 포트와 실행 프로그램 확인하기
    sudo netstat -nlpt
    
    # 특정 포트(예: 22번 포트)가 사용 중인지 확인하기
    netstat -an | grep :22
    
    # 네트워크 인터페이스별 송수신 패키의 통계 및 에러 확인하기
    netstat -i
    ```
    

<br>


### 2. 네트워크 설정 관리

- nmcli 실습
    
    ```bash
    # 현재 네트워크 장치의 물리적/논리적 상태 확인하기
    # 장치(Device)의 연결 상태와 하드웨어 명칭 확인
    nmcli device status
    
    # 논리적 연결(Connection) 프로필 목록 확인
    nmcli connection show
    
    # 기존 IP 주소를 유지하면서 보조 IP(Secondary) 추가하기
    # 기존 192.168.100.10 설정에 192.168.100.11 주소를 추가 (+ 기호 사용) [cite: 52]
    sudo nmcli con mod ens160 +ipv4.addresses 192.168.100.11/24
    
    # DNS 서버 주소를 구글(8.8.8.8)로 지정
    sudo nmcli con mod ens160 ipv4.dns "8.8.8.8"
    
    # IP 할당 방식을 수동(Static)으로 확정하고 적용하기
    # 할당 방식을 manual로 변경 (DHCP 사용 중단)
    sudo nmcli con mod ens160 ipv4.method manual
    
    # 변경된 설정을 인터페이스에 즉시 반영 (재시작)
    sudo nmcli con up ens160
    ```
    
- **네트워크 설정 확인 - 최종 검증 실습**
    
    ```bash
    # 인터페이스에 두 개의 IP(10.10, 10.11)가 모두 올라왔는지 확인
    ip -c addr show ens160
    
    # 설정된 기본 게이트웨이(Gateway) 정보가 유효한지 확인
    ip route
    
    # 외부 호스트(8.8.8.8)로 3번의 신호를 보내 응답 확인
    ping -c 3 8.8.8.8
    
    # 설정한 DNS가 도메인 명을 IP로 잘 변환하는지 확인
    nslookup google.com
    ```
    
- **추가한 보조 IP 주소 제거하기 (원복)**
    
    ```bash
    # 설정되어 있는 192.168.100.11 주소만 삭제 (- 기호 사용)
    sudo nmcli con mod ens160 -ipv4.addresses 192.168.100.11/24
    
    # 변경된 설정을 인터페이스에 즉시 반영 (재시작)
    sudo nmcli con up ens160
    
    # 인터페이스에 IP(10.10, 10.11)가 삭제되었는지 확인
    ip -c addr show ens160
    ```
    

<br>


### **Hands-on LAB: 신규 개발 서버 네트워크 구축 및 연결성 점검**

<aside>
📢 시나리오
새로운 프로젝트를 위해 애플리케이션 전용 서버(server2)가 할당되었습니다. 관리자는 이 서버가 외부 통신(DNS)이 가능한지 확인하고, 개발 효율을 위해 보조 IP를 추가로 할당해야 합니다. 또한, 현재 서버에서 열려 있는 보안 포트를 점검하여 개발자에게 안전한 환경을 인계하는 일련의 과정을 수행하십시오.

</aside>

---

<br>


**Mission 1. 서버 식별 정보 및 하드웨어 상태 점검**

- server2를 부팅한 후  user 계정으로 로그인 합니다.
- 개발자가 접속할 서버의 이름(Hostname)을 변경하고, 현재 물리적인 연결 상태를 확인합니다.
    - **서버 호스트 네임 변경:** 서버 식별을 위해 이름을 `application`으로 변경하세요.
    - **인터페이스 상태 확인:** `ip` 명령어를 사용하여 인터페이스의 MAC 주소와 활성화(`UP`) 여부를 확인하세요.
    - **라우팅 경로 확인:** 외부로 나가는 기본 게이트웨이가 설정되어 있는지 확인하세요.
        
        ```bash
        sudo hostnamectl hostname application
        ip link
        ip route
        ```
        

<br>


**Mission 2. 네트워크 수동 설정 및 다중 IP 할당 (nmcli)**

- 서버 운영 환경에 맞춰 고정 IP를 설정하고, 서비스 확장용 보조 IP를 추가합니다.
    - **고정 IP 및 보조 IP 추가:** 기존 `192.168.100.20` 주소는 유지하고, 보조 주소 `192.168.100.21`을 추가하세요.
    - **DNS 서버 설정:** 구글 DNS(`8.8.8.8`)를 네임서버로 지정하세요.
    - **설정 적용:** IP 할당 방식을 `manual`로 변경하고 설정을 즉시 활성화하세요.
        
        ```bash
        # 보조 IP 추가 (+ 기호 사용)
        sudo nmcli con mod ens160 +ipv4.addresses 192.168.100.21/24
        
        # DNS 설정 및 수동(Static) 모드 전환
        sudo nmcli con mod ens160 ipv4.dns "8.8.8.8" ipv4.method manual
        
        # 설정 반영
        sudo nmcli con up ens160
        ```
        

<br>


**Mission 3. 네트워크 무결성 검증 및 포트 점검 (Ping & Netstat)**

- 설정된 네트워크가 실제로 외부와 통신이 되는지, 보안상 열린 포트는 없는지 확인합니다.
    - **연결성 테스트:** 외부 호스트(`8.8.8.8`)로 3번의 패킷을 보내 응답 속도를 확인하세요.
    - **도메인 질의 확인:** `nslookup`을 사용하여 `google.com`의 IP를 정상적으로 받아오는지 확인하세요.
    - **포트 및 서비스 점검:** `netstat`을 사용하여 현재 서버에서 대기 중(`Listening`)인 포트와 해당 프로그램을 확인하세요.
        
        ```bash
        # 1. IP 할당 최종 확인
        ip -c addr show ens160
        
        # 2. 외부 연결성 및 DNS 확인
        ping -c 3 8.8.8.8
        nslookup google.com
        
        # 3. 오픈 포트 및 대기 서비스 확인 (관리자 권한)
        sudo netstat -nlpt
        ```
