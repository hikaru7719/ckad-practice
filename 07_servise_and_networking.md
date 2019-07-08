# Service and Networking
## nginx podを作成し、80番portを解放する
```
kubectl run nginx --image=nginx --restart=Never --port=80 --expose
service/nginx created
pod/nginx created
```
podと同時にserviceが作成されていることがわかる。

## ClusterIPが作成されたことを確認する。エンドポイントも同様に確認する
```
kubectl get service
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.100.0.1     <none>        443/TCP   13d
nginx        ClusterIP   10.100.15.50   <none>        80/TCP    95s
```
```
kubectl get endpoints
NAME         ENDPOINTS                                AGE
kubernetes   192.168.190.192:443,192.168.90.135:443   13d
nginx        192.168.215.89:80                        2m19s
```

## ClusterIPを確認し、busybox podからwgetを実行する
```
kubectl get service nginx
kubectl get service nginx                                                          ✔  1593  14:22:42
NAME    TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
nginx   ClusterIP   10.100.15.50   <none>        80/TCP    4m37s
```

```
kubectl run busybox --rm --image=busybox --restart=Never -it -- sh
/ # wget -o- 10.100.15.50:80
Connecting to 10.100.15.50:80 (10.100.15.50:80)
saving to 'index.html'
index.html           100% |************************************************************************************************************************************************************|   612  0:00:00 ETA
'index.html' saved
/ # exit
pod "busybox" deleted
```

## ClusterIPをNodePortに変更し、NodePortのIPに対して、wgetを実行する
```
kubectl edit svc nginx

typeをNodePortに変更する
```
```
kubectl get service nginx
NAME    TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
nginx   NodePort   10.100.15.50   <none>        80:30623/TCP   10m

wget -o- 10.100.15.50:30623
```

## image 'dgkanatsios/simpleapp', replicas=3のfooというDeploymentを作成する。Labelはapp=fooとし、8080番portのトラフィックを許可する。
```
kubectl run foo --image=dgkanatsios/simpleapp --labels=app=foo --port=8080 --replicas=3
```

## IPアドレスを取得し、8080番portにwgetを実行する
```
kubectl get pod -l app=foo -o wide
NAME                   READY   STATUS    RESTARTS   AGE    IP                NODE                                                 NOMINATED NODE
foo-6884ddd565-2ftqb   1/1     Running   0          110s   192.168.171.86    ip-192-168-189-227.ap-northeast-1.compute.internal   <none>
foo-6884ddd565-b74nq   1/1     Running   0          110s   192.168.197.185   ip-192-168-196-192.ap-northeast-1.compute.internal   <none>
foo-6884ddd565-slfgr   1/1     Running   0          110s   192.168.64.214    ip-192-168-82-187.ap-northeast-1.compute.internal    <none>
```

```
kubectl run busybox --rm --image=busybox --restart=Never -it -- sh
/ # wget -o- 192.168.171.86:8080
Connecting to 192.168.171.86:8080 (192.168.171.86:8080)
saving to 'index.html'
index.html           100% |************************************************************************************************************************************************************|    54  0:00:00 ETA
'index.html' saved
/ # ls index.html
index.html
/ # cat index.html
Hello world from foo-6884ddd565-2ftqb and version 2.0
/ #
```

## foo　deploymentを6262番ポートで解放する。その後endpointを確認する。
```
kubectl expose deployment foo --port=6262 --target-port=8080
kubectl get servicfe foo
NAME   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
foo    ClusterIP   10.100.50.159   <none>        6262/TCP   61s

kubectl get endpoints foo
NAME   ENDPOINTS                                                      AGE
foo    192.168.171.86:8080,192.168.197.185:8080,192.168.64.214:8080   2m6s
```

## busybox podを用いてfoo serviceにアクセスする。アクセスしたときに毎回異なるhost nameが返ってくることを確認する。
```
kubectl get service
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
foo          ClusterIP   10.100.50.159   <none>        6262/TCP       5m5s
kubernetes   ClusterIP   10.100.0.1      <none>        443/TCP        15d
nginx        NodePort    10.100.15.50    <none>        80:30623/TCP   32h

kubectl run busybox --image=busybox --restart=Never --rm -it -- sh
/ # wget -o- http://foo:10.100.50.159
wget: bad port spec 'foo:10.100.50.159'
/ # wget -o- http://foo:6262
/ # cat index.html
Hello world from foo-6884ddd565-b74nq and version 2.0
/ # exit

kubectl delete service foo
kubectl delete deployment foo
```

## replocasが２のnginx deploymentを作成し、80番ポートをCluster IPで解放する。その後、access:trueのラベルがついたpodにのみアクセスできるようなNetwork Policyを作成する
```
kubectl run nginx --image=nginx  --replicas=2 --port=80 --expose

vim policy.yml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: aceess-nginx
spec:
  podSelector:
    matchLabels:
      run: nginx
  ingress:
  - from:
    - podSelector:
        matchLabels:
          access: 'true'
```

```
kubectl run busybox --image=busybox --restart=Never --labels=access='true' --rm -it -- wget nginx:80
```