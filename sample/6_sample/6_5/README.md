# ------------------------------------------------
# 6.5.1 NodePort Serviceの作成
# ------------------------------------------------
```kubectl
kubectl apply -f sample-nodeport.yaml

# Serviceの一覧を表示
kubectl get services

# 一時的にPodを立ち上げて、NodePort Serviceの名前解決を確認
kubectl run --image=centos:6 --restart=Never --rm -i testpod -- dig sample-nodeport.default.svc.cluster.local

kubectl delete -f sample-nodeport.yaml
```

# ------------------------------------------------
# 6.5.2 NodePortの注意点
# ------------------------------------------------

NodePortで利用できるポート範囲は、
  - GKEを含む多くのKubernetes環境では30000～32767（Kubernetesのデフォルト値）
  - 範囲外の値を設定しようとするとエラー

# ------------------------------------------------
# 6.5.3 ノード間通信の排除（ノードをまたいだロードバランシングの排除）
# ------------------------------------------------

NodePort Serviceでは
Kubernetes Node上のNodePortに到達したリクエストは、
さらにノードをまたいだPodへもロードバランシングされる

```kubectl
# Podが起動しているノードとそのIPアドレスを確認
kubectl get pods -o custom-columns="NAME:{metadata.name},Node:{spec.nodeName},NodeIP:{status.hostIP}
```

curl -s http://10.101.79.230:30081
