# 学習環境の構築

## Dockerのインストール

k8s学習用の環境をUbuntu 22.04上に構築する。学習環境としては`kind`もしくは`minikube`を使用するがともにDockerが必要になるため、まずは下記のコマンドを実行してDockerをインストールする。

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

asdfを使用して`kubectl`をインストールする。※asdfのインストール、使用法について割愛

```
$ asdf plugin add kubectl
$ asdf list all kubectl
...中略...
1.26.0
...中略...
```

`kubectl`のどのバージョンを使用するかを確認しておくこと。公式ドキュメントに下記の記載がある。

```
kubectlのバージョンは、クラスターのマイナーバージョンとの差分が1つ以内でなければなりません。たとえば、クライアントがv1.2であれば、v1.1、v1.2、v1.3のマスターで動作するはずです。
```

ひとまず1.26.0をインストールすることにする。

```
$ asdf install kubectl 1.26.0
Downloading kubectl from https://dl.k8s.io/release/v1.26.0/bin/linux/amd64/kubectl
$ asdf global kubectl 1.26.0
```

`kubectl`とクラスタのバージョンの確認は下記のコマンドを実行する。

```
$ kubectl version --output=yaml
clientVersion:
  buildDate: "2022-12-08T19:58:30Z"
  compiler: gc
  gitCommit: b46a3f887ca979b1a5d14fd39cb1af43e7e5d12d
  gitTreeState: clean
  gitVersion: v1.26.0
  goVersion: go1.19.4
  major: "1"
  minor: "26"
  platform: linux/amd64
kustomizeVersion: v4.5.7
serverVersion:
  buildDate: "2022-10-25T19:35:11Z"
  compiler: gc
  gitCommit: 434bfd82814af038ad94d62ebe59b133fcb50506
  gitTreeState: clean
  gitVersion: v1.25.3
  goVersion: go1.19.2
  major: "1"
  minor: "25"
  platform: linux/amd64
```

続いて`kubectx`をインストールしておく。`kubectx`を使用することでコンテキストを簡単に切り替えることができる。kindとminikubeを併用する(したい場面があるかわからないが)場合などに便利。バージョンは気にせずlatestで入れておく。

```
$ asdf plugin add kubectx
$ asdf install kubectx latest
$ asdf global kubectx latest
```

`kubectx`コマンドを実行するとコンテキストの一覧が表示される

```
$ kubectx
kind-kind
minikube
```

切り替えたいクラスターを引数として渡せばコンテキストを切り替えることができる。

```
$ kubectx kind-kind
Switched to context "kind-kind".
```

# kubectlの基本的な使い方

使用頻度の高いコマンドは(今のところ)以下6つ。

- kubectl apply
- kubectl get
- kubectl exec
- kubectl describe
- kubectl logs
- kubectl delete

## apply

マニフェストファイルを適用してリソースを作成するために使用する。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hello-world
spec:
  containers:
    - name: hello-world
      image: hello-world
```

上記の内容でファイルを作成し下記のコマンドを実行する。

```
$ kubectl apply -f hello-world.yaml
pod/hello-world created
```

## get

リソースの一覧を表示するために使用する。例えば上記のhello-worldを実行した状態で下記のコマンドを実行する。

### kubectl get

```
$ kubectl get all
NAME              READY   STATUS    RESTARTS   AGE
pod/hello-world   0/1     Pending   0          5s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   19s
```

`kubectl get all`で全て(※といっても厳密にはすべては表示されず明示しないと表示されないものがある)のリソースを表示できる。リソースの種類を指定することで表示するリソースを絞ることもできる。たとえばPodのみを表示する場合は下記のようにする。

```
$ kubectl get po
NAME          READY   STATUS             RESTARTS      AGE
hello-world   0/1     CrashLoopBackOff   4 (69s ago)   2m52s
```

`po`は`pod`の省略形であり、どちらの名前を使用してもよい。指定できるリソース種類の全一覧は`kubectl api-resources`で確認できる。よく使いそうなものだけここに列挙しておく。

| リソース名 | 省略名 |
|-|-|
|pod|po|
|service|svc|
|replicaset|rs|
|deployment|deploy|
|statefulset|sts|
|configmap|cm|
|secret|-|
|service|svc|
|namespace|ns|
|node|no|

`kubectl get`に続けてリソース名を指定する際にカンマ区切りでリソース名を並べることができる。

```
$ kubectl get po,no
NAME              READY   STATUS             RESTARTS        AGE
pod/hello-world   0/1     CrashLoopBackOff   7 (3m11s ago)   14m

NAME                      STATUS   ROLES           AGE   VERSION
node/kind-control-plane   Ready    control-plane   14m   v1.25.3
```

上記の例ではPodとノードの一覧を表示している。

### kubectl get -o wide

`-o wide`オプションを指定することでより詳細な情報を取得することができる。

```
$ kubectl get po -o wide
NAME          READY   STATUS             RESTARTS        AGE   IP           NODE                 NOMINATED NODE   READINESS GATES
hello-world   0/1     CrashLoopBackOff   8 (4m24s ago)   20m   10.244.0.2   kind-control-plane   <none>           <none>
```

### kubectl get -o yaml

`kubectl get`に`-o yaml`オプションを指定することでさらに詳細な情報をyaml形式で取得することができる。出力したyamlをマニフェストファイルのテンプレートとして利用する際などに使う。例としてSecretをコマンドで作成してからyamlとして出力する。

```
$ kubectl create secret generic sec1 \
  --from-literal=key1="value1"
secret/sec1 created
```

下記コマンドでyamlとして出力する。

```
$ kubectl get secret -o yaml
apiVersion: v1
items:
- apiVersion: v1
  data:
    key1: dmFsdWUx
  kind: Secret
  metadata:
    creationTimestamp: "2023-02-05T06:03:57Z"
    name: sec1
    namespace: default
    resourceVersion: "2407"
    uid: c6180b68-d072-412f-afdf-d742c890afaf
  type: Opaque
kind: List
metadata:
  resourceVersion: ""
```

出力したyamlを不要な箇所を削る等の編集を加えて他のマニフェストに組み込むなどの用途に役立つ。
