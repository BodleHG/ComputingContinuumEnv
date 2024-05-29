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