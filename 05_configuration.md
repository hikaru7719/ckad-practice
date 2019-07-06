# configmap

## configという名前のConfigMapを作成する。値はfoo=lala,foo2=loloとする
```
kubectl create configmap config --from-literal=foo=lala --from-literal=foo2=lolo
```

## ConfiglMapの値を確認する
```
kubectl get cm config -o yaml --export
apiVersion: v1
data:
  foo: lala
  foo2: lolo
kind: ConfigMap
metadata:
  creationTimestamp: null
  name: config
  selfLink: /api/v1/namespaces/default/configmaps/config
```
or
```
kubeclt describe cm config
Name:         config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
foo:
----
lala
foo2:
----
lolo
Events:  <none> 
```

## fileからConfigMapを作成し、その値を確認する
```
echo -e "foo3=lili\nfoo4=lele" > config.txt
kubectl create configmap config2 --from-file=config.txt
kubectl get cm config2 -o yaml --export
```

## envfileからConfigMapを作成し、その値を確認する
```
echo -e "val1=val1\n#this is a comment\nval2=val2\n#anothercomennt" > config.env
kubectl create cm config3 --from-env-file=config.env
kubeclt get cm config3 -o yaml --export
apiVersion: v1
data:
  val1: val1
  val2: val2
kind: ConfigMap
metadata:
  creationTimestamp: null
  name: config3
  selfLink: /api/v1/namespaces/default/configmaps/config3
```
env fileのコメント文は含まれずにConfigMapが作成される

## fileからConfigMapを作成する。Keyはspecialとする
```
echo -e "val3=val3\nval4=val4\n" > special.txt
kubectl create cm config4 --from-file=special=specila.txt
kubectl get cm config4 -o yaml --export
apiVersion: v1
data:
  special: |+
    val3=val3
    val4=val4

kind: ConfigMap
metadata:
  creationTimestamp: null
  name: config4
  selfLink: /api/v1/namespaces/default/configmaps/config4
```

## optionsというConfigMapを作成する。ConfigMapの内容はval5=val5。その後新しくnginx podを作成し、CogfigMapをnginx podから読み込む。環境変数optionに対して、ConfigMapのval5の値を設定する。
```
kubectl create configmap options --from-literal=val5=val5
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > nginx.yml
vim nginx.yml

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
    env:
    - name: option
      valueFrom:
        configMapKeyRef:
          name: options
          key: val5
  dnsPolicy: ClusterFirst
  restartPolicy: Neve
```
```
kubectl apply -f nginx.yml
kubectl exec nginx -- env | grep option
option=val5
```

## anotheroneというConfigMapを作成する。値はvar6=var6 var7=var7とする。このConfgiMapを新しく作成するNginx Podの環境変数としてロードする。
```
kubectl create configmap anotherone --from-literal=var6=var6 --from-literal=var7=var7
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > nginx2.yml
vim nginx2.yml

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
    envFrom:
    - configMapRef:
        name: anotherone
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

kubeclt apply -f nginx2.yml
kubectl exec nginx -- env | grep var
var6=var6
var7=var7
```

## ConfigMapを作成し、それをVolumeとしてロードする
```
 kubectl create configmap cmvolume --from-literal=var8=val8 --from-literal=var9=val9
 kubect run nginx --image=nginx --restart=Never -o yaml --dry-run > nginx3.yml
 vim nginx3.yml

 apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  volumes:
  - name: myvolume
    configMap:
      name: cmvolume
  containers:
  - image: nginx
    name: nginx
    resources: {}
    volumeMounts:
    - name: myvolume
      mountPath: /etc/lala
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```
```
kubectl apply -f nginx3.yml

kubectl exec -it nginx -- /bin/bash
cd /etc/lala
ls
var8  var9
catr var8
val8
```