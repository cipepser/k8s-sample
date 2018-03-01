# k8s触ってみる

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
 Version:	17.12.0-ce
 API version:	1.35
 Go version:	go1.9.2
 Git commit:	c97c6d6
 Built:	Wed Dec 27 20:03:51 2017
 OS/Arch:	darwin/amd64

Server:
 Engine:
  Version:	17.12.0-ce
  API version:	1.35 (minimum version 1.12)
  Go version:	go1.9.2
  Git commit:	c97c6d6
  Built:	Wed Dec 27 20:12:29 2017
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

```
❯ kubectl cluster-info
Kubernetes master is running at https://192.168.99.100:8443

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

```
❯ kubectl get nodes
NAME       STATUS    ROLES     AGE       VERSION
minikube   Ready     <none>    2m        v1.9.0
```

## References
* [Kubernetesに入門したい ](https://speakerdeck.com/hihihiroro/kubernetesniru-men-sitai)