# ------------------------------------------------
# 6.8.1 ExternalName Serviceの作成
# ------------------------------------------------
```kubectl
kubectl apply -f sample-externalname.yaml

# Serviceの一覧を表示
kubectl get services

# 一時的にPodを立ち上げて、ExternalNameのCNAMEの名前解決が行われることの確認
kubectl run --image=centos:6 --restart=Never --rm -i testpod -- dig sample-externalname.default.svc.cluster.local CNAME

kubectl delete -f sample-externalname.yaml
```

# ------------------------------------------------
# 6.8.2 外部サービスとの疎結合の確保
# ------------------------------------------------
クラスタ内では、Pod宛の通信にServiceの名前解決を使うことで疎結合性を保っていました。
一方で、SaaSやIaaSなどの外部にあるサービスを利用する際にも、可能な限り疎結合にする必要があります。

アプリケーションなどに外部のエンドポイントを書き込んでしまうと、
切り替えを実施する際にアプリケーション側の設定変更が必要になってしまいます。

一方でExternalNameを利用すると、
アクセス先の切り替えをExternalName Serviceの変更を行うだけで可能になります（クラスタ内DNSによる名前解決の結果が変わるため）。
切り替えへの対応がKubernetes上のオペレーションで完結することになり、外部サービスとの疎結合性を保てる。

# ------------------------------------------------
# 6.8.3 外部サービスと内部サービス間の切り替え
# ------------------------------------------------
またExternalNameの利用により、外部サービスとの疎結合性の確保に加え、
外部サービスとKubernetes上にデプロイされたクラスタ内サービスとの切り替えも柔軟に行えるようになります。

アプリケーション側はServiceのFQDN（store.default.svc.cluster.local）を指定することで、
その後名前解決されるとExternalNameのCNAMEレコードかClusterIPのAレコードが返る形になるため、
アプリケーション側の改修をせずに内部サービスと外部サービスを切り替えることが可能です。
1つ注意点を挙げるとすれば、ClusterIP ServiceからExternalName Serviceに切り替える場合、spec.clusterIPを明示的に空にする必要があります。

