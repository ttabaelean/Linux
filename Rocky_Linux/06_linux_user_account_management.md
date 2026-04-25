### 1. 사용자 계정 기본 정보

- server1에 user 계정으로 로그인 한 후 다음의 명령어를 실습하세요.
    
    ```bash
    #/etc/passwd 파일을 통해 특정 사용자(예: developer)의 UID, 홈 디렉토리, 기본 쉘을 확인합니다.
    grep "developer" /etc/passwd
    
    # /etc/shadow 파일을 통해 암호의 암호화 여부 및 최종 변경일, 만료 예정일 등을 확인합니다.
    sudo grep "developer" /etc/shadow
    
    # /etc/group 파일을 통해 특정 그룹의 GID와 해당 그룹에 속한 보조 그룹 멤버 목록을 확인합니다.
    grep "developer" /etc/group
    
    # /etc/gshadow 파일을 통해 그룹의 관리자 권한을 가진 계정과 그룹 암호 설정 여부를 확인합니다.
    sudo grep "developer" /etc/gshadow
    
    # 시스템 명령어를 통해 현재 세션의 UID, GID, 소속된 모든 그룹 정보를 한눈에 출력합니다.
    id
    
    ```
    

<br>


### 2. 로그인 사용자 정보 확인 실습

- **현재 접속 중인 사용자 요약 정보 확인**
    - 현재 시스템에 로그인한 사용자들의 이름, 터미널 번호, 접속 시간을 확인합니다.
        
        ```bash
        # 현재 로그인한 사용자의 목록을 간단히 출력합니다.
        who
        
        # 현재 로그인한 사용자의 상세 정보와 시스템 부하(load average), 현재 실행 중인 작업을 확인합니다.
        w
        ```
        
- **특정 사용자의 상세 정보 및 시스템 상태 확인**
    - 항목별 제목을 포함하여 가독성을 높이거나, 특정 계정의 활동만 집중적으로 분석합니다.
        
        ```bash
        # 출력 항목의 제목(NAME, LINE, TIME 등)을 함께 표시하여 정보를 확인합니다.
        who -H
        
        # 특정 사용자(예: user)가 현재 어떤 명령어를 실행하고 있는지 상세히 확인합니다.
        w user
        
        # 일부 항목을 제외한 짧은 형식으로 사용자 활동 정보만 간략히 확인합니다.
        w -s
        ```
        
- **시스템 환경 및 세션 정보 확인**
    - 현재 사용자의 계정 정보와 시스템의 마지막 부팅 시간, 런레벨 등을 확인합니다.
        
        ```bash
        # 내가 현재 로그인한 계정의 정보만 확인합니다.
        who -m
        
        # 시스템이 마지막으로 재부팅된 날짜와 시간을 확인합니다.
        who -b
        
        # 현재 시스템의 운영 모드(Run-level)를 확인합니다.
        who -r
        
        # 시스템 명령어를 통해 현재 세션의 UID, GID, 소속된 그룹 정보를 한눈에 출력합니다.
        id
        
        ```
        

<br>


### 3.  계정 관리 명령어

- **리눅스 그룹 관리 실습 (groupadd, groupmod, groupdel)**
    - 프로젝트 팀 단위의 권한 관리를 위해 새로운 그룹을 생성하고 정보를 수정하는 과정을 실습합니다.
        
        ```bash
        # 새로운 프로젝트 그룹(devteam)을 특정 GID와 함께 생성합니다.
        sudo groupadd -g 4000 devteam
        grep devteam /etc/group
        
        # 그룹의 이름을 변경(devteam -> devops)하고 GID를 수정합니다.
        # 그룹 이름을 devteam으로 변경합니다.
        sudo groupmod -n devops devteam
        grep devteam /etc/group
        grep devops /etc/group
        
        # 변경된 그룹의 GID를 4050으로 수정합니다.
        sudo groupmod -g 4050 devops
        grep "devops" /etc/group
        ```
        
- **리눅스 사용자 관리 실습 (useradd, usermod, userdel)**
    - 사용자 계정을 생성하고, 앞서 만든 그룹에 소속시키거나 계정 설정을 변경하는 과정을 실습합니다.
    - 새로운 사용자(devuser01)를 생성하면서 보조 그룹(devops)에 지정합니다.
        
        ```bash
        # 계정 생성 시 -G 옵션으로 보조 그룹을 지정합니다.
        sudo useradd -G devops devuser01
        grep devuser01 /etc/passwd
        tail -5 /etc/group
        
        # 생성된 사용자의 암호를 설정합니다.
        echo "password123" | sudo passwd --stdin devuser01
        sudo grep devuser01 /etc/shadow
        ```
        
    - 기존 사용자의 설정을 수정(홈 디렉토리 변경 또는 그룹 추가)합니다.
        
        ```bash
        # 사용자의 코멘트(설명) 정보를 수정합니다.
        sudo usermod -c "Project Developer" devuser01
        grep devuser01 /etc/passwd
        
        # 사용자를 새로운 추가 그룹에 포함시킵니다.
        sudo usermod -aG wheel devuser01
        grep wheel /etc/group
        id devuse01

        # 사용자에게 할당된 그룹을 제거합니다.
        sudo gpasswd -d devuser01 wheel
        grep wheel /etc/group
        id devuse01        
        ```
        
    - 사용자 계정의 상세 정보와 소속 그룹을 최종 확인합니다.
        
        ```bash
        # /etc/passwd 파일에서 계정 정보를 확인합니다.
        grep "devuser01" /etc/passwd
        
        # id 명령어로 소속된 모든 그룹(GID)을 확인합니다.
        id devuser01
        ```
        
    - 사용자 계정을 삭제합니다. (홈 디렉토리까지 모두 삭제)
        
        ```bash
        sudo userdel -r devuser01
        ```
        

<br>


### 4. 패스워드 관리

- 사용자 계정의 암호를 새롭게 설정하거나 변경합니다.
    - **일반적인 변경**: `sudo passwd devuser01` 명령 후 대화형으로 암호를 입력합니다.
    - **자동화 방식 (stdin)**: 파이프를 이용해 한 줄로 암호를 즉시 설정합니다.
        
        ```bash
        # devuser01의 암호를 "pass123"으로 자동 설정합니다.
        sudo useradd devuser01
        echo "pass123" | sudo passwd --stdin devuser01
        ```
        
- 사용자 계정의 암호 및 계정 만료 정보를 상세히 확인합니다.
    - `chage -l` 옵션을 사용하여 현재 적용된 정책(만료일, 변경 주기 등)을 출력합니다.
        
        ```bash
        sudo chage -l devuser01
        sudo grep devuser01 /etc/shadow
        ```
        
    - **기업 보안 정책에 맞춰 암호 변경 주기와 계정 만료일을 설정합니다.**
        - **최대 사용 기간(-M)**: 암호를 90일마다 바꾸도록 설정합니다.
        - **계정 만료일(-E)**: 특정 날짜(예: 프로젝트 종료일)에 계정이 자동으로 잠기도록 설정합니다.
        
        ```bash
        # 암호 최대 사용 기간 90일, 계정 만료일을 2026-12-31로 지정합니다.
        sudo chage -M 90 -E 2026-12-31 devuser01
        ```
        
    - **설정된 보안 정책이 시스템 파일에 어떻게 기록되었는지 검증합니다.**
    • `/etc/shadow` 파일의 필드 값 변화를 확인하여 정책 반영 여부를 체크합니다.
        
        ```bash
        sudo grep "devuser01" /etc/shadow
        ```
        
        • 실습에 사용한 계정을 삭제합니다.(홈 디렉토리까지 모두 삭제)
        
        ```bash
        sudo userdel -r devuser01
        sudo groupdel devops
        ```
        

<br>


### **Hands-on LAB: 신규 프로젝트 팀 계정 생성 및 보안 정책 적용**

<aside>
📢 시나리오
당신은 시스템 엔지니어로서 새로운 보안 프로젝트 팀을 위한 인프라 환경을 구축해야 합니다. 보안 팀용 그룹을 생성하고, 신입 사원인 `smlee` 계정을 생성하여 해당 그룹에 배정하세요. 또한, 기업 보안 규정에 따라 암호 만료 정책을 적용하고 현재 접속 중인 사용자들의 활동을 점검하여 안전한 운영 환경을 조성하십시오.

</aside>

---

<br>


**Mission 1. 프로젝트 그룹 생성 및 관리 (Group Management)**

- 보안 프로젝트를 위한 전용 그룹을 생성하고, 관리 체계 변경에 따라 정보를 수정합니다.
    - **그룹 생성**: GID `4000`번을 가진 `devops` 그룹을 생성하세요.
    - **그룹 수정**: 팀 명칭 변경 계획에 따라 그룹 이름을 `devteam`으로, GID를 `4050`으로 변경하세요.
    - **그룹 확인**: `/etc/group` 파일을 통해 변경 사항이 정상 반영되었는지 검토하세요.
    
    ```bash
    # 1. 그룹 생성
    sudo groupadd -g 4000 devops
    grep devops /etc/group
    
    # 2. 그룹 이름 및 GID 수정
    sudo groupmod -n devteam devops
    sudo groupmod -g 4050 devteam
    
    # 3. 설정 확인
    grep "devteam" /etc/group
    ```
    

<br>


**Mission 2. 사용자 계정 생성 및 보안 정책 적용 (User & Password)**

- 신규 사용자 `smlee`를 생성하여 프로젝트 그룹에 배정하고, 엄격한 암호 정책을 적용합니다.
    - **계정 생성**: `smlee` 사용자를 생성하고 보조 그룹으로 `devteam`을 지정하세요.
    - **암호 설정**: 대화형 방식이 아닌 한 줄 명령어로 `pass123` 암호를 즉시 부여하세요.
    - **보안 정책**: 암호를 90일마다 변경하도록 설정하고, 2026년 12월 31일에 계정이 만료되도록 조치하세요.
    
    ```bash
    # 1. 사용자 추가 및 그룹 배정
    sudo useradd -G devteam smlee
    grep smlee /etc/passwd
    grep smlee /etc/group
    
    # 2. 패스워드 자동 설정
    echo "pass123" | sudo passwd --stdin smlee
    
    # 3. 암호 및 계정 만료 정책 적용
    sudo chage -M 90 -E 2026-12-31 smlee
    
    # 4. 정책 적용 상태 상세 확인
    sudo chage -l smlee
    ```
    

<br>


**Mission 3. 실시간 접속 사용자 점검 및 계정 정리 (Monitoring & Clean up)**

- 현재 시스템을 사용하는 인원을 점검하고, 프로젝트 종료를 가정하여 계정을 삭제합니다.
    - **사용자 점검**: 현재 누가 접속 중이며, 어떤 작업을 수행하여 부하를 주는지 확인하세요.
    - **계정 삭제**: 실습이 완료된 `smlee` 계정과 `devteam` 그룹을 삭제하세요. (홈 디렉토리 포함)
    
    ```bash
    # 1. 현재 접속 사용자 및 실행 중인 작업 확인
    who -H
    w
    
    # 2. 사용자 계정 삭제 (홈 디렉토리 포함)
    sudo userdel -r smlee
    
    # 3. 그룹 삭제
    sudo groupdel devteam
    ```
