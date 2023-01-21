# Ingressとは

Serviceの前に置く。L7ロードバランサー。

## そもそもL7ロードバランサーとは

ロードバランサーは以下の二種類に分けられる模様。

- L4: レイヤー4（IPアドレスとポート番号による）負荷分散
- L5: レイヤー7で(URLやHTTPヘッダによる)負荷分散

L7ロードバランサーではホストベース、パスベースで振り分け先が決定されるため、基本的には同じ振り分け先が使われ続ける。振り分け先がサーバA,サーバBのように2つ以上設定されている場合には、ロードバランサーに設定した振り分けルール（ラウンドロビンやURLルーティング、等）に基づいて通信が届く。

```
minikube addons enable ingress
```

```
$ kubectl get ing,svc,deploy,po
NAME                                               CLASS   HOSTS   ADDRESS        PORTS   AGE
ingress.networking.k8s.io/ingress-sample-ingress   nginx   *       192.168.49.2   80      73s

NAME                             TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
service/ingress-sample-service   ClusterIP   10.100.79.9   <none>        80/TCP    73s
service/kubernetes               ClusterIP   10.96.0.1     <none>        443/TCP   6m10s

NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/ingress-sample-deployment   2/2     2            2           73s

NAME                                             READY   STATUS    RESTARTS   AGE
pod/ingress-sample-deployment-6cc9c4fd67-4czkj   1/1     Running   0          73s
pod/ingress-sample-deployment-6cc9c4fd67-cd7fn   1/1     Running   0          73s
```
