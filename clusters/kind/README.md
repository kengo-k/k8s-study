# インストール

公式ページインストール方法にしたがい下記コマンドでインストールを行った。

```
$ curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.17.0/kind-linux-amd64
$ chmod +x ./kind
$ sudo mv ./kind /usr/local/bin/kind
```

# クラスターを作成する

`kind get clusters`で事前にクラスターが作成されていないことを確認しておく。

```
$ kind get clusters
No kind clusters found.
```

`kind create cluster`でクラスターを作成する。

```
$ kind create cluster
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.25.3) 🖼
 ✓ Preparing nodes 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Have a nice day! 👋
```

正常にクラスタが作成されたように見える。kindではノードがDockerコンテナとして作成されるため実際にコンテナが作成されているかどうか確認する。

```
$ docker ps --format "table {{.ID}}\t{{.Image}}\t{{.Names}}"
CONTAINER ID   IMAGE                                 NAMES
32496b214d9a   kindest/node:v1.25.3                  kind-control-plane
46b4be0c9489   gcr.io/k8s-minikube/kicbase:v0.0.36   minikube
```

`kind-control-plane`(マスターノード)が作成されていることが確認できた。クラスター作成の際にクラスタ構成を設定ファイルで指定しなかったためデフォルトではシングルノードでクラスターが作成されている。続いて`kubectl get nodes`でノードの一覧を表示してみる。

```
$ kubectl get no
NAME                 STATUS   ROLES           AGE     VERSION
kind-control-plane   Ready    control-plane   5m34s   v1.25.3
```

`docker ps`の出力と同様にシングルノードで作成されていることが確認できた

# 実際にPodを作成してみる

サンプル用Podを起動する。

```
$ cd 01-hello
$ kubectl apply -f hello.yaml
pod/hello-sample-pod created

$ kubectl get po
NAME               READY   STATUS      RESTARTS      AGE
hello-sample-pod   0/1     Completed   3 (31s ago)   52s

$ kubectl logs hello-sample-pod

Hello from Docker!
...以下省略...
```
