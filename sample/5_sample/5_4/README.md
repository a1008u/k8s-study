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

# 終了する場合はこれを実施
kubectl delete -f sample-deployment.yaml
```

# ------------------------------------------------
# 5.4.2 Deploymentのアップデート
# ------------------------------------------------
```kubectl
# 対象のReplicaSetの状態をYAML形式で出力
kubectl get replicasets sample-deployment-7dfb996c6b -o yaml
```

# ------------------------------------------------
# 5.4.3 変更のロールバック
# ------------------------------------------------

>CI/CDのパイプラインからロールバックする場合、「kubectl rollout」コマンドよりも、古いマニフェストを再度「kubectl apply」コマンドを実行して適用したほうが相性が良いため、
>実際には、下記のロールバックは利用することは少ないでしょう。

```kubectl
# 変更履歴の確認
kubectl rollout history deployment sample-deployment

# 初期状態のDeployment(該当revisionの詳細情報の取得)
kubectl rollout history deployment sample-deployment --revision 1

# 1度更新したあとのDeployment(該当revisionの詳細情報の取得)
kubectl rollout history deployment sample-deployment --revision 2

# リビジョンを指定してロールバックする場合
kubectl rollout undo deployment sample-deployment --to-revision 1

# 1つ前にロールバックする場合（デフォルトの--to-revision 0が指定されて一つ前に戻る）
kubectl rollout undo deployment sample-deployment

# ロールバック後は前のReplicaSetでPodが起動される
kubectl get replicasets
```

# ------------------------------------------------
# 5.4.4 Deployment更新の一時停止
# ------------------------------------------------
Deploymentの変更と適用を分けることができる。（イメージはビルドとデプロイを分ける感じ）
```kubectl
# 更新の一時停止
kubectl rollout pause deployment sample-deployment

# nginxの書き換え
kubectl set image deployment sample-deployment nginx-container=nginx:1.13

# Deploymentのアップデートの状況を確認(ロールバックもできない)
kubectl rollout status deployment sample-deployment

# 更新の一時停止解除
kubectl rollout resume deployment sample-deployment

```

# ------------------------------------------------
# 5.4.5 rolling-updateコマンドとReplicationController
# ------------------------------------------------
非推奨

# ------------------------------------------------
# 5.4.6 Deploymentのアップデート戦略
# ------------------------------------------------

spec.strategy.typeで設定され、デフォルトは「RoolingUpdate」となる。

戦略的には下記2点がある。
  Recreate 
    一度全てのPodを削除してから再度Podを作成するためダウンタイムが発生するが、余分なリソースを使わないかつ切り替え官僚が早い
    一度既存のReplicaSetのレプリカ数を0にしてリソースを解放する。その後、新たなReplicaSetを作成する。
  
  RollingUpdate 


```kubectl
# ReplicaSetの一覧を表示（リソースの状態に変化があった場合は追従して出力）
kubectl get replicasets --watch
```
# ------------------------------------------------
# 5.4.7 より詳細なアップデートパラメータ
# ------------------------------------------------

下記パラメータを設定ができる。詳細はsample-deployment-prams.yamlに記載
  minReadySeconds
  revisionHistoryLimit
  progressDeadlineSeconds

```kubectl
```

# ------------------------------------------------
# 5.4.8 Deploymentのスケーリング
# ------------------------------------------------
kubectl scale または kubectl apply -fを利用する
```kubectl
# kubectlコマンドを利用したスケーリング
kubectl scale deployment sample-deployment --replicas=5

# レプリカ数を3から5に変更したマニフェストをapply
sed -e 's|replicas: 3|replicas: 5|' sample-deployment.yaml | kubectl apply -f -
```

# ------------------------------------------------
# 5.4.9 マニュフェストを書かずにDeploymentを作成する
# ------------------------------------------------
```kubectl
# マニフェストを使わずにコマンドラインからのDeploymentの作成
kubectl run sample-deployment-cli --image nginx:1.12 --replicas 3 --port 80
```
