## 쿠버네티스 대시보드

대시보드 Service 추가
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```

대시보드 Service 확인
```bash
kubectl get svc -n kubernetes-dashboard
```

대시보드 yaml파일 수정
```bash
kubectl edit svc kubernetes-dashboard -n kubernetes-dashboard
```
위 이미지에서 아래 이미지로 yaml 수정

<img src = "https://github.com/BodleHG/ComputingContinuumEnv/assets/89232601/a039d7ce-ceba-44f7-a03b-e44f61deb995">

<img src = "https://github.com/BodleHG/ComputingContinuumEnv/assets/89232601/da0325d7-4135-481c-b9e4-4e4bd58ca2ae">

Service Account 설정
```bash
cat <<EOF | kubectl create -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
EOF
```

Cluster Role 설정
```bash
cat <<EOF | kubectl create -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
EOF
```

토큰 발행
```bash
kubectl -n kubernetes-dashboard create token admin-user
```

발행된 토큰을 통해 대시보드 접근


## Grafana & Prometheus 설치

helm 설치
```bash

# helm 설치
$ wget  https://get.helm.sh/helm-v3.9.4-linux-amd64.tar.gz
$ tar xvfz helm-v3.9.4-linux-amd64.tar.gz
$ sudo mv linux-amd64/helm /usr/bin/

# helm repo 추가
$ helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

# helm repo Update
$ helm repo update
```

kube-prometheus-stack 다운로드
```bash
$ helm pull prometheus-community/kube-prometheus-stack
# ls 명령어로 kube-prometheus-stack-xx.x.x.tgz의 버전 확인
$ ls
$ tar xfz kube-prometheus-stack-59.0.0.tgz
$ cd kube-prometheus-stack/
```

게정 비밀번호 수정
```bash
adminPassword: sysailab@612
```

<img src = "https://github.com/BodleHG/ComputingContinuumEnv/assets/89232601/c860d3cd-7408-4492-bad0-9f090d6641a9">

```bash
kubectl create namespace monitoring
```

```bash
helm install prometheus . -n monitoring -f values.yaml
```

설치 확인
```bash
helm list -n monitoring
kubectl get pod -n monitoring
kubectl get svc -n monitoring
```

```bash
kubectl edit service -n monitoring prometheus-grafana
```

여기서는 grafana 포트를 31001로 설정
<img src = "https://github.com/BodleHG/ComputingContinuumEnv/assets/89232601/e280e69a-869b-4255-a64f-adc2d93ce0c3">



```bash
kubectl edit service -n monitoring prometheus-kube-prometheus-prometheus
```

여기서는 Prometheus 포트를 30090으로 설정

<img src = "https://github.com/BodleHG/ComputingContinuumEnv/assets/89232601/62e6f011-27f5-42f8-a18a-41d28312dadd">


<img src = "https://github.com/BodleHG/ComputingContinuumEnv/assets/89232601/cf702069-2f26-42aa-b60c-8278d3e2197a">

<img src = "https://github.com/BodleHG/ComputingContinuumEnv/assets/89232601/05ac700e-31e1-4433-acd2-f2a23ab80826">

<img src = "https://github.com/BodleHG/ComputingContinuumEnv/assets/89232601/99703ade-7208-420c-86d9-69f266bd1c37">

<img src = "https://github.com/BodleHG/ComputingContinuumEnv/assets/89232601/4430cc44-2a08-42c8-adb2-6eb31ab2e78a">

<img src = "https://github.com/BodleHG/ComputingContinuumEnv/assets/89232601/33c09c49-e011-43d2-b86e-91c30662cdf5">

https://grafana.com/grafana/dashboards/15759-kubernetes-views-nodes/

<img src = "https://github.com/BodleHG/ComputingContinuumEnv/assets/89232601/2c2689c9-db47-415d-b9eb-9a5e7b2c4c4d">

<img src = "https://github.com/BodleHG/ComputingContinuumEnv/assets/89232601/a605cef7-137e-41e0-866d-1a842cce6501">

<img src = "https://github.com/BodleHG/ComputingContinuumEnv/assets/89232601/a1c18ad6-8e1d-4886-866d-d5a7a5138dd1">

<img src = "https://github.com/BodleHG/ComputingContinuumEnv/assets/89232601/ac943587-6728-4163-9b1d-c82efafd15eb">


