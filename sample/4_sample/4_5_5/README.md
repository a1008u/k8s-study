# sample-podの操作

## Podの作成
```kubectl
# pod作成
kubectl apply -f sample-pod-multi.yaml
```

## yamlの起動例
```
ls -R ./
sample-pod.yaml
sample-pod2.yaml
dir/sample-pod3.yaml

# カレントディレクトリのyamlかつ再帰的に全てのyamlを実行する
kubectl apply -f ./ -R
```

```kubectl
# ラベルを使って、リソースを
kubectl get pods -l label1
kubectl get pods -L label1
```