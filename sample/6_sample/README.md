# 6

## 6.1 
Discovery&LBリソース
クラスタ上のコンテナに対するエンドポイントの提供や、ラベルに一致するコンテナのディスカバリに利用されるリソースです。

利用者が直接利用するもの
  - L4ロードバランシングを提供するServiceリソース
  - L7ロードバランシングを提供するIngressリソース

Serviceリソースには、提供目的としてクラスタ内部からの接続用やクラスタ外部からの接続用というように、
接続口となる仮想IPなどのエンドポイントの提供目的が異なる7種類のService typeがあります。


## 6.2 
podとコンテナの関係
同じPod内であればすべてのコンテナは同じIPアドレスが割り当てられています。
よって、同一Podのコンテナへ通信を行う場合にはlocalhost宛に通信を行う。

あるPodのコンテナから別Podのコンテナへ通信を行う場合にはPodのIPアドレス宛に通信を行う。

kubernetesクラスタ内部ネットワーク
Kubernetesクラスタは、クラスタを構築するとノードにPodのための内部ネットワークを自動的に構成する。

Serviceを利用することによって得られる大きなメリット
  - Pod宛トラフィックのロードバランシング
  - サービスディスカバリとクラスタ内DNS

## 6.3 ClusterIP Service
ClusterIP Serviceは、Kubernetesの最も基本となる「type: ClusterIP」のService  
「type: ClusterIP」のServiceを作成すると、  
Kubernetesクラスタ内からのみ疎通性があるInternal Networkに作り出される仮想IPが割り当てられます。  
ClusterIP宛の通信は、各ノード上で実行しているシステムコンポーネントのkube-proxyがPod向けに転送を行います。  
（実装はProxy-modeによって異なります）

ClusterIPは、Kubernetes Cluster外からトラフィックを受ける必要がない箇所などで、クラスタ内ロードバランサとして利用します。
デフォルトでは、Kubernetes APIに接続するための「kubernetes」Serviceが作成されており、ClusterIPが発行される。
```kubectl
kubectl get services
```


## 6.4 ExternalIP Service
特定のKubernetes Nodeの「IPアドレス:Port」で受信したトラフィックを、
コンテナに転送する形で外部疎通性を確立するService


## 6.5 NodePort Service
ExternalIP Serviceに類似したService
ExternalIPは、指定したKubernetes NodeのIPアドレス:Portで受信したトラフィックをコンテナに転送する形で、外部疎通性を確立していました。  
それに対してNodePortは、全てのKubernetes NodeのIPアドレス:Portで受信したトラフィックをコンテナに転送する形で、外部疎通性を確立します。  
ExternalIP Serviceで全ノードからトラフィックを受け付けるように設定するのに近い機能ですが、  
厳密に言うとListenする際に0.0.0.0:Portを使用してすべてのIPアドレスでBindするような形になります。  


## 6.6 LoacBalancer Service
プロダクション環境でクラスタ外からトラフィックを受ける場合に、
一番実用的で使い勝手が良いServiceです。

LoadBalancer Serviceでは、Kubernetesクラスタ外のLoadBalancerに外部疎通性のある仮想IPを払い出すことが可能です。
外部のLoadBalancerを利用するため、Kubernetesクラスタが構築されているインフラがこの仕組みに対応している必要があります。
現状ではGCP／AWS／Azure／OpenStackを始めとしたクラウドプロバイダが、LoadBalancer Serviceを利用できる環境の主流です
（最近ではMetalLBなども登場し、クラウドプロバイダ以外の環境での選択肢も増えてきています）。

また、Docker for Macでは、LoadBalancer Serviceを払い出すときに仮想IPアドレスではなくlocalhost（127.0.0.1）のIPアドレスを使用するため、注意して下さい。本書ではGKEを例に説明しますが、基本的な挙動はほとんどのKubernetes環境でも変わりません。

NodePortやExternalIPでは、
結局のところいずれかのKubernetes Nodeに割り当てられたIPアドレス宛に通信を行うため、
そのノードが単一障害点（Single Point of Failure、SPoF）となってしまいました。
しかし「type: LoadBalancer」では、Kubernetes Node群とは別に外部のロードバランサを利用するので、ノードの障害に強いという特徴があります。
イメージとしてはNodePort Serviceを作り、クラスタ外のLoadBalancerからKubernetes Node宛にバランシングするような形です。
Kubernetes Nodeに障害が発生した場合には、そのノードに対してトラフィックを転送しないようにすることで、自動的に復旧するようになっています。
ただしLoadBalancerがノードの障害を検知して転送先から除外するまでの時間が必要なので、この間一定の通信断は発生します。
これについては、従来のLoadBalancerと仮想マシンの場合と同様です。


## 6.7 Headless Service(None)
対象となる個々のPodのIPアドレスが直接返ってくるServiceです
StatefulSetがHeadless Serviceを利用している場合に限り、
Pod名でIPアドレスをディスカバリすることが可能



## 6.8 ExternalName Service
通常のServiceリソースとは異なり、Service名の名前解決に対して外部のドメイン宛のCNAMEを返す。

使用する場面
  別の名前を設定したい場合
  クラスタ内からのエンドポイントを切り替えやすくしたい場合

## 6.9 None-Selector Service
ExternalName Serviceでは、Service名で名前解決を行うと外部のドメインに転送されました。
それに対してNone-Selector Serviceでは、Service名で名前解決を行うと自分で指定したメンバに対してロードバランシングを行います。

Kubernetes内に自由な宛先でここまで説明した形式のロードバランサが作れる機能です。
「type: LoadBalancer」を指定することも可能ですが、Kuberneteクラスタ外部のロードバランサで受信したトラフィックがKubernetesクラスタに転送された後、再度Kuberneteクラスタ外部に転送する形になるため、有用な用途がほとんどありません。
これは「type: NodePort」なども同様です。

externalNameを指定しておらず、Selectorが存在しないサービスを作成した後、
Endpointsリソースを手動で作ることで柔軟なServiceを作ることが可能です。

例えば、Kubernetesクラスタの外部のアプリケーションサーバに対してリクエストを分散する場合にも、
KubernetesのServiceを使って行うことが可能です。
また、プロダクション環境とステージング環境で、
クラスタ外部とクラスタ内部のサービスを使い分けたとしても、
アプリケーション側からは同じClusterIPとして見せることが可能になります。

外部に対してリクエストを送るためのServiceとしては、ExternalName Serviceとほぼ同じですが、
ExternalName ServiceはCNAMEが返したのに対して、
None-Selector ServiceではClusterIPのAレコードが返ります。
そのため、KubernetesのServiceの使用感のまま、外部向けにロードバランシングを行うことが可能になります。


## 6.10 Ingress(L7ロードバランシング)


## まとめ
KubernetesではPodのサービスディスカバリやL4ロードバランシング機能を提供するためにServiceリソースが用意されています。
また似たようなリソースとして、L7ロードバランシング機能を提供するIngressも用意されています。

Service
  L4ロードバランシング
  クラスタ内DNSによる名前解決
  ラベルを利用したPodのサービスディスカバリ

Ingress
  L7ロードバランシング
  HTTPS終端
  パスベースルーティング

Serviceではエンドポイントを提供する複数のtypeが用意されており、要件に合わせて選択することが可能です。
基本的には、
内部向けエンドポイントを払い出したいときに使う「type: ClusterIP」
外部向けエンドポイントを払い出したいときに使う「type: LoadBalancer」

Serviceの種類
  |Serviceの種類|IPエンドポイントの内容|
  |---|---|
  |ClusterIP|Kubernetes Cluster内でのみ疎通可能な仮想IP|
  |ExternalIP|特定のKubernetes NodeのIPアドレス|
  |NodePort|全Kubernetes Nodeの全IPアドレス（0.0.0.0）|
  |LoadBalancer|クラスタ外で提供されているLoadBalancerの仮想IP|
  |Headless（None）|PodのIPアドレスを用いたDNS Round Robin|
  |ExternalName|CNAMEを用いた疎結合性の確保|
  |None-Selector|自由に宛先メンバを設定可能な様々な種別のエンドポイント|

Ingressにはtypeなどはありませんが、実装により仕様が大きく異なるケースがある。
Ingressの種類
  |Ingressの種類|実装例|
  |---|---|
  |クラスタ外のロードバランサを利用したIngress|GKE|
  |クラスタ内にIngress用のPodをデプロイするIngress|Nginx Ingress／Nghttpx Ingress|