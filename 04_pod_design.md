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
