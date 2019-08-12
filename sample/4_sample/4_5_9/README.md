# コンテナイメージの更新
```kubectl
kubectl describe pod sample-pod

# nginxのversionを更新
kubectl set image pod sample-pod nginx-container=nginx:1.13

# 確認
kubectl describe pod sample-pod | grep "Image:"
```

# 注意
  setコマンドを使うとマニュフェストを更新することなく設定を更新できますが、
  マニュフェストと一致しなくなってしまうので、乱用はしないようにしましょう。