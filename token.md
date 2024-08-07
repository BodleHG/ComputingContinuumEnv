control plane join 토큰
```bash
kubeadm join 192.168.50.201:6443 --token nqv7wq.qofb6unn1vmlxmu1 \
        --discovery-token-ca-cert-hash sha256:9e02209e5b997a874e48708b6e423144d0bacb43bedfcf0777b1deeb70460b3b \
        --control-plane
```

worker node join 토큰
```bash
sudo kubeadm join 192.168.50.201:6443 --token nqv7wq.qofb6unn1vmlxmu1 \
        --discovery-token-ca-cert-hash sha256:9e02209e5b997a874e48708b6e423144d0bacb43bedfcf0777b1deeb70460b3b

$ sudo kubeadm join 192.168.50.201:6443 --token ieiy19.uf5rurm6h18qwenn --discovery-token-ca-cert-hash sha256:9e02209e5b997a874e48708b6e423144d0bacb43bedfcf0777b1deeb70460b3b

```

kubeadm join 192.168.50.201:6443 --token ieiy19.uf5rurm6h18qwenn --discovery-token-ca-cert-hash sha256:9e02209e5b997a874e48708b6e423144d0bacb43bedfcf0777b1deeb70460b3b


쿠버네티스 대시보드 토큰

https://192.168.50.201:31000
```text
eyJhbGciOiJSUzI1NiIsImtpZCI6InFrME9MZGRZNHF3eWRJNVBQY0hjb1JOUkQ0X2lORGVsYnkyOTBtZ2ZGWDAifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNzE2OTc1Njk5LCJpYXQiOjE3MTY5NzIwOTksImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsInNlcnZpY2VhY2NvdW50Ijp7Im5hbWUiOiJhZG1pbi11c2VyIiwidWlkIjoiZjFmNGQwODYtM2EwYS00NmYxLWJhNGQtYjczMWM3ODc4NWZmIn19LCJuYmYiOjE3MTY5NzIwOTksInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlcm5ldGVzLWRhc2hib2FyZDphZG1pbi11c2VyIn0.fxVXGYmxBPciv48L0izB4w4x1WgoAPGTOUdgxdmeD82FUupHt7JlFa3iyRs_Pl6mCRLTaF5OhSfCXFPn3cxBwkZ9RFU0DXVIzPtJf6Doq-aojj224G5DcG5Bq9z6SOzRrEqWGDq77KgZH0lN9ysNXynRtLeacEvUrMFnkTdb1dwAgX-1UqFND185pKrbL7UHpM3GfyjGd-yWcy0tM8lNY-x3tjKLApvvC8ttVOXvh0MS8FIjztUoN7cnCkEVqDhrLkfbvYz1cBnF09zdJyhRMOtcd7RmqJixP_Iqvrw8IdL72cRUEMHtEvK1xpI_hS_wtU-D0Jeg32g354qJj1suFQ
```