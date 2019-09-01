# ------------------------------------------------
# 6.4.1 ClusterIP Serviceの作成
# ------------------------------------------------
```kubectl
# Kubernetesノードの一覧とアドレスを出力
kubectl get nodes -o custom-columns="NAME:{metadata.name},IP:{status.addresses[].address}"

kubectl apply -f sample-externalip.yaml

# ExternalIPの確認
$ kubectl get services

# 一時的にPodを立ち上げて、AレコードでClusterIPが他のPodから解決できることの確認
kubectl run --image=centos:6 --restart=Never --rm -i testpod -- dig sample-externalip.default.svc.cluster.local

// 終了
kubectl delete -f sample-externalip.yaml
```