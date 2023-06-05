ДЗ "5"

* Создаем ветку kubernetes-security
```
rudcha@rudcha:~/darudcha_platform$ sudo git branch kubernetes-security
rudcha@rudcha:~/darudcha_platform$ sudo git switch kubernetes-security
Switched to branch 'kubernetes-security'
rudcha@rudcha:~/darudcha_platform$ ls -la
total 24
drwxr-xr-x  4 root   root   4096 мая 30 11:27 .
drwxr-xr-x 35 rudcha rudcha 4096 мая 29 22:57 ..
drwxr-xr-x  8 root   root   4096 мая 30 11:29 .git
drwxr-xr-x  4 root   root   4096 мая  7 11:17 .github
-rw-r--r--  1 root   root   1075 мая  7 11:07 LICENSE
-rw-r--r--  1 root   root     49 мая 30 11:27 README.md
rudcha@rudcha:~/darudcha_platform$
rudcha@rudcha:~/darudcha_platform$ sudo mkdir kubernetes-security
```


* Заводим таск 
* task01 - Создать Service Account bob , дать ему роль admin в рамках всего кластера, cоздать Service Account dave без доступа к кластеру
* AccountServices_bob.yaml
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: bob
```

```
rudcha@rudcha:~/darudcha_platform/kubernetes-security/task01$ kubectl apply -f ServiceAccount_bob.yaml 
serviceaccount/bob created
rudcha@rudcha:~/darudcha_platform/kubernetes-security/task01$ kubectl get sa
NAME      SECRETS   AGE
admin     0         23d
bob       0         20s
default   0         24d
```

* дать ему роль admin в рамках всего кластера
```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: bob
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: admin
subjects:
  - kind: ServiceAccount
    name: bob
    namespace: default
```

```
rudcha@rudcha:~/darudcha_platform/kubernetes-security/task01$ kubectl get clusterrolebinding | grep bob
bob                                                    ClusterRole/admin                                                                  6m12s
```

```
rudcha@rudcha:~/darudcha_platform/kubernetes-security/task01$ kubectl apply -f ServiceAccount_dave.yaml 
serviceaccount/dave created
rudcha@rudcha:~/darudcha_platform/kubernetes-security/task01$ kubectl get sa
NAME      SECRETS   AGE
bob       0         20m
dave      0         10s
default   0         24d

```

* task02, Создать namespasec - prometheus, SA - carol с правами get , list , watch.

```
apiVersion: v1
kind: Namespace
metadata:
  name: prometheus
```
rudcha@rudcha:~/darudcha_platform/kubernetes-security/task02$ sudo vim 01_namespases.yaml
rudcha@rudcha:~/darudcha_platform/kubernetes-security/task02$ kubectl apply -f 01_namespases.yaml 
namespace/prometheus created
rudcha@rudcha:~/darudcha_platform/kubernetes-security/task02$ kubectl get ns
NAME                   STATUS   AGE
default                Active   24d
ingress-nginx          Active   12d
kube-node-lease        Active   24d
kube-public            Active   24d
kube-system            Active   24d
kubernetes-dashboard   Active   23d
metallb-system         Active   12d
prometheus             Active   5s
```

```
rudcha@rudcha:~/darudcha_platform/kubernetes-security/task02$ sudo vim 02_ServiseAccount_carol.yaml
rudcha@rudcha:~/darudcha_platform/kubernetes-security/task02$ kubectl apply -f 02_ServiseAccount_carol.yaml 
serviceaccount/carol created
rudcha@rudcha:~/darudcha_platform/kubernetes-security/task02$ kubectl get sa
NAME      SECRETS   AGE
bob       0         32m
dave      0         12m
default   0         24d
rudcha@rudcha:~/darudcha_platform/kubernetes-security/task02$ kubectl get sa -n prometheus
NAME      SECRETS   AGE
carol     0         18s
default   0         4m
```
rudcha@rudcha:~/darudcha_platform/kubernetes-security/task02$ cat 03_ServiceRoleBinging_carol.yaml 
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  # "namespace" omitted since ClusterRoles are not namespaced
  name: get-watch-ls
rules:
- apiGroups: [""]
  #
  # at the HTTP level, the name of the resource for accessing Secret
  # objects is "secrets"
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```
* Роль описали выше теперь задаем доступ:

```
rudcha@rudcha:~/darudcha_platform/kubernetes-security/task02$ cat 04_ClusterRoleBinding.yaml 
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: pod-get-watch-ls-binging
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: get-watch-ls
subjects:
  - kind: Group
    name: system:serviceaccounts:prometheus
    apiGroup: rbac.authorization.k8s.io
```

```
kubectl apply -f 04_ClusterRoleBinding.yaml 
clusterrolebinding.rbac.authorization.k8s.io/pod-get-watch-ls-binging created
```

```
rudcha@rudcha:~/darudcha_platform/kubernetes-security/task02$ kubectl get clusterrole | grep get-watch-ls
get-watch-ls                                                           2023-06-05T08:53:45Z
```

* task03 - Создать Namespace dev, Создать Service Account jane в Namespace dev, Дать jane роль admin в рамках Namespace dev, Создать Service Account ken в Namespace dev, Дать ken роль view в рамках Namespace dev

```
rudcha@rudcha:~/darudcha_platform/kubernetes-security/task03$ vim 01_namespases.yaml 
rudcha@rudcha:~/darudcha_platform/kubernetes-security/task03$ sudo vim 01_namespases.yaml 
rudcha@rudcha:~/darudcha_platform/kubernetes-security/task03$ kubectl apply -f 01_namespases.yaml 
namespace/dev created
rudcha@rudcha:~/darudcha_platform/kubernetes-security/task03$ kubectl get ns
NAME                   STATUS   AGE
default                Active   28d
dev                    Active   6s
ingress-nginx          Active   16d
kube-node-lease        Active   28d
kube-public            Active   28d
kube-system            Active   28d
kubernetes-dashboard   Active   28d
metallb-system         Active   17d
prometheus             Active   4d22h
```

```
rudcha@rudcha:~/darudcha_platform/kubernetes-security/task03$ cat 02_ServiceAccount_jane.yaml 
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jane
  namespace: dev
```

```
kubectl apply -f 02_ServiceAccount_jane.yaml 
serviceaccount/jane created
rudcha@rudcha:~/darudcha_platform/kubernetes-security/task03$ kubectl get sa -n dev
NAME      SECRETS   AGE
default   0         3m30s
jane      0         1s
```

```
rudcha@rudcha:~/darudcha_platform/kubernetes-security/task03$ cat 03_ServiceRole_jane.yaml 
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev
  name: admin
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["*"]
  verbs: ["*"]
```

```
rudcha@rudcha:~/darudcha_platform/kubernetes-security/task03$ cat 04_RoleBinding_jane.yaml 
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: jane-admin
  namespace: dev
subjects:
  - kind: ServiceAccount
    name: jane
    apiGroup: ""
roleRef:
  kind: Role
  name: admin
  apiGroup: rbac.authorization.k8s.io
```


```
rudcha@rudcha:~/darudcha_platform/kubernetes-security/task03$ kubectl apply -f 05_Service_Account_ken.yaml 
serviceaccount/ken created
rudcha@rudcha:~/darudcha_platform/kubernetes-security/task03$ kubectl apply -f 06_ServiceRole_ken.yaml 
role.rbac.authorization.k8s.io/view created
rudcha@rudcha:~/darudcha_platform/kubernetes-security/task03$ kubectl apply -f 07_RoleBinding.yaml 
rolebinding.rbac.authorization.k8s.io/ken-view created
