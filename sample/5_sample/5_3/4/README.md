# 5.3.4 ReplicaSetのスケーリング
## 1つ目の方法はマニフェストを書き換えて「kubectl apply」コマンドを実行する方法
```kubectl
# レプリカ数を3から5に変更したマニフェストをapply
$sed -e 's|replicas: 3|replicas: 5|' sample-pod.yaml | kubectl apply -f -replicaset.apps "sample-rs" configured”
```

## 2つ目「kubectl scale」コマンドを利用する場合
```kubectl
# レプリカ数を5に増加
kubectl scale rs sample-rs --replicas 5


```

```kubectl
kubectl apply -f sample-pod.yaml

# 上記どちらかのスケーリングコマンドを実行

# スケーリングを確認
kubectl get replicasets

kubectl delete -f sample-pod.yaml 
```