nginxを起動し他のデバッグ用Podからアクセスして動作確認する。

マニフェストファイル:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: debug
spec:
  containers:
    - name: debug
      image: centos:7
      command:
        - "sh"
        - "-c"
      args:
        - |
          while true
          do
            sleep ${DELAY}
          done
      env:
        - name: "DELAY"
          value: "5"

---
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - name: nginx
      image: nginx:1.17.2-alpine
```

nginxのPodと動作確認用のdebugのPodを定義した。

マニフェストファイルを適用:
```
$ kubectl apply -f multi_pod.yaml
pod/debug created
pod/nginx created
```

動作確認する際にIPアドレスが必要となる。`-o wide`オプションを使用することでIPアドレスを確認できる。

```
$ kubectl get pod -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
debug   1/1     Running   0          72s   172.17.0.3   minikube   <none>           <none>
nginx   1/1     Running   0          72s   172.17.0.4   minikube   <none>           <none>
```

下記コマンドでdebugのPodに入ることができる。Podに入ることができたらcurlコマンドでnginxのPodのIPアドレスを指定して実行する。

```
$ kubectl exec -it debug -- sh
sh-4.2# curl 172.17.0.4
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...以下略...
```

nginxのページを取得できれば成功。またIPアドレスではなくPod名でアクセスすることも可能。Podの名前は下記の形式となっている。

```
<IPの"."を"-"で置き換えたもの>.<namespace>.pod.<cluster domain>
```

今回の場合はnamespaceは`default`なので

```
172-17-0-4.default.pod.cluster.local
```

となる。実際に試してみる。

```
sh-4.2# curl 172-17-0-4.default.pod.cluster.local
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...以下略...
```

Pod名でアクセスすることが確認できた。
