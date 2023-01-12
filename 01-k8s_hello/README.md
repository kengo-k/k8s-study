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
