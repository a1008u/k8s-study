# ------------------------------------------------
# 5.8.1 CronJobの作成
# ------------------------------------------------
```kubectl
# CronJobの作成
kubectl apply -f sample-cronjob.yaml

# CronJobの確認
kubectl get cronjobs

# スケジュール時間になっていないときはJobが存在しない
kubectl get jobs

# LAST-SCHEDULEが更新されている
kubectl get cronjobs

# CronJobが作成したJobが存在
kubectl get jobs

kubectl delete -f sample-cronjob.yaml
```


# ------------------------------------------------
# 5.8.2 CronJobの一時停止
# ------------------------------------------------
```kubectl
# クライアントからCronJobの一時停止を実行
kubectl patch cronjob sample-cronjob -p '{"spec":{"suspend":true}}'

# CronJobが一時停止している状態
kubectl get cronjobs
```

# ------------------------------------------------
# 5.8.3 同時実行に関する制御
# ------------------------------------------------
CronJobでは、Jobを作成するという特性上、同時実行に関するポリシーを設定することが可能です。
Jobの実行が意図した間隔内で正常終了している時は、明示的に指定せずとも同時実行されることなく新たなJobが実行されます。
一方で、古いJobがまだ実行している際にはポリシーで新たなJobを実行するかどうかの制御を行いたいケースがあります。

設定方法
spec.concurrencyPolicy
  Allow - 同時実行に対して制御しない（デフォルト）
  Forbid - 前のJobが終了していない場合、次のJobは実行しない
  Replace - 前のJobをキャンセルし、Jobを開始する（前のJobのレプリカ数を0に変更し、前のJobに紐付く各Podの削除処理を行う）

```kubectl
```

# ------------------------------------------------
# 5.8.4 実行開始期限に関する制御
# ------------------------------------------------
CronJobは、指定した時刻になるとKubernetes MasterがJobを作成します。
そのため、Kubernetes Masterが一時的にダウンしていた場合など、開始時刻が遅れた場合に許容できる秒数（spec.startingDeadlineSeconds）を指定することが可能です。
例えば、毎時00分に起動するJobを「毎時00～05分にだけ実行可能」に設定する場合には、300 secに設定しておく必要があります。
デフォルトでは、どんなに開始時間が遅れた場合でもJobを作成するようになっています。


```kubectl
```

# ------------------------------------------------
# 5.8.5 CronJobの履歴
# ------------------------------------------------
設定方法
spec.successfulJobsHistoryLimit - 成功したJobを保存する数
spec.failedJobsHistoryLimit - 失敗したJobを保存する数


```kubectl
# CronJobが作成したJobの確認（一定時間経過後）
kubectl get jobs
```

# ------------------------------------------------
# 5.8.6 マニフェストを書かずにCronJobを作成する ＊非推奨
# ------------------------------------------------
```kubectl
# マニフェストを使わずにコマンドラインからのCronJobの作成
kubectl run sample-cronjob --schedule "*/1 * * * *" \”
```
