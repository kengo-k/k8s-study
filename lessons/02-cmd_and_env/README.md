コマンドの実行と環境変数を設定する方法を確認する。まずは下記の内容でマニフェストファイルを作成する。


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: command
spec:
  containers:
    - name: echo-hello
      image: ubuntu:22.04
      command: ["/bin/bash", "-c"]
      args: ["echo Hello, ${NAME}!"]
      env:
        - name: "NAME"
          value: "World"
```

`env`で環境変数`NAME`を指定してechoコマンドの引数に指定している。`kubectl apply`でマニフェストファイルを適用する。

```
$ kubectl apply -f pod.yaml
pod/command created
```

ログを確認する。

```
$ kubectl logs command
Hello, World!
```

環境変数が正しく出力されていることを確認できた。
