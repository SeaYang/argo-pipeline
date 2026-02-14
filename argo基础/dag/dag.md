# 通过demo演示dag

[dag-demo.yaml](./dag-demo.yaml)

```shell
root@k8s-master:/home/seayang/argo-base# kubectl create -f dag-demo.yaml -n argo
workflow.argoproj.io/dag-diamond-tg5xq created
```

<img src="../images/dag-demo.png" alt="dag-demo" width="100%">
