```kubectl
kubectl apply -f sample-pod.yaml

# ReplicaSetの確認
kubectl get replicasets -o wide

＃ReplicaSetがPodの管理に使用しているラベル
kubectl get pods -l app=sample-app -o wide

kubectl delete -f sample-pod.yaml 
```

# セルフヒーリングの確認
```kubectl

# 止めるためのPod名（NAME）を確認
kubectl get pods -l app=sample-app -o wide

# 特定のPodを削除
kubectl delete pod sample-rs-45zsf 

# 新規Podが作成されて合計3つPodが存在することを確認
kubectl get pods -o wide

# ReplicaSetの詳細情報の表示
kubectl describe rs sample-rs

# Podの一覧を表示
kubectl get pods -L app
```
