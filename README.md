# 概要
オリジナルのnginxイメージを作成し、ローカルのk8sクラスタにて起動し、外部からアクセスできるようにする手順。
個人的なチュートリアル。

## 前提
dockerがインストール済
minikubeがインストール済
kubectrlがインストール済
docker hubにアカウントがあること

## minikube起動
minikube start

## オリジナルのnginxイメージ作成
### HTML作成

mkdir html
touch html/index.html
vi html/index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Test Nginx</title>
</head>
<body>
  <p>Test Nginx</p>
</body>
</html>
```

### Dockerfile作成
touch Dockerfile
```
FROM nginx:1.19.0

COPY ./html /usr/share/nginx/html
```

### イメージビルド
docker build --tag miyazi777/test-nginx .

### コンテナを起動してテスト
docker run --rm --name ts-nginx -p 80:80 miyazi777/test-nginx

### docker hubに登録
docker push miyazi777/test-nginx

## minikube上のkubernetesにデプロイ
### deployment作成
#### マニフェストを作成
touch nginx-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-nginx-deployment
  labels:
    component: test-nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      component: test-nginx
  template:
    metadata:
      labels:
        component: test-nginx
    spec:
      containers:
      - name: test-nginx
        image: miyazi777/test-nginx
```

#### デプロイ

kubectl apply -f nginx-deployment.yaml

kubectl get deployments

### サービス作成

#### サービス用のマニフェスト作成

touch nginx-service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: test-nginx-service
spec:
  selector:
    component: test-nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30001
  type: NodePort
```

#### サービスを適用
kubectl apply -f nginx-service.yaml

kubectl get service

## 動作確認
### ipアドレス確認
minikube ip

curl http://<ip-address>:30001/


