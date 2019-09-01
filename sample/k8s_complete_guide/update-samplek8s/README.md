```
kubectl apply -f client-pod.yaml

kubectl describe pod client-pod

kubectl delete -f client-pod.yaml

kubectl apply -f client-deployment.yaml

kubectl get pods
kubectl get deployments

kubectl get pods -o wide
```

## imageのupdateの対応
docker imageを作成時にバージョンを着けて、docker hubに格納  
kubectl（命令）を用いて、podのimageをupdateする。  

```
# 事前にdocker imageをdocker hubに格納しましょう(今回はv5として格納した)
kubectl set image deployment/client-deployment client=stephengrider/multi-client:v5
```

## docker serverの確認
```
# docker-clientの見先をvm上のminikubeに変更
eval $(minikube docker-env)

```