```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-sample
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
      env: dev
  template:
    metadata:
      name: nginx
      labels:
        app: web
        env: dev
    spec:
      containers:
        - name: nginx
          image: nginx:1.17.2-alpine
```

- replicasでPodの数を指定する
- selector.matchLabelsとtemplate.metadata.labelsを一致させること

```
$ kubectl apply -f replicaset.yaml
replicaset.apps/replicaset-sample created
```

```
$ kubectl get all
NAME                          READY   STATUS    RESTARTS   AGE
pod/replicaset-sample-szr5d   1/1     Running   0          89s
pod/replicaset-sample-wrdr7   1/1     Running   0          89s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   2d13h

NAME                                DESIRED   CURRENT   READY   AGE
replicaset.apps/replicaset-sample   2         2         2       89s
```
