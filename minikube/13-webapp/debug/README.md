# デバッグ用のPodを作成する

## DockerfileからDockerイメージをビルドする

下記の内容でDockerfileを作成する
```Dockerfile
FROM centos:7

COPY . /tmp/debug

RUN \
  mv /tmp/debug/mongodb-org-4.0.repo /etc/yum.repos.d; \
  yum install -y mongodb-org-shell-4.0.5 mongodb-org-tools-4.0.5; \
  yum install -y iproute net-tools; \
  curl -o /usr/local/bin/jq -L https://github.com/stedolan/jq/releases/download/jq-1.6/jq-linux64; \
  chmod +x /usr/local/bin/jq; \
  yum clean all;
```
`docker build`コマンドでDockerイメージを作成する
```
$ docker build -t debug .
```

イメージが作成されたかどうかを確認する
```
$ docker image ls
REPOSITORY                    TAG       IMAGE ID       CREATED          SIZE
debug                         latest    1567d0767788   58 seconds ago   236MB
```

コンテナを作成してmongoコマンドがインストールされているか確認する
```
$ docker run -it debug sh
sh-4.2# mongo
MongoDB shell version v4.0.5
...以下略...
```

## デバッグ用Podの動作確認を行う

Dockerコンテナの動作確認ができたのでkubectlからPodに接続して同様の確認をする。下記の内容でデバッグ用Podのマニフェストファイルを作成する

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: debug
  labels:
    app: web
    env: dev
spec:
  containers:
    - name: debug
      image: debug
      imagePullPolicy: Never
      command:
        - "sh"
        - "-c"
      args:
        - |
          while true
          do
            sleep 5
          done
```

動作確認する
```
$ kubectl exec -it debug -- sh
sh-4.2# mongo
MongoDB shell version v4.0.5
...中略...
sh-4.2# jq
jq - commandline JSON processor [version 1.6]
```
