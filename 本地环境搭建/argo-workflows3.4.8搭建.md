# argo-workflows搭建

基于k8s 1.23.3 安装argo workflow

## 新建namespace

```shell
kubectl create namespace argo
```

## 安装argo-workflows

### 安装命令

master节点执行

```shell
kubectl apply -n argo -f https://github.com/argoproj/argo-workflows/releases/download/v3.4.8/install.yaml
```

本仓库已经保存了[argo3.4.8.yml](./yml/argo3.4.8.yml)，也可以将该文件复制到master节点，然后安装

```shell
kubectl apply -n argo -f argo3.4.8.yaml
```

建议先把yml里面的镜像先手动拉取下来，除此之外，也手动拉取一下这个镜像：quay.io/argoproj/argoexec:v3.4.8

### 验证是否安装成功

```shell
root@k8s-master:/home/seayang# kubectl get pods -n argo
NAME                                   READY   STATUS    RESTARTS   AGE
argo-server-c7d4d7f8b-4w52t            1/1     Running   0          5m37s
workflow-controller-8646cbc66f-9nxw6   1/1     Running   0          5m37s
```

### 获取页面访问token

```shell
root@k8s-master:/home/seayang# kubectl get secret -n argo
NAME                      TYPE                                  DATA   AGE
argo-server-token-6jw2c   kubernetes.io/service-account-token   3      7m9s
argo-token-9bbjk          kubernetes.io/service-account-token   3      7m9s
default-token-w78rw       kubernetes.io/service-account-token   3      2d22h
root@k8s-master:/home/seayang# ARGO_TOKEN="Bearer $(kubectl get secret argo-server-token-6jw2c -n argo  -o=jsonpath='{.data.token}' | base64 --decode)"
root@k8s-master:/home/seayang# echo $ARGO_TOKEN
Bearer eyJhbGciOiJSUzI1NiIsImtpZCI6ImpIQ1MybHV2cGV6aG9RZ0pFc2tQRmhHNkRlNkNZTVRZdGFDMEhGMkFESlkifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJhcmdvIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImFyZ28tc2VydmVyLXRva2VuLTZqdzJjIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFyZ28tc2VydmVyIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiNTkxNDJkMTEtMmYxOS00OTJmLTllZGUtNzYxNWM4MzViMmE1Iiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50OmFyZ286YXJnby1zZXJ2ZXIifQ.nm55u12ZD5rFF3K8UJ5dOGk2RnFciFxt81g3w-tK8JIPPsg4oVpZp-z-JBLVWMfkGP41wYrb2vqJRFydn2M0cAg0YOE1YcPeTSSv2p9PPMM0BVBaPb0J8k1BOohoFsisdRKtbNzUSxcJiabAH4q4ZWIPtrZOMRka9uIwrkcvh7J8D0OKhGGomgS2cjEwVkFyhz3WXB4kmTsafJ7PMdpsOIfb39WyTFEuoddnUc2Hj8ju47nQkZxZfSYsF4cxoHFy-XQqygvtyBxegiSRYEHBH2Lo9EUqTGpyGPMfF2z4xAdBI6GbNURKzTEbieV_iG-WiGSTmkLtc946Yi4m-HrxgQ
```

### 开启页面访问

```shell
root@k8s-master:/home/seayang# kubectl -n argo port-forward --address 0.0.0.0 deployment/argo-server 2746:2746
Forwarding from 0.0.0.0:2746 -> 2746
```

浏览器访问：https://192.168.10.128:2746

**注意**一定要https开头，而不能http开头，会进入argo的登录页面，如下所示

<img src="./images/argo页面登录.png" alt="argo页面登录" width="100%">

将前面的ARGO_TOKEN的值，复制粘贴到中间的白色框内，然后点LOGIN按钮，进入下面的页面

<img src="./images/argo登录后的页面.png" alt="argo登录后的页面" width="100%">

## argo-workflows初体验

master节点默认不可调度，所以所有的pod都会调度到worker节点，在worker节点先手动拉取镜像：docker.1ms.run/library/alpine:3.23.2，然后在master执行

```shell
root@k8s-master:/home/seayang# kubectl create -n argo -f - <<EOF
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: test-alpine-
spec:
  entrypoint: main
  templates:
  - name: main
    container:
      image: docker.1ms.run/library/alpine:3.23.2
      imagePullPolicy: Never
      command: [echo]
      args: ["Hello Argo! My pod is haha"]
EOF
```

输出

```shell
workflow.argoproj.io/test-alpine-5hm48 created
```

### 命令查看

```shell
root@k8s-master:/home/seayang# kubectl get workflows -n argo
NAME                STATUS      AGE     MESSAGE
test-alpine-5hm48   Succeeded   8m58s
```

### 页面查看

工作流列表页

<img src="./images/argo-workflow列表页.png" alt="argo-workflow列表页" width="100%">

点击进入详情

<img src="./images/workflow详情页.png" alt="workflow详情页" width="100%">

点击绿色的圆圈

<img src="./images/workflow详情弹框.png" alt="workflow详情弹框" width="100%">

可以查看各种信息，这里暂时不一一列举了
