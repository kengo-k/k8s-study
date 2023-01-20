```
$ kubectl get po,svc
NAME                                   READY   STATUS    RESTARTS   AGE
pod/debug                              0/1     Error     0          40s
pod/statefulset-sample-statefulset-0   1/1     Running   0          6m35s

NAME                                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes                   ClusterIP   10.96.0.1    <none>        443/TCP   7m11s
service/statefulset-sample-service   ClusterIP   None         <none>        80/TCP    6m35s
```

```
$ kubectl run debug -it --restart=Never --image=centos:7 --command -- sh
```
