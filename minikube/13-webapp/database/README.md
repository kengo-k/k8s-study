```
$ docker build -t weblog-db:v1.0.0 .
```

```
$ docker run -d  weblog-db:v1.0.0
59b024a2f66684e64acf78d49a8ccc3d92a13eaeb213b98ac6a757b17f7f3e96
```

```
$ docker ps
CONTAINER ID   IMAGE              COMMAND                  CREATED        STATUS        PORTS       NAMES
36689f9598ee   weblog-db:v1.0.0   "docker-entrypoint.s…"   23 hours ago   Up 23 hours   27017/tcp   beautiful_kapitsa
```

```
$ docker exec -it beautiful_kapitsa sh
```

```
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
