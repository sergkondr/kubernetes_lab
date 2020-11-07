# Kubernetes lab
This repo is used for some checks and getting lab expirience. It is based on [kubernetes blog article](https://kubernetes.io/blog/2019/03/15/kubernetes-setup-using-ansible-and-vagrant/) with minor changes.


### Start VM
```
vagrant up k8s-master node-1
```
and then `vagrant ssh k8s-master`


### Init master
```
sudo kubeadm init \
  --apiserver-advertise-address="192.168.50.10" \
  --apiserver-cert-extra-sans="192.168.50.10" \
  --node-name k8s-master
```
We don't use flag `--pod-network-cidr=X.X.X.X/X` purposely.

And then add kubeconfig to home-dir:
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```


### Manual allocation of pod subnets
- Patch cidr for each node include master
```
kubectl patch node k8s-master -p '{"spec":{"podCIDR":"192.168.0.0/24"}}'
```
- [Install kuberouter](###Install kuberouter)
- Join node to the cluster



### Install kuberouter
```
kubectl apply -f kubernetes/kuberouter.yaml
```
This file is based on [official deployment file](https://raw.githubusercontent.com/cloudnativelabs/kube-router/master/daemonset/generic-kuberouter-all-features.yaml) but it contains some fixes:
1. Strict master IP instead of the service IP according to [this issue](https://github.com/cloudnativelabs/kube-router/issues/625#issuecomment-453305576)
2. Fix volume mounts to use kubeconfig


### Reset cluster
```
sudo kubeadm reset
sudo rm -rf /etc/cni/net.d
rm $HOME/.kube/config
```
