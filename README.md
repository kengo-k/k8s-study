# 学習環境の構築

## Dockerのインストール

k8s学習用の環境をLinuxに構築する。ディストリビューションはUbuntu 22.04を使用。下記のコマンドを実行してDockerをインストールする。

```
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
$ echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
$ sudo apt-get update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io
```

動作確認用にhello-worldを実行する。

```
$ docker run hello-world
docker: Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Post "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/containers/create": dial unix /var/run/docker.sock: connect: permission denied.
See 'docker run --help'.
```

デフォルトではrootユーザしか実行できないため権限不足のエラーが出てしまう。下記のコマンドを実行し`docker`グループを作成、作業用のユーザを`docker`グループに所属させる。変更後にdockerを再起動する。

```
# groupadd docker
# service docker restart
# usermod -a -G docker user1
```

再起動後、再度hello-worldを実行する。

```
$ docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
2db29710123e: Pull complete
Digest: sha256:aa0cc8055b82dc2509bed2e19b275c8f463506616377219d9642221ab53cf9fe
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
...以下略...
```

正常に実行されればOK。

# kubectlのインストール

下記のコマンドを実行しkubectlをインストールする。

```
curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
```

インストール後、ファイルに実行権限を付与しPATHの通ったディレクトリに配置すること

# minikubeのインストール

下記のコマンドを実行しminikubeをインストールする。

```
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
```

インストール後、ファイルに実行権限を付与しPATHの通ったディレクトリに配置すること

## minikubeの基本的な使用方法を確認

```
$ minikube status
🤷  Profile "minikube" not found. Run "minikube profile list" to view all profiles.
👉  To start a cluster, run: "minikube start"
```

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

```
$ minikube status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

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
