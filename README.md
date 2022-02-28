# upgradeKubernetesCluster

https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/

# 1. Condition before upgrade
Master, Worker8 and 9's version is older than Worker1 and 2, because of some reasons such as the timing to join the kubernetes cluster. But, the version should be consitent between them.
```
$ kubectl get nodes -o wide
NAME      STATUS   ROLES                  AGE     VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
master    Ready    control-plane,master   19d     v1.23.3   192.168.33.100   <none>        Ubuntu 20.04.3 LTS   5.4.0-81-generic   docker://20.10.7
worker1   Ready    node                   5h13m   v1.23.4   192.168.33.101   <none>        Ubuntu 20.04.3 LTS   5.4.0-97-generic   docker://20.10.7
worker2   Ready    node                   5h12m   v1.23.4   192.168.33.102   <none>        Ubuntu 20.04.3 LTS   5.4.0-97-generic   docker://20.10.7
worker8   Ready    node                   19d     v1.23.3   192.168.33.108   <none>        Ubuntu 20.04.3 LTS   5.4.0-89-generic   docker://20.10.7
worker9   Ready    node                   19d     v1.23.3   192.168.33.109   <none>        Ubuntu 20.04.3 LTS   5.4.0-89-generic   docker://20.10.7
```

# 2. Check the upgrade plan
You can learn which version should be upgrade to.
```
$ kubeadm upgrade plan
[upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[preflight] Running pre-flight checks.
[upgrade] Running cluster health checks
[upgrade] Fetching available versions to upgrade to
[upgrade/versions] Cluster version: v1.23.3
[upgrade/versions] kubeadm version: v1.23.3
[upgrade/versions] Target version: v1.23.4
[upgrade/versions] Latest version in the v1.23 series: v1.23.4

Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   CURRENT       TARGET
kubelet     3 x v1.23.3   v1.23.4
            2 x v1.23.4   v1.23.4

Upgrade to the latest version in the v1.23 series:

COMPONENT                 CURRENT   TARGET
kube-apiserver            v1.23.3   v1.23.4
kube-controller-manager   v1.23.3   v1.23.4
kube-scheduler            v1.23.3   v1.23.4
kube-proxy                v1.23.3   v1.23.4
CoreDNS                   v1.8.6    v1.8.6
etcd                      3.5.1-0   3.5.1-0

You can now apply the upgrade by executing the following command:

	kubeadm upgrade apply v1.23.4

Note: Before you can perform this upgrade, you have to update kubeadm to v1.23.4.

_____________________________________________________________________


The table below shows the current state of component configs as understood by this version of kubeadm.
Configs that have a "yes" mark in the "MANUAL UPGRADE REQUIRED" column require manual config upgrade or
resetting to kubeadm defaults before a successful upgrade can be performed. The version to manually
upgrade to is denoted in the "PREFERRED VERSION" column.

API GROUP                 CURRENT VERSION   PREFERRED VERSION   MANUAL UPGRADE REQUIRED
kubeproxy.config.k8s.io   v1alpha1          v1alpha1            no
kubelet.config.k8s.io     v1beta1           v1beta1             no
_____________________________________________________________________

```

```
$ sudo apt-cache madison kubeadm
   kubeadm |  1.23.4-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.23.3-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.23.2-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.23.1-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.23.0-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.22.7-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.22.6-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.22.5-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.22.4-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.22.3-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.22.2-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.22.1-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.22.0-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   ...
```

# 3. Unhold kubeadm and hold it again after upgrading it
```
$ apt-mark unhold kubeadm && \
  apt-get update && apt-get install -y kubeadm=1.23.4-00 && \
  apt-mark hold kubeadm
```

# 4. Activate new version thru kubeadm command
```
$ sudo kubeadm upgrade apply v1.23.4
[upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
...

[upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.23.4". Enjoy!
[upgrade/kubelet] Now that your control plane is upgraded, please proceed with upgrading your kubelets if you haven't already done so.


$ kubeadm upgrade node
[upgrade] Reading configuration from the cluster...
[upgrade] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
W0227 12:39:28.253378 1413873 utils.go:69] The recommended value for "resolvConf" in "KubeletConfiguration" is: /run/systemd/resolve/resolv.conf; the provided value is: /run/systemd/resolve/resolv.conf
...
[upgrade] The control plane instance for this node was successfully updated!
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[upgrade] The configuration for this node was successfully updated!

```

# 5. Unhold kubelet and kubectl and hold them again after upgrading them
```
$ apt-mark unhold kubelet kubectl && \
  apt-get update && apt-get install -y kubelet=1.23.4-00 kubectl=1.23.4-00 && \
  apt-mark hold kubelet kubectl
```

# 6. Condition after upgrade
```
kubectl get nodes -o wide
NAME      STATUS   ROLES                  AGE     VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
master    Ready    control-plane,master   19d     v1.23.4   192.168.33.100   <none>        Ubuntu 20.04.3 LTS   5.4.0-81-generic   docker://20.10.7
worker1   Ready    node                   5h26m   v1.23.4   192.168.33.101   <none>        Ubuntu 20.04.3 LTS   5.4.0-97-generic   docker://20.10.7
worker2   Ready    node                   5h25m   v1.23.4   192.168.33.102   <none>        Ubuntu 20.04.3 LTS   5.4.0-97-generic   docker://20.10.7
worker8   Ready    node                   19d     v1.23.3   192.168.33.108   <none>        Ubuntu 20.04.3 LTS   5.4.0-89-generic   docker://20.10.7
worker9   Ready    node                   19d     v1.23.3   192.168.33.109   <none>        Ubuntu 20.04.3 LTS   5.4.0-89-generic   docker://20.10.7
```

# 7. Worker Node
Very similar to Master Node.
```
apt-mark unhold kubeadm && apt-get update && apt-get install -y kubeadm=1.23.4-00 && apt-mark hold kubeadm
kubeadm upgrade node
apt-mark unhold kubelet kubectl && apt-get update && apt-get install -y kubelet=1.23.4-00 kubectl=1.23.4-00 && apt-mark hold kubelet kubectl
```
