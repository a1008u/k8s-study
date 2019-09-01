# ------------------------------------------------
# 6.6.1 LoadBalancer Serviceの作成
# ------------------------------------------------
```kubectl
kubectl apply -f sample-lb.yaml

# Serviceの一覧を表示
kubectl get services sample-lb

# Serviceの一覧を表示
kubectl get services

# 一時的にPodを立ち上げて、NodePort Serviceの名前解決を確認
kubectl run --image=centos:6 --restart=Never --rm -i testpod -- dig sample-nodeport.default.svc.cluster.local

kubectl delete -f sample-lb.yaml
```


# ------------------------------------------------
# 6.6.2 ノード間通信の排除（ノードを跨いだロードバランシングの排除）
# ------------------------------------------------
利用しないほうがいい


# ------------------------------------------------
# 6.6.3 LoadBalancerが払い出す仮想IPの静的な指定
# ------------------------------------------------

# ------------------------------------------------
# 6.6.4 LoadBalancerのファイアウォールルールの設定
# ------------------------------------------------
LoadBalancer側でアクセス制御を行う場合は、
spec.loadBalancerSourceRangesに接続を許可する送信元IPアドレスの範囲を指定します。




