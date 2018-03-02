# Kubernetes 速習会

[Kubernetes 速習会](https://qiita.com/koudaiii/items/d0b3b0b78dc44d97232a)を参考に自分の手を動かしてみる

`k8s-sokusyu.yaml`がないので、hello world以降をやる。

## alias

`kubectl`を打つのがめんどくさくなってきたので、aliasを貼る。
`.zshrc`や`.bashrc`に記載するか、以下の`k`は`kubectl`であることに注意。

```
alias k='kubectl'
```

## hello world

`https://github.com/cipepser/k8s-sample/4`以下に`docker-hello-world`をcloneする

```
❯ git submodule add https://github.com/koudaiii/docker-hello-world 4/docker-hello-world
```

### namespace

namespaceを自分の名前に変える

```
❯ git diff kubernetes/namespace.yaml
diff --git a/kubernetes/namespace.yaml b/kubernetes/namespace.yaml
index e4fa54e..bcf0f8f 100644
--- a/kubernetes/namespace.yaml
+++ b/kubernetes/namespace.yaml
@@ -1,4 +1,4 @@
 apiVersion: v1
 kind: Namespace
 metadata:
-  name: docker-hello-world
+  name: cipepser
```

自分のnamespaceを作成

```
❯ kubectl create -f kubernetes/namespace.yaml
namespace "cipepser" created
```

namespaceができたことを確認

```
❯ kubectl get namespace
NAME          STATUS    AGE
cipepser      Active    18s
default       Active    10h
docker        Active    10h
kube-public   Active    10h
kube-system   Active    10h
```

Podを見てみる

```
❯ kubectl get po --namespace=kube-system
NAME                                         READY     STATUS    RESTARTS   AGE
etcd-docker-for-desktop                      1/1       Running   0          10h
kube-apiserver-docker-for-desktop            1/1       Running   0          10h
kube-controller-manager-docker-for-desktop   1/1       Running   0          10h
kube-dns-6f4fd4bdf-5g56r                     3/3       Running   0          10h
kube-proxy-gvm58                             1/1       Running   0          10h
kube-scheduler-docker-for-desktop            1/1       Running   0          10h
```

`puroduction.yaml`の`namespace`も自分の名前に書き換える

```
❯ git diff kubernetes/production.yaml
diff --git a/kubernetes/production.yaml b/kubernetes/production.yaml
index 5195ed6..533e711 100644
--- a/kubernetes/production.yaml
+++ b/kubernetes/production.yaml
@@ -1,7 +1,7 @@
 apiVersion: extensions/v1beta1
 kind: Deployment
 metadata:
-  namespace: docker-hello-world
+  namespace: cipepser
   name: docker-hello-world
   labels:
     name: docker-hello-world
```

自分のspaceで動かす  
※ここから`kubectl`を`k`でalias張った

```
❯ k create -f kubernetes/production.yaml --namespace=cipepser
deployment "docker-hello-world" created
```

###  go server

`git clone`する

```
❯ git submodule add https://github.com/koudaiii/go_server 4/go_server
```

アプリ起動

```
❯ cd 4/go_server/kubernetes

❯ k create -f blue.yaml --namespace=cipepser
deployment "blue" created

❯ k create -f green.yaml --namespace=cipepser
deployment "green" created

❯ k create -f service-go.yaml --namespace=cipepser
service "go" created
```

nginx起動する前にpodをwatchする(以下はnginx起動後のため、nginxが増えている状態)

```
❯ k get po -w --namespace=cipepser
NAME                                 READY     STATUS    RESTARTS   AGE
blue-7f5c4c44cf-5n2s2                1/1       Running   0          1h
docker-hello-world-b5f7848d5-2lgns   0/1       Pending   0          1h
docker-hello-world-b5f7848d5-4x5wm   1/1       Running   0          1h
docker-hello-world-b5f7848d5-cqxjx   0/1       Pending   0          1h
docker-hello-world-b5f7848d5-ctf4z   0/1       Pending   0          1h
docker-hello-world-b5f7848d5-gmkwb   0/1       Pending   0          1h
docker-hello-world-b5f7848d5-h868l   0/1       Pending   0          1h
docker-hello-world-b5f7848d5-q9qcg   0/1       Pending   0          1h
docker-hello-world-b5f7848d5-qsclv   0/1       Pending   0          1h
docker-hello-world-b5f7848d5-v99dn   0/1       Pending   0          1h
docker-hello-world-b5f7848d5-x6gv2   0/1       Pending   0          1h
green-76b9bf4dd8-kxd87               1/1       Running   0          1h
nginx-7455b586c5-skxdq   0/1       Pending   0         <invalid>
nginx-7455b586c5-skxdq   0/1       Pending   0         <invalid>
nginx-7455b586c5-skxdq   0/1       ContainerCreating   0         <invalid>
nginx-7455b586c5-skxdq   1/1       Running   0         28s
```

上記のwatchとは別タブで、nginx起動

```
❯ k create -f deployment-nginx.yaml --namespace=cipepser
service "nginx" created // ここ消してしまったのでこんなメッセージだったはずのもの

❯ k create -f service-nginx.yaml --namespace=cipepser
service "nginx" created
```

サービスを確認する

```
❯ k get svc --namespace=cipepser
NAME      TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
go        NodePort       10.97.95.122   <none>        8080:31397/TCP   1h
nginx     LoadBalancer   10.96.228.55   localhost     80:31914/TCP     2m
```

deploymentsを確認する

```
❯ k get deployment --namespace=cipepser
NAME                 DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
blue                 1         1         1            1           1h
docker-hello-world   10        10        10           1           1h
green                1         1         1            1           1h
nginx                1         1         1            1           3m
```

サービスの詳細

```
❯ k describe svc --namespace=cipepser
Name:                     go
Namespace:                cipepser
Labels:                   app=go
                          name=app
Annotations:              <none>
Selector:                 app=go,color=blue,name=app
Type:                     NodePort
IP:                       10.97.95.122
LoadBalancer Ingress:     localhost
Port:                     go  8080/TCP
TargetPort:               8080/TCP
NodePort:                 go  31397/TCP
Endpoints:                10.1.0.5:8080
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>


Name:                     nginx
Namespace:                cipepser
Labels:                   app=nginx
Annotations:              <none>
Selector:                 app=nginx
Type:                     LoadBalancer
IP:                       10.96.228.55
LoadBalancer Ingress:     localhost
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  31914/TCP
Endpoints:                10.1.0.7:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

### color

blue-green deployment

```
❯ k describe svc go --namespace=cipepser
Name:                     go
Namespace:                cipepser
Labels:                   app=go
                          name=app
Annotations:              <none>
Selector:                 app=go,color=blue,name=app
Type:                     NodePort
IP:                       10.97.95.122
LoadBalancer Ingress:     localhost
Port:                     go  8080/TCP
TargetPort:               8080/TCP
NodePort:                 go  31397/TCP
Endpoints:                10.1.0.5:8080
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

deploymentのreplicasを`1`から`10`に変更

```
❯ k edit deployment green --namespace=cipepser
21 spec:
22   minReadySeconds: 30
23   replicas: 10  # ここを10にしてみました
24   selector:
25     matchLabels:
26       app: go

deployment "green" edited
```

svcのcolorを`blue`から`green`に変更

```
❯ k edit svc go --namespace=cipepser

25   selector:
26     app: go
27     color: green       ## blue -> green  へ変更
28     name: app
29   sessionAffinity: None

service "go" edited
```

変更されている

```
❯ k get deployment --namespace=cipepser
NAME                 DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
blue                 1         1         1            1           1h
docker-hello-world   10        10        10           1           2h
green                10        10        10           10          1h
nginx                1         1         1            1           18m
```

Endpointsも増えている

```
❯ k describe svc go --namespace=cipepser
Name:                     go
Namespace:                cipepser
Labels:                   app=go
                          name=app
Annotations:              <none>
Selector:                 app=go,color=green,name=app
Type:                     NodePort
IP:                       10.97.95.122
LoadBalancer Ingress:     localhost
Port:                     go  8080/TCP
TargetPort:               8080/TCP
NodePort:                 go  31397/TCP
Endpoints:                10.1.0.10:8080,10.1.0.11:8080,10.1.0.12:8080 + 7 more...
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

### health check

#### http Get

http Getでhealth checkするために`liveness-httpGet.yaml`に以下の内容を記載

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginxhttp
spec:
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
        livenessProbe:
          httpGet:
            # Path to probe; should be cheap, but representative of typical behavior
            path: /index.html
            port: 80
          initialDelaySeconds: 30
          timeoutSeconds: 1
```

デプロイ

```
❯ k create -f liveness-httpGet.yaml --namespace=cipepser
deployment "nginxhttp" created
```

確認

```
❯ k describe pods nginxhttp --namespace=cipepser
(中略)
Events:
  Type    Reason                 Age   From                         Message
  ----    ------                 ----  ----                         -------
  Normal  Scheduled              1m    default-scheduler            Successfully assigned nginxhttp-759fbc6564-jlz24 to docker-for-desktop
  Normal  SuccessfulMountVolume  1m    kubelet, docker-for-desktop  MountVolume.SetUp succeeded for volume "default-token-f8285"
  Normal  Pulling                1m    kubelet, docker-for-desktop  pulling image "nginx"
  Normal  Pulled                 1m    kubelet, docker-for-desktop  Successfully pulled image "nginx"
  Normal  Created                1m    kubelet, docker-for-desktop  Created container
  Normal  Started                1m    kubelet, docker-for-desktop  Started container
```

#### exec


http Getでhealth checkするために`liveness-exec.yaml`に以下の内容を記載

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - args:
    - /bin/sh
    - -c
    - echo ok > /tmp/health; sleep 10; rm -rf /tmp/health; sleep 600
    image: gcr.io/google_containers/busybox
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/health
      initialDelaySeconds: 15
      timeoutSeconds: 1
    name: liveness
```

デプロイ

```
❯ k create -f liveness-exec.yaml --namespace=cipepser
pod "liveness-exec" created
```

確認するとhealth checkに失敗しているのがわかる。

```
❯ k describe pods liveness-exec --namespace=cipepser
(中略)
Events:
  Type     Reason                 Age               From                         Message
  ----     ------                 ----              ----                         -------
  Normal   Scheduled              2m                default-scheduler            Successfully assigned liveness-exec to docker-for-desktop
  Normal   SuccessfulMountVolume  2m                kubelet, docker-for-desktop  MountVolume.SetUp succeeded for volume "default-token-f8285"
  Warning  Unhealthy              36s (x5 over 1m)  kubelet, docker-for-desktop  Liveness probe failed: cat: can't open '/tmp/health': No such file or directory
  Normal   Pulling                6s (x3 over 2m)   kubelet, docker-for-desktop  pulling image "gcr.io/google_containers/busybox"
  Normal   Killing                6s (x2 over 1m)   kubelet, docker-for-desktop  Killing container with id docker://liveness:Container failed liveness probe.. Container will be killed and recreated.
  Normal   Pulled                 4s (x3 over 2m)   kubelet, docker-for-desktop  Successfully pulled image "gcr.io/google_containers/busybox"
  Normal   Created                4s (x3 over 2m)   kubelet, docker-for-desktop  Created container
  Normal   Started                3s (x3 over 2m)   kubelet, docker-for-desktop  Started container
  ```

podの削除

```
❯ k delete pods liveness-exec --namespace=cipepser
pod "liveness-exec" deleted
```

















## References
* [Kubernetes 速習会](https://qiita.com/koudaiii/items/d0b3b0b78dc44d97232a)