# misc

## Research Tools

- [connected paper](https://www.connectedpapers.com/)
- [ccf-deadlines](https://ccfddl.github.io/)
- [paraphraser & grammar-check](https://quillbot.com/)
- [ccfrank](https://github.com/WenyanLiu/CCFrank4dblp)

## Note for Experiment

- cni config uninitialized: `kubectl apply -f coreos/flannel/master/Documentation/kube-flannel.yml`
- `minikube` 使用 aliyun 仓库拉取 google 的 k8s.gcr.io 镜像：`--image-repository`

technique stack:

- openwhisk: faas platform
- helm: package management of k8s
- kubernetes: production-grade container orchestration
- kind: build cluster for k8s
