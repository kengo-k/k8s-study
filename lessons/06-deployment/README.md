```
$ kubectl apply -f deployment.yaml
deployment.apps/nginx created
```

```
$ kubectl get all
NAME                         READY   STATUS    RESTARTS   AGE
pod/nginx-7cdc558d96-665vl   1/1     Running   0          53s
pod/nginx-7cdc558d96-mpb7r   1/1     Running   0          53s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   2d13h

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   2/2     2            2           53s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-7cdc558d96   2         2         2       53s
```


```
$ kubectl rollout history deployment.apps/nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         <none>
```

`deploy/nginx`でも可

```
$ kubectl apply -f deploymentV2.yaml
deployment.apps/nginx configured
```

```
$ kubectl rollout history deployment.apps/nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
2         <none>
3         <none>
```
