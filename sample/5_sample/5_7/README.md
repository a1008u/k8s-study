# ------------------------------------------------
# 5.7.1 ReplicaSetとの違いとJobの使い所
# ------------------------------------------------

JobとReplicaSetとの大きな違いは、「起動するPodが停止することを前提にして作られているかどうか」です。

```kubectl
```


# ------------------------------------------------
# 5.7.2 Jobの作成
# ------------------------------------------------
```kubectl
kubectl apply -f sample-job.yaml

# Jobの一覧を表示
kubectl get jobs

# Jobが作成したPodを確認(終了時の確認にも)
kubectl get pods

kubectl delete -f sample-job.yaml
```


# ------------------------------------------------
# 5.7.3 restartPolicyによる挙動の違い
# ------------------------------------------------
Jobのマニフェストではspec.template.spec.restartPolicyに、OnFailureまたはNeverのどちらかを指定する必要があります。

## restartPolicy: Neverの場合
Pod障害時には新規のPodが作成されます。
Dockerではspec.containers[].commandで指定されたプログラムはプロセスID=1で実行されます。
プロセスID=1でsleepコマンドを実行していると、SIGKILLシグナルを送ってもプロセスを停止することができないため、シェルから別のプロセスを立ち上げるような形で起動しておきます。

```kubectl
kubectl apply -f sample-job-never-restart.yaml

# Jobが作成したPodを確認
kubectl get pods

# コンテナ上のsleepプロセスの停止
kubectl exec -it sample-job-never-restart-j6pnx -- sh -c 'kill -9 `pgrep sleep`'

# 新たなPodが起動
$ kubectl get pods

kubectl delete -f sample-job-never-restart.yaml
```

## restartPolicy: OnFailureの場合
再度同一のPodを利用してJobを再開します。
下記のマニフェストを使って、restartPolicyにOnFailureを指定した場合の動作確認を行います。
Podが起動するノードやPodのIPアドレスは変わることはありませんが、PersistentVolume（永続化領域）やKubernetes Node上の領域（hostPath）をマウントしていない場合には、データ自体は紛失します。

```kubectl
kubectl apply -f sample-job-onfailer-restart.yaml

# Jobが作成したPodを確認
kubectl get pods

# コンテナ上のsleepプロセスの停止
kubectl exec -it sample-job-onfailer-restart-6pjvs -- sh -c 'kill -9 `pgrep sleep`'

# 新たなPodが起動
$ kubectl get pods

kubectl delete -f sample-job-onfailer-restart.yaml
```

# ------------------------------------------------
# 5.7.4 並列Jobとワークキュー型の実行
# ------------------------------------------------

下記パラメータについて
  成功回数を指定するcompletions
  並列度を指定するparallelism
これらの項目のデフォルト値は1なので、明示的に指定しなくても動作は変わりません。
この2つのパラメータは、Jobを並列化して実行する際に利用するオプションとなります。


```kubectl
# Jobの並列度は途中で変更可能。マニュフェストを書き換えてapplyをする。
# 並列度を2 -> 3に変更したマニュフェストをapply
sed -e 's|parallelism: 2|parallelism: 3|' sample-paralleljob.yaml | kubectl apply -f -
```


