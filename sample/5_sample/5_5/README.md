# ------------------------------------------------
# 5.5.1 DaemonSetの作成
# ------------------------------------------------
```kubectl
# DaemonSetの作成
kubectl apply -f sample-ds.yaml

# Podの一覧を表示
kubectl get pods -o wide

# 終了する場合はこれを実施
kubectl delete -f sample-ds.yaml
```

# ------------------------------------------------
# 5.5.2 DaemonSetのアップデート戦略
# ------------------------------------------------
下記の2つを利用することができます。
  RollingUpdate デフォルト(Deploymentと同じ)
  OnDelete DaemonSetのマニュフェストが変更された際にPodの更新はせず、別の要因でPodが再度作成されるときに新しい定義でPodを作成

## onDelete
任意のタイミングでPodのアップデートを行う場合には、
DaemonSetに紐づくPodを「kubectl delete pod」コマンドにより手動で停止し、
セルフヒーリングの機能によって新しいPodを作成してください。

```kubectl
# DaemonSetの作成
kubectl apply -f sample-ds-ondelete.yaml

# 1つのPodだけ再作成してアップデートする
kubectl delete pod sample-ds-ondelete-qzddt
```

## RollingUpdate
DaemonSetもDeploymentと同じく、RollingUpdateを利用したアップデートが可能です。  
一度に超過可能なPodの数（maxSurge）を設定することはできません。＊1ノード1podのため  
一度に停止可能なPod数（maxUnavailable）のみを指定してRollingUpdateを行います。  

例  
maxUnavailable=2の場合、Podを2つずつ同時にアップデートしていくような形になります。  
また、maxUnavailableを0にすることはできません。  
