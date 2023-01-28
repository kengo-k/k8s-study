# minikubeのインストール

下記のコマンドを実行しminikubeをインストールする。

```
$ curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
```

インストール後、ファイルに実行権限を付与しPATHの通ったディレクトリに配置すること

## minikubeの基本的な使用方法を確認

ステータスを確認

```
$ minikube status
🤷  Profile "minikube" not found. Run "minikube profile list" to view all profiles.
👉  To start a cluster, run: "minikube start"
```

まだ一度も起動してないためnot foundとなる。startしろと書いてあるのでその通りにする。

```
$ minikube start
😄  minikube v1.28.0 on Ubuntu 22.04
✨  Automatically selected the docker driver
📌  Using Docker driver with root privileges
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
💾  Downloading Kubernetes v1.25.3 preload ...
...以下略...
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

起動が成功したっぽいので再度ステータスを確認する。

```
$ minikube status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

正常に起動できた模様。最後にアドオンの一覧を確認しておく。

```
$ minikube addons list
|-----------------------------|----------|--------------|--------------------------------|
|         ADDON NAME          | PROFILE  |    STATUS    |           MAINTAINER           |
|-----------------------------|----------|--------------|--------------------------------|
| ambassador                  | minikube | disabled     | 3rd party (Ambassador)         |
| auto-pause                  | minikube | disabled     | Google                         |
| cloud-spanner               | minikube | disabled     | Google                         |
| csi-hostpath-driver         | minikube | disabled     | Kubernetes                     |
| dashboard                   | minikube | disabled     | Kubernetes                     |
| default-storageclass        | minikube | enabled ✅   | Kubernetes                     |
...以下略...
```

ずらずらと一覧が表示されていればOK

## minikube環境をすべて削除してクリーンな状態に戻す

```
minikube delete --all
```

##

```
$ minikube docker-env
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.49.2:2376"
export DOCKER_CERT_PATH="/home/kengo/.minikube/certs"
export MINIKUBE_ACTIVE_DOCKERD="minikube"
```

```
$ sudo ./minikube start --vm-driver=none
GUEST_MISSING_CONNTRACK が原因で終了します: 申し訳ありませんが、Kubernetes 1.25.3 は root アカウントのパス中にインストールされた conntrack が必要です
```

```
$ sudo apt install conntrack
```

```
🐳  NOT_FOUND_CRI_DOCKERD が原因で終了します:

💡  提案:

    Kubernetes v1.24+ の none ドライバーと docker container-runtime は cri-dockerd を要求します。

    これらの手順を参照して cri-dockerd をインストールしてください:

    https://github.com/Mirantis/cri-dockerd#build-and-install
```
