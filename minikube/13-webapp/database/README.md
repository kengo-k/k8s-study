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

```
$ kubectl apply -f weblog-db-storage.yaml
persistentvolume/storage-volume created
persistentvolumeclaim/storage-claim created
```
```
$ kubectl get pvc,pv
NAME                                  STATUS   VOLUME           CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/storage-claim   Bound    storage-volume   1Gi        RWX            slow           22s

NAME                              CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                   STORAGECLASS   REASON   AGE
persistentvolume/storage-volume   1Gi        RWX            Retain           Bound    default/storage-claim   slow                    22ss
```

```
$ openssl rand -base64 1024 | tr -d '\r\n' | cut -c 1-1024 > keyfile
```

```
$ kubectl create secret generic mongo-secret \
> --from-literal=root_username=admin \
> --from-literal=root_password=Passw0rd \
> --from-file=./keyfile
secret/mongo-secret created
```

```
$ kubectl get secret mongo-secret -o yaml
apiVersion: v1
data:
  keyfile: dk5uMG9iOW5hM09oWGo0dTRhU0psdDRQM1k4L3pRTW1xV0VrYm5tQnRpRVBrbkd6Mzd4UWQ5MEE1Zmh5ZzBNUDFra1FuL1NmeU1jaE5xanNwZzI0RXdHVzBQQmdBRGNFcVVaaU5xK2FIVjdkdUVQZFNQOVZ3b1NPSEJRb0MrL085NG1VZU5QMExjeDViNEdTL2VKUTFWaEhQYlFHSlE2RTl5cVJIRFBZVk5NRFpHV215ZnFyNlZaV0hqamNGRUlWeHZVK0dJakhjTVVZbGRaN0RCVFVFZWhra1FydDJDbFRVSTZTeG1qd253RmhyMklheEdiUHVsNmRJSmMyL2JFb1A2MGFvSDBlbDdhOWlUejN3R3JvZ3VmWE9uVUs1d3l6VXp5YUx3bC9adldPMkttaUFWTnhTb2FIOXVwY3hYZTF5cXhZZUk5VE9iSHZWakxRY3lzTUFvb2t2Nm1vbHZTN0pFbU1qdWdsdVBHczQzWDI3aVdtMXY1N0ZXUGV0R08xUFlMdXF0YWQ4aDdJZDNBVldXVUthYlJoNjByVUdWTFcrZlJ3RUlIN2hMdGQ5L3RPdE5JMkJSNk40WDNYNnB1K0d1ZVRnaUtSZ0F1emhid2FkY1lnN21oei9GRzkxaGdHTnp1SzlpZWZlWG0zK2hPTHJIMTVjOFlrRHNXVTBmOUt2TVV5andiaDNKNFZKU3JYbXpra0VLbUFPOVVFUUYwMm10MXVEc2ZlSDRGRUdnaDBEempKY3RZclVHVFZKZzF5SXB4dTJMTVV5T2RBUGNla1hITTREdVZHTW5Gek9qVC9saVRPUFkvOTRoMWVlVzk3cWd1TUJBR2plKzdaZjZxbHhOcHZMbFpYM1pwSUtGSE9mVVJVMWZwQWVVc29DOHA3QkN1ZS93M3ZQNWRzTldNT3hyQjhKYk05aEtIWTdsVWU5UGZkL05qT1hLNy9Sa3lsaFhkTzFDU014Z2dVSXY0ZmdvU0RSNDhVbGZuVzRaUzBISHVtWG8wODE2aFZNRFVydjM3OUtkNEdRdGJHWjRPaktGK3o5WEl5aHZmeThXSzJieUNhRExtWXEybUZFS1VVNEI1TnhMbVhJbFZ3UlpreTg5OHpGaTdXdmIwcWh0TUhCd3FLcHBDVkJ0ZWM0V3h0TzQ3Z29wMDFMem1VUS9hQ2c4SThKR293TzFubWZMMkFIVFBlNmRYbzY0NFJCMkE3MmNYMXdHQ3NjSGY2ak9LV0w2bjVpcDhZMTNKVEo2WERwa0RxZitLU05XTDJ3NWFZWUR4MnJFM00wcnZkTmVXVmdzcXcyN2h5aFZ6ZlVOUWFDSG9lU3dsa3pKbTlNTEJ2Z2NlOENNbjJvaWZpZ0N0VAo=
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

```
$ kubectl delete secret/mongo-secret
secret "mongo-secret" deleted
```

```
$ kubectl apply -f weblog-db-storage.yaml
persistentvolume/storage-volume created
persistentvolumeclaim/storage-claim created
secret/mongo-secret unchanged
pod/mongodb created
```

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
