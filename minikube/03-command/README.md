```
apiVersion: v1
kind: Pod
metadata:
  name: command
spec:
  containers:
    - name: echo-hello
      image: ubuntu:22.04
      command: ["/bin/bash", "-c"]
      args: ["echo Hello, ${NAME}!"]
      env:
        - name: "NAME"
          value: "World"
```

```
$ kubectl apply -f command.yaml
pod/command created
```

```
$ kubectl logs command
Hello, World!
```
