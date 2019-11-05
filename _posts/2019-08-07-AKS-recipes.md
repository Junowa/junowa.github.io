---
layout: post
author: junowa
---


Some AKS basic recipes...

The source code is here: https://github.com/Junowa/test-kubernetes/


# Simple pod troubleshooting

To create from cli a simple deployment:
```
kubectl -n jno run -it --rm alpine-ssh --image=alpine
```

This command will create a deployment and open an interactive session to it.

After exiting the interactive session, the deployment will be deleted.


If you don't use the interactive fleg **-it**, the pod will be created and will terminated (because it has nothing else to do), resulting in a CrashLoopBackOff state.

https://serverfault.com/questions/924243/back-off-restarting-failed-container-error-syncing-pod-in-minikube




# AKS services

https://docs.microsoft.com/fr-fr/azure/aks/concepts-network

## Load balancer service

- type load-balancer service creates a front end ip in the load balancer
- type load-balancer service destruction removes the front end ip address

```
apiVersion: apps/v1
kind: Deployment
metadata: 
  labels:
    app: hello-example-8080
  name: hello-example-8080    # Name of the deployment
spec:
  selector:      #The selector field defines how the Deployment finds which Pods to manage.
    matchLabels:
      app: hello-example-8080
  replicas: 1
  template:
    metadata:
      labels:
        app: hello-example-8080   #Must match with selector.matchLabels
    spec:
      containers:
      - image: gcr.io/google-samples/node-hello:1.0
        name: hello-example-8080
        ports:
        - containerPort: 8080

---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: hello-example-8080
  name: lb-hello-example-8080   # Name of the service
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: hello-example-8080   # The service targets TCP/8080 on any Pod with the app=hello-example-8080 label
  type: LoadBalancer
```

## Ingress

https://docs.microsoft.com/bs-cyrl-ba/azure/aks/ingress-basic

The ingress controller creates an Azure Load-Balancer (and frontend IP).

Get the frontend IP:

```
$ kubectl -n ingress-controller get svc
```

Then we can create a simple pod and ClusterIP service to expose the pod.

```
The ingress controller creates a azure load balancer and a frontend IP address.
kind: Pod
apiVersion: v1
metadata:
  name: apple-app
  labels:
    app: apple
spec:
  containers:
    - name: apple-app
      image: hashicorp/http-echo
      args:
        - "-text=apple"

---
kind: Service
apiVersion: v1
metadata:
  name: apple-service
spec:
  selector:
    app: apple
  ports:
    - port: 5678 # Default port for image
```

Finally, let's create the ingress:

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
        - path: /apple
          backend:
            serviceName: apple-service
            servicePort: 5678
```


You can reach your pod using the ingress controller frontend IP.

```
curl http://ingress_controller_frontend_ip/apple
```


# AKS Storage


https://docs.microsoft.com/fr-fr/azure/aks/azure-disks-dynamic-pv


To create persitent storage:
- request a Persistent Volume Claim (PVC) through the storage provider to create the Persistent Volume (PV)
- mount it to the pod container.


In AKS, two initial StorageClasses are defined:
- default (azure standard)
- managed-premium (azure premium)

```
$ kubectl get sc default -o yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"storage.k8s.io/v1beta1","kind":"StorageClass","metadata":{"annotations":{"storageclass.beta.kubernetes.io/is-default-class":"true"},"labels":{"kubernetes.io/cluster-service":"true"},"name":"default","namespace":""},"parameters":{"cachingmode":"ReadOnly","kind":"Managed","storageaccounttype":"Standard_LRS"},"provisioner":"kubernetes.io/azure-disk"}
    storageclass.beta.kubernetes.io/is-default-class: "true"
  creationTimestamp: "2019-09-26T12:35:54Z"
  labels:
    kubernetes.io/cluster-service: "true"
  name: default
  resourceVersion: "37791024"
  selfLink: /apis/storage.k8s.io/v1/storageclasses/default
  uid: 2edbf09e-e05a-11e9-bf11-3aa52db08f12
parameters:
  cachingmode: ReadOnly
  kind: Managed
  storageaccounttype: Standard_LRS
provisioner: kubernetes.io/azure-disk
reclaimPolicy: Delete
volumeBindingMode: Immediate
```


## Create a StorageClass (optionally)

You can also use the default storage class.

```
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: managed-premium-retain
provisioner: kubernetes.io/azure-disk
reclaimPolicy: Retain
parameters:
  storageaccounttype: Premium_LRS
  kind: Managed
```

The reclaiming policy can be:
- Retain 
- Delete
- Recycle


### Parameters

We use here a Azure managed disk.

Premium (SSD) and LRS (Local Redundant storage) means SSD disk are replicated inside the same datacenter.

### Reclaim Policy

**Retain:**

The PVC is deleted but the PV still exists and the volume is considered "released".
The administrator can manually recalim the volume:
1. Delete the PV (the associated storage asset in external infra still exists)
2. Manually clean up data on storage asset
3. Manually delete the associated storage asset

**Delete:**

Deletion removes both PV and associated storage asset.

**Recycle:**

The Recycle reclaim policy performs a basic scrub (rm -rf /thevolume/*) on the volume and makes it available again for a new claim.

**Protection:**

kubernetes.io/pv-protection -> empeche la suppression du pvc tant que le pod n'est pas proprement supprimé.


## Create PVC

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: azure-managed-disk
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: managed-premium
  resources:
    requests:
      storage: 5Gi
```

The access modes are:

- ReadWriteOnce – the volume can be mounted as read-write by a single node
- ReadOnlyMany – the volume can be mounted read-only by many nodes
- ReadWriteMany – the volume can be mounted as read-write by many nodes

After pvc creation, pv is automatically created. 
The Volumes state goes from pending to bound to a claim.

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"PersistentVolumeClaim","metadata":{"annotations":{},"name":"azure-managed-disk","namespace":"jno"},"spec":{"accessModes":["ReadWriteOnce"],"resources":{"requests":{"storage":"5Gi"}},"storageClassName":"default"}}
    pv.kubernetes.io/bind-completed: "yes"
    pv.kubernetes.io/bound-by-controller: "yes"
    volume.beta.kubernetes.io/storage-provisioner: kubernetes.io/azure-disk
  creationTimestamp: "2019-11-05T09:32:11Z"
  finalizers:
  - kubernetes.io/pvc-protection
  name: azure-managed-disk
  namespace: jno
  resourceVersion: "6323918"
  selfLink: /api/v1/namespaces/jno/persistentvolumeclaims/azure-managed-disk
  uid: 249bebbf-ffaf-11e9-92b9-8e2741e0c46d
spec:
  accessModes:
  - ReadWriteOnce
  dataSource: null
  resources:
    requests:
      storage: 5Gi
  storageClassName: default
  volumeMode: Filesystem
  volumeName: pvc-249bebbf-ffaf-11e9-92b9-8e2741e0c46d
status:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 5Gi
  phase: Bound
  ```

## Create a deployment and mount the volume

Herafter, we create a deployment and mount the data volume into the container.
By default, the number of replica is 1.

But if you set **replicas: 3** for instance, all pods will share the same  data volume.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  selector: 
    matchLabels:
      app: my-app 
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: alpine
        command: [ "/bin/sh", "-c", "--" ]
        args: [ "while true; do sleep 30; done;" ]
        volumeMounts:
          - name: pv-data
            mountPath: /data
      volumes:
        - name: pv-data
          persistentVolumeClaim:
            claimName: azure-managed-disk
```

Let's increase the number of replicas.
In this case, all pods will access the same volume, so:
- either you update the pvc access mode from ReadWriteOnce to ReadWriteMany, 
- either you assign pods to a specific node.

Let's choose the second option:
```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  replicas: 3
  selector: 
    matchLabels:
      app: my-app 
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: alpine
        command: [ "/bin/sh", "-c", "--" ]
        args: [ "while true; do sleep 30; done;" ]
        volumeMounts:
          - name: pv-data
            mountPath: /data
      nodeSelector:
        kubernetes.io/hostname: aks-default-18766590-2
      volumes:
        - name: pv-data
          persistentVolumeClaim:
            claimName: azure-managed-disk
```

Now, we can see that the data volume is shared across the pods.
In this case, the application must be able to handle the concurrent read-write of the same file.

If you don't want to deal with that, you can use a VolumeClaimTemplate with a StatefulSet.
Kubernetes creates one PersistentVolume for each VolumeClaimTemplate. 

Each Pod will receive a single PersistentVolume with a StorageClass of my-storage-class and 1 Gib of provisioned storage. 

```
---
apiVersion: apps/v1 #  for k8s versions before 1.9.0 use apps/v1beta2  and before 1.8.0 use extensions/v1beta1
kind: StatefulSet
metadata:
  name: my-app
spec:
  replicas: 3 # by default is 1
  serviceName: "my-app-svc" # serviceName is mandatory
  selector:
    matchLabels:
      app: my-app # has to match .spec.template.metadata.labels
  template:
    metadata:
      labels:
        app: my-app # has to match .spec.selector.matchLabels
    spec:
      containers:
      - name: my-app
        image: alpine
        command: [ "/bin/sh", "-c", "--" ]
        args: [ "while true; do sleep 30; done;" ]
        volumeMounts:
          - name: pv-data
            mountPath: /data

  volumeClaimTemplates:
  - metadata:
      name: pv-data
    spec:
      accessModes:
        - ReadWriteOnce
      #storageClassName: "my-storage-class" #default storage class
      resources:
        requests:
          storage: 50Mi
```

After creation, you will have 3 pods. Each pod is associated to its own pvc.