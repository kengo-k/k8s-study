# 概要

最初のステップとしてhello-worldを実行する。下記のコマンドを実行する。

## コマンドから直接podを起動する

```
$ kubectl run hello-world --image hello-world --restart=Never
pod/hello-world created
```

podが作成されたと表示された。ただし出力はpodのコンテナ内で行われているためコンソールには何も表示されない。hello-worldが実行されていることを確認するためにまず`kubectl get pod`を実行しPodの一覧を表示する。

```
$ kubectl get pod
NAME          READY   STATUS      RESTARTS   AGE
hello-world   0/1     Completed   0          9m37s
```

Pod名が`hello-world`であることが確認できたので`kubectl logs`で出力を確認する。

```
$ kubectl logs pod/hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.
...以下略...
```

dockerのhello-worldイメージを実行したときと同じ出力が確認できる。確認が終わったので下記のコマンドでpodを削除しておく。

```
$ kubectl delete pod/hello-world
pod "hello-world" deleted
```

本当に削除されたかどうかを最後に確認しておく。

```
$ kubectl get pod
No resources found in default namespace.
```

## 同じ内容をマニフェストファイルから実行する

下記の内容でマニフェストファイル(yaml)を作成する。名前は任意。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test
  namespace: default
  labels:
    env: study
spec:
  containers:
    - name: hello-world
      image: hello-world
```

`kubectl apply -f <マニフェストファイル名>`で実行する

```
$ kubectl apply -f pod.yaml
pod/test created
```

作成されたPod名はマニフェストファイルの`metadata.name`で指定した名前となっていることがわかる。Pod一覧を確認してみる。

```
$ kubectl get pod
NAME   READY   STATUS             RESTARTS      AGE
test   0/1     CrashLoopBackOff   1 (16s ago)   45s
```

Pod名がtestとなっていることが確認できた。


### kubectl getの詳細

Podの一覧表示に`kubectl get pod`を使用したが`kubectl get`はその後に続けて表示したいリソースのタイプを指定できる。`kubectl get pod`はPodリソースのみを表示する、という意味になる。`kubectl get all`で全リソースを表示できる。

```
$ kubectl get all
NAME       READY   STATUS      RESTARTS      AGE
pod/test   0/1     Completed   2 (35s ago)   64s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   21h
```

先ほどは`test`と表示されていた行が今度は`pod/test`となっていることがわかる。これはtestリソースの種類がPodであることを示している。`service/kubernates`はkubernatesリソースの種類がServiceであることを示す。

`kubectl get -f <マニフェストファイル>`でマニフェストファイルで指定されたリソースを表示できる。

```
$ kubectl get -f hello.yaml
NAME   READY   STATUS             RESTARTS      AGE
test   0/1     CrashLoopBackOff   2 (41s ago)   97s
```

同様に削除もマニフェストファイルを指定して実行できる。


```
$ kubectl delete -f hello.yaml
pod "test" deleted
```

削除されたことを確認する。

```
$ kubectl get pod
No resources found in default namespace.
```

削除された模様。

### kubectl get で指定できるリソースの種類について

下記コマンドで全てのリソース種類と省略形をリスト表示できる。

```
$ kubectl api-resources
NAME                              SHORTNAMES   APIVERSION                             NAMESPACED   KIND
bindings                                       v1                                     true         Binding
componentstatuses                 cs           v1                                     false        ComponentStatus
configmaps                        cm           v1                                     true         ConfigMap
endpoints                         ep           v1                                     true         Endpoints
events                            ev           v1                                     true         Event
limitranges                       limits       v1                                     true         LimitRange
namespaces                        ns           v1                                     false        Namespace
nodes                             no           v1                                     false        Node
persistentvolumeclaims            pvc          v1                                     true         PersistentVolumeClaim
persistentvolumes                 pv           v1                                     false        PersistentVolume
pods                              po           v1                                     true         Pod
podtemplates                                   v1                                     true         PodTemplate
replicationcontrollers            rc           v1                                     true         ReplicationController
resourcequotas                    quota        v1                                     true         ResourceQuota
secrets                                        v1                                     true         Secret
serviceaccounts                   sa           v1                                     true         ServiceAccount
services                          svc          v1                                     true         Service
mutatingwebhookconfigurations                  admissionregistration.k8s.io/v1        false        MutatingWebhookConfiguration
validatingwebhookconfigurations                admissionregistration.k8s.io/v1        false        ValidatingWebhookConfiguration
customresourcedefinitions         crd,crds     apiextensions.k8s.io/v1                false        CustomResourceDefinition
apiservices                                    apiregistration.k8s.io/v1              false        APIService
controllerrevisions                            apps/v1                                true         ControllerRevision
daemonsets                        ds           apps/v1                                true         DaemonSet
deployments                       deploy       apps/v1                                true         Deployment
replicasets                       rs           apps/v1                                true         ReplicaSet
statefulsets                      sts          apps/v1                                true         StatefulSet
tokenreviews                                   authentication.k8s.io/v1               false        TokenReview
localsubjectaccessreviews                      authorization.k8s.io/v1                true         LocalSubjectAccessReview
selfsubjectaccessreviews                       authorization.k8s.io/v1                false        SelfSubjectAccessReview
selfsubjectrulesreviews                        authorization.k8s.io/v1                false        SelfSubjectRulesReview
subjectaccessreviews                           authorization.k8s.io/v1                false        SubjectAccessReview
horizontalpodautoscalers          hpa          autoscaling/v2                         true         HorizontalPodAutoscaler
cronjobs                          cj           batch/v1                               true         CronJob
jobs                                           batch/v1                               true         Job
certificatesigningrequests        csr          certificates.k8s.io/v1                 false        CertificateSigningRequest
leases                                         coordination.k8s.io/v1                 true         Lease
endpointslices                                 discovery.k8s.io/v1                    true         EndpointSlice
events                            ev           events.k8s.io/v1                       true         Event
flowschemas                                    flowcontrol.apiserver.k8s.io/v1beta2   false        FlowSchema
prioritylevelconfigurations                    flowcontrol.apiserver.k8s.io/v1beta2   false        PriorityLevelConfiguration
ingressclasses                                 networking.k8s.io/v1                   false        IngressClass
ingresses                         ing          networking.k8s.io/v1                   true         Ingress
networkpolicies                   netpol       networking.k8s.io/v1                   true         NetworkPolicy
runtimeclasses                                 node.k8s.io/v1                         false        RuntimeClass
poddisruptionbudgets              pdb          policy/v1                              true         PodDisruptionBudget
clusterrolebindings                            rbac.authorization.k8s.io/v1           false        ClusterRoleBinding
clusterroles                                   rbac.authorization.k8s.io/v1           false        ClusterRole
rolebindings                                   rbac.authorization.k8s.io/v1           true         RoleBinding
roles                                          rbac.authorization.k8s.io/v1           true         Role
priorityclasses                   pc           scheduling.k8s.io/v1                   false        PriorityClass
csidrivers                                     storage.k8s.io/v1                      false        CSIDriver
csinodes                                       storage.k8s.io/v1                      false        CSINode
csistoragecapacities                           storage.k8s.io/v1                      true         CSIStorageCapacity
storageclasses                    sc           storage.k8s.io/v1                      false        StorageClass
volumeattachments                              storage.k8s.io/v1                      false        VolumeAttachment
```

上記の出力を見るとわかる通り省略形は存在しないものもある
