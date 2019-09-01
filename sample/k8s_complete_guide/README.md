# k8sの環境構築（開発環境）
## minikubeをインストール
``` bash (mac)
# brewインストール確認（ない場合はインストール）
which brew

# Kubectlをインストール
brew install kubectl
which kubectl

# virtual boxのインストール

# minikubeのインストール
brew cask install minikube
which minikube

```

```
# minikube実行
minikube start

# minikube状態確認
minikube status

kubectl cluster-info
```

# samplek8sの実行
```
kubectl apply -f client-pod.yaml

kubectl apply -f client-node-port.yaml

# 
kubectl get pods
kubectl get service

# vmで動いているminikube ipを確認する（minikubeにアクセスするにはこのipが必要）
minikube ip

# http://192.168.99.100:31515
```