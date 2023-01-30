# kindでマルチノードなクラスタを作成する

kindを使うことで簡単にマルチノードクラスタを作成できるので実際に試してみる。まずは下記の内容でクラスタの定義ファイルを作成する。

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
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
b138c48f26c1   kindest/node:v1.25.3                  kind-worker
197de9f2d8d3   kindest/node:v1.25.3                  kind-control-plane
12569285c685   kindest/node:v1.25.3                  kind-worker2
46b4be0c9489   gcr.io/k8s-minikube/kicbase:v0.0.36   minikube
```

マスターノード1つ、ワーカーノード2つ作成されていることが確認できた。念のため`kubectl get nodes`も見ておく。

```
$ kubectl get no
NAME                 STATUS   ROLES           AGE     VERSION
kind-control-plane   Ready    control-plane   2m46s   v1.25.3
kind-worker          Ready    <none>          2m24s   v1.25.3
kind-worker2         Ready    <none>          2m24s   v1.25.3
```

実際に複数のPodを作成しワーカーノードに振り分けられるかどうかを確認する。ReplicaSetのサンプルを使用して動作確認してみる。

```
$ cd 05-replicaset
$ kubectl apply -f replicaset.yaml
replicaset.apps/replicaset-sample created

NAME                      READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
replicaset-sample-524v7   1/1     Running   0          71s   10.244.1.5   kind-worker    <none>           <none>
replicaset-sample-5vm8c   1/1     Running   0          71s   10.244.2.5   kind-worker2   <none>           <none>
replicaset-sample-csd4g   1/1     Running   0          71s   10.244.1.4   kind-worker    <none>           <none>
replicaset-sample-hzg57   1/1     Running   0          71s   10.244.2.6   kind-worker2   <none>           <none>
replicaset-sample-lwrdk   1/1     Running   0          71s   10.244.1.6   kind-worker    <none>           <none>
replicaset-sample-qg6z7   1/1     Running   0          71s   10.244.2.3   kind-worker2   <none>           <none>
replicaset-sample-tx79m   1/1     Running   0          71s   10.244.2.4   kind-worker2   <none>           <none>
replicaset-sample-vmvlk   1/1     Running   0          71s   10.244.2.2   kind-worker2   <none>           <none>
replicaset-sample-xtn9x   1/1     Running   0          71s   10.244.1.3   kind-worker    <none>           <none>
replicaset-sample-z8ktc   1/1     Running   0          71s   10.244.1.2   kind-worker    <none>           <none>
```

`NODE`列の表示からPodがワーカーノードに振り分けられていることが確認できた。
