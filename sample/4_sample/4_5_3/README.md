# sample-podの操作

## Podの作成
```kubectl
# pod作成
kubectl create -f sample-pod.yaml
```

## Podの操作
```kubectl
# Podの一覧取得
kubectl get pods

# リソースの削除
kubectl delete -f sample-pod.yaml

# リソースの即時強制削除
kubectl delete -f sample-pod.yaml --grace-priod 0 --force
```

## yamlの更新と反映
```kubectl
# applyを利用して更新
kubectl apply -f sample-pod.yaml

# sample-podのimage確認
kubectl get pods sample-pod -o jsonpath="{.spec.containers[].image}"
```

## リソースの作成はcreateでなくapplyを利用する
create --save-configやapplyを利用することで、下記のマニュフェストを利用することができます。
  - 前回適用したマニュフェスト
  - 現在クラスタに登録されているマニュフェスト
  - 今回適用するマニュフェスト

しかし、  
createでsaveのオプションをつけないと、  
現在クラスタに登録さてているマニュフェストが作成されてないため、  
差分チェックをすることができなくなる可能性があります。  
そのため、リソース作成時にもapplyを利用したほうがいい。  

