# Docker for Mac with Kubernetes

[Docker for Mac with Kubernetes](https://qiita.com/taishin/items/920d62a641c9cd58f289)を参考にk8sを触ってみる

## kubectlのuninstall

2でinstallしたkubectlを削除する

```
❯ brew uninstall kubectl
Uninstalling /usr/local/Cellar/kubernetes-cli/1.9.3... (172 files, 65.4MB)
```

dockerの`Preference`で`Enable Kubernetes`にチェックを入れる


## contextの変更

現在のcontextを確認

```
❯ kubectl config get-contexts
CURRENT   NAME                 CLUSTER                      AUTHINFO             NAMESPACE
          docker-for-desktop   docker-for-desktop-cluster   docker-for-desktop
*         minikube             minikube                     minikube
```

`docker-for-desktop`に変更

```
❯ kubectl config use-context docker-for-desktop
Switched to context "docker-for-desktop".
```

変わったことを確認

```
❯ kubectl config get-contexts
CURRENT   NAME                 CLUSTER                      AUTHINFO             NAMESPACE
*         docker-for-desktop   docker-for-desktop-cluster   docker-for-desktop
          minikube             minikube                     minikube
```

ノードの詳細を確認

```
❯ kubectl describe nodes
Name:               docker-for-desktop
Roles:              master
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/hostname=docker-for-desktop
                    node-role.kubernetes.io/master=
Annotations:        node.alpha.kubernetes.io/ttl=0
                    volumes.kubernetes.io/controller-managed-attach-detach=true
Taints:             <none>
CreationTimestamp:  Thu, 01 Mar 2018 23:11:33 +0900
Conditions:
  Type             Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----             ------  -----------------                 ------------------                ------                       -------
  OutOfDisk        False   Fri, 02 Mar 2018 00:02:03 +0900   Thu, 01 Mar 2018 23:11:27 +0900   KubeletHasSufficientDisk     kubelet has sufficient disk space available
  MemoryPressure   False   Fri, 02 Mar 2018 00:02:03 +0900   Thu, 01 Mar 2018 23:11:27 +0900   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure     False   Fri, 02 Mar 2018 00:02:03 +0900   Thu, 01 Mar 2018 23:11:27 +0900   KubeletHasNoDiskPressure     kubelet has no disk pressure
  Ready            True    Fri, 02 Mar 2018 00:02:03 +0900   Thu, 01 Mar 2018 23:11:27 +0900   KubeletReady                 kubelet is posting ready status
Addresses:
  InternalIP:  192.168.65.3
  Hostname:    docker-for-desktop
Capacity:
 cpu:     2
 memory:  2046948Ki
 pods:    110
Allocatable:
 cpu:     2
 memory:  1944548Ki
 pods:    110
System Info:
 Machine ID:
 System UUID:                620345C4-0000-0000-8DDB-6F1E1779F7FA
 Boot ID:                    62c066a7-950b-4b2d-b892-e0d6c23fadc3
 Kernel Version:             4.9.75-linuxkit-aufs
 OS Image:                   Docker for Mac
 Operating System:           linux
 Architecture:               amd64
 Container Runtime Version:  docker://18.3.0
 Kubelet Version:            v1.9.2
 Kube-Proxy Version:         v1.9.2
ExternalID:                  docker-for-desktop
Non-terminated Pods:         (8 in total)
  Namespace                  Name                                          CPU Requests  CPU Limits  Memory Requests  Memory Limits
  ---------                  ----                                          ------------  ----------  ---------------  -------------
  docker                     compose-5d4f4d67b6-h48lp                      0 (0%)        0 (0%)      0 (0%)           0 (0%)
  docker                     compose-api-7bb7b5968f-9t9xh                  0 (0%)        0 (0%)      0 (0%)           0 (0%)
  kube-system                etcd-docker-for-desktop                       0 (0%)        0 (0%)      0 (0%)           0 (0%)
  kube-system                kube-apiserver-docker-for-desktop             250m (12%)    0 (0%)      0 (0%)           0 (0%)
  kube-system                kube-controller-manager-docker-for-desktop    200m (10%)    0 (0%)      0 (0%)           0 (0%)
  kube-system                kube-dns-6f4fd4bdf-5g56r                      260m (13%)    0 (0%)      110Mi (5%)       170Mi (8%)
  kube-system                kube-proxy-gvm58                              0 (0%)        0 (0%)      0 (0%)           0 (0%)
  kube-system                kube-scheduler-docker-for-desktop             100m (5%)     0 (0%)      0 (0%)           0 (0%)
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  CPU Requests  CPU Limits  Memory Requests  Memory Limits
  ------------  ----------  ---------------  -------------
  810m (40%)    0 (0%)      110Mi (5%)       170Mi (8%)
Events:
  Type    Reason                   Age                From                         Message
  ----    ------                   ----               ----                         -------
  Normal  NodeAllocatableEnforced  51m                kubelet, docker-for-desktop  Updated Node Allocatable limit across pods
  Normal  NodeHasSufficientDisk    51m (x8 over 51m)  kubelet, docker-for-desktop  Node docker-for-desktop status is now: NodeHasSufficientDisk
  Normal  NodeHasSufficientMemory  51m (x8 over 51m)  kubelet, docker-for-desktop  Node docker-for-desktop status is now: NodeHasSufficientMemory
  Normal  NodeHasNoDiskPressure    51m (x7 over 51m)  kubelet, docker-for-desktop  Node docker-for-desktop status is now: NodeHasNoDiskPressure
```

## システムコンテナ

デフォルトで見えないシステムコンテナを見えるようにするため、
dockerの`Preference`で`Show system containers (advanced)`にチェックを入れる


## References
* [Docker for Mac with Kubernetes](https://qiita.com/taishin/items/920d62a641c9cd58f289)