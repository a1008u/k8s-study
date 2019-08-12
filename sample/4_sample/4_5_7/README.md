```kubectl
# 更新時に削除されない
kubectl apply -f ./

# 更新時に削除される
kubectl apply -f ./ --prune -l system=a

# 下記のようにpod1はprunedとなる
# pod/sample-pod2 unchanged
# pod/sample-pod1 pruned
# rmで、対象のリソースが削除されているためpodも削除される。（prunedのおかげで紐付けて消せる）
rm sample-pod1.yaml
kubectl apply -f ./ --prune -l system=a
```