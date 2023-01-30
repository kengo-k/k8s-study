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

上記のcri-dockerdのインストールURLに従いインストールを完了し再度`minikube start`を実行する。

```
$ sudo ./minikube start --vm-driver=none
😄  Ubuntu 22.04 上の minikube v1.28.0
✨  既存のプロファイルを元に、none ドライバーを使用します
👍  minikube クラスター中のコントロールプレーンの minikube ノードを起動しています
🔄  「minikube」のために既存の none bare metal machine を再起動しています...
ℹ️  OS リリースは Ubuntu 22.04.1 LTS です

❌  RUNTIME_ENABLE が原因で終了します: Temporary Error: sudo crictl version: exit status 1
stdout:

stderr:
sudo: crictl: コマンドが見つかりません
```

エラー内容にしたがいcrictlをインストールする。インストールには下記のページを参考にした。

- https://computingforgeeks.com/install-cri-o-container-runtime-on-ubuntu-linux/


```sh
OS=xUbuntu_20.04
CRIO_VERSION=1.25
echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/ /"|sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
echo "deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$CRIO_VERSION/$OS/ /"|sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$CRIO_VERSION.list

curl -L https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$CRIO_VERSION/$OS/Release.key | sudo apt-key add -
curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/Release.key | sudo apt-key add -

sudo apt-get update
sudo apt-get install cri-tools
```

再度実行する。

```
$ sudo ./minikube start --vm-driver=none
😄  Ubuntu 22.04 上の minikube v1.28.0
✨  既存のプロファイルを元に、none ドライバーを使用します
👍  minikube クラスター中のコントロールプレーンの minikube ノードを起動しています
🔥  none の「minikube」を削除しています...
🤦  StartHost に失敗しましたが、再度試してみます: boot lock: unable to open /tmp/juju-mkc8ab01ad3ea83211c505c81a7ee49a8e3ecb89: permission denied
😿  none bare metal machine の開始に失敗しました。「minikube delete」実行で解決するかも知れません: boot lock: unable to open /tmp/juju-mkc8ab01ad3ea83211c505c81a7ee49a8e3ecb89: permission denied

❌  HOST_JUJU_LOCK_PERMISSION が原因で終了します: Failed to start host: boot lock: unable to open /tmp/juju-mkc8ab01ad3ea83211c505c81a7ee49a8e3ecb89: permission denied
💡  提案: 'sudo sysctl fs.protected_regular=0' を実行するか、'--driver=docker' のような root を必要としないドライバーを試してください
🍿  関連イシュー: https://github.com/kubernetes/minikube/issues/6391
```

提案に従ってみる。

```
$ sudo sysctl fs.protected_regular=0
fs.protected_regular = 0
```

```
$ sudo ./minikube start --vm-driver=none
😄  Ubuntu 22.04 上の minikube v1.28.0
✨  既存のプロファイルを元に、none ドライバーを使用します
👍  minikube クラスター中のコントロールプレーンの minikube ノードを起動しています
🤹  localhost (CPU=8、メモリー=15907MB、ディスク=234024MB) 上で実行しています...
ℹ️  OS リリースは Ubuntu 22.04.1 LTS です
🐳  Docker 20.10.22 で Kubernetes v1.25.3 を準備しています...
    ▪ kubelet.resolv-conf=/run/systemd/resolve/resolv.conf
    > kubectl.sha256:  64 B / 64 B [-------------------------] 100.00% ? p/s 0s
    > kubeadm.sha256:  64 B / 64 B [-------------------------] 100.00% ? p/s 0s
    > kubelet.sha256:  64 B / 64 B [-------------------------] 100.00% ? p/s 0s
    > kubeadm:  41.77 MiB / 41.77 MiB [-----------] 100.00% 71.02 MiB p/s 800ms
    > kubectl:  42.93 MiB / 42.93 MiB [------------] 100.00% 43.10 MiB p/s 1.2s
    > kubelet:  108.95 MiB / 108.95 MiB [----------] 100.00% 68.64 MiB p/s 1.8s
    ▪ 証明書と鍵を作成しています...
    ▪ コントロールプレーンを起動しています...
    ▪ RBAC のルールを設定中です...
🤹  ローカルホスト環境を設定中です...

❗  'none' ドライバーは既存 VM の統合が必要なエキスパートに向けて設計されています。
💡  多くのユーザーはより新しい 'docker' ドライバーを代わりに使用すべきです (root 権限が必要ありません！)
📘  追加の詳細情報はこちらを参照してください: https://minikube.sigs.k8s.io/docs/reference/drivers/none/

❗  kubectl と minikube の構成は /root に保存されます
❗  kubectl か minikube コマンドを独自のユーザーとして使用するためには、そのコマンドの再配置が必要な場合があります。たとえば、独自の設定を上書きするためには、以下を実行します

    ▪ sudo mv /root/.kube /root/.minikube $HOME
    ▪ sudo chown -R $USER $HOME/.kube $HOME/.minikube

💡  これは環境変数 CHANGE_MINIKUBE_NONE_USER=true を設定して自動的に行うこともできます
🔎  Kubernetes コンポーネントを検証しています...
    ▪ gcr.io/k8s-minikube/storage-provisioner:v5 イメージを使用しています
🌟  有効なアドオン: default-storageclass, storage-provisioner
💡  kubectl が見つかりません。kubectl が必要な場合、'minikube kubectl -- get pods -A' を試してください
🏄  終了しました！kubectl がデフォルトで「minikube」クラスターと「default」ネームスペースを使用するよう設定されました
```

kubectlが使えるかどうか確認してみる。

```
$ kubectl get po
E0129 12:25:58.700268   16262 memcache.go:238] couldn't get current server API group list: Get "http://localhost:8080/api?timeout=32s": dial tcp 127.0.0.1:8080: connect: connection refused
E0129 12:25:58.700560   16262 memcache.go:238] couldn't get current server API group list: Get "http://localhost:8080/api?timeout=32s": dial tcp 127.0.0.1:8080: connect: connection refused
E0129 12:25:58.702210   16262 memcache.go:238] couldn't get current server API group list: Get "http://localhost:8080/api?timeout=32s": dial tcp 127.0.0.1:8080: connect: connection refused
E0129 12:25:58.703697   16262 memcache.go:238] couldn't get current server API group list: Get "http://localhost:8080/api?timeout=32s": dial tcp 127.0.0.1:8080: connect: connection refused
E0129 12:25:58.705396   16262 memcache.go:238] couldn't get current server API group list: Get "http://localhost:8080/api?timeout=32s": dial tcp 127.0.0.1:8080: connect: connection refused
The connection to the server localhost:8080 was refused - did you specify the right host or port?
```

一応できたようだがroot権限が必要になるため色々面倒になる模様。

```
sudo ./kubectl get po
No resources found in default namespace.
```

いつもの見慣れた表示ができたので動作している模様。毎回sudoが必要になるのであれば面倒なのでこのまま使うかどうか考える必要がありそう。面倒くさいので上記の出力に記載されている再配置を試してみる。

```
# 念のため既存のディレクトリをパックアップしておく
$ mv .kube .kube.org
$ mv .minikube .minikube.org
$ sudo rm -rf .minikube .kube
$ sudo mv /root/.kube /root/.minikube $HOME
$ sudo chown -R $USER $HOME/.kube $HOME/.minikube
```

一般ユーザでkubectlできるか試す。
```
$ kubectl get po
Error in configuration:
* unable to read client-cert /root/.minikube/profiles/minikube/client.crt for minikube due to open /root/.minikube/profiles/minikube/client.crt: permission denied
* unable to read client-key /root/.minikube/profiles/minikube/client.key for minikube due to open /root/.minikube/profiles/minikube/client.key: permission denied
* unable to read certificate-authority /root/.minikube/ca.crt for minikube due to open /root/.minikube/ca.crt: permission denied
```

いまだに/root配下のファイルを見ている模様。ためしに設定関連のファイルがないか探したところ`.kube/config`ファイルに/root以下のファイルを参照する箇所が残っていたので上記でmvして再配置した自ユーザのデイレクトリに書き換えてから再度実行する。

```
$ kubectl get po
No resources found in default namespace.
```

一般ユーザで動作できているように見える。実際に簡単なPodを動作させてみる。`01-hello`で作成したhello-worldのPodを使ってみる。

```
$ kubectl apply -f pod.yaml
pod/test created
```

Podが作成された模様。ステータスもチェックしてみる。

```
$ kubectl get po
NAME                 READY   STATUS    RESTARTS   AGE
test                 0/1     Pending   0          7s
```

Pendingのまま起動しない。原因はまだ把握できていないがとりあえず

```
$ kubectl get node
NAME                STATUS     ROLES           AGE   VERSION
ubuntu-on-macmini   NotReady   control-plane   23m   v1.25.3
```

どうやらcontrol-plane(つまりマスターノード)がNotReadyになっている模様。まだ正しく動作できていない
