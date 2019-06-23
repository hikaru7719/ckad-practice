# core concept

## namespace（mynamespace）を作成しを、nginx podを立ち上げる。

```
kubectl create namespace mynamespace
kubectl run nginx --image=nginx --restart=Never -n mynamespace
```

contextの確認
```
kubectl config current-context
```

contextを変更
namespaceを作成した、mynamespaceに設定する
```
kubectl config set-context $(kubectl config current-context) --namespace=mynamespace
```

この状態でget podすると作成したnginxが確認できる。
```
kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
nginx   0/1     Pending   0          8m44s
```

contextをdefault namespaceに戻す
```
kubectl config set-context $(kubectl config current-context) --namespace=default
```
## YAMLを使ってnginx podを作成する。

podの情報をyamlファイルに書き出す。
```
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > pod.yml
```

```
cat pod.yml

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

```

pod.ymlでmynamespaceにpodを作成する。
```
kubectl create -f pod.yml -n mynamespace
Error from server (AlreadyExists): error when creating "pod.yml": pods "nginx" already exists
```

name:nginxのpodが既に存在すると怒られる。
既存のnginxを削除。
nameで削除する場合は以下のコマンド。
```
kubectl delete pod nginx -n mynamespace
```
-lでラベル一致でpodを選択できる。

```
kubectl create -f pod.yml -n mynamespace
pod/nginx created
```

## busyboxのpodを作成し、pod上でenvコマンドを実行する。
```
kubectl run busybox --image=busybox --restart=Never -it -- env
```
このコマンドを実行後標準出力に環境変数の一覧が表示される。

```
kubectl logs busybox
```
でもう一度確認することができる。

## busyboxのpodを作成し、pod上でenvコマンドを実行する。（yaml）
```
kubect run busybox --image=busybox --restart=Never --dry-run -o yaml --command -- env > busybox.yml 
```
マニフェスト用のyamlファイルが出力される。

```
cat busybox.yml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  containers:
  - command:
    - env
    image: busybox
    name: busybox
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

kubernetesマニフェストではcommandが起動時に実行されるコマンドで、引き数がある場合にはargsで指定する。

```
kubectl apply -f busybox.yml
kubectl logs busybox
```
マニフェストを適用し、コンテナのログをみる。

## mynmsというnamespaceを作成するマニフェストを取得する(実際に作成することはしない)
```
kubectl create namespace mynms --dry-run -o yaml > mynms.yml
```

```
cat mynms.yml
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: null
  name: mynms
spec: {}
status: {}
```
