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

# SecurityContext

## UID 101として実行するnignx Podの作成
```
 kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > security_context.yml
 vim security_context.yml
 apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  securityContext:
    runAsUser: 101
  containers:
  - image: nginx
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

## capabilityにNetAdmin,SYS_TIMEを付与してnginx podを実行する
```
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > capability.yml
vim capability.yml
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
    securityContext:
      capabilities:
        add: ["NET_ADMIN", "SYS_TIME"]
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

# RequestLimit
## requestがcpu=100m, memory=256Mi limitsがcpu=200m, memory=512Miのnginx podの作成
```
kubectl run nginx --image=nginx --restart=Never --requests='cpu=100m,memory=256' --limits='cpu=200m,memory=512Mi'
```
```
kubectl get pod nginx -o yaml --export
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
  selfLink: /api/v1/namespaces/default/pods/nginx
spec:
  containers:
  - image: nginx
    imagePullPolicy: Always
    name: nginx
    resources:
      limits:
        cpu: 200m
        memory: 512Mi
      requests:
        cpu: 100m
        memory: "256"
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-bntv4
      readOnly: true
  dnsPolicy: ClusterFirst
  nodeName: ip-192-168-196-192.ap-northeast-1.compute.internal
  priority: 0
  restartPolicy: Never
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: default-token-bntv4
    secret:
      defaultMode: 420
      secretName: default-token-bntv4
status:
  phase: Pending
  qosClass: Burstable
```

# Secrets
## fileからkey/valueを取得し,nameがmysecret2のSecreteを作成する
```
echo admin > username
kubectl create secret generic mysecret2 --from-file=username
```

## 作成したSecretを確認する
```
kubectl get secret mysecret2 -o yaml --export
apiVersion: v1
data:
  username: YWRtaW4K
kind: Secret
metadata:
  creationTimestamp: null
  name: mysecret2
  selfLink: /api/v1/namespaces/default/secrets/mysecret2
type: Opaque

echo YWRtaW4K | base64 -D
admin
```

```
kubectl get secret mysecret2 -o jsonpath='{.data.username}{"\n"}' | base64 -D
admin
```
## nginx podを作成し、mysecret2を/etc/fooにマウントする

```
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > nginx_secret.yml
vim nginx_secret.yml
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  volumes:
  - name: foo
    secret:
      secretName: mysecret2
  containers:
  - image: nginx
    name: nginx
    resources: {}
    volumeMounts:
    - name: foo
      mountPath: /etc/foo
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```
```
kubectl apply -f nginx_secret.yml
kubectl exec -it nginx -- /bin/sh
root@nginx:/# cd /etc/foo
root@nginx:/etc/foo# ls
username
root@nginx:/etc/foo# cat username
admin
root@nginx:/etc/foo#
```

## secretをpodの環境変数としてロードする
```
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > nginx_secret_env.yml
vim nginx_secret_env.yml
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
    - name: USERNAME
      valueFrom:
        secretKeyRef:
          name: mysecret2
          key: username
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```
```
kubectl apply -f nginx_secret_env.yml
kubectl exec nginx -- env | grep USERNAME
USERNAME=admin
```

# ServiceAccount

## 全てのNameSpaceのServiceAccountを取得する
```
kubectl get sa --all-namespaces
NAMESPACE     NAME         SECRETS   AGE
default       default      1         12d
kube-public   default      1         12d
kube-system   aws-node     1         12d
kube-system   coredns      1         12d
kube-system   default      1         12d
kube-system   kube-proxy   1         12d
``` 

## myuserというServiceAccountを作成する
```
kubeclt create sa myuser
```

## nginx podを作成し、ServiceAccountとしてmyuserを指定する
```
kubectl run nginx --image=nginx --restart=Never -o yaml --dry-run > nginx_service_account.yml
vim nginx_service_account.yml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  serviceAccountName: myuser
  containers:
  - image: nginx
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```
```
kubectl describe pod nginx
```
service accountのトークンがマウントされていることがわかる。
