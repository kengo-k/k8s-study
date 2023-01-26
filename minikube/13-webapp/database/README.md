# データベース用PodのStatefulSetを作成する

## Podに使用するイメージを作成する

PodのDockerイメージを作成するために下記の内容でDockerfileを作成する

```Dockerfile
FROM alpine:3.9

COPY docker-entrypoint.sh /usr/local/bin

RUN \
  adduser -g mongodb -DH -u 1000 mongodb; \
  apk --no-cache add mongodb=4.0.5-r0; \
  chmod +x /usr/local/bin/docker-entrypoint.sh; \
  mkdir -p /data/db; \
  chown -R mongodb:mongodb /data/db;

VOLUME /data/db
EXPOSE 27017

ENTRYPOINT ["docker-entrypoint.sh"]
CMD [ "mongod" ]

```

コンテナ起動時に実行する`docker-entrypoint.sh`を作成する。

```sh
#! /bin/sh
INIT_FLAG_FILE=/data/db/init-completed
INIT_LOG_FILE=/data/db/init-mongod.log

start_mongod_as_daemon() {
echo
echo "> start mongod ..."
echo
mongod \
  --fork \
  --logpath ${INIT_LOG_FILE} \
  --quiet \
  --bind_ip 127.0.0.1 \
  --smallfiles;
}

create_user() {
echo
echo "> create user ..."
echo
if [ ! "$MONGO_INITDB_ROOT_USERNAME" ] || [ ! "$MONGO_INITDB_ROOT_PASSWORD" ]; then
  return
fi
mongo "${MONGO_INITDB_DATABASE}" <<-EOS
  db.createUser({
    user: "${MONGO_INITDB_ROOT_USERNAME}",
    pwd: "${MONGO_INITDB_ROOT_PASSWORD}",
    roles: [{ role: "root", db: "${MONGO_INITDB_DATABASE:-admin}"}]
  })
EOS
}

create_initialize_flag() {
echo
echo "> create initialize flag file ..."
echo
cat <<-EOF > "${INIT_FLAG_FILE}"
[$(date +%Y-%m-%dT%H:%M:%S.%3N)] Initialize scripts if finigshed.
EOF
}

stop_mongod() {
echo
echo "> stop mongod ..."
echo
mongod --shutdown
}

if [ ! -e ${INIT_FLAG_FILE} ]; then
  echo
  echo "--- Initialize MongoDB ---"
  echo
  start_mongod_as_daemon
  create_user
  create_initialize_flag
  stop_mongod
fi

exec "$@"
```

イメージを作成する
```
$ docker build -t webapp-db:v1.0.0 .
```

Dockerコンテナを起動する
```
$ docker run -d  webapp-db:v1.0.0
59b024a2f66684e64acf78d49a8ccc3d92a13eaeb213b98ac6a757b17f7f3e96
```

Dockerコンテナが起動しているか確認する
```
$ docker ps
CONTAINER ID   IMAGE              COMMAND                  CREATED        STATUS        PORTS       NAMES
36689f9598ee   webapp-db:v1.0.0   "docker-entrypoint.s…"   23 hours ago   Up 23 hours   27017/tcp   beautiful_kapitsa
```

Dockerコンテナの中に入りデータベースが起動していることを確認する
```
$ docker exec -it beautiful_kapitsa sh
/ # mongo
MongoDB shell version v4.0.5
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("08e87bab-4c50-45f8-82ad-4db4f8a3194e") }
MongoDB server version: 4.0.5
...中略...
---

> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
```

## データベース用のPodを作成する

下記の内容でマニフェストファイルを作成する。データベース用のPodとPodにマウントするストレージを定義する。

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: storage-volume
  labels:
    app: web
    type: storage
spec:
  storageClassName: slow
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: "/data/storage"

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: storage-claim
  labels:
    app: web
    type: storage
spec:
  storageClassName: slow
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi

---
apiVersion: v1
kind: Pod
metadata:
  name: mongodb
  labels:
    app: web
    type: database
spec:
  volumes:
    - name: storage
      persistentVolumeClaim:
        claimName: storage-claim
  containers:
    - name: mongodb
      image: webapp-db:v1.0.0
      imagePullPolicy: Never
      volumeMounts:
        - mountPath: "/data/db"
          name: storage
      command:
        - "mongod"
        - "--bind_ip_all"
```

マニフェストファイルを適用する

```
$ kubectl apply -f webapp-db-pod.yaml
persistentvolume/storage-volume created
persistentvolumeclaim/storage-claim created
pod/mongodb created
```

リソースが作成されたかどうかを確認する

```
$ kubectl get po,pv,pvc
NAME          READY   STATUS    RESTARTS   AGE
pod/mongodb   1/1     Running   0          63s

NAME                              CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                   STORAGECLASS   REASON   AGE
persistentvolume/storage-volume   1Gi        RWX            Retain           Bound    default/storage-claim   slow                    63s

NAME                                  STATUS   VOLUME           CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/storage-claim   Bound    storage-volume   1Gi        RWX            slow           63s
```

## データベースで使用するシークレットを作成する

下記コマンドで認証につかうkeyfileを作成する

```
$ openssl rand -base64 1024 | tr -d '\r\n' | cut -c 1-1024 > keyfile
```

さきほど作成したkeyfileとユーザ名、パスワードを含むシークレットを作成する

```
$ kubectl create secret generic mongo-secret \
> --from-literal=root_username=admin \
> --from-literal=root_password=Passw0rd \
> --from-file=./keyfile
secret/mongo-secret created
```

作成したシークレットからマニフェストの定義を出力する

```
$ kubectl get secret mongo-secret -o yaml
apiVersion: v1
data:
  keyfile: dk5uMG...長すぎるので省略...
  root_password: UGFzc3cwcmQ=
  root_username: YWRtaW4=
kind: Secret
metadata:
  creationTimestamp: "2023-01-24T09:42:19Z"
  name: mongo-secret
  namespace: default
  resourceVersion: "52806"
  uid: 3593bbec-2247-4802-b8aa-4c3bbd9adcdf
type: Opaque
```

上記のマニフェストファイルを後ほどPodの定義に組み込むために使用する。作成したシークレット自体はもう不要なので削除しておく。

```
$ kubectl delete secret/mongo-secret
secret "mongo-secret" deleted
```

先ほど作成したマニフェストにシークレットの定義とデータベースの認証を追加する。

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: storage-volume
  labels:
    app: web
    type: storage
spec:
  storageClassName: slow
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: "/data/storage"

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: storage-claim
  labels:
    app: web
    type: storage
spec:
  storageClassName: slow
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi

---
apiVersion: v1
kind: Secret
metadata:
  name: mongo-secret
  namespace: default
  labels:
    app: weblog
    env: dev
type: Opaque
data:
  root_username: YWRtaW4=
  root_password: UGFzc3cwcmQ=
  keyfile: dk5uM...長すぎるので省略...

---
apiVersion: v1
kind: Pod
metadata:
  name: mongodb
  labels:
    app: web
    type: database
spec:
  volumes:
    - name: storage
      persistentVolumeClaim:
        claimName: storage-claim
    - name: secret
      secret:
        secretName: mongo-secret
        items:
          - key: keyfile
            path: keyfile
            mode: 0700
  containers:
    - name: mongodb
      image: webapp-db:v1.0.0
      imagePullPolicy: Never
      env:
        - name: "MONGO_INITDB_ROOT_USERNAME"
          valueFrom:
            secretKeyRef:
              name: mongo-secret
              key: root_username
        - name: "MONGO_INITDB_ROOT_PASSWORD"
          valueFrom:
            secretKeyRef:
              name: mongo-secret
              key: root_password
        - name: "MONGO_INITDB_DATABASE"
          value: "admin"
      volumeMounts:
        - mountPath: "/data/db"
          name: storage
        - mountPath: /home/mongodb
          name: secret
      args:
        - "mongod"
        - "--auth"
        - "--bind_ip_all"
```

修正したマニフェストファイルを適用する
```
$ kubectl apply -f webapp-db-pod-with-secret.yaml
persistentvolume/storage-volume created
persistentvolumeclaim/storage-claim created
secret/mongo-secret configured
pod/mongodb created
```

データベースの認証が有効になっていることを確認する
```
$ kubectl exec -it mongodb -- sh
/ # mongo
MongoDB shell version v4.0.5
connecting to: mongodb://127.0.0.1:27017/?
...中略...
> show dbs
2023-01-24T10:00:50.428+0000 E QUERY    [js] Error: listDatabases failed:{
        "ok" : 0,
        "errmsg" : "command listDatabases requires authentication",
        "code" : 13,
        "codeName" : "Unauthorized"
} :
...中略...
> use admin
switched to db admin
> db.auth("admin", "Passw0rd")
1
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
```

最初の`show dbs`で認証エラーとなること。ユーザ名とパスワードを指定して`show dbs`が成功していることから認証が正しく機能していることを確認することができた。

## TODO StatefulSet化


```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: storage-volume-0
  namespace: default
  labels:
    app: weblog
    type: storage
spec:
  storageClassName: slow
  capacity:
    storage: 1Gi
  accessModes:
  - ReadWriteMany
  hostPath:
    path: "/data/pv0000"
    type: Directory

---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: storage-volume-1
  namespace: default
  labels:
    app: weblog
    type: storage
spec:
  storageClassName: slow
  capacity:
    storage: 1Gi
  accessModes:
  - ReadWriteMany
  hostPath:
    path: "/data/pv0001"
    type: Directory

---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: storage-volume-2
  namespace: default
  labels:
    app: weblog
    type: storage
spec:
  storageClassName: slow
  capacity:
    storage: 1Gi
  accessModes:
  - ReadWriteMany
  hostPath:
    path: "/data/pv0002"
    type: Directory

---
apiVersion: v1
kind: Secret
metadata:
  name: mongo-secret
  namespace: default
  labels:
    app: weblog
    type: database
type: Opaque
data:
  root_username: YWRtaW4=
  root_password: UGFzc3cwcmQ=
  keyfile: REZDeX...長いので省略...

---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mongo
  namespace: default
  labels:
    app: weblog
    type: database
spec:
  selector:
    matchLabels:
      app: weblog
      type: database
  serviceName: db-svc
  replicas: 3
  template:
    metadata:
      name: mongodb
      namespace: default
      labels:
        app: weblog
        type: database
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: mongodb
        image: webapp-db:v1.0.0
        imagePullPolicy: Never
        args:
        - "mongod"
        - "--auth"
        - "--clusterAuthMode=keyFile"
        - "--keyFile=/home/mongodb/keyfile"
        - "--replSet=rs0"
        - "--bind_ip_all"
        env:
        - name: "MONGO_INITDB_ROOT_USERNAME"
          valueFrom:
            secretKeyRef:
              name: mongo-secret
              key: root_username
        - name: "MONGO_INITDB_ROOT_PASSWORD"
          valueFrom:
            secretKeyRef:
              name: mongo-secret
              key: root_password
        - name: "MONGO_INITDB_DATABASE"
          value: "admin"
        volumeMounts:
        - mountPath: /data/db
          name: storage
        - mountPath: /home/mongodb
          name: secret
      volumes:
      - name: secret
        secret:
          secretName: mongo-secret
          items:
          - key: keyfile
            path: keyfile
            mode: 0700
  volumeClaimTemplates:
  - metadata:
      name: storage
    spec:
      storageClassName: slow
      accessModes:
      - ReadWriteMany
      resources:
        requests:
          storage: 1Gi
```

```
$ kubectl apply -f webapp-db-statefulset.yaml
persistentvolume/storage-volume-0 created
persistentvolume/storage-volume-1 created
persistentvolume/storage-volume-2 created
secret/mongo-secret created
statefulset.apps/mongo created
```

```
$ kubectl get po,pv,pvc,sts,secret
NAME          READY   STATUS    RESTARTS   AGE
pod/mongo-0   1/1     Running   0          15s
pod/mongo-1   1/1     Running   0          11s
pod/mongo-2   1/1     Running   0          8s

NAME                                CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                     STORAGECLASS   REASON   AGE
persistentvolume/storage-volume-0   1Gi        RWX            Retain           Bound    default/storage-mongo-0   slow                    15s
persistentvolume/storage-volume-1   1Gi        RWX            Retain           Bound    default/storage-mongo-1   slow                    15s
persistentvolume/storage-volume-2   1Gi        RWX            Retain           Bound    default/storage-mongo-2   slow                    15s

NAME                                    STATUS   VOLUME             CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/storage-mongo-0   Bound    storage-volume-0   1Gi        RWX            slow           15s
persistentvolumeclaim/storage-mongo-1   Bound    storage-volume-1   1Gi        RWX            slow           11s
persistentvolumeclaim/storage-mongo-2   Bound    storage-volume-2   1Gi        RWX            slow           8s

NAME                     READY   AGE
statefulset.apps/mongo   3/3     15s

NAME                  TYPE     DATA   AGE
secret/mongo-secret   Opaque   3      15s
```
