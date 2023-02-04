kindで簡単にマルチノードクラスタを作成できるので試してみる。まずは下記の内容でクラスタの定義ファイルを作成する。

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
```

`nodes`設定で複数の`worker`を定義することでマルチノード構成になる。この例ではマスター1つ、ワーカー2つのクラスタとなる。ファイル作成後、下記コマンドを実行してクラスタを再作成する

```
$ kind delete clusters kind
$ kind create cluster --config=./multi_nodes_cluster.yaml
```

正常に処理が終了すれば複数のノード(Dockerコンテナ)が作成されているはずなので確認する。

```
$ docker ps --format "table {{.ID}}\t{{.Image}}\t{{.Names}}"
CONTAINER ID   IMAGE                                 NAMES
e9f32f52b91a   kindest/node:v1.25.3                  kind-worker
dd085ea2d128   kindest/node:v1.25.3                  kind-control-plane
1abc6cdb879e   kindest/node:v1.25.3                  kind-worker2
46b4be0c9489   gcr.io/k8s-minikube/kicbase:v0.0.36   minikube
```

マスターノード1つ、ワーカーノード2つ作成されていることが確認できた。念のため`kubectl get nodes`も見ておく。

```
$ kubectl get no
NAME                 STATUS   ROLES           AGE   VERSION
kind-control-plane   Ready    control-plane   14m   v1.25.3
kind-worker          Ready    <none>          14m   v1.25.3
kind-worker2         Ready    <none>          14m   v1.25.3
```

Dockerコンテナと同じ名前を持つノードが存在していることが確認できた。続いて実際に複数のPodを作成しワーカーノードに振り分けられるかどうかを確認する。下記のマニフェストファイルを作成する。

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kind-multi-node-deployment
  labels:
    app: web
spec:
  replicas: 5
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - image: nginx
        name: nginx
        ports:
        - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: kind-multi-node-service
spec:
  selector:
    app: web
  ports:
    - port: 80
```

`replicas`の値が5となっているため5つのPodがワーカノードに振り分けられた起動するはずである。実際にapplyして確認してみる。

```
$ kubectl apply -f multi_nodes_app.yaml
deployment.apps/kind-multi-node-deployment unchanged
service/kind-multi-node-service created
```

作成されたpodとdeploymentを確認する。

```
$ kubectl get po,deploy
NAME                                              READY   STATUS    RESTARTS   AGE
pod/kind-multi-node-deployment-695858cd9d-2wlqf   1/1     Running   0          9m17s
pod/kind-multi-node-deployment-695858cd9d-gm428   1/1     Running   0          9m13s
pod/kind-multi-node-deployment-695858cd9d-qw9qt   1/1     Running   0          9m14s
pod/kind-multi-node-deployment-695858cd9d-z768h   1/1     Running   0          9m17s
pod/kind-multi-node-deployment-695858cd9d-zm72c   1/1     Running   0          9m17s

NAME                                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/kind-multi-node-deployment   5/5     5            5           9m36s
```

5つのPodが作成されていることは確認できたがどのノードに振り分けられているかが確認できなかった。`kukectl get po -o wide`を使用して詳細を表示することでノードが確認できる。

```
$ kubectl get po -o wide
NAME                                          READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
kind-multi-node-deployment-695858cd9d-2wlqf   1/1     Running   0          10m   10.244.1.4   kind-worker    <none>           <none>
kind-multi-node-deployment-695858cd9d-gm428   1/1     Running   0          10m   10.244.2.7   kind-worker2   <none>           <none>
kind-multi-node-deployment-695858cd9d-qw9qt   1/1     Running   0          10m   10.244.2.6   kind-worker2   <none>           <none>
kind-multi-node-deployment-695858cd9d-z768h   1/1     Running   0          10m   10.244.2.5   kind-worker2   <none>           <none>
kind-multi-node-deployment-695858cd9d-zm72c   1/1     Running   0          10m   10.244.1.5   kind-worker    <none>           <none>
```

`NODE`列の表示からPodがワーカーノードに振り分けられていることが確認できた。念のためページが表示できているか確認しておく。作成されたPodから適当に一つ選んで確認する。

```
$ kubectl exec -it kind-multi-node-deployment-695858cd9d-2wlqf -- sh
# curl localhost
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...以下省略...
```
