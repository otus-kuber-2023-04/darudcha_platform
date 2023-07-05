ДЗ №7

* Создадим директорию kubernetes-operators/deploy
* Cоздадим CustomResource deploy/cr.yml
```
rudcha@rudcha:~/darudcha_platform/kubernetes-operators$ cat cr.yml 
apiVersion: otus.homework/v1
kind: MySQL
metadata:
 name: mysql-instance
spec:
 image: mysql:5.7
 database: otus-database
 password: otuspassword # Так делать не нужно, следует использовать secret
 storage_size: 1Gi
usless_data: "useless info"
rudcha@rudcha:~/darudcha_platform/kubernetes-operators$ kubectl apply -f deploy/cr.yml
error: resource mapping not found for name: "mysql-instance" namespace: "" from "deploy/cr.yml": no matches for kind "MySQL" in version "otus.homework/v1"
ensure CRDs are installed first
```
* Создадим CRD deploy/crd.yml gist
```
rudcha@rudcha:~/darudcha_platform/kubernetes-operators$ cat deploy/crd.yml 
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: mysqls.otus.homework # имя CRD должно иметь формат plural.group
spec:
  scope: Namespaced # Данный CRD будер работать в рамках namespace
  group: otus.homework # Группа, отражается в поле apiVersion CR
  versions: # Список версий
    - name: v1
      served: true # Будет ли обслуживаться API-сервером данная версия
      storage: true # Версия описания, которая будет сохраняться в etcd
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                cronSpec:
                  type: string
                image:
                  type: string
                replicas:
                  type: integer
  names: # различные форматы имени объекта CR
    kind: MySQL # kind CR
    plural: mysqls
    singular: mysql
    shortNames:
      - ms
```
* Создадим CRD и Cоздаем CR.
```
rudcha@rudcha:~/darudcha_platform/kubernetes-operators$ kubectl apply -f deploy/crd.yml 
customresourcedefinition.apiextensions.k8s.io/mysqls.otus.homework created
rudcha@rudcha:~/darudcha_platform/kubernetes-operators$ kubectl apply -f deploy/cr.yml
mysql.otus.homework/mysql-instance created
rudcha@rudcha:~/darudcha_platform/kubernetes-operators$ kubectl get mysqls.otus.homework
NAME             AGE
mysql-instance   7s
rudcha@rudcha:~/darudcha_platform/kubernetes-operators$ kubectl delete mysqls.otus.homework mysql-instance
mysql.otus.homework "mysql-instance" deleted
```

* Добавим в спецификацию CRD ( spec ) параметры validation 
```
schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              description: MySQLSpec defines the desired state of MySQL
              properties:
                # свойства спецификации кластера (.spec)
                image:
                  description: image for MySQL
                  type: string
                database:
                  description: MySQL database name
                  type: string
                password:
                  description: MySQL admin password
                  type: string
                storage_size:
                  description: MySQL storage size
                  type: string
              required:
              - image
              - database
              - password
              - storage_size
              type: object
            # назначение ключа объекта yaml .status (пустой объект)
            status:
              description: MySQLStatus defines the observed state of MySQL
              type: object
          type: object
```

* В папке kubernetes-operators/build создайте файл mysqloperator.py и в дирректории kubernetes-operators/build/templates создайте шаблоны.
```
rudcha@rudcha:~/darudcha_platform/kubernetes-operators/build$ kopf run mysqloperator.py
/usr/local/lib/python3.10/dist-packages/kopf/_core/reactor/running.py:179: FutureWarning: Absence of either namespaces or cluster-wide flag will become an error soon. For now, switching to the cluster-wide mode for backward compatibility.
  warnings.warn("Absence of either namespaces or cluster-wide flag will become an error soon."
[2023-06-14 12:47:33,513] kopf._core.engines.a [INFO    ] Initial authentication has been initiated.
[2023-06-14 12:47:33,517] kopf.activities.auth [INFO    ] Activity 'login_via_pykube' succeeded.
[2023-06-14 12:47:33,519] kopf.activities.auth [INFO    ] Activity 'login_via_client' succeeded.
[2023-06-14 12:47:33,519] kopf._core.engines.a [INFO    ] Initial authentication has finished.
/home/rudcha/darudcha_platform/kubernetes-operators/build/mysqloperator.py:14: YAMLLoadWarning: calling yaml.load() without Loader=... is deprecated, as the default Loader is unsafe. Please read https://msg.pyyaml.org/load for full details.
  json_manifest = yaml.load(yaml_manifest)
[2023-06-14 12:47:33,698] kopf.objects         [INFO    ] [default/mysql-instance] Handler 'mysql_on_create' succeeded.
[2023-06-14 12:47:33,698] kopf.objects         [INFO    ] [default/mysql-instance] Creation is processed: 1 succeeded; 0 failed.
```

* Удалим все ресурсы, созданные контроллером:
```
rudcha@rudcha:~/darudcha_platform/kubernetes-operators/build$ k9s 
rudcha@rudcha:~/darudcha_platform/kubernetes-operators/build$ kubectl delete mysqls.otus.homework mysql-instance
mysql.otus.homework "mysql-instance" deleted
rudcha@rudcha:~/darudcha_platform/kubernetes-operators/build$ kubectl delete deployments.apps mysql-instance
deployment.apps "mysql-instance" deleted
rudcha@rudcha:~/darudcha_platform/kubernetes-operators/build$ kubectl delete pvc mysql-instance-pvc
persistentvolumeclaim "mysql-instance-pvc" deleted
rudcha@rudcha:~/darudcha_platform/kubernetes-operators/build$ kubectl delete pv mysql-instance-pv
persistentvolume "mysql-instance-pv" deleted
rudcha@rudcha:~/darudcha_platform/kubernetes-operators/build$ kubectl delete svc mysql-instance
service "mysql-instance" deleted
rudcha@rudcha:~/darudcha_platform/kubernetes-operators/build$
```

* Далее, я использую kind cluster и деплоим kopf run mysql-operator.py
```
rudcha@rudcha:~/darudcha_platform/kubernetes-operators/build/templates$ kubectl get pvc
NAME                        STATUS   VOLUME                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
backup-mysql-instance-pvc   Bound    backup-mysql-instance-pv   1Gi        RWO                           3m20s
mysql-instance-pvc          Bound    mysql-instance-pv          2Gi        RWO                           3m44s
```
```
rudcha@rudcha:~/darudcha_platform/kubernetes-operators/build/templates$ kubectl exec -it $MYSQLPOD -- mysql -potuspassword -e "select * from test;" otus-database
mysql: [Warning] Using a password on the command line interface can be insecure.
+----+-------------+
| id | name        |
+----+-------------+
|  1 | some data   |
|  2 | some data-2 |
+----+-------------+
```
