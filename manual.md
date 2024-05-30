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

ssh 구성이 변경되어 공개키 삭제 해야될 시(Host key verification failed)
```ssh-keygen -R 대상IP```

## **Ubuntu 노트북 대기모드 해제 (덮개 닫아도 전원 유지)**

logind.conf 파일을 연 후, 24번째의 HandleLidSwitch 값을 ignore로 변경
```bash
$ sudo vi /etc/systemd/logind.conf
$ systemctl restart systemd-logind
```

<img src = "https://github.com/BodleHG/ComputingContinuumEnv/assets/89232601/c38a0d22-316c-46e0-8742-c48393d67205">


## **Ubuntu Nvidia 그래픽카드 드라이버 설정**

사용중인 그래픽 카드 모델, 그래픽 드라이버 확인
```bash
$ sudo lshw -c display
```
설치가능한 NVIDIA 그래픽 드라이버 목록
```bash
$ sudo ubuntu-drivers devices
```
목록 중 그래픽 드라이버 설치 (Open 버전의 경우 일부 모니터 인식이 안될 수 있음)
```bash
$ sudo apt install nvidia-driver-535
```
시스템 재부팅
```bash
$ sudo shutdown now
```
재부팅 시 무한 로딩 혹은 검은 화면이 계속된다면 BIOS 메뉴에서 **Secure Boot 해제** 후 재부팅


NVIDIA 그래픽 드라이버 확인
```bash
$ nvidia-smi
```
```bash
$ sudo lshw -c display
```

## **모든 쿠버네티스 참여 노드 설정**
Ubnutu Swap 메모리 해제
```bash
$ sudo swapoff -a
```
부팅 시 Ubnutu Swap 메모리 해제

(/etc/fstab 파일 내부의 2번째 라인 맨 앞에 #을 붙이라는 명령어로, 장치에 따라 2번째 라인에 루트 파티션에 대한 내용이 있을 수도 있음. **반드시 /etc/fstab 파일 내용 확인**)
```bash
# 아래 코드 주석 처리
/swapfile none swap sw 0 0

# 혹은 아래 코드 실행 (권장하지 않음)
$ sed -i '2s/^/#/' /etc/fstab
```
<img src = "https://github.com/BodleHG/ComputingContinuumEnv/assets/89232601/7b418629-c4f1-424c-be10-eed9e0a70d9e">

스왑 메모리 비활성화 확인
```bash
$ free -h
```

네트워크 설정
br_netfilter 요소를 활성화하여 iptables가 bridge 인터페이스의 패킷을 제어하도록 함. 이를 통해 bridge 인터페이스를 사용하는 프로세스(파드, 컨테이너)간 통신 가능
```bash
$ sudo modprobe br_netfilter # br_netfilter 요소 활성화

# 노드가 재부팅되더라도 br_netfilter가 활성화하도록 설정
$ sudo su 
$ echo "br_netfilter" > /etc/modules-load.d/br_netfilter.conf
$ exit
```

br_netfilter 설정 확인
```bash
$ lsmod 
```
<img src = "https://github.com/BodleHG/ComputingContinuumEnv/assets/89232601/c90e4082-15f6-4c6e-a251-af4ea1740a8d">

```bash
$ modinfo br_netfilter
```
<img src = "https://github.com/BodleHG/ComputingContinuumEnv/assets/89232601/a0e3b724-4152-430a-84f2-e263fca8fba4">

iptables에서 br_netfilter 사용 활성화
```bash
$ cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

# 설정 적용
$ sudo sysctl --system
```

iptables 설정 되었는지 확인
```bash
$ cd /proc/sys/net/bridge
$ ls -ahl
$ cat bridge-nf-call-arptables
$ cat bridge-nf-call-ip6tables
$ cd ~
```
<img src = "https://github.com/BodleHG/ComputingContinuumEnv/assets/89232601/59be9595-611c-4e2d-b114-f14c63358396">
<img src = "https://github.com/BodleHG/ComputingContinuumEnv/assets/89232601/5b0ee4f6-e781-4a5e-a31d-e843d4b00768">


컨테이너 런타임 설치(containerd)
```bash
$ sudo apt-get -y update
$ sudo apt-get install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
$ sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg
$ sudo add-apt-repository -y "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

$ sudo apt-get -y update
$ sudo apt-get install -y containerd.io
$ containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
$ sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml

$ sudo systemctl restart containerd
$ sudo systemctl enable containerd
```

노드의 IP 설정
```bash
$ sudo su

$ cat <<-'EOF' >/etc/default/kubelet
KUBELET_EXTRA_ARGS=--node-ip=자신의ip
EOF

# 예시
$ cat <<-'EOF' >/etc/default/kubelet
KUBELET_EXTRA_ARGS=--node-ip=192.168.50.202
EOF

$ exit
```

kubelet, kubeadm, kubectl 설치
```bash
$ sudo apt-get update
apt 업데이트, ca 관련 패키지 다운로드 설정
$ sudo apt-get install -y apt-transport-https ca-certificates curl gpg

$ sudo mkdir -p -m 755 /etc/apt/keyrings

$ curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

$ echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

$ sudo apt-get update

$ sudo apt-get install -y kubelet kubeadm kubectl

$ sudo apt-mark hold kubelet kubeadm kubectl
```

## **Control Plane 구성**
kubeadm을 통한 클러스터 초기화, control plane role 배정 (Flannel CNI 사용)
```bash
# --pod-network-cidr=10.244.0.0/16 : Flannel에서 기본적으로 권장하는 네트워크 대역 설정
# --control-plane-endpoint : control plane의 IP 주소
# --apiserver-advertise-address : control plane의 IP 주소
$ sudo kubeadm init \
--pod-network-cidr=10.244.0.0/16 \
--control-plane-endpoint=192.168.50.201 \
--apiserver-advertise-address=192.168.50.201
```

위 명령어 실행 시 시간이 좀 걸림(여러 이미지 파일 다운로드)

다음과 같은 화면 보이면, 설치가 성공한 것임.

발행된 토큰 반드시 저장(노드 추가 시 필요)

<img src = "https://github.com/BodleHG/ComputingContinuumEnv/assets/89232601/978d20a1-8072-4f56-9875-3161c2c1ab8d">

kubeadm 토큰 발행
```bash
# 다른 control plane 추가
$ kubeadm join 192.168.50.201:6443 --token nqv7wq.qofb6unn1vmlxmu1 \
        --discovery-token-ca-cert-hash sha256:9e02209e5b997a874e48708b6e423144d0bacb43bedfcf0777b1deeb70460b3b \
        --control-plane

# Worker Node 추가
$ kubeadm join 192.168.50.201:6443 --token nqv7wq.qofb6unn1vmlxmu1 \
        --discovery-token-ca-cert-hash sha256:9e02209e5b997a874e48708b6e423144d0bacb43bedfcf0777b1deeb70460b3b
```

kubectl 명령어 수행을 위한 권한 부여
```bash
$ mkdir -p $HOME/.kube
$ sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
$ sudo chown -R $(계정이름):$(계정이름) /home/$(계정이름)/.kube
# 예시
$ sudo chown -R sysailab612:sysailab612 /home/sysailab612/.kube
```

Flannel CNI 설치 및 적용
```bash
$ kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

CNI 설치 후 잠깐 기다려야 함

노드의 ROLE 및 STATUS READY 확인
```bash
$ kubectl get node
```

<img src = "https://github.com/BodleHG/ComputingContinuumEnv/assets/89232601/d7b78dc8-b5dd-40cb-a086-d56338eaf726">


### 노드의 Role 설정

참여 노드의 ROLE 변경
```bash
$ kubectl label node $(NodeName) node-role.kubernetes.io/$(Role)=
# 예시
$ kubectl label node 612odyssey1 node-role.kubernetes.io/worker=
```


## **Worker Node 구성**
발행된 control plane의 토큰을 이용하여 join
```bash
#예시
$ sudo kubeadm join 192.168.50.201:6443 --token nqv7wq.qofb6unn1vmlxmu1 \
        --discovery-token-ca-cert-hash sha256:9e02209e5b997a874e48708b6e423144d0bacb43bedfcf0777b1deeb70460b3b
```

Control Plane에서 worker node join 여부 확인
```bash
$ kubectl get node
```
<img src = "https://github.com/BodleHG/ComputingContinuumEnv/assets/89232601/d7b78dc8-b5dd-40cb-a086-d56338eaf726">

### 부록 - Worker Node에서 kubectl 명령어를 사용하고 싶다면

Control Plane의 admin.conf 파일을 Worker Node에 저장하여, 권한을 부여해야 함
```bash
# worker node에서
$ mkdir -p $HOME/.kube
# control plane에서
$ sudo scp /etc/kubernetes/admin.conf $(controlplane계정)@$(controlplaneIP주소):$HOME/.kube/config
# 예시
$ sudo scp /etc/kubernetes/admin.conf sysailab612@192.168.50.202:$HOME/.kube/config
# worker node에서
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
# worker node에서
$ sudo chown -R $(계정이름):$(계정이름) /home/$(계정이름)/.kube
# 예시
$ sudo chown -R sysailab612:sysailab612 /home/sysailab612/.kube
```

## **삭제, 초기화 관련**

kubeadm을 이용해 구축한 클러스터 초기화
```bash
$ sudo kubeadm reset
$ sudo rm -r /etc/cni/net.d/*
$ sudo rm -r /etc/cni/net.d
$ sudo rm -r ~/.kube/config
$ sudo systemctl restart kubelet
```

kubernetes 삭제
```bash
sudo systemctl stop kubelet
sudo systemctl stop docker

sudo rm -rf /var/lib/cni/
sudo rm -rf /var/lib/kubelet/
sudo rm -rf /run/calico   # 설치한 cni를 삭제
sudo rm -rf /etc/cni/
sudo rm -rf /etc/kubernetes
sudo rm -rf /root/.kube/
sudo rm -rf /root/.k8s/
sudo rm -rf /var/lib/etcd/

sudo ip link delete cni0   # 설치한 cni의 네트워크 인터페이스 삭제
sudo ip link delete calico   # 설치한 cni의 네트워크 인터페이스 삭제

sudo apt remove --purge kubelet kubectl kubeadm

sudo apt autoremove

systemctl start docker
```

## 우분투 공장초기화
```bash
sudo dpkg --configure -a
sudo apt-get update
sudo apt-get -f install
sudo apt-get full-upgrade
sudo apt-get install --reinstall ubuntu-desktop
sudo apt-get autoremove
sudo apt-get clean
```