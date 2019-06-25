# pod design

# label and annotation

##  label app=v1でnginx1,nginx2,nginx3のpodを作成する
```
kubectl run nginx1 --image=nginx --restart=Never --labels=app=v1
kubectl run nginx2 --image=nginx --restart=Never --labels=app=v1
kubectl run nginx3 --image=nginx --restart=Never --labels=app=v1
```

## podの全てのlabelを表示する
```
kubectl get pod --show-labels
NAME     READY   STATUS    RESTARTS   AGE   LABELS
nginx1   1/1     Running   0          84s   app=v1
nginx2   1/1     Running   0          77s   app=v1
nginx3   1/1     Running   0          71s   app=v1
```

## pod nginx2のlabelをapp=v2に変更する
```
kubectl label pod nginx2 app=v2 --overwrite
kubectl get pod --show-labels
NAME     READY   STATUS    RESTARTS   AGE     LABELS
nginx1   1/1     Running   0          5m19s   app=v1
nginx2   1/1     Running   0          5m12s   app=v2
```

## podのappラベルの値を取得する
```
kubectl get pod -L app
NAME     READY   STATUS    RESTARTS   AGE     APP
nginx1   1/1     Running   0          6m29s   v1
nginx2   1/1     Running   0          6m22s   v2
nginx3   1/1     Running   0          6m16s   v1
```

## app=v2のラベルのpodだけ取得する
```
kubectl get pod -l app=v2
NAME     READY   STATUS    RESTARTS   AGE
nginx2   1/1     Running   0          9m10s
```

## 作ったpodのappラベルを取り除く
```
kubectl label pod nginx1 nginx2 nginx3 app-

kubectl get pod --show-labels
NAME     READY   STATUS    RESTARTS   AGE   LABELS
nginx1   1/1     Running   0          12m   <none>
nginx2   1/1     Running   0          12m   <none>
nginx3   1/1     Running   0          11m   <none>
```

## 作ったpodに"description='my description'" annotationをつける
```
kubectl annotate pod nginx1 nginx2 nginx3 description='my description'
```

## podのアノテーションを確認する
```
kubectl describe pod nginx1 | grep -i 'annotations'
Annotations:        description: my description
```

## アノテーションを削除する
```
kubectl annotate pod nginx{1..3} description-
```

## podを削除する
```
kubectl delete pod nginx{1..3}
```

# Deployment

## Deploymentを作成する。（image nginx:1.7.8, replicas 2, 80portを解放）
```
kubectl create deployment nginx --image=nginx:1.7.8 --dry-run -o yaml > deployment.yml
```

vimでreplicasを2に変更し、ports以下を追加する。
```
vim deployment.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.7.8
        name: nginx
        resources: {}
        ports:
          - containerPort: 80
status: {}
```

## 作成したdeploymentのYAMLを取得する
```
kubectl get deploy nginx --export -o yaml
```

## deploymentによって作成されたreplica setのYAMLを取得する
```
kubectl describe deployment nginx

・
・
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  10m   deployment-controller  Scaled up replica set nginx-8b56f79ff to 2
```

```
kubectl get rs nginx-8b56f79ff -o yaml --export
```

もしくは
```
kubectl get rs -l app=nginx -o yaml
```

## podのYAMLを取得する
```
kubectl get pod
NAME                    READY   STATUS    RESTARTS   AGE
nginx-8b56f79ff-gr6dp   1/1     Running   0          15m
nginx-8b56f79ff-nwfll   1/1     Running   0          15m
```

```
kubectl get pod -l app=nginx
```

```
kubectl get pod nginx-8b56f79ff-gr6dp -o yaml --export
```

## rolloutの状態を確認する
```
kubectl rollout status deploy nginx
```

## nginxのimageをnginx:1.7.9にアップデートする
```
kubectl set image deploy nginx nginx=nginx:1.7.9
```
もしくは
```
kubectl edit deploy nginx
```

## rolloutのhistoryをチェックし、replcasetがOKであることを確認する
```
kubectl rollout history deploy nginx

kubectl get deploy nginx

kubectl get rs
NAME               DESIRED   CURRENT   READY   AGE
nginx-5c689d88bb   2         2         2       10m
nginx-8b56f79ff    0         0         0       33m
```

## 最新のrolloutをUNDOし、立ち上がったpodのimageがnginx:1.7.8であることを確認する
```
kubectl rollout undo deploy nginx
kubectl describe pod nginx-8b56f79ff-f9k8m | grep -i image
```

## 間違ったimageでdeploymentをupdateする
```
kubectl set image deploy nginx nginx=nginx:1.91
```

## rolloutで失敗したことを確認する
```
kubectl rollout status deploy nginx
kubeclt get pod
```

## Deploymentをリビジョン2に戻し、imageがnginx:1.7.9であることを確認する
```
kubectl rollout undo deploy nginx --to-revision=2

kubectl describe deploy nginx | grep -i image
Image:        nginx:1.7.9

kubectl rollout status deploy nginx
deployment "nginx" successfully rolled out

```

## Deploymentのリビジョン3の詳細を確認する
```
kubectl rollout history deploy nginx --revision=3
deployment.extensions/nginx with revision #3
Pod Template:
  Labels:	app=nginx
	pod-template-hash=8b56f79ff
  Containers:
   nginx:
    Image:	nginx:1.7.8
    Port:	80/TCP
    Host Port:	0/TCP
    Environment:	<none>
    Mounts:	<none>
  Volumes:	<none>
```

## Deploymentをreplicas=5にscaleする
```
kubectl scale deploy nginx --replicas=5
```
```
kubectl get pod
kubectl describe deploy nginx
```

## AutoScaleの設定をする
```
kubectl autoscale deploy nginx --min=5 --max=10 --cpu-percent=80
```

## rolloutのpauseする
```
kubectl rollout pause deploy nginx
```

## nginxのimageを1.9.1に設定しても、updateされないことを確認する
```
kubectl edit deploy nginx
kubectl rollout history deploy nginx
```

## rolloutをresumeしてimageがupdateされていることを確認する
```
kubectl rollout resume deploy nginx
kubectl rollout history deploy nginx --revision=6
deployment.extensions/nginx with revision #6
Pod Template:
  Labels:	app=nginx
	pod-template-hash=6987cdb55b
  Containers:
   nginx:
    Image:	nginx:1.9.1
    Port:	80/TCP
    Host Port:	0/TCP
    Environment:	<none>
    Mounts:	<none>
  Volumes:	<none>
```

## deploymentとHPAを削除する
```
kubeclt delete deploy nginx
kubectl delete hpa nginx
```