### 1. 리눅스 디스크 정보 보기 실습

- 시스템의 전체적인 디스크 사용 현황과 물리적인 장치 연결 상태를 점검합니다.
- **파일시스템별 디스크 공간 사용 현황 출력**
    
    ```bash
    df -h
    ```
    
- 특정 디렉토리(/var/log)가 차지하는 실제 디스크 사용량 확인
    
    ```bash
    sudo du -h /var/log
    ```
    
- 현재 연결된 모든 블럭 디바이스(디스크, 파티션) 목록 확인
    
    ```bash
    lsblk
    ```
    
- 디스크 장치의 고유 식별 번호(UUID) 및 파일시스템 타입 확인
    
    ```bash
    sudo blkid
    ```
    

<br>


### 2. 리눅스 파일 시스템 추가 실습

- VMWare에서 5GB 디스크를 새롭게 추가하였고, 해당 디스크를 사용할 수 있도록 만드는 전체 워크플로우 실습입니다.
- Step 1: 디스크 파티션 나누기 (fdisk)
    - 새로 장착된 `/dev/nvme0n2` 디스크를 논리적으로 분할합니다.
    
    ```bash
    sudo fdisk /dev/nvme0n2 
    # Command: n (새 파티션 생성) -> p (기본 파티션) -> 1 (번호) -> Enter -> +2G (크기 설정) [cite: 491-495]
    # Command: w 
    
    lsblk
    ```
    
- Step 2: 파일시스템 생성/포맷 (mkfs)
    - 생성한 파티션(`/dev/nvme0n2p1`)을 리눅스에서 사용할 수 있도록 XFS 형식으로 포맷합니다.
    
    ```bash
    sudo mkfs -t xfs /dev/nvme0n2p1
    ```
    
- Step 3: 장치 마운트 (mount)
    - 포맷된 장치를 시스템의 특정 디렉토리에 연결합니다.
    
    ```bash
    # 1. 마운트할 디렉토리 생성
    sudo mkdir /backup
    
    # 2. 장치와 디렉토리 연결
    sudo mount /dev/nvme0n2p1 /backup/
    
    # 3. 마운트 결과 확인
    df -h /backup/
    ```
    
- Step 4: 부팅 시 자동 마운트 설정 (/etc/fstab)
    - 시스템이 재부팅 되어도 자동으로 마운트 되도록 설정 파일을 편집합니다.
    
    ```bash
    # 1. /etc/fstab 파일 하단에 설정 추가 [cite: 619, 620]
    # 형식: [장치명] [마운트지점] [파일시스템] [옵션] [덤프/체크순서]
    sudo vi /etc/fstab
    ...
    /dev/nvme0n2p1  /backup  xfs  defaults  0 0
    
    # 2. 변경된 설정 시스템에 반영
    sudo systemctl daemon-reload
    
    # 3. 자동 마운트 테스트
    sudo mount -a
    
    # 4. 마운트 결과 확인
    mount | tail -l
    ls -l /backup
    ```
    

<br>


### **Hands-on LAB: 로그 데이터를 위한 파일시스템 추가**

<aside>
📢 시나리오
관리자는 새롭게 추가한 5GB 디스크(`/dev/nvme0n2`) 중 2GB만 `/backup`으로 사용하고 있습니다. 이제 남은 3GB의 파티션 공간을 활용하여, 시스템 로그 데이터가 급증할 경우를 대비한 로그 전용 저장소(`/logdata`)를 구축하고 시스템 운영의 안정성을 확보하십시오.

</aside>

---

<br>



**Mission 1. 디스크 사용 현황 및 가용 공간 점검**

- 추가 파티션 작업을 하기 전, 현재 시스템의 물리적 연결 상태와 사용 중인 파티션 정보를 확인합니다.
    - **연결 상태 확인**: 현재 `/dev/nvme0n2` 디스크의 전체 크기와 이미 생성된 `p1` 파티션의 크기를 대조하세요.
    
    ```bash
    # 1. 전체 블럭 디바이스 목록 및 트리 구조 확인
    sudo -i
    lsblk
    
    ```
    

<br>


**Mission 2. 남은 공간을 활용한 두 번째 파티션 생성 (fdisk)**

- 디스크의 나머지 3GB 영역을 새로운 논리 파티션으로 분할합니다.
    - **파티션 추가**: `n` 명령어를 사용하여 남은 모든 섹터를 할당하는 두 번째 파티션(`p2`)을 생성하세요.
    - **설정 저장**: 변경된 파티션 테이블을 디스크에 기록하고 커널이 인식하도록 하세요.
    
    ```bash
    # 1. 파티션 관리 도구 실행
    fdisk /dev/nvme0n2
    
    # 2. fdisk 내부 작업 단계
    # Command: n
    # Select: p 
    # Partition number: 2
    # First sector: (엔터 - p1 이후 시작점 자동 선택)
    # Last sector: (엔터 - 남은 모든 공간 할당)
    # Command: w (저장 및 종료)
    
    # 3. 새로운 파티션(nvme0n2p2) 생성 결과 확인
    lsblk
    ```
    

<br>


**Mission3. 파일시스템 생성 및 로그 전용 디렉토리 연결 (Mount)**

- 운영체제가 데이터를 쓰고 읽을 수 있도록 포맷하고 시스템에 연결합니다.
    - **포맷**: 새로운 파티션을 고성능 `xfs` 파일시스템으로 포맷하세요.
    - **마운트**: 시스템 운영 로그를 저장할 `/logdata` 디렉토리를 생성하고 장치를 연결하세요.
        
        ```bash
        # 1. xfs 파일시스템 생성 (포맷)
        mkfs -t xfs /dev/nvme0n2p2
        
        # 2. 마운트 포인트 생성 및 장치 연결
        mkdir /logdata
        mount /dev/nvme0n2p2 /logdata
        
        # 3. 연결 상태 및 용량(약 3GiB) 확인
        df -hT /logdata
        ```
        

<br>


**Mission 4. 시스템 무결성을 위한 자동 마운트 설정 (/etc/fstab)*

- 서버가 재부팅되어도 로그 저장소가 항상 유지되도록 설정합니다.
    - **식별자 확인**: 장치명 대신 보안성이 높은 `UUID`를 사용하여 등록을 준비하세요.
    - **설정 반영**: `/etc/fstab`에 등록 후 시스템 데몬을 리로드하여 설정을 확정하세요.
        
        ```bash
        # 1. 새로 생성한 p2 파티션의 UUID 확인
        blkid /dev/nvme0n2p2
        
        # 2. /etc/fstab 파일 편집 (UUID 예시: a1b2c3d4...)
         vi /etc/fstab
        UUID=확인된_UUID /logdata xfs defaults 0 0
        
        # 3. 설정 반영 및 자동 마운트 테스트
        systemctl daemon-reload
        mount -a
        
        # 4. 최종 마운트 정보 요약 확인
        mount | grep logdata
        ```
