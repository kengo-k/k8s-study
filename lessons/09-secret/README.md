# コマンドからSecretを直接作成する

## リテラルを指定して作成する場合

`kubectl create secret`コマンドで`--from-literal`オプションを指定する。

```
$ kubectl create secret generic <secret name> --from-literal=<secret key>="secret-value1"
```

## ファイルを指定して作成する場合

`kubectl create secret`コマンドで`--from-file`オプションを指定する。

```
$ kubectl create secret generic <secret name> --from-file=</path/to/secret-file>
```

## 実際にSecretを作成して確認する

`--from-literal`と`--from-file`オプションを同時に指定して二つの機密情報を格納するSecretを作成する。まずは`--from-file`で指定するファイル`secret-key2`を下記の内容で作成する。


```
secret-value2
```

下記コマンドでSecretを作成する。

```
$ kubectl create secret generic secret-sample-secret \
  --from-literal=secret-key1="secret-value1" \
  --from-file=./secret-key2
secret/secret-sample-secret created
```

`kubectl get secret`でSecretの一覧が作成されていることを確認。

```
$ kubectl get secret
NAME                   TYPE     DATA   AGE
secret-sample-secret   Opaque   2      4m50s
```

`-o`オプションを使用してyaml形式で内容を出力してみる。

```
$ kubectl get secret secret-sample-secret -o yaml
apiVersion: v1
data:
  secret-key1: c2VjcmV0LXZhbHVlMQ==
  secret-key2: c2VjcmV0LXZhbHVlMgo=
kind: Secret
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"secret-key1":"c2VjcmV0LXZhbHVlMQ==","secret-key2":"c2VjcmV0LXZhbHVlMgo="},"kind":"Secret","metadata":{"annotations":{},"name":"secret-sample-secret","namespace":"default"}}
  creationTimestamp: "2023-01-18T10:49:32Z"
  name: secret-sample-secret
  namespace: default
  resourceVersion: "58210"
  uid: c009fc52-de25-41c9-9d81-2a6dc47f16c7
type: Opaque
```

暗号化された内容が表示されていることがわかる。ただし厳密には暗号化ではなくBase64エンコードされているだけなので簡単に復元できてしまうことに注意が必要。(つまり実質的には機密情報を守る役割を果たせているわけではない)

実際に復号化してみる。

```
$ echo "c2VjcmV0LXZhbHVlMQ==" | base64 --decode
secret-value1%
```

逆の操作を行うにはBase64でエンコードする。

```
$ echo -n secret-value1 | base64
c2VjcmV0LXZhbHVlMQ==
```

手動でSecretのマニフェストファイルを作る場合は上記のようにbase64でエンコードした値を使用して作成することができる。

## SecretをPodに組み込む

先ほど出力したsecretのyaml情報を利用してSecretをPodに組み込んでみる。内容としてはほぼ[08-confng](../08-config/)のConfigMap相当の部分がSecretに置き換わっただけとなる。

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: secret-sample-secret
data:
  secret-key1: c2VjcmV0LXZhbHVlMQ==
  secret-key2: c2VjcmV0LXZhbHVlMgo=

---
apiVersion: v1
kind: Pod
metadata:
  name: secret-sample-pod
  namespace: default
spec:
  containers:
  - name: nginx
    image: nginx:1.17.2-alpine
    env:
    - name: SECRET_KEY1
      valueFrom:
        secretKeyRef:
          name: secret-sample-secret
          key: secret-key1
    volumeMounts:
    - name: secret-storage1
      mountPath: /home/nginx
  volumes:
  - name: secret-storage1
    secret:
      secretName: secret-sample-secret
      items:
        - key: secret-key2
          path: secret-key2

```

`kubectl apply`が正常に実行できたら端末にログインしてSecretを読み込めるか確認する。

```
$ kubectl apply -f secret.yaml
secret/secret-sample-secret created
pod/secret-sample-pod created
```

SecretとPodの作成が正常に終了することが確認できたので端末でSecretの内容を確認する。

```
$ kubectl exec -it secret-sample-pod -- sh
/ # echo $SECRET_KEY1
secret-value1
/ # cat /home/nginx/secret-key2
secret-value2
```
