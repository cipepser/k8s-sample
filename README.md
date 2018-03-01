# k8s触ってみる

## 構成要素

[Kubernetesの構成を理解する](https://speakerdeck.com/hhiroshell/kubernetesfalsegou-cheng-woli-jie-suru)
が参考になる

## 実際に動かしてみる

[Kubernetesに入門したい ](https://speakerdeck.com/hihihiroro/kubernetesniru-men-sitai)
を参考にk8sを触ってみる

## Install

※Docker for Macはすでにinstall済み

```
❯ brew install kubectl
```

```
❯ brew cask install minikube
```

## Version確認

```
❯ docker version
Client:
 Version:	18.03.0-ce-rc1
 API version:	1.37
 Go version:	go1.9.4
 Git commit:	c160c73
 Built:	Thu Feb 22 02:34:03 2018
 OS/Arch:	darwin/amd64
 Experimental:	false
 Orchestrator:	swarm

Server:
 Engine:
  Version:	18.03.0-ce-rc1
  API version:	1.37 (minimum version 1.12)
  Go version:	go1.9.4
  Git commit:	c160c73
  Built:	Thu Feb 22 02:42:37 2018
  OS/Arch:	linux/amd64
  Experimental:	true
```

```
❯ kubectl version
Client Version: version.Info{Major:"1", Minor:"9", GitVersion:"v1.9.3", GitCommit:"d2835416544f298c919e2ead3be3d0864b52323b", GitTreeState:"clean", BuildDate:"2018-02-09T21:51:54Z", GoVersion:"go1.9.4", Compiler:"gc", Platform:"darwin/amd64"}
```

```
❯ minikube version
minikube version: v0.25.0
```

## k8sクラスタ作成

minikube起動(5分くらいかかった)

```
❯ minikube start
Starting local Kubernetes v1.9.0 cluster...
Starting VM...
Downloading Minikube ISO
 142.22 MB / 142.22 MB [===========================================] 100.00% 0s
Getting VM IP address...
Moving files into cluster...
Downloading localkube binary
 162.41 MB / 162.41 MB [===========================================] 100.00% 0s
 0 B / 65 B [---------------------------------------------------------]   0.00%
 65 B / 65 B [=====================================================] 100.00% 0sSetting up certs...
Connecting to cluster...
Setting up kubeconfig...
Starting cluster components...
Kubectl is now configured to use the cluster.
Loading cached images from config file.
```

クラスタ情報を見る

```
❯ kubectl cluster-info
Kubernetes master is running at https://192.168.99.100:8443

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

ノード情報を見る

```
❯ kubectl get nodes
NAME       STATUS    ROLES     AGE       VERSION
minikube   Ready     <none>    2m        v1.9.0
```

## k8sクラスタにアプリをデプロイ

デプロイ

```
❯ kubectl run kubernetes-bootcamp --image=docker.io/jocatalin/kubernetes-bootcamp:v1 --port=8080
deployment "kubernetes-bootcamp" created
```

デプロイしたアプリの情報を見る

```
❯ kubectl get deployments
NAME                  DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   1         1         1            0           15s
```

k8sクラスタにアクセスするためのproxyを起動

```
❯ kubectl proxy
Starting to serve on 127.0.0.1:8001
```

`crul`を別タブで叩く

```
❯ curl http://localhost:8001/version
{
  "major": "",
  "minor": "",
  "gitVersion": "v1.9.0",
  "gitCommit": "925c127ec6b946659ad0fd596fa959be43f0cc05",
  "gitTreeState": "clean",
  "buildDate": "2018-01-26T19:04:38Z",
  "goVersion": "go1.9.1",
  "compiler": "gc",
  "platform": "linux/amd64"
}
```

PODへアクセスするために環境変数に保存

```
❯ export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')

❯ echo Name of the Pod: $POD_NAME
Name of the Pod: kubernetes-bootcamp-5d7f968ccb-d7cjb
```

PODにアクセス

```
❯ curl http://localhost:8001/api/v1/proxy/namespaces/default/pods/$POD_NAME/
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-5d7f968ccb-d7cjb | v=1
```

PODの情報

```
❯ kubectl get pods
NAME                                   READY     STATUS    RESTARTS   AGE
kubernetes-bootcamp-5d7f968ccb-d7cjb   1/1       Running   0          2h

```

アクセスログを見る

```
❯ kubectl logs $POD_NAME
Kubernetes Bootcamp App Started At: 2018-03-01T09:38:31.190Z | Running On:  kubernetes-bootcamp-5d7f968ccb-d7cjb

Running On: kubernetes-bootcamp-5d7f968ccb-d7cjb | Total Requests: 1 | App Uptime: 7920.929 seconds | Log Time: 2018-03-01T11:50:32.119Z

```

もう一回アクセスしてみる

```
❯ curl http://localhost:8001/api/v1/proxy/namespaces/default/pods/$POD_NAME/
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-5d7f968ccb-d7cjb | v=1
```

アクセスが増えている

```
❯ kubectl logs $POD_NAME
Kubernetes Bootcamp App Started At: 2018-03-01T09:38:31.190Z | Running On:  kubernetes-bootcamp-5d7f968ccb-d7cjb

Running On: kubernetes-bootcamp-5d7f968ccb-d7cjb | Total Requests: 1 | App Uptime: 7920.929 seconds | Log Time: 2018-03-01T11:50:32.119Z
Running On: kubernetes-bootcamp-5d7f968ccb-d7cjb | Total Requests: 2 | App Uptime: 7983.077 seconds | Log Time: 2018-03-01T11:51:34.267Z
```

PODの詳細

```
❯ kubectl describe pods
Name:           kubernetes-bootcamp-5d7f968ccb-d7cjb
Namespace:      default
Node:           minikube/192.168.99.100
Start Time:     Thu, 01 Mar 2018 18:37:56 +0900
Labels:         pod-template-hash=1839524776
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.17.0.4
Controlled By:  ReplicaSet/kubernetes-bootcamp-5d7f968ccb
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://8ac0c067e72ac365032beedda041b02aae3d3fccc72dba66adfb0dce9a6491d3
    Image:          docker.io/jocatalin/kubernetes-bootcamp:v1
    Image ID:       docker-pullable://jocatalin/kubernetes-bootcamp@sha256:0d6b8ee63bb57c5f5b6156f446b3bc3b3c143d233037f3a2f00e279c8fcc64af
    Port:           8080/TCP
    State:          Running
      Started:      Thu, 01 Mar 2018 18:38:31 +0900
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-b8pj8 (ro)
Conditions:
  Type           Status
  Initialized    True
  Ready          True
  PodScheduled   True
Volumes:
  default-token-b8pj8:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-b8pj8
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     <none>
Events:          <none>

```

POD内でコマンド実行

```
❯ kubectl exec $POD_NAME env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=kubernetes-bootcamp-5d7f968ccb-d7cjb
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
KUBERNETES_SERVICE_HOST=10.96.0.1
NPM_CONFIG_LOGLEVEL=info
NODE_VERSION=6.3.1
HOME=/root

```

同じ要領で`ls`コマンドも叩ける

```
❯ kubectl exec $POD_NAME ls
bin
boot
core
dev
etc
home
lib
lib64
media
mnt
opt
proc
root
run
sbin
server.js
srv
sys
tmp
usr
var
```

bashでPOD内に入る

```
❯ kubectl exec -ti $POD_NAME bash
root@kubernetes-bootcamp-5d7f968ccb-d7cjb:/# env
NODE_VERSION=6.3.1
HOSTNAME=kubernetes-bootcamp-5d7f968ccb-d7cjb
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT=tcp://10.96.0.1:443
TERM=xterm
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_HOST=10.96.0.1
NPM_CONFIG_LOGLEVEL=info
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
PWD=/
SHLVL=1
HOME=/root
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
_=/usr/bin/env
root@kubernetes-bootcamp-5d7f968ccb-d7cjb:/# exit
```


## k8sクラスタ外にアプリの公開

```
❯ kubectl get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   2h
```

外部へ公開。スライドのとおりだとうまくいかない。バージョンが違うから？

```
❯ kubectl expose deployment kubernetes-bootcamp --type="NodePort" --port=8080 service "kubernetes-bootcamp" exposed
service "kubernetes-bootcamp" exposed
Error from server (NotFound): deployments.extensions "service" not found
```

TODO: errorが起こる

一応以下のようにservicesを見返すと立ち上がっている

```
❯ kubectl get services
NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
kubernetes            ClusterIP   10.96.0.1       <none>        443/TCP          2h
kubernetes-bootcamp   NodePort    10.106.41.252   <none>        8080:31399/TCP   11s
```

PORTとIPを取得して、Nodeにアクセス

```
❯ export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}')
❯ echo $NODE_PORT
31399

❯ export NODE_IP=$(minikube ip)
❯ echo $NODE_IP
192.168.99.100

❯ curl http://$NODE_IP:$NODE_PORT
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-5d7f968ccb-d7cjb | v=1
```

kubernetes-bootcampが動いているPODとservicesを表示

```
❯ kubectl get pods -l run=kubernetes-bootcamp
NAME                                   READY     STATUS    RESTARTS   AGE
kubernetes-bootcamp-5d7f968ccb-d7cjb   1/1       Running   1          3h

❯ kubectl get services -l run=kubernetes-bootcamp
NAME                  TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
kubernetes-bootcamp   NodePort   10.106.41.252   <none>        8080:31399/TCP   32m
```

`app=v1`というラベルを付与
上2行は、すでにやっているもの


```
❯ export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')

❯ echo Name of the Pod: $POD_NAME
Name of the Pod: kubernetes-bootcamp-5d7f968ccb-d7cjb

❯ kubectl label pod $POD_NAME app=v1
pod "kubernetes-bootcamp-5d7f968ccb-d7cjb" labeled
```

PODの詳細

```
❯ kubectl describe pods/$POD_NAME
Name:           kubernetes-bootcamp-5d7f968ccb-d7cjb
Namespace:      default
Node:           minikube/192.168.99.100
Start Time:     Thu, 01 Mar 2018 18:37:56 +0900
Labels:         app=v1
                pod-template-hash=1839524776
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.17.0.3
Controlled By:  ReplicaSet/kubernetes-bootcamp-5d7f968ccb
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://f3705aa4a0c03b2e9a87441b04883a8a14c0de1f8a783b9e2dc0758ff8544c72
    Image:          docker.io/jocatalin/kubernetes-bootcamp:v1
    Image ID:       docker-pullable://jocatalin/kubernetes-bootcamp@sha256:0d6b8ee63bb57c5f5b6156f446b3bc3b3c143d233037f3a2f00e279c8fcc64af
    Port:           8080/TCP
    State:          Running
      Started:      Thu, 01 Mar 2018 21:30:39 +0900
    Last State:     Terminated
      Reason:       Error
      Exit Code:    255
      Started:      Thu, 01 Mar 2018 18:38:31 +0900
      Finished:     Thu, 01 Mar 2018 21:30:05 +0900
    Ready:          True
    Restart Count:  1
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-b8pj8 (ro)
Conditions:
  Type           Status
  Initialized    True
  Ready          True
  PodScheduled   True
Volumes:
  default-token-b8pj8:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-b8pj8
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     <none>
Events:
  Type    Reason                 Age   From               Message
  ----    ------                 ----  ----               -------
  Normal  SuccessfulMountVolume  11m   kubelet, minikube  MountVolume.SetUp succeeded for volume "default-token-b8pj8"
  Normal  SandboxChanged         11m   kubelet, minikube  Pod sandbox changed, it will be killed and re-created.
  Normal  Pulled                 11m   kubelet, minikube  Container image "docker.io/jocatalin/kubernetes-bootcamp:v1" already present on machine
  Normal  Created                11m   kubelet, minikube  Created container
  Normal  Started                11m   kubelet, minikube  Started container
```

ラベルからPODを表示

```
❯ kubectl get pods -l app=v1
NAME                                   READY     STATUS    RESTARTS   AGE
kubernetes-bootcamp-5d7f968ccb-d7cjb   1/1       Running   1          3h
```

### サービスの削除

削除する

```
❯ kubectl delete services -l run=kubernetes-bootcamp
service "kubernetes-bootcamp" deleted
```

サービスが消えたことを確認

```
❯ kubectl get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   3h
```

Nodeへ直接アクセスもできない

```
❯ curl http://$NODE_IP:$NODE_PORT
curl: (7) Failed to connect to 192.168.99.100 port 31399: Connection refused
```

でもアプリは生きている

```
❯ kubectl exec -ti $POD_NAME curl localhost:8080
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-5d7f968ccb-d7cjb | v=1
```














## References
* [Kubernetesに入門したい ](https://speakerdeck.com/hihihiroro/kubernetesniru-men-sitai)
* [Kubernetesの構成を理解する](https://speakerdeck.com/hhiroshell/kubernetesfalsegou-cheng-woli-jie-suru)