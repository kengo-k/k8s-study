# kindで構築したクラスタ上にて、ホストディレクトリをPodにマウントする

## ポイント

kindではノードがDockerコンテナとして起動し、ノード内でさらにPod(containerdという実装が使われる)が起動するという構成になっている。つまり

- ホスト -> ノード(Docker) -> Pod(containerd)

となっているため

- まずノード上でホストのディレクトリがマウントされていることを確認
- Podでノードにマウントされたディレクトリがさらにマウントされていることを確認

という手順を踏む。

## クラスタを作成する

下記の内容でクラスタを作成する

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
  extraMounts:
  - containerPath: /data
    hostPath: /data
```

クラスタを作成したら`docker ps`で作成されたコンテナ一覧を確認する。

```
$ docker ps --format "table {{.ID}}\t{{.Image}}\t{{.Names}}"
CONTAINER ID   IMAGE                   NAMES
3d99394e7943   kindest/node:v1.25.3    kind-control-plane
6a9690fe9b8e   kindest/node:v1.25.3    kind-worker
```

`kind-worker`の中にはいってマウントされているか確認する。

```
$ docker exec -it kind-worker sh
# cd /data
# ls
pv0000  pv0001  pv0002  storage
```

マウントされていることが確認できた。

```
$ kubectl exec -it pod/nginx -- sh
# cd /test
# ls
pv0000  pv0001  pv0002  storage
```
