Kubernetesには、
Pod内の全コンテナの/etc/hostsを書き換える機能が用意されており、
spec.hostAliasesで指定することが可能です。

```kubectl
kubectl apply -f sample-pod.yaml

kubectl get pods

# Podの詳細情報を表示
kubectl get pods --output wide

# ---------------------------
# hostsの確認
kubectl exec -it sample-hostaliases cat /etc/hosts
exit
# ---------------------------
kubectl delete -f sample-pod.yaml 