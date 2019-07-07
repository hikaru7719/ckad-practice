# Obsrevability

# Liveness and readiness proves

## liveness probeを設定したnginx podを作成し、lsコマンドを実行する。その後liveness probeを確認する

```
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > nginx_liveness.yml
vim nginx_liveness.yml
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
    livenessProbe:
      exec:
        command:
        - ls
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```
kubectl apply -f nginx_liveness.yml
kubectl describe pod nginx | grep liveness
   Liveness:       exec [ls] delay=0s timeout=1s period=10s #success=1 #failure=3
```

## nginx_liveness.ymlを修正し、podがStartして5秒後に開始し、Periodを10秒後に設定する
```
kubectl explain pod.spec.containers.livenessProbe

KIND:     Pod
VERSION:  v1

RESOURCE: livenessProbe <Object>

DESCRIPTION:
     Periodic probe of container liveness. Container will be restarted if the
     probe fails. Cannot be updated. More info:
     https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle#container-probes

     Probe describes a health check to be performed against a container to
     determine whether it is alive or ready to receive traffic.

FIELDS:
   exec	<Object>
     One and only one of the following should be specified. Exec specifies the
     action to take.

   failureThreshold	<integer>
     Minimum consecutive failures for the probe to be considered failed after
     having succeeded. Defaults to 3. Minimum value is 1.

   httpGet	<Object>
     HTTPGet specifies the http request to perform.

   initialDelaySeconds	<integer>
     Number of seconds after the container has started before liveness probes
     are initiated. More info:
     https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle#container-probes

   periodSeconds	<integer>
     How often (in seconds) to perform the probe. Default to 10 seconds. Minimum
     value is 1.

   successThreshold	<integer>
     Minimum consecutive successes for the probe to be considered successful
     after having failed. Defaults to 1. Must be 1 for liveness. Minimum value
     is 1.

   tcpSocket	<Object>
     TCPSocket specifies an action involving a TCP port. TCP hooks not yet
     supported

   timeoutSeconds	<integer>
     Number of seconds after which the probe times out. Defaults to 1 second.
     Minimum value is 1. More info:
     https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle#container-probes
```
```
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > nginx_liveness2.yml
vim nginx_liveness2.yml
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
    livenessProbe:
      initialDelaySeconds: 5
      periodSeconds: 10
      exec:
        command:
        - ls

  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```
```
kubectl apply -f nginx_liveness2.yml
kubectl describe pod nginx | grep -i liveness
 Liveness:       exec [ls] delay=5s timeout=1s period=10s #success=1 #failure=3
```

## nginx podをport80で解放し、/のPATHに対して、HTTP Readness Probeを実行する
```
kubectl run nignx --image=nginx --restart=Never --dry-run -o yaml > nginx_readiness.yml
vim nginx_readiness.yml
```
```
kubectl apply -f nginx_readiness.yml
kubectl describe pod nginx | grep -i readiness
```

# Ligging
## BusyBox podを作成し、i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 1; doneを実行し、ログを確認する
```
kubectl run busybox --image=busybox --restart=Never -- /bin/sh -c 'i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 1; done'

kubectl logs busybox -f
0: Sun Jul  7 04:34:51 UTC 2019
1: Sun Jul  7 04:34:52 UTC 2019
2: Sun Jul  7 04:34:53 UTC 2019
3: Sun Jul  7 04:34:54 UTC 2019
4: Sun Jul  7 04:34:55 UTC 2019
5: Sun Jul  7 04:34:56 UTC 2019
6: Sun Jul  7 04:34:57 UTC 2019
7: Sun Jul  7 04:34:58 UTC 2019
8: Sun Jul  7 04:34:59 UTC 2019
9: Sun Jul  7 04:35:00 UTC 2019
10: Sun Jul  7 04:35:01 UTC 2019
11: Sun Jul  7 04:35:02 UTC 2019
12: Sun Jul  7 04:35:03 UTC 2019
13: Sun Jul  7 04:35:04 UTC 2019
14: Sun Jul  7 04:35:05 UTC 2019
```

# Debugging
## busybox podを作成し、ls /notexsist コマンドを実行し、エラーを確認する
```
kubectl run busybox --restart=Never --image=busybox -- /bin/sh -c 'ls /notexist'
kubectl logs busybox
ls: /notexsist: No such file or directory

kubectl describe po busybox
Name:               busybox
Namespace:          default
Priority:           0
PriorityClassName:  <none>
Node:               ip-192-168-196-192.ap-northeast-1.compute.internal/192.168.196.192
Start Time:         Sun, 07 Jul 2019 13:38:52 +0900
Labels:             run=busybox
Annotations:        <none>
Status:             Failed
IP:                 192.168.194.7
Containers:
  busybox:
    Container ID:  docker://93308dedbdb2a6c10a10c32be36de6ddf34b31edee4f5c49b69c36216ac691dd
    Image:         busybox
    Image ID:      docker-pullable://busybox@sha256:c94cf1b87ccb80f2e6414ef913c748b105060debda482058d2b8d0fce39f11b9
    Port:          <none>
    Host Port:     <none>
    Args:
      /bin/sh
      -c
      ls /notexsist
    State:          Terminated
      Reason:       Error
      Exit Code:    1
      Started:      Sun, 07 Jul 2019 13:38:56 +0900
      Finished:     Sun, 07 Jul 2019 13:38:56 +0900
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-bntv4 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             False
  ContainersReady   False
  PodScheduled      True
Volumes:
  default-token-bntv4:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-bntv4
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age   From                                                         Message
  ----    ------     ----  ----                                                         -------
  Normal  Scheduled  17s   default-scheduler                                            Successfully assigned default/busybox to ip-192-168-196-192.ap-northeast-1.compute.internal
  Normal  Pulling    16s   kubelet, ip-192-168-196-192.ap-northeast-1.compute.internal  pulling image "busybox"
  Normal  Pulled     13s   kubelet, ip-192-168-196-192.ap-northeast-1.compute.internal  Successfully pulled image "busybox"
  Normal  Created    13s   kubelet, ip-192-168-196-192.ap-northeast-1.compute.internal  Created container
  Normal  Started    13s   kubelet, ip-192-168-196-192.ap-northeast-1.compute.internal  Started container

kubectl delete po busybox
```

## busybox podを作成し、notexsistコマンドを実行し、エラーを確認する
```
kubectl run busybox --image=busybox --restart=Never -- notexsit
kubectl logs busybox
# Startに失敗したので何も表示されない

kubeclt describe pod busybox
Name:               busybox
Namespace:          default
Priority:           0
PriorityClassName:  <none>
Node:               ip-192-168-82-187.ap-northeast-1.compute.internal/192.168.82.187
Start Time:         Sun, 07 Jul 2019 13:43:05 +0900
Labels:             run=busybox
Annotations:        <none>
Status:             Failed
IP:                 192.168.68.201
Containers:
  busybox:
    Container ID:  docker://a89499e1839bbeb9cf76aaaf31679f3cd665bed7609fbdbc98549c01c8a3f10c
    Image:         busybox
    Image ID:      docker-pullable://busybox@sha256:c94cf1b87ccb80f2e6414ef913c748b105060debda482058d2b8d0fce39f11b9
    Port:          <none>
    Host Port:     <none>
    Args:
      notexsit
    State:          Terminated
      Reason:       ContainerCannotRun
      Message:      OCI runtime create failed: container_linux.go:348: starting container process caused "exec: \"notexsit\": executable file not found in $PATH": unknown
      Exit Code:    127
      Started:      Sun, 07 Jul 2019 13:43:08 +0900
      Finished:     Sun, 07 Jul 2019 13:43:08 +0900
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-bntv4 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             False
  ContainersReady   False
  PodScheduled      True
Volumes:
  default-token-bntv4:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-bntv4
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason     Age   From                                                        Message
  ----     ------     ----  ----                                                        -------
  Normal   Scheduled  14s   default-scheduler                                           Successfully assigned default/busybox to ip-192-168-82-187.ap-northeast-1.compute.internal
  Normal   Pulling    13s   kubelet, ip-192-168-82-187.ap-northeast-1.compute.internal  pulling image "busybox"
  Normal   Pulled     11s   kubelet, ip-192-168-82-187.ap-northeast-1.compute.internal  Successfully pulled image "busybox"
  Normal   Created    11s   kubelet, ip-192-168-82-187.ap-northeast-1.compute.internal  Created container
  Warning  Failed     11s   kubelet, ip-192-168-82-187.ap-northeast-1.compute.internal  Error: failed to start container "busybox": Error response from daemon: OCI runtime create failed: container_linux.go:348: starting container process caused "exec: \"notexsit\": executable file not found in $PATH": unknown

kubectl get events | grep -i error
3m22s       Warning   Failed      Pod    Error: failed to start container "busybox": Error response from daemon: OCI runtime create failed: container_linux.go:348: starting container process caused "exec: \"notexsit\": executable file not found in $PATH": unknown 
```

## nodeのcpu/memory utilizationを確認する
```
kubectl top nodes
```