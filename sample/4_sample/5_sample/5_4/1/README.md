```kubectl
# アップデートの履歴を保存するオプションをつけて
kubectl apply -f sample-deployment.yaml --record

# ReplicaSetの一覧をYAML形式で表示
kubectl get replicasets -o yaml | head

# Deploymentの確認
kubectl get deployments

# ReplicaSetの確認
kubectl get replicasets

# Podの確認
kubectl get pods

# ------------------------------------------------
# Deploymentの変更があると、ReplicaSetが新規作成する
# ------------------------------------------------

# nginxの書き換え
kubectl set image deployment sample-deployment nginx-container=nginx:1.13

# Deploymentのアップデートの状況を確認
kubectl rollout status deployment sample-deployment

# Deploymentの確認
kubectl get deployments

# ReplicaSetの確認
kubectl get replicasets

# Podの確認
kubectl get pods

# 対象のReplicaSetの状態をYAML形式で出力
kubectl get replicasets sample-deployment-7dfb996c6b -o yaml

kubectl delete -f sample-deployment.yaml
```