```
$ docker build -t debug .
```

```
$ docker image ls
REPOSITORY                    TAG       IMAGE ID       CREATED          SIZE
debug                         latest    1567d0767788   58 seconds ago   236MB
```

```
$ docker run -it debug sh
sh-4.2# mongo
MongoDB shell version v4.0.5
...以下略...
```

```
$ kubectl exec -it debug -- sh
sh-4.2# mongo
MongoDB shell version v4.0.5
...中略...
sh-4.2# jq
jq - commandline JSON processor [version 1.6]
```
