# darudcha_platform
darudcha Platform repository
ДЗ №4

* Создаем каталог kubernetes-volumes, делаем отдельную ветку в git
```
rudcha@rudcha:~/darudcha_platform$ mkdir kubernetes-volumes
mkdir: cannot create directory ‘kubernetes-volumes’: Permission denied
rudcha@rudcha:~/darudcha_platform$ sudo mkdir kubernetes-volumes
rudcha@rudcha:~/darudcha_platform$ ls -la
total 28
drwxr-xr-x  5 root   root   4096 мая 24 12:57 .
drwxr-xr-x 35 rudcha rudcha 4096 мая 24 10:33 ..
drwxr-xr-x  8 root   root   4096 мая 24 12:56 .git
drwxr-xr-x  4 root   root   4096 мая  7 11:17 .github
drwxr-xr-x  2 root   root   4096 мая 24 12:57 kubernetes-volumes
-rw-r--r--  1 root   root   1075 мая  7 11:07 LICENSE
-rw-r--r--  1 root   root     49 мая 24 12:56 README.md
rudcha@rudcha:~/darudcha_platform$ sudo git branch kubernetes-volumes
rudcha@rudcha:~/darudcha_platform$ sudo git branch 
  kubernetes-controllers
  kubernetes-intro
  kubernetes-networks
* kubernetes-prepare
  kubernetes-volumes
  main
rudcha@rudcha:~/darudcha_platform$ sudo git switch kubernetes-volumes 
Switched to branch 'kubernetes-volumes'
rudcha@rudcha:~/darudcha_platform$ ls -la
total 28
drwxr-xr-x  5 root   root   4096 мая 24 12:57 .
drwxr-xr-x 35 rudcha rudcha 4096 мая 24 10:33 ..
drwxr-xr-x  8 root   root   4096 мая 24 12:59 .git
drwxr-xr-x  4 root   root   4096 мая  7 11:17 .github
drwxr-xr-x  2 root   root   4096 мая 24 12:57 kubernetes-volumes
-rw-r--r--  1 root   root   1075 мая  7 11:07 LICENSE
-rw-r--r--  1 root   root     49 мая 24 12:56 README.md
rudcha@rudcha:~/darudcha_platform$ cd kubernetes-volumes/
rudcha@rudcha:~/darudcha_platform/kubernetes-volumes$ ls -la
total 8
drwxr-xr-x 2 root root 4096 мая 24 12:57 .
drwxr-xr-x 5 root root 4096 мая 24 12:57 ..

```

* Запускаем кластер kind
```
rudcha@rudcha:~/darudcha_platform/kubernetes-volumes$ kind create cluster
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.26.3) 🖼
 ✓ Preparing nodes 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Have a nice day! 👋

```

* Создаем statefulset.yaml и применяем
```
rudcha@rudcha:~/darudcha_platform/kubernetes-volumes$ kubectl apply -f minio-statefulset.yaml 
statefulset.apps/minio created
rudcha@rudcha:~/darudcha_platform/kubernetes-volumes$ kubectl get pods
NAME      READY   STATUS    RESTARTS   AGE
minio-0   0/1     Pending   0          5s
rudcha@rudcha:~/darudcha_platform/kubernetes-volumes$ k9s 
rudcha@rudcha:~/darudcha_platform/kubernetes-volumes$ kubectl get pods
NAME      READY   STATUS    RESTARTS   AGE
minio-0   1/1     Running   0          28s
rudcha@rudcha:~/darudcha_platform/kubernetes-volumes$ kubectl get pvc
NAME           STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
data-minio-0   Bound    pvc-eec187c1-0515-412a-ad55-fce4297b9248   10Gi       RWO            standard       32s
```

* Проверка работы MinIO
```
rudcha@rudcha:~/darudcha_platform/kubernetes-volumes$ sudo vim minio-headlessservice.yaml 
rudcha@rudcha:~/darudcha_platform/kubernetes-volumes$ kubectl apply -f minio-headlessservice.yaml 
service/minio created
rudcha@rudcha:~/darudcha_platform/kubernetes-volumes$ kubectl get statefulsets
NAME    READY   AGE
minio   1/1     4m7s
rudcha@rudcha:~/darudcha_platform/kubernetes-volumes$ kubectl get pods
NAME      READY   STATUS    RESTARTS   AGE
minio-0   1/1     Running   0          4m14s
rudcha@rudcha:~/darudcha_platform/kubernetes-volumes$ kubectl get pvc
NAME           STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
data-minio-0   Bound    pvc-eec187c1-0515-412a-ad55-fce4297b9248   10Gi       RWO            standard       4m22s
rudcha@rudcha:~/darudcha_platform/kubernetes-volumes$ kubectl describe pvc data-minio-0
Name:          data-minio-0
Namespace:     default
StorageClass:  standard
Status:        Bound
Volume:        pvc-eec187c1-0515-412a-ad55-fce4297b9248
Labels:        app=minio
Annotations:   pv.kubernetes.io/bind-completed: yes
               pv.kubernetes.io/bound-by-controller: yes
               volume.beta.kubernetes.io/storage-provisioner: rancher.io/local-path
               volume.kubernetes.io/selected-node: kind-control-plane
               volume.kubernetes.io/storage-provisioner: rancher.io/local-path
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      10Gi
Access Modes:  RWO
VolumeMode:    Filesystem
Used By:       minio-0
Events:
  Type    Reason                 Age                From                                                                                                Message
  ----    ------                 ----               ----                                                                                                -------
  Normal  WaitForFirstConsumer   5m3s               persistentvolume-controller                                                                         waiting for first consumer to be created before binding
  Normal  Provisioning           5m3s               rancher.io/local-path_local-path-provisioner-75f5b54ffd-dmzp5_2c97dae1-8757-4ac6-8adf-0d626f5be4a6  External provisioner is provisioning volume for claim "default/data-minio-0"
  Normal  ExternalProvisioning   5m (x2 over 5m3s)  persistentvolume-controller                                                                         waiting for a volume to be created, either by external provisioner "rancher.io/local-path" or manually created by system administrator
  Normal  ProvisioningSucceeded  4m57s              rancher.io/local-path_local-path-provisioner-75f5b54ffd-dmzp5_2c97dae1-8757-4ac6-8adf-0d626f5be4a6  Successfully provisioned volume pvc-eec187c1-0515-412a-ad55-fce4297b9248
```
