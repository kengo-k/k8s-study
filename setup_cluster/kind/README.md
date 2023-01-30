# インストール

[公式ページ](https://kind.sigs.k8s.io/docs/user/quick-start/#installing-from-release-binaries)のインストール方法にしたがいインストールを行う。

```
$ curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.17.0/kind-linux-amd64
$ chmod +x ./kind
$ sudo mv ./kind /usr/local/bin/kind
```

# クラスターを作成する

クラスターが作成されていないことを確認する

```
$ kind get clusters
No kind clusters found.
```

クラスターを作成する

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

作成された模様。コンテナが作成されていると思われるので確認してみる。

```
$ docker ps
CONTAINER ID   IMAGE                  COMMAND                  CREATED          STATUS          PORTS                       NAMES
f7f8e1d4084d   kindest/node:v1.25.3   "/usr/local/bin/entr…"   56 seconds ago   Up 51 seconds   127.0.0.1:43781->6443/tcp   kind-control-plane
```

control-plane(マスターノード)が作成されていることが確認できた。

```
$ kubectl config get-contexts
CURRENT   NAME        CLUSTER     AUTHINFO    NAMESPACE
*         kind-kind   kind-kind   kind-kind
          minikube    minikube    minikube    default
```

kubectlのcontextも切り替わっていることが確認できた。
