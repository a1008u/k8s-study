# 5
## 5.1 Workloadsリソース

workloadsリソースは、クラスタ上にコンテナを起動させるのに利用するリソース。

Workloadsの関係性
![workloadsリソースの関係](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F251519%2Feaad1faf-0bac-ea07-06bf-38f7f373f9b5.png?ixlib=rb-1.2.2&auto=compress%2Cformat&fit=max&s=9c337e68bacf7b5d28f2ab268f06f999)

# 5.2 Pod
workloadsの最小単位のリソース。

## 5.2.1 Podのデザパ
  1.サイドカーパターン - メインコンテナに機能を追加する
  2.アンパサダーパターン - 外部システムとのやりとりの代理を行う
  3.アダプタパターン - 外部からのアクセスのインターフェースとなる

Pod内では、containerPortの衝突が起きないようにしよう。

## 5.2.6 Pod名の制限
  1.利用可能な文字は英小文字と数字
  2.利用可能な記号は"-","."
  3.始まりと終わりの文字は英小文字

---

# 5.3 ReplicaSet/ReplicationController
Podのレプリカを作成し、指定した数のPodを維持し続けるリソースです。  

ReplicaSetのラベルと別マニュフェストのラベル名が同じ場合、  
ReplicaSetはPodを増やし過ぎたと誤認して、  
指定の数まで停止または削除してしまう。  

## 5.4 Deployment
Deploymentは複数のReplicaSetを管理することで、  
ローリングアップデートやロールバックなどを実現するリソース  

Deploymentを使うと、  
新しいReplicaSet上でコンテナが起動したことや、  
ヘルスチェックが通っていることを確認しながら切り替えを行ってくれますし、  
ReplicaSetの移行過程におけるPod数の細かい指定も可能です。  

## 5.5 DaemonSet
  ReplicaSet
  各Kubernetes Node上に合計でN個のPodをNodeのリソース状況に合わせて配置していきます。
  ＊各Node上のPodの数が等しくなるとも限りませんし、全てのNode上に確実に配置されるとも限りません。

  DaemonSet
  各ノードにPodを1つずつ配置するリソースです。
  ユースケースとしては、fluentdやdatadogなど、全体Nod場で必ず動作させたいプロセスのために利用する。

## 5.6 StatefulSet
ReplicaSetの特殊な形。
  ReplicaSetの違い。
  1.作成されるPod名のサフィックスは数字のインデックスが付与されたものになる
  2.データを永続化するための仕組みを有している

## 5.7 job
 コンテナを利用して一度限りの処理を実行させるリソース。
 N並列で実行しながら、指定した回数のコンテナの実行（正常終了）を保証するリソースです。

## 5.8 Cronjob
  Cronのようにスケジュールされた時間にJobを作成します。
  CornJobがJobを管理し、JobがPodを管理するような3層の親子関係になっています。

## 5.9 まとめ
|Workloadsリソース|	使い分け方|
|---|---|
|Pod|	デバッグや確認用途などに利用する|
|ReplicationController|	非推奨のためReplicaSetを利用する|
|ReplicaSet|	Podをスケールさせて管理する ＊基本的にはDeployment経由で利用する|
|Deployment|	スケールさせるワークロードに利用する|
|DaemonSet|	各ノードに1つずつPodを配置したい時に利用する|
|StatefulSet|	データの永続化などステートを持つワークロードに利用する|
|Job|	ワークキューやタスクなどコンテナの終了が必要なワークロードに利用する|
|CronJob|	定期的にJobを作成したい時に利用する|