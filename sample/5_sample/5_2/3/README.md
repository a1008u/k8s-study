```kubectl
kubectl apply -f sample-pod.yaml

kubectl get pods

# Podの詳細情報を表示
kubectl get pods --output wide

# コンテナへのログイン
kubectl exec -it sample-2pod /bin/bash 

# ---------------------------

# インストール
apt-get update && apt-get -y install iproute procps

# コンテナ内からのIPアドレスの確認
ip a | grep "inet"

# コンテナ内からのbind（listen）しているポートの確認
ss -napt | grep LISTEN

＃コンテナ内からのプロセスの確認
ps aux

exit

# ---------------------------

# 複数コンテナを内包するPodの場合は特定のコンテナを指定可能
kubectl exec -it sample-2pod -c nginx-container ls
kubectl exec -it sample-2pod -c redis-container ls

# オプションを含むコマンドの実行
kubectl exec -it sample-pod -- ls --all --classify

kubectl delete -f sample-pod.yaml 
```
