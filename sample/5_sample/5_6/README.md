# ------------------------------------------------
# 5.6.1 StatefulSetの作成
# ------------------------------------------------
```kubectl
kubectl apply -f sample-statefulset.yaml

kubectl get statefulsets

# Podの一覧を表示
kubectl get pods -o wide

# StatefulSetに紐づくPersistentVolumeClaim（PersistentVolume要求）の確認（出力一部改変）
kubectl get persistentvolumeclaims

# StatefulSetに紐づくPersistentVolume（永続化領域）の確認（出力一部改変）
kubectl get persistentvolumes

kubectl delete -f sample-statefulset.yaml
```

# ------------------------------------------------
# 5.6.2 StatefulSetのスケーリング
# ------------------------------------------------

スケールアウトの場合には、Podを同時に1つずつ作成/削除する。
スケールインの場合には、インデックスが大きいもの（最新ものから）から削除する。

  特徴
    0番目のPodは常に一番最初に作られ、一番最後に削除されるため、
    0番目のPodをマスターノードとするような冗長化構成を持つアプリケーションに適しています。

```kubectl
# kubectlコマンドを利用したスケーリング
kubectl scale statefulset sample-statefulset --replicas=5

# レプリカ数を3から5に変更したマニフェストをapply
$ sed -e 's|replicas: 3|replicas: 5|' sample-statefulset.yaml | kubectl apply -f -
```

# ------------------------------------------------
# 5.6.3 StatefulSetのライフサイクル
# ------------------------------------------------
  1.シーケンシャルな実行（1つ1つpodを作成）
  2.パラレル実行（spec.podManagementPolicyをParallelに設定）
```kubectl
```

# ------------------------------------------------
# 5.6.4 StatefulSetのアップデート戦略
# ------------------------------------------------
RollingUpdateとOnDeleteの2つが用意されています。

  OnDelete
  StatefulSetのマニフェストが変更された際にPodの更新はせず、別の要因でPodが再起動されるときに新しい定義でPodを作成します。
  StatefulSetのマニフェストを変更してイメージなどを差し替えたとしても、既存のPodのアップデートは行いません。
  StatefulSetでは永続化領域を持つデータベースやクラスタなどに利用されることが多いため、
  手動でアップデートを行っていきたい場合にはOnDeleteを使って任意のタイミングでや次回再起動時に更新することができるようになっています。
  また、type以外に指定可能な設定項目はありません。


  RollingUpdate ＊デフォルト
  即時Podの更新を行っていきます。
  しかし、StatefulSetでは永続化データがあるため、Deploymentとは異なり、余分なPodを作成してのローリングアップデートはできません。
  また、一度に停止可能なPod数（maxUnavailable）を指定してのローリングアップデートもできず、1つのPodごとにReadyになったことを確認して行われていきます。
  ＊spec.podManagementPolicyがParallelに設定されている場合でも、並列に処理されることはなく、1つのPodずつアップデートを行います。

  StatefulSetのRollingUpdateでは、partitionという特有の値を設定することが可能です。
  partitionを設定することで、全体のPodのうち、どのPodまでを更新するかを指定することができるようになります。
  これにより全体に影響を与えることなく、部分的に更新を適用して様子を見るといった安全性の高いアップデートを行うことが可能です。

```kubectl
```

# ------------------------------------------------
# 5.6.5 永続化領域のデータ保持の確認
# ------------------------------------------------

下記は、sample-statefulset.yamlで実行しています。

```kubectl

kubectl apply -f sample-statefulset.yaml

# 約1GBの永続化領域がマウントされていることの確認（/dev/sdb > /usr/share/nginx/html）
kubectl exec -it sample-statefulset-0 -- df -h  | grep /dev/sd”

# 永続化領域にsample.htmlがないことを確認
kubectl exec -it sample-statefulset-0 ls /usr/share/nginx/html/sample.html

# 永続化領域にsample.htmlを作成
kubectl exec -it sample-statefulset-0 touch /usr/share/nginx/html/sample.html

# 永続化領域にsample.htmlがあることを確認
kubectl exec -it sample-statefulset-0 ls /usr/share/nginx/html/sample.html

# Podの予期せぬ停止その1（Podの削除）
kubectl delete pod sample-statefulset-0

# Podの予期せぬ停止その2（nginxプロセスの停止）
kubectl exec -it sample-statefulset-0 -- /bin/bash -c 'kill 1'

# Pod停止後もファイルは欠損していない
kubectl exec -it sample-statefulset-0 ls /usr/share/nginx/html/sample.html

# Podの一覧を表示
kubectl get pods -o wide

```

# ------------------------------------------------
# 5.6.6 StatefulSetの削除とPersistentVolumeの削除
# ------------------------------------------------
StatefulSetを作成すると、
Podに対してPersistentVolumeClaim（PersistentVolume要求）を設定することができるため、
PersistentVolume（永続化領域）も同時に確保することができる。

この確保した永続化領域は、StatefulSetが削除されても同時に解放されることはありません。
理由としては、StatefulSetが永続化領域を解放する前にボリュームからデータをバックアップする猶予を与えるため。
＊StatefulSetを削除後、StatefulSetがPersistentVolumeClaimで確保したPersistentVolumeを解放せずに再度StatefulSetを作成した場合には、永続化領域のデータはそのままPodが起動することになります。


```kubectl
# StatefulSetの削除
kubectl delete statefulset sample-statefulset

# StatefulSetに紐づくPersistentVolumeClaim（PersistentVolume要求）の確認（出力一部改変）
kubectl get persistentvolumeclaims

# StatefulSetに紐づくPersistentVolume（永続化領域）の確認（出力一部改変）
kubectl get persistentvolumes

# 再度StatefulSetの作成
kubectl apply -f sample-statefulset.yaml

# StatefulSetを一度削除したあとも、永続化領域にsample.htmlがあることを確認
kubectl exec -it sample-statefulset-0 ls /usr/share/nginx/html/sample.html

# StatefulSetが確保した永続化領域の解放
kubectl delete persistentvolumeclaims www-sample-statefulset-{0..4}
```
