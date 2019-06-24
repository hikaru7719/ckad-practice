# Multi container pods

## 2つのコンテナが入ったpodを作成する。どちらのimageもbusyboxで"echo hello; sleep 3600"コマンドを実行する。2つめのコンテナに入り、lsコマンドを実行する。

```
kubectl run busybox --image=busybox --restart=Never --dry-run -o yaml  -- /bin/sh -c 'echo hello;sleep 3600' > two_container.yml
```

```
vim two_container.yml
```
container以下を修正。

```
cat two_container.yml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  containers:
  - args:
    - /bin/sh
    - -c
    - echo hello;sleep 3600
    image: busybox
    name: busybox
    resources: {}
  - args:
    - /bin/sh
    - -c
    - echo hello;sleep 3600
    image: busybox
    name: busybox2
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```
kubectl create -f two_contaier.yml
kubectl exec -it busybox -c busybox2 -- /bin/sh
/ # ls
bin   dev   etc   home  proc  root  sys   tmp   usr   var
/ # exit
```