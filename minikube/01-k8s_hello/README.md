```
$ kubectl run hello-world --image hello-world --restart=Never
pod/hello-world created
```

```
$ kubectl get pod
NAME          READY   STATUS      RESTARTS   AGE
hello-world   0/1     Completed   0          9m37s
```

```
$ kubectl logs pod/hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.
...以下略...
```

dockerのhello-worldを実行したときと同様の出力が確認できる。


```
$ kubectl delete pod/hello-world
pod "hello-world" deleted
```

```
$ kubectl get pod
No resources found in default namespace.
```

```yaml:hello.yaml
apiVersion: v1
kind: Pod
metadata:
  name: test
  namespace: default
  labels:
    env: study
spec:
  containers:
    - name: hello-world
      image: hello-world
```

```
$ kubectl apply -f hello.yaml
pod/test created
```

```
$ kubectl get pod
NAME   READY   STATUS             RESTARTS      AGE
test   0/1     CrashLoopBackOff   1 (16s ago)   45s
```

```
$ kubectl get -f hello.yaml
NAME   READY   STATUS             RESTARTS      AGE
test   0/1     CrashLoopBackOff   2 (41s ago)   97s
```

```
$ kubectl get all
NAME       READY   STATUS      RESTARTS      AGE
pod/test   0/1     Completed   2 (35s ago)   64s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   21h
```

```
$ kubectl delete -f hello.yaml
pod "test" deleted
```

```
$ kubectl get pod
No resources found in default namespace.
```
