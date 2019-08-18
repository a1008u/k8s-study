# 参考
https://github.com/MasayaAoyama/kubernetes-perfect-guide

# 4.APIリソースとkubectl

## 4.5 CLIツールkubectl
kubectlの設定情報の確認方法
```
cat  ~/.kube/config
```

configの情報
clusters　クラスターの情報
contexts　cluseterとusersの紐付け情報が格納。また、カレントのclusterの情報も保持
    cluster: docker-desktop
    user: docker-desktop
    name: docker-desktop
users　認証情報など


### configの設定変更(kubectlを利用)
```kubectl
# クラスタの定義
kubectl config set-cluster prd-cluster --server=https://localhost:6443

# 認証情報の定義
kubectl config set-credentials admin-user --client-certificate=./sample.crt --client-key=./sample.key --embed-certs=true

# Contextの定義
kubectl config set-context prd-admin --cluster=prd-cluster --user=admin-user --namespace=default
```

### contextの操作
```kubectl
# contextの切り替え
kubectl config user-context pre-admin

# 現在のcontextを表示
kubectl config current-context

# コマンドの実行ごとに、Contextを指定することも可能
kubectl --context prd-admin get pod
```

## 4.5.6 アノテーションとラベル

```
apiVersion: v1
kind: Pod
metadata:
  name: sample-pod
  annotations:
    annotation1: val1
    annotation2: val2
  labels:
    label1: lal1
    label2: lal2
spec:
  containers:
    - name: nginx-container
      image: nginx:1.13
```

### アノテーション
リソースに対するメモ書き  
システムコンポーネントと結びつくことで、処理ができる。  

アノテーションの役割
  1.システムコンポーネントのためにデータを保存
  2.全ての環境では利用できない設定を行う（GKEやEKSなどの外部接続など）
  3.正式に組み込まれる前の昨日の設定を行う

### ラベル
ソースを分別するための情報

ラベルの役割
  1.開発者が利用するラベル
  2.システムが利用するラベル


### 4.5.10 リソースの情報取得(get)
```kubectl
# pods一覧を表示
kubectl get pods
kubectl get pods sample-pod
kubectl get pods -l app -L app
kubectl get pods -o JSON
kubectl get pods -o YAML
kubectl get pods -o JSON sample-pod
kubectl get pods -o YAML sample-pod

# nodes一覧を表示
kubectl get nodes

# nodesとpodsを両方表示
kubectl get all
```

### 4.5.10 リソースの詳細情報取得(describe)
```kubectl
kubectl describe pod sample-pod

kubectl describe node docker-desktop
```

### 4.5.11 リソースの使用料(top)
```kubectl
kubectl top node
kubectl -n kube-system top node
```

### 4.5.12 Pod上でのコマンドの実行(exec)
```kubectl
kubectl exec -it sample-pod /bin/sh

# 複数コンテナが入ったPodの特定のコンテナでexec
kubectl exec -it sample-pod -c nginx-container /bin/sh

# 引数があるコマンドを実行の場合
kubectl exec -it sample-pod -- /bin/sh -l

kubectl exec -it sample-pod -- /bin/sh -c "ls --all --classify | grep media"
```

### 4.5.13 Podのログ確認(logs)
```kubectl
kubectl logs sample-pod
kubectl logs sample-pod -c nginx-container
kubectl logs -f sample-pod
kubectl logs --since=1h --tail=10 --timestamps=true sample-pod
```


### 4.5.16 コンテナとローカルマシン間でのファイルのコピー(cp)
```kubectl
# ample-pod:/etc/hostnameをhostnameとしてカレントディレクトリにコピー
kubectl cp sample-pod:/etc/hostname hostname

# ホスト環境にあるhostnameをsample-pod:tmp/newfileとしてコピー
kubectl cp hostname sample-pod:tmp/newfile
kubectl exec -it sample-pod ls /tmp
```

### 4.5.17 ローカルマシンからPodへのポートフォワーディング(port-forward)
```kubectl
kubectl port-forward sample-pod 8888:80
curl -I localhost:8888
```

deploymentやserviceにも接続することができるが、複数のpodのうち一つのみに接続している
```kubectl
kubectl port-forward deployment/sample-deployment 8888:80

kubectl port-forward service/sample-service 8888:80
```