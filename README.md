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
