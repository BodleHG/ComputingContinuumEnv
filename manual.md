## **CLUSTER 구성**
- Control plane
  - AI Server : 192.168.50.200
- Worker Node
  - workstation : 192.168.50.201
  - odyssey1(ubuntu) : 192.168.50.202
  - odyssey2(ubuntu) : 192.168.50.203
  - raspberrypi4(ubuntu) : 192.168.50.204
  - raspberrypi3(raspbian) : 192.168.50.205

## **OpenSSH 설정**

- Windows
  1. 설정 -> 시스템 -> 선택적 기능 -> 선택적 기능 추가 -> OpenSSH Server 다운
  2. 혹은, 아래 명령어 입력
    ``` powershell
    $ Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
    ``` 
  3. ssh 서비스 시작        
    ``` powershell
    $ Start-Service sshd
    ```
  4. 시스템 시작 시 ssh 서비스 자동 시작
    ```powershell
    $ Set-Service -Name sshd -StartupType 'Automatic'
    ```
  5. TCP 인바운드 규칙 설정
    ```powershell
    $ New-NetFirewallRule -Name sshd -DisplayName 'OpenSSH-Server-In-TCP' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
    ```

- Ubuntu
  1. Open SSH Server 설치
    ```bash
    $ sudo apt update
    $ sudo apt install openssh-server
    ```
  2. SSH Server 실행
    ```bash
    $ sudo systemctl enable ssh
    $ sudo systemctl start ssh
    ```
  3. 방화벽 허용
    ```bash
    $ sudo ufw allow ssh
    ```

## **Docker Desktop 설치 및 쿠버네티스 구축(Only WorkerNode)**
1. wsl 업데이트(wsl 초기설정이 안되어있을 때)
    ```powershell
    $ wsl --update
    ```
2. wsl 설치
    ```powershell
    $ wsl --install
    ```
3. PC 재부팅
4. [Docker Desktop 설치](https://www.docker.com/products/docker-desktop) 설치
5. Docker Desktop -> setting -> kubernetes 활성화





