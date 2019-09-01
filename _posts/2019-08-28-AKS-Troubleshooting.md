---
layout: post
author: junowa
---

This post gathers AKS troubleshooting stories.

## Pod avec UNKNOW status

En utilisant le ```kubectl describe```, on s'aperçoit que le disque du noeud est saturé.

Ceci est vérifié en se connectant au noeud en ssh avec ```df -h``` puis avec ```docker system df```.
```
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
udev            3.4G     0  3.4G   0% /dev
tmpfs           695M  1.6M  694M   1% /run
/dev/sda1        30G   25G  5.0G  83% /
tmpfs           3.4G     0  3.4G   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           3.4G     0  3.4G   0% /sys/fs/cgroup
/dev/sdb1       281G   63M  267G   1% /mnt
tmpfs           695M     0  695M   0% /run/user/1002
```

```
# docker system df
TYPE                TOTAL               ACTIVE              SIZE                RECLAIMABLE
Images              111                 19                  16.99GB             13.91GB (81%)
Containers          46                  30                  244.2MB             2B (0%)
Local Volumes       2                   2                   48.8kB              0B (0%)
Build Cache         0                   0                   0B                  0B
```

En effet, **/var/lib/docker/overlay2** consomme une grande partie du disque.

Le problème est rapidement résolu par ```docker system prune -a -f```qui va supprimer les images, les conteneurs et les réseaux non utilisés.


Cette suppression devrait être gérée par le mécanisme de Garbage Collector du cluster.

Vérifions la configuration **kubelet** sur le noeud AKS (/etc/default/kubelet):

```
KUBELET_OPTS=

KUBELET_CONFIG=--address=0.0.0.0 --allow-privileged=true --authorization-mode=Webhook --azure-container-registry-config=/etc/kubernetes/azure.json --cadvisor-port=0 --cgroups-per-qos=true --cloud-config=/etc/kubernetes/azure.json --cloud-provider=azure --cluster-dns=10.0.0.10 --cluster-domain=cluster.local --enforce-node-allocatable=pods --event-qps=0 --eviction-hard=memory.available<100Mi,nodefs.available<10%,nodefs.inodesFree<5% --feature-gates=PodPriority=true --image-gc-high-threshold=85 --image-gc-low-threshold=80 --image-pull-progress-deadline=30m --keep-terminated-pod-volumes=false --kube-reserved=cpu=79m,memory=1638Mi --kubeconfig=/var/lib/kubelet/kubeconfig --max-pods=30 --network-plugin=cni --node-status-update-frequency=10s --non-masquerade-cidr=10.201.96.128/25 --pod-infra-container-image=k8s.gcr.io/pause-amd64:3.1 --pod-manifest-path=/etc/kubernetes/manifests --pod-max-pids=100
KUBELET_IMAGE=k8s.gcr.io/hyperkube-amd64:v1.11.5
KUBELET_REGISTER_SCHEDULABLE=true
KUBELET_NODE_LABELS=node-role.kubernetes.io/agent=,kubernetes.io/role=agent,agentpool=default,storageprofile=managed,storagetier=Standard_LRS,kubernetes.azure.com/cluster=MC_dev_kubernetes_westeurope
```

https://kubernetes.io/docs/concepts/cluster-administration/kubelet-garbage-collection/#user-configuration

Sur AKS, par défaut, nous trouvons:
```
--image-gc-high-threshold=85 
--image-gc-low-threshold=80
```

Au-dessus de 85% de disque, le garbage collector devrait nettoyer le cluster.

Note: ces options seront bientôt dépréciées dans les prochaines versions.
