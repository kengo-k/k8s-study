# kindでマルチノードなクラスタを作成する

kindを使うことで簡単にマルチノードクラスタを作成できるので実際に試してみる。まずは下記の内容でクラスタの定義ファイルを作成する。

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  extraPortMappings:
    - containerPort: 30080
      hostPort: 30070
- role: worker
- role: worker
```


クラスタを実際に作成する前にすでにクラスタが存在していないか確認する。存在する場合は削除する。

```sh
$ kind get clusters
kind # 存在した模様

 # クラスター`kind`を削除
$ kind delete clusters kind
Deleted clusters: ["kind"]

# クラスタを作成
$ kind create cluster --config=./multi_nodes_cluster.yaml
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.25.3) 🖼
 ✓ Preparing nodes 📦 📦 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
 ✓ Joining worker nodes 🚜
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Not sure what to do next? 😅  Check out https://kind.sigs.k8s.io/docs/user/quick-start/
```

マルチノードなクラスターが作成されているはず。まずはDockerのコンテナを確認する。

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
$ kubectl get no -o wide
NAME                 STATUS   ROLES           AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
kind-control-plane   Ready    control-plane   2m25s   v1.25.3   172.18.0.3    <none>        Ubuntu 22.04.1 LTS   5.15.0-58-generic   containerd://1.6.9
kind-worker          Ready    <none>          2m3s    v1.25.3   172.18.0.4    <none>        Ubuntu 22.04.1 LTS   5.15.0-58-generic   containerd://1.6.9
kind-worker2         Ready    <none>          2m3s    v1.25.3   172.18.0.2    <none>        Ubuntu 22.04.1 LTS   5.15.0-58-generic   containerd://1.6.9
```

実際に複数のPodを作成しワーカーノードに振り分けられるかどうかを確認する。下記のマニフェストファイルを作成する。

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: web-nginx
  name: web-nginx
spec:
  replicas: 5
  selector:
    matchLabels:
      app: web-nginx
  template:
    metadata:
      labels:
        app: web-nginx
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
  name: web-nginx
spec:
  selector:
    app: web-nginx
  type: NodePort
  ports:
    - port: 80
      nodePort: 30080
```

`replicas`の値が5となっているため5つのPodがワーカノードに振り分けられた起動するはずである。実際にapplyして確認してみる。

```
$ kubectl apply -f app.yaml
deployment.apps/web-nginx created
service/web-nginx created

$ kubectl get po -o wide
NAME                         READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
web-nginx-7dc47b4fd9-6td28   1/1     Running   0          31s   10.244.2.4   kind-worker2   <none>           <none>
web-nginx-7dc47b4fd9-8rlt5   1/1     Running   0          31s   10.244.2.3   kind-worker2   <none>           <none>
web-nginx-7dc47b4fd9-g8s2b   1/1     Running   0          31s   10.244.1.3   kind-worker    <none>           <none>
web-nginx-7dc47b4fd9-jd77z   1/1     Running   0          31s   10.244.2.2   kind-worker2   <none>           <none>
web-nginx-7dc47b4fd9-wvsg6   1/1     Running   0          31s   10.244.1.2   kind-worker    <none>           <none>
```

`NODE`列の表示からPodがワーカーノードに振り分けられていることが確認できた。

最後に実際にnginxのページが表示されるか確認する。クラスターの定義ファイルにて

```
  extraPortMappings:
    - containerPort: 30080
      hostPort: 30070
```

の記述からホストの30070にアクセスすればページが表示されるはず。

```
$ curl localhost:30070
...省略...
<h1>Welcome to nginx!</h1>
...省略...
```
