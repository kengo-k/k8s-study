# kindでマルチノードなクラスタを作成する

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
```

```
$ kind create cluster \
  --name=kind-multi-nodes \
  --config=./multi_nodes_cluster.yaml
```

```
$ docker ps
CONTAINER ID   IMAGE                                 COMMAND                  CREATED             STATUS             PORTS                                                                                                                                  NAMES
db1ddc4a1973   kindest/node:v1.25.3                  "/usr/local/bin/entr…"   50 seconds ago      Up 44 seconds                                                                                                                                             kind-multi-nodes-worker
34ebc8d9a4fc   kindest/node:v1.25.3                  "/usr/local/bin/entr…"   50 seconds ago      Up 44 seconds                                                                                                                                             kind-multi-nodes-worker2
446e3b3b8a61   kindest/node:v1.25.3                  "/usr/local/bin/entr…"   50 seconds ago      Up 45 seconds      127.0.0.1:44671->6443/tcp                                                                                                              kind-multi-nodes-control-plane
46b4be0c9489   gcr.io/k8s-minikube/kicbase:v0.0.36   "/usr/local/bin/entr…"   About an hour ago   Up About an hour   127.0.0.1:32772->22/tcp, 127.0.0.1:32771->2376/tcp, 127.0.0.1:32770->5000/tcp, 127.0.0.1:32769->8443/tcp, 127.0.0.1:32768->32443/tcp   minikube
```
