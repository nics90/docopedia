![](https://miro.medium.com/max/875/0*YAMvBnBHhf7hTqy2.jpg)

credits Me and Open source

**Persistent Volumes**

A PersistentVolume (PV) is a storage resource in the cluster that has been provisioned by an administrator or dynamically provisioned using Storage Classes.

**Static Provisioning:**

A cluster administrator creates several PVs. They carry the details of the real storage, which is available for use by cluster users.

**awsElasticBlockStore:**

Before you can use an EBS volume with a Pod, you need to create it.

```
aws ec2 create-volume \--availability-zone=eu-west-1a \--size=100 \--volume-type=gp2PersistentVolume spec:
```

Here,

```
--- apiVersion: v1gcePersistentDisk: ~kind: PersistentVolumemetadata:   name: test-volumespec:   accessModes:     - ReadWriteOnce  awsElasticBlockStore:     fsType: ext4    volumeID: ~  capacity:     storage: 100Gi  storageClassName: ebs-disk
```

Before creating a PersistentVolume, you must create the PD.

```
gcloud beta compute disks create --size=200GB my-data-disk \--region us-central1 \ --replica-zones us-central1-a,us-central1-b
```

**PersistentVolume spec:**

```
--- apiVersion: v1kind: PersistentVolumemetadata:   name: test-volumespec:   accessModes:     - ReadWriteOnce  capacity:     storage: 200Gi  gcePersistentDisk:     fsType: ext4    pdName: my-data-disk  storageClassName: gcp-disk
```

Check persistent Volumes

```
kubectl get pvNAME CAPACITY ACCESS MODES RECLAIM POLICY STATUS CLAIM STORAGECLASS REASON AGEtest-volume 200Gi RWO Delete Available gcp-disk 6s
```

**azureDisk:**

Before creating a PersistentVolume, you must create a virtual disk in Azure.

PersistentVolume spec:

```
--- apiVersion: v1kind: PersistentVolumemetadata:   name: test-volumespec:   accessModes:     - ReadWriteOnce  azureDisk:     diskName: test.vhd    diskURI: "https://someaccount.blob.microsoft.net/vhds/test.vhd"  capacity:     storage: 500Gi  storageClassName: azure-disk
```

**azureFile**:

You will need to create a Kubernetes secret that holds both the account name and key.

```
kubectl create secret generic azure-secret \  — from-literal=azurestorageaccountname=< … > \  — from-literal=azurestorageaccountkey=< … >
```

Before creating a PersistentVolume, create Azure Files share.

**PersistentVolume spec:**

```
--- apiVersion: v1kind: PersistentVolumemetadata:   name: sample-storagespec:   accessModes:     - ReadWriteMany  azureFile:     readOnly: false    secretName: azure-secret    shareName: k8stest  capacity:     storage: 10Gi  persistentVolumeReclaimPolicy: Retain  storageClassName: azure-file-share
```

**NFS:**

Before creating a PersistentVolume, You will need NFS server details.

PersistentVolume spec:

```
--- apiVersion: v1kind: PersistentVolumemetadata:   name: nfsspec:   accessModes:     - ReadWriteMany  capacity:     storage: 1Mi  nfs:     path: /    server: nfs-server.mydomain.com  storageClassName: nfs
```

**Dynamic Provisioning:**

When none of the static PVs match a user’s PersistentVolumeClaim, the cluster may try to dynamically provision a volume, especially for the PVC.

This provisioning is based on StorageClasses, the PVC must request a storage class and the administrator must have created and configured that class for dynamic provisioning to occur.

**StorageClasses:**

Volume implementations are configured through StorageClass resources.

If you set up a Kubernetes cluster on GCP, AWS, Azure, or any other cloud platform, a default StorageClass creates for you which uses the standard persistent disk type.

**List storage class:**

**AWS:**

```
kubectl get storageclassNAME PROVISIONER AGEdefault (default) kubernetes.io/aws-ebs 3d
```

**GCP:**

```
kubectl get storageclassNAME PROVISIONER AGEstandard (default) kubernetes.io/gce-pd 3d
```

**StorageClass Configuration:**

```
--- apiVersion: storage.k8s.io/v1kind: StorageClassmetadata:   name: standardprovisioner: kubernetes.io/aws-ebsreclaimPolicy: RetainvolumeBindingMode: Immediate
```

**Capacity:**

Generally, a PV will have a specific storage capacity. This is set using the PV’s capacity attribute.

Currently, storage size is the only resource that can be set or requested.

**Provisioner**:

Storage classes have a provisioner that determines what volume plugin is used for provisioning PVs.

**Reclaim Policy:**

It can be either Delete or Retain. Default is Delete.

**Volume Binding Mode:**

The volumeBindingMode field controls when volume binding and dynamic provisioning should occur. Immediate is default and specifying the WaitForFirstConsumer mode.

The following plugins support Wait For First Consumer with dynamic provisioning:

1.  AWSElasticBlockStore
2.  GCEPersistentDisk
3.  AzureDisk

**Access Modes:**

PersistentVolumes support the following access modes:

1.  ReadWriteOnce: The Volume can be mounted as a read-write by a single node.
2.  ReadOnlyMany: The Volume can be mounted read-only by many nodes.
3.  ReadWriteMany: The Volume can be mounted as a read-write by many nodes.

PersistentVolumes that are backed by Compute Engine persistent disks don’t support this access mode.

**Persistent Volume Claims**

A persistent volume claim (PVC) is a request for storage by a user from a PV. Claims can request specific size and access modes (e.g: they can be mounted once read/write or many times read-only).

If none of the static persistent volumes match the user’s PVC request, the cluster may attempt to dynamically create a PV that matches the PVC request based on storage class.

List PVs:

```
kubectl get pvNAME         CAPACITY  ACCESS MODES  RECLAIM POLICY  STATUS     CLAIM  STORAGECLASS  REASON  AGEtest-volume  200Gi     RWO           Delete          Available         gcp-disk              6s
```

Persistent volume claim manifest:

```
--- apiVersion: v1kind: PersistentVolumeClaimmetadata:   name: test-pvcspec:   accessModes:     - ReadWriteOnce  resources:     requests:       storage: 200Gi  storageClassName: gcp-disk
```

Create PVC:

```
kubectl create -f test-pvc.yamlpersistentvolumeclaim/test-pvc created
```

List PVCs:

```
kubectl get pvcNAME       STATUS   VOLUME        CAPACITY   ACCESS MODES   STORAGECLASS   AGEtest-pvc   Bound    test-volume   200Gi      RWO            gcp-disk       7s
```

Use PVC to a Pod:

```
--- apiVersion: v1kind: Podmetadata:   labels:     name: webserver  name: nginx-webserverspec:   containers:     -       image: nginx      name: webserver      ports:         -           containerPort: 80          name: http      volumeMounts:         -           mountPath: /usr/local/nginx/html          name: app-data  volumes:     -       name: app-data      persistentVolumeClaim:         claimName: test-pvc
```

Create PVC without a static PV:

You can create a PVC based on storage class specifications. If you omit the storage class, it will use the default storage class.

```
kubectl get storageclassNAME                 PROVISIONER            AGEstandard (default)   kubernetes.io/gce-pd   29m
```

Then,

```
--- apiVersion: v1kind: PersistentVolumeClaimmetadata:   name: wordpress-pvcspec:   accessModes:     - ReadWriteOnce  resources:     requests:       storage: 300Gi  storageClassName: standard
```

and check

```
kubectl apply -f wordpress-pvc.yamlpersistentvolumeclaim/wordpress-pvc created
```

List dynamically created PVC and PV:

```
kubectl get pvc wordpress-pvcNAME            STATUS  VOLUME                                    CAPACITY  ACCESS MODES  STORAGECLASS  AGEwordpress-pvc   Bound   pvc-325160ee-fb3a-11e9-903e-42010a800149  300Gi     RWO           standard      19skubectl get pvNAME                                      CAPACITY  ACCESS MODES  RECLAIM POLICY  STATUS  CLAIM                 STORAGECLASS  REASON  AGEpvc-325160ee-fb3a-11e9-903e-42010a800149  300Gi     RWO           Delete          Bound   default/wordpress-pvc standard              43s
```

WordPress deployment with PVC:

```
--- apiVersion: apps/v1kind: Deploymentmetadata:   labels:     app: wordpress  name: wordpressspec:   selector:     matchLabels:       app: wordpress  template:     metadata:       labels:         app: wordpress    spec:       containers:         -           env:             -               name: WORDPRESS_DB_HOST              value: mysql            -               name: WORDPRESS_DB_PASSWORD              valueFrom:                 secretKeyRef:                   key: password                  name: mysql-pass          image: "wordpress:latest"          name: wordpress          ports:             -               containerPort: 80              name: wordpress          volumeMounts:             -               mountPath: /var/www/html              name: wordpress-persistent-storage      volumes:         -           name: wordpress-persistent-storage          persistentVolumeClaim:             claimName: wordpress-pvc
```

**Volume Claim Template**

Volume claim templates are a list of claims that pods are allowed to reference. The StatefulSet controller is responsible for mapping network identities to claims in a way that maintains the identity of a pod.

Every claim in this list must have at least one matching (by name) volumeMount in one container in the template.

You can also define storage class to leverage dynamic provisioning of persistent volumes so you won’t have to create them manually.

```
--- volumeClaimTemplates:   -     metadata:       name: data    spec:       accessModes:         - ReadWriteOnce      resources:         requests:           storage: 1Gi      storageClassName: standard
```

WordPress StatefulSet with volume claim templates:

```
--- apiVersion: apps/v1kind: StatefulSetmetadata:   labels:     app: wordpress  name: wordpressspec:   replicas: 3  selector:     matchLabels:       app: wordpress  serviceName: wordpress  template:     metadata:       labels:         app: wordpress    spec:       containers:         -           env:             -               name: WORDPRESS_DB_HOST              value: mysql            -               name: WORDPRESS_DB_PASSWORD              valueFrom:                 secretKeyRef:                   key: password                  name: mysql-pass          image: "wordpress:latest"          name: wordpress          ports:             -               containerPort: 80              name: wordpress          volumeMounts:             -               mountPath: /var/www/html              name: wordpress-data  volumeClaimTemplates:     -       metadata:         name: wordpress-data      spec:         accessModes:           - ReadWriteOnce        resources:           requests:             storage: 10Gi        storageClassName: standard
```

For Reference

![](https://miro.medium.com/max/875/0*cRMUa8dBX1dHpj5L)

👋 [**Join us today**](https://avmconsulting.net/) **!!**

️Follow us on [LinkedIn](https://www.linkedin.com/company/avm-consulting-inc/), [Twitter](https://twitter.com/AvmConsulting), [Facebook](https://www.facebook.com/AVM-Consulting-Inc-114225960307418), and [Instagram](https://www.instagram.com/avmconsulting_inc/)

![](https://miro.medium.com/max/875/1*UOzO1SzC0rQg-0hBwZjFDA.png)

[https://avmconsulting.net/](https://avmconsulting.net/)

If this post was helpful, please click the clap 👏 button below a few times to show your support! ⬇