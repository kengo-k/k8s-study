```
kind create cluster --config=cluster.yaml
kubectl apply -f app.yaml
```

```
$ kubectl create deploy app-a --image=nginx
$ kubectl create deploy app-b --image=nginx
```

```
$ kubectl expose deploy/app-a --type NodePort --port 80 --target-port 80
$ kubectl expose deploy/app-b --type NodePort --port 80 --target-port 80
```

```
$ kubectl apply -f ingress.yaml
```
