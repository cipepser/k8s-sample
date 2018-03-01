# k8s触ってみる

[Kubernetesに入門したい ](https://speakerdeck.com/hihihiroro/kubernetesniru-men-sitai)
を参考にk8sを触ってみる

## Install

※Docker for Macはすでにinstall済み

```
> brew install kubectl
```

```
> brew cask install minikube
```

## Version確認

```
> docker version
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
> kubectl version
Client Version: version.Info{Major:"1", Minor:"9", GitVersion:"v1.9.3", GitCommit:"d2835416544f298c919e2ead3be3d0864b52323b", GitTreeState:"clean", BuildDate:"2018-02-09T21:51:54Z", GoVersion:"go1.9.4", Compiler:"gc", Platform:"darwin/amd64"}
The connection to the server localhost:8080 was refused - did you specify the right host or port?
```

```
> minikube version
minikube version: v0.25.0
```


## References
* [Kubernetesに入門したい ](https://speakerdeck.com/hihihiroro/kubernetesniru-men-sitai)