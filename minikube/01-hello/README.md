# 概要

最初のステップとしてhello-worldを実行する。下記のコマンドを実行する。

## コマンドから直接podを起動する

```
$ kubectl run hello-world --image hello-world --restart=Never
pod/hello-world created
```

podが作成されたと表示された。ただし出力はpodのコンテナ内で行われているためコンソールには何も表示されない。hello-worldが実行されていることを確認するためにまず`kubectl get pod`を実行しPodの一覧を表示する。

```
$ kubectl get pod
NAME          READY   STATUS      RESTARTS   AGE
hello-world   0/1     Completed   0          9m37s
```

Pod名が`hello-world`であることが確認できたので`kubectl logs`で出力を確認する。

```
$ kubectl logs pod/hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.
...以下略...
```

dockerのhello-worldイメージを実行したときと同じ出力が確認できる。確認が終わったので下記のコマンドでpodを削除しておく。

```
$ kubectl delete pod/hello-world
pod "hello-world" deleted
```

本当に削除されたかどうかを最後に確認しておく。

```
$ kubectl get pod
No resources found in default namespace.
```

## 同じ内容をマニフェストファイルから実行する

下記の内容でマニフェストファイル(yaml)を作成する。名前は任意。

```yaml
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

`kubectl apply -f <マニフェストファイル名>`で実行する

```
$ kubectl apply -f pod.yaml
pod/test created
```

作成されたPod名はマニフェストファイルの`metadata.name`で指定した名前となっていることがわかる。Pod一覧を確認してみる。

```
$ kubectl get pod
NAME   READY   STATUS             RESTARTS      AGE
test   0/1     CrashLoopBackOff   1 (16s ago)   45s
```

Pod名がtestとなっていることが確認できた。


### kubectl getの詳細

Podの一覧表示に`kubectl get pod`を使用したが`kubectl get`はその後に続けて表示したいリソースのタイプを指定できる。`kubectl get pod`はPodリソースのみを表示する、という意味になる。`kubectl get all`で全リソースを表示できる。

```
$ kubectl get all
NAME       READY   STATUS      RESTARTS      AGE
pod/test   0/1     Completed   2 (35s ago)   64s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   21h
```

先ほどは`test`と表示されていた行が今度は`pod/test`となっていることがわかる。これはtestリソースの種類がPodであることを示している。`service/kubernates`はkubernatesリソースの種類がServiceであることを示す。

`kubectl get -f <マニフェストファイル>`でマニフェストファイルで指定されたリソースを表示できる。

```
$ kubectl get -f hello.yaml
NAME   READY   STATUS             RESTARTS      AGE
test   0/1     CrashLoopBackOff   2 (41s ago)   97s
```

同様に削除もマニフェストファイルを指定して実行できる。


```
$ kubectl delete -f hello.yaml
pod "test" deleted
```

削除されたことを確認する。

```
$ kubectl get pod
No resources found in default namespace.
```

削除された模様。
