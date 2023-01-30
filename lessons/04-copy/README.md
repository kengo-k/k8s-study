- ホストからPodへファイルを転送する
- Podに入りファイルを編集する
- Podからホストにファイルを戻す

マニフェストファイル:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: debug
  namespace: default
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
          value: "86400"
```

上記ファイルをapplyしてPodが実行されている状態で下記のコマンドを実行する

```
$ kubectl cp test.txt debug:/test.txt
```

転送がおそらく成功しているはずなのでPodに入り下記コマンドを実行する。

```
$ kubectl exec -it debug -- sh
sh-4.2# cat /test.txt
Hello
sh-4.2# echo "World" >> /test.txt
sh-4.2# cat /test.txt
Hello
World
```

Pod内で編集を行ったのでPodのファイルをホストへコピーする。

```
$ kubectl cp debug:/test.txt test2.txt
tar: Removing leading `/' from member names
```

ホストにtest2.txtがコピーされていればOK
