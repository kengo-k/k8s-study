kindで構築したクラスタ上でサービスを作成し外部から参照できるようにする。

## クラスタを作成する

下記の内容でクラスタ定義ファイルを作成する。

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  extraPortMappings:
    - containerPort: 30080
      hostPort: 80
```

`extraPortMappings`によりホストの80番ポートからノードの30080番へ転送をする設定を追加している。ファイル作成後、下記コマンドを実行してクラスタを再作成する。

```
$ kind delete clusters kind
$ kind create cluster --config=./port_mappings_cluster.yaml
```

## マニフェストファイルを作成する

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kind-port-mappings-pod
  labels:
    app: web
    env: dev
spec:
  containers:
    - name: nginx
      image: nginx:1.17.3-alpine

---
apiVersion: v1
kind: Service
metadata:
  name: kind-port-mappings-service
spec:
  type: NodePort
  selector:
    app: web
    env: dev
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```

`Service`の`ports`設定でノードの30080からPodの80へ転送するように設定を追加している。ファイルを作成しマニフェストを適用する。

```
$ kubectl apply -f port_mappings_app.yaml
pod/kind-port-mappings-pod created
service/kind-port-mappings-service created
```

ホスト上からlocalhostへアクセスしてnginxのページが表示されればOK。

```
$ curl localhost
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...省略...
```
