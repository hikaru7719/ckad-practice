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

## myrqと言うResouceQuotaを作成するマニフェストを取得する（実際に作成することはしない）
ResourceQuotaはnamespace単位でリソースの制限を行うためのものである。
```
kubectl create resoucequota myrq --hard=cpu=1,memory=1G,pods=2 --dry-run -o yaml > quota.yml
```

```
cat quota.yml
apiVersion: v1
kind: ResourceQuota
metadata:
  creationTimestamp: null
  name: myrq
spec:
  hard:
    cpu: "1"
    memory: 1G
    pods: "2"
```

## 全てのnamespaceのpodの情報を取得する
```
kubectl get pods --all-namespaces
```

## nginxのpodを作成し、80版ポートのトラフィックを許可する
```
kubectl run nginx --image=nginx --restart=Never --port=80
```

## nginxのpodのimageをnginx:1.7.1に変更し、podが一度削除され、再作成されたことを確認する
```
kubectl set image pod/nginx nginx=nginx:1.7.1
kubectl describe pod nginx
kubectl get pod nginx --watch
```

## nginx podのipを取得し、wgetでindex.htmlを取得する。
```
kubectl get pod -o wide
kubectl run busybox --image=busybox --restart=Never --rm -it -- sh
wget . . . .:80
```

## nginxのpodの情報をyamlで取得する（クラスタ固有の情報は除外）
--exportオプションをつけるとクラスタ固有の情報が除外される。現在はdeprecated
```
kubectl get pod nginx -o yaml --export
```

## nginx podの潜在的な問題についての情報を取得する（例；podがまだstartしてない等）
```
kubectl describe pod nginx
```

## nginx podのログを取得する
```
kubectl logs nginx
```

## podのクラッシュや再起動時、過去のインスタンスのログを取得する。
```
kubectl logs nginx -p
```

## nginx podに接続する
```
kubectl exec ngixn -it -- /bin/sh
```

## busybox podを作成し、hello worldを出力し、exitする。
```
kubectl run busybox --image=busybox --restart=Never -it -- echo "Hello World"
```

## busybox podを作成し、hello worldを出力し、そのコンテナを削除する
```
kubectl run busybox --image=busybox --restart=Never --rm -it -- /bin/sh -c 'echo hello world'
hello world
pod "busybox" deleted
```

## nginx podを作成し、環境変数にval1=val1をセットし、確認する
```
kubectl run nginx --image=nginx --restart=Never --env=val1=val1
kubectl exec -it nginx -- /bin/sh
env | grep val1

or
kubectl describe pod nginx | grep val
``` 