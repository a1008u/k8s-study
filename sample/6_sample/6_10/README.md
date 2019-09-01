# ------------------------------------------------
# 6.10.1 リソースとコントローラ
# ------------------------------------------------
Ingressをより理解しやすくするため、少しKubernetesの内部実装の話をします。

Kubernetesは分散システムであり、マニフェストで定義したリソースをKubernetesに登録することから始まります。
登録されただけでは何も処理は行われず、実際に処理を行うコントローラと呼ばれるシステムコンポーネントが必要です。
各リソースは登録された後に様々なコントローラが実際に処理をすることでシステムが動作するようになっています。

例
Deploymentには対応したReplicaSetを作成したり、
レプリカ数を変更しながらローリングアップデートを行うDeployment Controllerと呼ばれるシステムコンポーネントがKubernetesクラスタ上で動作しています。
Deployment Controllerがない場合、マニフェストでDeploymentリソースを作成したとしてもReplicaSetは作られません。

# ------------------------------------------------
# 6.10.2 リソースとコントローラ
# ------------------------------------------------
ひとくちにIngressと言っても、その指し示していることは様々です。
例を挙げると「Ingressリソース」とは、マニフェストで登録されるAPIリソースのことを意味しますし、
「Ingress Controller」はIngressリソースがKubernetesに登録された際に、何らかの処理を行うものになります。
処理の例
  GCPのGCLB（Google Cloud Load Balancing）を操作することによるL7ロードバランサの設定
  Nginxの設定を変更してリロードを実施するなど


# ------------------------------------------------
# 6.10.3 Ingressの種類
# ------------------------------------------------
1.クラスタ外のロードバランサを利用したIngress
  GKE Ingress
2.クラスタ内にIngress用のPodをデプロイするIngress
  Nginx Ingress
  Nghttpx Ingress

## 1.クラスタ外のロードバランサを利用したIngress
Ingressリソースを作成するだけでLoadBalancerの仮想IPが払い出されて、利用可能になります。

そのため、IngressのトラフィックはGCPのGCLB（Google Cloud Load Balancing）がトラフィックを受信した後、
GCLBでHTTPS終端やパスベースルーティングなどを行い、NodePortへトラフィックを転送することで対象のPodまで到達します。
順番を簡略にすると次の2ステップの経路をたどる。

［クライアント］-> ［L7 LoadBalancer（NodePort経由）］-> ［転送先のPod］


## 2.クラスタ内にIngress用のPodをデプロイするIngress
Ingressリソースで定義したL7相当のロードバランサ処理を行わせるためのIngress用のPodを、クラスタ内に作成する必要があります。
また作成したIngress用のPodに対して、
クラスタ外からアクセスできるように別途Ingress用のPod宛にLoadBalancer Serviceを作成するなどの準備が必要になります。
さらに、Ingress用のPodがHTTPSの終端やパスベースのルーティングなどL7相当の処理を行うため、
負荷に応じてPodのレプリカ数のオートスケーリングも考慮する必要があります。

Ingress用のPodにNginxを使用したNginx Ingressの場合、
ロードバランサがNginx Podまで転送し、その後NginxがL7相当の処理を行い対象のPodへ転送します。
このとき、Nginx Podから対象のPodまではNodePortは通らず、
直接PodのIPアドレス宛に送られます。順番を簡略にすると次の3ステップの経路を辿ります。

［クライアント］-> ［L4 LoadBalancer（type: LoadBalancer）］-> ［Nginx Pod（Nginx Ingress Controller）］-> ［転送先のPod］

# ingressの例は下記参照
https://kubernetes.github.io/ingress-nginx/deploy/


# ------------------------------------------------
# 6.10.4 Ingressリソースを作成する前の事前準備
# ------------------------------------------------
```kubectl
# sample-ingress-apps-1/path1/index.htmlへのリクエストでホスト名を返す
kubectl exec -it sample-ingress-apps-1 -- mkdir /usr/share/nginx/html/path1/
kubectl exec -it sample-ingress-apps-1 -- cp /etc/hostname /usr/share/nginx/html/path1/index.html

# sample-ingress-apps-2/path2/index.htmlへのリクエストでホスト名を返す
kubectl exec -it sample-ingress-apps-2 -- mkdir /usr/share/nginx/html/path2
kubectl exec -it sample-ingress-apps-2 -- cp /etc/hostname /usr/share/nginx/html/path2/index.html

# sample-ingress-apps-3/index.htmlへのリクエストでホスト名を返す
kubectl exec -it sample-ingress-default -- cp /etc/hostname /usr/share/nginx/html/index.html

# -----------------------
# httpsを利用したい場合
# -----------------------

# 自己署名証明書の作成
$ openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout ~/tls.key -out ~/tls.crt -subj "/CN=sample.example.com"

# Secretの作成（証明書ファイルを指定した場合）
$ kubectl create secret tls --save-config tls-sample --key ~/tls.key --cert ~/tls.crt

```

# ------------------------------------------------
# 6.10.5 Ingressリソースの作成
# ------------------------------------------------
IngressリソースはL7ロードバランサのため、特定のホスト名に対して「リクエストパス > Serviceバックエンド」のペアで転送ルールを設定します。
また、1つのIPアドレスで複数のホスト名を扱うことも可能です。
spec.rules[].http.paths[].backend.servicePortに設定するPort番号は、Serviceのspec.ports[].portを指定してください。

また、Ingressリソースのマニフェストに定義可能な設定は、
「クラスタ外のロードバランサを利用したIngress」と「クラスタ内にIngress用のPodをデプロイするIngress」のどちらの方式であってもほとんど同じです。

## sample-ingress.yamlとsample-ingress-apps.yamlの関係（endpoint）
［/path1/*］
  -> ［sample-ingress-svc-1 Service］
  -> ［sample-ingress-apps-1 Pod（/path1/index.html）］
［/path2/*］
  -> ［sample-ingress-svc-2 Service］
  -> ［sample-ingress-apps-2 Pod（/path2/index.html）］
［*］
  -> ［sample-ingress-default Service］
  -> ［sample-ingress-default Pod（/index.html）］


# ------------------------------------------------
# 6.10.6 GKEの場合
# ------------------------------------------------
GKEの場合には、デフォルトでGKE用のIngress Controllerがデプロイされており、
特に意識することなくIngressリソースごとに自動でロードバランサの仮想IPが払い出されます。
```gcloud

```

# ------------------------------------------------
# 6.10.7 Nginx Ingressの場合
# ------------------------------------------------
```kubectl

```