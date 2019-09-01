# ------------------------------------------------
# 6.3.1 ClusterIP Serviceの作成
# ------------------------------------------------
```kubectrl

// deploymentとserviceを起動
kubectl apply -f sample-deployment.yaml
kubectl apply -f sample-clusterip.yaml

// 状態を確認
kubectl get pods
kubectl get service

// 疎通確認
kubectl run --image=centos:6 --restart=Never --rm -i testpod -- curl -s http://sample-clusterip:8080

// 終了
kubectl delete -f sample-deployment.yaml
kubectl delete -f sample-clusterip.yaml
```

# ------------------------------------------------
# 6.3.2 ClusterIP 仮想IPの静的な指定
# ------------------------------------------------

ClusterIPを自動付与ではなく手動で指定する場合には、spec.clusterIPを指定することで固定できる。

また、すでに作成されているServiceに対して、後からClusterIPを変更することはできない。









