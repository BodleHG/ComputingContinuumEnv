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
Ubnutu Swap 메모리 해제 *권장하지 않음*

(/etc/fstab 파일 내부의 2번째 라인 맨 앞에 #을 붙이라는 명령어로, 장치에 따라 2번째 라인에 루트 파티션에 대한 내용이 있을 수도 있음. **반드시 /etc/fstab 파일 내용 확인**)
```bash
$ sed -i '2s/^/#/' /etc/fstab
```

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



노드간 통신을 위한 iptables 에 브릿지 관련 설정 추가
```bash
$ cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF
```

```bash
$ sudo modprobe br_netfilter
$ sudo su
$ echo "br_netfilter" > /etc/modules-load.d/br_netfilter.conf
$ exit
```



```
$ cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
$ sudo sysctl --system
```

kubeadm, kubelet, kubectl 설치
```bash
$ sudo apt-get update
# apt 업데이트, ca 관련 패키지 다운로드 설정
$ sudo apt-get install -y apt-transport-https ca-certificates curl gpg

$ sudo mkdir -p -m 755 /etc/apt/keyrings

$ curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

$ echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

$ sudo apt-get update

$ sudo apt-get install -y kubelet kubeadm kubectl

$ kubeadm version
$ kubelet --version
$ kubectl version
```

Docker Deamon이 사용하는 드라이버를 cgroupfs대신 systemd를 사용하도록 설정

(kubernetes에서 권장하는 드라이버가 systemd이기 때문)

```bash
$ sudo mkdir /etc/docker
cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF

# $ sudo mkdir -p /etc/systemd/system/docker.service.d
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker
$ sudo systemctl restart kubelet
```

## **Control Plane 구성**
kubeadm을 통한 네트워크 설정 (Flannel Pod 네트워크 애드온 사용)
```bash
$ sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address={Host's IP Address}
```

kubeadm 에러 발생 시(Ubuntu 20.04 / containerd.io 1.3.7 이상 발생 에러)
```bash
$ sudo rm /etc/containerd/config.toml
$ sudo systemctl restart containerd
$ sudo kubeadm init
```

아래 예시와 같은 토큰 발행 시 복사
```bash
$ kubeadm join 192.168.50.201:6443 --token 70sj08.96cywuuzxroks8wm \
        --discovery-token-ca-cert-hash sha256:f6858f1ba3012d0186e6749d28b977903ee202acd53c42e688869e7964fd7ad9
```

모든 사용자가 kube 명령어를 사용 가능하도록 설정
```bash
$ sudo mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

```bash
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
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

---
우분투 공장초기화
sudo dpkg --configure -a
sudo apt-get update
sudo apt-get -f install
sudo apt-get full-upgrade
sudo apt-get install --reinstall ubuntu-desktop
sudo apt-get autoremove
sudo apt-get clean



---
# READY 상태 된 테스트본
Ubnutu Swap 메모리 해제
```bash
$ sudo swapoff -a
```
Ubnutu Swap 메모리 해제 *권장하지 않음*

(/etc/fstab 파일 내부의 2번째 라인 맨 앞에 #을 붙이라는 명령어로, 장치에 따라 2번째 라인에 루트 파티션에 대한 내용이 있을 수도 있음. **반드시 /etc/fstab 파일 내용 확인**)
```bash
$ sed -i '2s/^/#/' /etc/fstab
```

스왑 메모리 비활성화 확인
```bash
$ free -h
```

sudo modprobe br_netfilter
sudo su
echo "br_netfilter" > /etc/modules-load.d/br_netfilter.conf
exit

lsmod # br_netfilter 상태 확인
modinfo br_netfilter # netfilter 상태 확인


cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system


cd /proc/sys/net/bridge # 설정 잘 됐는지 확인
ls -ahl
cat bridge-nf-call-arptables
cat bridge-nf-call-ip6tables
cd ~

컨테이너 런타임 설치
sudo apt-get -y update
sudo apt-get install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg
sudo add-apt-repository -y "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

sudo apt-get -y update
sudo apt-get install -y containerd.io
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml

sudo systemctl restart containerd
sudo systemctl enable containerd


IP 설정
sudo su

자신의 ip
cat <<-'EOF' >/etc/default/kubelet
KUBELET_EXTRA_ARGS=--node-ip=자신의ip
EOF

cat <<-'EOF' >/etc/default/kubelet
KUBELET_EXTRA_ARGS=--node-ip=192.168.50.202
EOF

exit

$ sudo apt-get update
apt 업데이트, ca 관련 패키지 다운로드 설정
$ sudo apt-get install -y apt-transport-https ca-certificates curl gpg

$ sudo mkdir -p -m 755 /etc/apt/keyrings

$ curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

$ echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

$ sudo apt-get update

$ sudo apt-get install -y kubelet kubeadm kubectl

sudo apt-mark hold kubelet kubeadm kubectl


## 컨트롤 플레인 설정
sudo kubeadm init \
--pod-network-cidr=10.244.0.0/16 \
--control-plane-endpoint=192.168.50.201 \
--apiserver-advertise-address=192.168.50.201

시간이 좀 걸림

$ mkdir -p $HOME/.kube
$ sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
sudo chown -R sysailab612:sysailab612 /home/sysailab612/.kube

kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

## 워커노드
sudo kubeadm join 192.168.50.201:6443 --token nqv7wq.qofb6unn1vmlxmu1 \
        --discovery-token-ca-cert-hash sha256:9e02209e5b997a874e48708b6e423144d0bacb43bedfcf0777b1deeb70460b3b

worker node에서
mkdir -p $HOME/.kube
contron plane에서
sudo scp /etc/kubernetes/admin.conf sysailab612@192.168.50.202:$HOME/.kube/config
worker node에서
sudo chown $(id -u):$(id -g) $HOME/.kube/config
sudo chown -R sysailab612:sysailab612 /home/sysailab612/.kube