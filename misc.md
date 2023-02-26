# misc

## Research Tools

- [connected paper](https://www.connectedpapers.com/)
- [ccf-deadlines](https://ccfddl.github.io/)
- [paraphraser & grammar-check](https://quillbot.com/)
- [ccfrank](https://github.com/WenyanLiu/CCFrank4dblp)

## Note for Deployment

入门使用 minikube 比 kind 的体验更佳：

- `minikube` 使用 aliyun 仓库拉取 google 的 k8s.gcr.io 镜像：`--image-repository`
- minikube 报错 `[preflight] The system verification failed.`：[更换 kubernetes-version](https://github.com/kubernetes/minikube/issues/15026)、[backup](https://github.com/kubernetes/minikube/issues/14477)
- cni config uninitialized: `kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml`
- [使用自定义的镜像仓库运行 addons - minikube home](https://minikube.sigs.k8s.io/docs/handbook/addons/custom-images/)、[使用提前预载的 image 运行 addons - github](https://github.com/kubernetes/minikube/issues/10877)

k8s 部署过程中可能遇到的问题：

- k8s 在部署的过程中同样遇到 cni config uninitialized 的问题，需要按照网络插件
- [不同的网络情况访问 k8s dashboard](https://segmentfault.com/a/1190000023130407)
- kubelet 未能跑起来可能的原因：[container-runtime 没有配置成功](https://kubernetes.io/zh-cn/docs/setup/production-environment/container-runtimes)
- 使用 containerd 作为容器运行时，crictl 默认使用 `k8s.io` 命名空间：`crictl` 等价于 `ctr -n k8s.io`，镜像迁移可以使用 [podman](https://stackoverflow.com/questions/74595501/how-to-move-a-containerd-image-to-different-namespace)
- metrics-server 部署的一些前置要求：[启动 API Aggregator](https://www.cnblogs.com/lfl17718347843/p/14283796.html)、[未配置 CA 证书时需要 tls 验证](https://www.yeluohuakai.com/posts/2022/05/8635d5dd)、下载镜像的过程中由于镜像存放在 `metrics-server` 的路径下国内镜像无法直接下载可以在 deployment 文件中修改镜像地址或者先 pull 再 tag
- 持久卷部署的一些供应商：[Google Compute Engine - presistent disk (gce-pd)](https://cloud.google.com/compute/docs/disks/add-persistent-disk)、[nfs](https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner)
- 国外镜像拉取失败可以采用阿里云的[容器镜像服务](https://cr.console.aliyun.com/cn-shenzhen/instances) 或者 [自建私有仓库](https://docs.docker.com/registry/deploying/)
- 使用 nfs 作为 provisioner 时，[添加 `no_root_squash` 避免权限问题](https://blog.csdn.net/m0_46090675/article/details/122276216)
- k8s 中 job 无法直接重启，只能先删除再启动：`kubectl get job -n openwhisk owdev-install-packages -o json | jq 'del(.spec.selector)' | jq 'del(.spec.template.metadata.labels)' > install-packages.json.bak`、`cat init-couchdb.json | kubectl replace -f - --force` ([ref](https://serverfault.com/questions/809632/is-it-possible-to-rerun-kubernetes-job))
- linkerd 安装 viz 插件使用镜像：`linkerd viz install --set 'defaultRegistry=registry.cn-shenzhen.aliyuncs.com/warpmatrix' --values ./linkerd.yaml | kubectl apply -f -`
- linkerd 使用镜像完成 inject 操作：`kubectl get -n emojivoto deployments.apps -o yaml | linkerd inject --registry registry.cn-shenzhen.aliyuncs.com/warpmatrix - | kubectl apply -f -`
- 部署 openwhisk 需要用到 npm，可以通过 `kubectl exec <pod-name> -- npm config set registry https://registry.npmmirror.com` 进行换源，查看 npm 的镜像源 `kubectl exec <pod-name> -- npm config get registry`
- openwhisk 安装包的时候由于 npm 版本过低，需要将 `openwhisk/ow-utils:1.0` 更换成 `openwhisk/ow-utils:c5970a6`
- 部署 wasmedge 若不使用超级用户权限安装到系统目录，则需要将库文件添加到加载路径。在 github 中有相应 [issue](https://github.com/containers/crun/issues/1046) 的描述。
- 部署 quark 时如果使用的是 proxmox 需要开启嵌套虚拟化的支持 ([ref](https://zhuanlan.zhihu.com/p/593472919))
- proxmox 虚拟机的扩容：[逻辑卷组的操作](https://einverne.github.io/post/2020/11/extend-proxmox-system-partition-and-pve-file-system.html)、[已有分区的扩展](https://cloud.tencent.com/developer/beta/article/1671893)

冷启动机制：

- openwhisk: without a reactive configuration
  > a stem cell configuration will be provisioned when an invoker starts. A prewarm container that is assigned to an action is moved to the busy pool and the invoker creates one more prewarm container to replenish the prewarm pool maintaining the same number of prewarm containers.
- openwhisk: with a reactive configuration
  - `# of prewarm containers to be created` = `# of cold starts` / `threshold` * `increment`
  - no activation arrives for long time, the number of prewarm containers will eventually converge to `minCount`

technique stack:

- openwhisk: faas platform
- helm: package management of k8s
- kubernetes: production-grade container orchestration
- kind: kubernetes in docker, build a standalone k8s cluster
