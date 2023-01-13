```
$ kubectl apply -f nginx.yaml
pod/nginx created
```

```
$ kubectl get pod
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          56s
```

```
$ kubectl delete -f nginx.yaml
pod "nginx" deleted
```

```
$ kubectl get pod
No resources found in default namespace.
```
