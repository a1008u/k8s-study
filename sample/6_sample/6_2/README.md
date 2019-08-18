# ------------------------------------------------
# 6.2.1 Pod宛トラフィックのロードバランシング
# ------------------------------------------------
Serviceは、受信したトラフィックを複数のPodにロードバランシング（負荷分散）する機能を提供します。

これに対してServiceを使用すると、下記の機能もついてくる。
- 複数のPodに対するロードバランシングを自動的に構成
- Serviceはロードバランシングの接続口となるエンドポイントも提供

エンドポイントには、下記などの様々な種類が提供されています。
- 外部のロードバランサが払い出す仮想IPアドレス（Virtual IPアドレス）
- クラスタ内でのみ利用可能な仮想IPアドレス（ClusterIP）

```kubectl
kubectl apply -f sample-deployment.yaml

kubectl get pods

# 出力時に特定のJSON PATH（例ではラベル）の値のみを出力
kubectl get pods sample-deployment-6cd85bd5f-2zttr -o jsonpath='{.metadata.labels}

kubectl get pods
# 指定したラベルを持つPod情報のうち、指定のJSON Pathをカラムにして出力
kubectl get pods -l app=sample-app -o custom-columns="NAME:{metadata.name},IP:{status.podIP}"

# serviceのマニュフェストを実行
kubectl apply -f sample-clusterip.yaml

# Serviceの情報を確認
kubectl get services sample-clusterip

# Serviceの詳細情報を確認
kubectl describe svc sample-clusterip

# Pod名を取得し、全Podで"hostname > /usr/share/nginx/html/index.html"を実行
for PODNAME in `kubectl get pods -l app=sample-app -o jsonpath='{.items[*].metadata.name}'`; do
    kubectl exec -it ${PODNAME} -- cp /etc/hostname /usr/share/nginx/html/index.html;
done

# 一時的にPodを立ち上げて、Serviceのエンドポイントに向けてリクエスト
kubectl run --image=centos:6 --restart=Never --rm -i testpod -- curl -s http://10.102.49.59:8080

kubectl delete -f sample-deployment.yaml
kubectl delete -f sample-clusterip.yaml
```


Kubernetesでは、サービスディスカバリの機能をServiceが提供しています。
サービスディスカバリとは、端的に言うと特定の条件の対象となるメンバを列挙したり、名前からエンドポイントを判別する機能です。
Kubernetesの文脈では、Serviceに属するPodを列挙したり、Serviceの名前からエンドポイントの情報を返すことを指します。

サービスディスカバリの方法は、主に次の3種類が用意されています。
DNSを利用したサービスディスカバリでは、クラスタに内蔵しているDNSサーバ（以降、クラスタ内DNSと呼ぶ）へ自動的に登録されるServiceのエンドポイント情報を利用します。

- 環境変数を利用したサービスディスカバリ
- DNS Aレコードを利用したサービスディスカバリ
- DNS SRVレコードを利用したサービスディスカバリ


### 環境変数を利用したサービスディスカバリ
Pod内からは、環境変数でも同じNamespaceのサービスが確認できるようになっています。
「-」が含まれるサービス名は「_」に変更された上で、大文字に変換されます。「docker --links ...」で実行した際と同じ形式で環境変数が保存されるため、
Docker単体で利用していた環境からの移行時にも利用しやすくなっています。
ただし、コンテナ起動後のServiceの追加や削除に伴って環境変数が再度読み込まれるわけではないため、思わぬ事故に繋がる可能性もあります。
ServiceよりもPodの方が先にできた場合は環境変数が登録されていないため、Podを再作成してみてください。
```kubectl
# 環境変数に登録されているServiceの情報を確認
kubectl exec -it sample-deployment-6cd85bd5f-jhwbl env | grep -i sample_clusterip
```

### DNS Aレコードを利用したサービスディスカバリ
払い出されたIPアドレスを使用する以外に、自動登録されたDNSレコードを使用できます。

```kubectl
# 一時的にPodを立ち上げて、コンテナ内からsample-clusterip:8080宛にHTTPリクエスト
kubectl run --image=centos:6 --restart=Never --rm -i testpod -- curl -s http://sample-clusterip:8080”

# 一時的にPodを立ち上げて、コンテナ内からsample-clusterip.default.svc.cluster.localの名前解決を試みる
kubectl run --image=centos:6 --restart=Never --rm -i testpod -- dig sample-clusterip.default.svc.cluster.local”

# 一時的にPodを立ち上げて、デフォルトで/etc/resolv.confにかかれているオプションを確認
kubectl run --image=centos:6 --restart=Never --rm -i testpod -- cat /etc/resolv.conf”

# 一時的にPodを立ち上げて、コンテナ内から10.11.253.80の名前解決を試みる
kubectl run --image=centos:6 --restart=Never --rm -i testpod -- dig -x
```


### DNS SRVレコードを利用したサービスディスカバリ
SRVレコードは、Port名とProtocolを利用することで、
サービスを提供しているPort番号を含めたエンドポイントをDNSで解決する仕組みです。

```kubectl
# 一時的にPodを立ち上げて、SRVレコードが他のPodから解決できることの確認
kubectl run --image=centos:6 --restart=Never --rm -i testpod -- dig _http-port._tcp.sample-clusterip.default.svc.cluster.local SRV
```


# ------------------------------------------------
# 6.2.2 
# ------------------------------------------------

# ------------------------------------------------
# 6.2.3 
# ------------------------------------------------


# ------------------------------------------------
# 6.2.4 
# ------------------------------------------------


# ------------------------------------------------
# 6.2.5 
# ------------------------------------------------


# ------------------------------------------------
# 6.2.6 
# ------------------------------------------------