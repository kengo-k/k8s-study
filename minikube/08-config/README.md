下記の内容でマニフェストファイルを作成する。

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  # Podから参照されるConfigMapの名前を定義
  name: config-sample-config
  namespace: default
data:
  # 複数の設定値を一括で定義する例。
  # ファイルとしてマウントするために使用する
  config1.conf: |
    key1: value1
    key2: value2
  # 個別に設定を定義する例
  key3: value3

---
apiVersion: v1
kind: Pod
metadata:
  name: config-sample-pod
  namespace: default
spec:
  containers:
  - name: nginx
    image: nginx:1.17.2-alpine
    env:
      # 環境変数KEY3を定義
    - name: KEY3
      # 環境変数KEY3にconfigMapのkey3の値を設定する
      valueFrom:
        configMapKeyRef:
          name: config-sample-config
          key: key3
    volumeMounts:
      # 指定したstorageを所定の位置にマウントする
    - name: storage1
      mountPath: /home/nginx
  # volumeMountsから使用するVolumeを定義する
  volumes:
    # volume名。volumeMountsから参照する際に使用する
  - name: storage1
    configMap:
      # 参照するConfigMapの名前を指定
      name: config-sample-config
      items:
        # ConfigMapで定義された名称を指定
      - key: config1.conf
        # マウントするファイル名を指定
        path: config1.conf
```

```
$ kubectl get all
NAME                    READY   STATUS    RESTARTS   AGE
pod/config-sample-pod   1/1     Running   0          4s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   4d
```

`kubectl get all`してもConfigMapは表示されない。ConfigMapを表示するにはリソースタイプとしてconfigmapを指定する。

```
$ kubectl get configmap
NAME                   DATA   AGE
config-sample-config   2      75s
kube-root-ca.crt       1      4d
```

もしくは省略名`cm`でも可能。

```
$ kubectl get cm
NAME                   DATA   AGE
config-sample-config   2      2m1s
kube-root-ca.crt       1      4d
```

環境変数が設定されていること＆設定ファイルがマウントされていることを確認する。

```
$ kubectl exec -it config-sample-pod -- sh
/ # echo $KEY3
value3

/ # cat /home/nginx/config1.conf
key1: value1
key2: value2
```
