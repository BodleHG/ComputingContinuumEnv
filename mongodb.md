MongoDB 데이터를 저장할 Volume 생성
```bash
sudo mkdir -p /data/db/mongo-0
sudo mkdir -p /data/db/mongo-1
sudo mkdir -p /data/db/mongo-2
```

MongoDB Namespace 생성
```bash
kubectl create namespace mongodb-replica
```

Replica Set 환경에서 각 노드가 서로 통신하기 위한 Secret 발급
```bash
$ openssl rand -base64 741 > ./mongo-replica-sets-key.txt
$ kubectl create secret generic mongo-shared-bootstrap-data -n mongodb-replica --from-file=internal-auth-mongodb-keyfile=./mongo-replica-sets-key.txt
```


Storage Class 및 PersistentVolume 생성
```bash
kubectl apply -f storageClass.yaml
```

MongoDB StatefulSet 배포
```bash
kubectl apply -f mongodb.yaml
```

배치가 완료되면 mongo Pod에 접속해 Replica Set 설정
```bash
# Mongo Pod 접속
$ kubectl exec -it mongo-0 -c mongo -n mongodb-replica -- /bin/bash
# Mongo 터미널 접속
$ mongosh
# RepSet 설정
$ rs.initiate({_id: "MainRepSet", version: 1, members: [
  { _id: 0, host : "mongo-0.mongo.mongodb-replica.svc.cluster.local:27017" },
  { _id: 1, host : "mongo-1.mongo.mongodb-replica.svc.cluster.local:27017" },
  { _id: 2, host : "mongo-2.mongo.mongodb-replica.svc.cluster.local:27017" }
]});

# RepSet 설정이 잘 되었는지 확인
$ rs.status()
```

관리자 계정 생성 및 권한 부여
```bash
$ use admin

$ db.createUser({
    user: "sysailab612",
    pwd: "sysailab@612",
    roles:[{role:"root",db:"admin"}]
    })
```

참고 - control plane에도 파드가 배치되도록 설정
```bash
kubectl taint nodes $(controlplane이름) node-role.kubernetes.io=control-plane:NoSchedule-
# 예시
kubectl taint nodes 612workstation node-role.kubernetes.io/control-plane:NoSchedule-
```

