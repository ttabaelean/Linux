# 02. 리눅스 로그인과 기본 환경 구성

### **1. 가상 머신(server1) 시스템 구성**

- ssh로 root 원격 로그인 허용
    
    ```bash
    cat /etc/ssh/sshd_config | grep -i root
    
    vi /etc/ssh/sshd_config
    ..
    PermitRootLogin yes
    
    systemctl restart sshd
    ```
    
- user 사용자에게 root 권한 할당
    
    ```bash
    # sudoers 설정
    vi /etc/sudoers
    ...
    user  ALL=(ALL)       NOPASSWD: ALL
    ```
    
- hosts 파일에 호스트이름과 IP 주소 설정
    
    ```bash
    # hosts 설정
    vi /etc/hosts
    ...
    192.168.100.10 server1
    192.168.100.20 server2
    192.168.100.30 database
    ```
    
- SELinux 비 활성화
    
    ```bash
    # SELinux disable 설정
    sudo sed -i s/^SELINUX=.*$/SELINUX=disabled/ /etc/selinux/config
    sudo setenforce 0
    ```
    
- 패키지 설치
    
    ```bash
    #서버 운영에 필수적인 도구를 설치
    sudo dnf -y install git wget net-tools sysstat
    ```
    
- 네트워크 상태 확인
    
    ```bash
    # 호스트 이름 확인 및 변경
    hostname
    ip addr
    ping 8.8.8.8  -c 3
    ```
    
- 시스템 종료
    
    ```bash
    # 시스템 종료
    sudo init 0
    
    ```
    

### 2. server2 설정

- 호스트 이름 설정
    
    ```bash
    hostnamectl hostname server2
    hostname
    
    ip addr
    
    ping -c 8.8.8.8
    ```
    

### 3. database 설정

- 호스트 이름 설정
    
    ```bash
    hostnamectl hostname database
    hostname
    
    ip addr
    
    ping -c 8.8.8.8
    ```
