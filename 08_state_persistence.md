# State Persistence
## Define volumes
## 2containerのbusybox podを作成する。どちらのコンテナも'sleep 3600'を実行する。また、/etc/fooにempty dirをマウントする。２つ目のbusyboxに接続し、/etc/foo/paswardに/etc/passwordの第一カラムの内容を記入。１つ目のbusyboxに接続し/etc/foo/passwordの内容を標準出力に書き出す。
```
kubectl run busybox --image=busybox --restart=Never -o yaml --dry-run -- /bin/sh -c 'sleep 3600' > busybox_emptydir.yml

vim busybox_emptydir.yml
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
    - sleep 3600
    image: busybox
    name: busybox
    resources: {}
    volumeMounts:
    - name: myvolume
      mountPath: /etc/foo
  - args:
    - /bin/sh
    - -c
    - sleep 3600
    image: busybox
    name: busybox2
    resources: {}
    volumeMounts:
    - name: myvolume
      mountPath: /etc/foo
  dnsPolicy: ClusterFirst
  restartPolicy: Never
  volumes:
  - name: myvolume
    emptyDir: {}
status: {}
```

```
kubectl apply -f busybox_emptydir.yml
kubectl exec -it busybox --container=busybox -- sh
/ # cat /etc/passwd
root:x:0:0:root:/root:/bin/sh
daemon:x:1:1:daemon:/usr/sbin:/bin/false
bin:x:2:2:bin:/bin:/bin/false
sys:x:3:3:sys:/dev:/bin/false
sync:x:4:100:sync:/bin:/bin/sync
mail:x:8:8:mail:/var/spool/mail:/bin/false
www-data:x:33:33:www-data:/var/www:/bin/false
operator:x:37:37:Operator:/var:/bin/false
nobody:x:65534:65534:nobody:/home:/bin/false
/ # cd /etc/foo
/etc/foo # ls
/etc/foo # vi passwd
/etc/foo # exit

kubectl exec -it busybox --container=busybox -- sh
 / # cat /etc/foo/passwd
root:x:0:0:root:/root:/bin/sh
/ # exit
```

## myvolumeという10GiのPersistent Volumeを作成する。accessModeはReadWriteOnece ReadWriteMenyでstorageClassNameはnormal、hostPathが/etc/fooとする。
```
vim pv.yml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: myvolume
spec:
  storageClassName: normal
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
    - ReadWriteMany
  hostPath:
    path: /etc/foo

kubectl apply -f ps.yml

kubectl get pv
NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
myvolume   10Gi       RWO,RWX        Retain           Available           normal                  55s
```

## mypvcというPersistentVolume Claimを作成する。strageClassNameはnormalでrequestが4Gi、accessModeがReadWriteOnceとする。
```
vim pvc.yml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: mypvc
spec:
  storageClassName: normal
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 4Gi
```
```
kubectl apply -f pvc.yml
kubectl get pvc 
NAME    STATUS   VOLUME     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
mypvc   Bound    myvolume   10Gi       RWO,RWX        normal         64s

kubectl get pv
NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM           STORAGECLASS   REASON   AGE
myvolume   10Gi       RWO,RWX        Retain           Bound    default/mypvc   normal                  10m
```

## sleep 3600を実行するbusyboc podを作成し、/etc/fooにpersistent volume claimをマウントする。その後podにアクセスし、/etc/passwdを/etc/foo/passwdにコピーする

```
ubectl run busybox --image=busybox --restart=Never -o yaml --dry-run -- /bin/sh -c 'sleep 3600' > busybox_pvc.yml
vim busybox_pvc.yml
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
    - sleep 3600
    image: busybox
    name: busybox
    resources: {}
    volumeMounts:
    - name: myvolume
      mountPath: /etc/foo
  dnsPolicy: ClusterFirst
  restartPolicy: Never
  volumes:
  - name: myvolume
    persistentVolumeClaim:
      claimName: mypvc
status: {}
```

```
kubectl exec busybox -it -- cp /etc/passwd /etc/foo/passwd
```
## 同様のpodをもう一つ作成し、/etc/foo/passwdが存在することを確認する
```
cp busybox_pvc.yml busybox_pvc2.yml
vim busybox_pvc2.yml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox2
spec:
  containers:
  - args:
    - /bin/sh
    - -c
    - sleep 3600
    image: busybox
    name: busybox
    resources: {}
    volumeMounts:
    - name: myvolume
      mountPath: /etc/foo
  dnsPolicy: ClusterFirst
  restartPolicy: Never
  volumes:
  - name: myvolume
    persistentVolumeClaim:
      claimName: mypvc
status: {}
```
```
kubectl exec -it busybox -- cat /etc/foo/passwd
root:x:0:0:root:/root:/bin/sh
daemon:x:1:1:daemon:/usr/sbin:/bin/false
bin:x:2:2:bin:/bin:/bin/false
sys:x:3:3:sys:/dev:/bin/false
sync:x:4:100:sync:/bin:/bin/sync
mail:x:8:8:mail:/var/spool/mail:/bin/false
www-data:x:33:33:www-data:/var/www:/bin/false
operator:x:37:37:Operator:/var:/bin/false
nobody:x:65534:65534:nobody:/home:/bin/false
```

## busybox podを作成し、ローカルに/etc/passwdをダウンロードする
```
kubectl run busybox --image=busybox --restart=Never -- sleep 3600
kubectl cp busybox:/etc/passwd ./passwd
```
