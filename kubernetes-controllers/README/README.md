# darudcha_platform
darudcha Platform repository

ДЗ №2

*Установка kind
```
# For AMD64 / x86_64
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.19.0/kind-linux-amd64
# For ARM64
[ $(uname -m) = aarch64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.19.0/kind-linux-arm64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```
*Конфигурация локального кластера kind-config.yaml
```
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
- role: worker
```
*Запуск кластера kind:
```
rudcha@rudcha:~/darudcha_platform/kubernetes-controllers$ kind create cluster --config /home/rudcha/kind-config.yaml 
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.26.3) 🖼
 ✓ Preparing nodes 📦 📦 📦 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
 ✓ Joining worker nodes 🚜 
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Not sure what to do next? 😅  Check out https://kind.sigs.k8s.io/docs/user/quick-start/
rudcha@rudcha:~/darudcha_platform/kubernetes-controllers$ kubectl get nodes
NAME                 STATUS   ROLES           AGE   VERSION
kind-control-plane   Ready    control-plane   48s   v1.26.3
kind-worker          Ready    <none>          18s   v1.26.3
kind-worker2         Ready    <none>          17s   v1.26.3
kind-worker3         Ready    <none>          18s   v1.26.3
```
*Запустим одну реплику микросервиса frontend c исправленным манифестом - 
```
kubectl apply -f frontend-replicaset.yaml 
replicaset.apps/frontend created
rudcha@rudcha:~/darudcha_platform/kubernetes-controllers$ k9s 
rudcha@rudcha:~/darudcha_platform/kubernetes-controllers$ cat frontend-replicaset.yaml 
apiVersion: apps/v1
kind: ReplicaSet
metadata:
 name: frontend
 labels:
   app: frontend
spec:
 replicas: 1
 selector:
    matchLabels:
      app: frontend
 template:
   metadata:
     labels:
       app: frontend
   spec:
     containers:
     - name: server
       image: darudcha/frontend:latest
       ports:
         - containerPort: 8000
       env:
           - name: PRODUCT_CATALOG_SERVICE_ADDR
             value: "productcatalogservice:3550"
           - name: CURRENCY_SERVICE_ADDR
             value: "currencyservice:7000"
           - name: CART_SERVICE_ADDR
             value: "cartservice:7070"
           - name: RECOMMENDATION_SERVICE_ADDR
             value: "recommendationservice:8080"
           - name: SHIPPING_SERVICE_ADDR
             value: "shippingservice:50051"
           - name: CHECKOUT_SERVICE_ADDR
             value: "checkoutservice:5050"
           - name: AD_SERVICE_ADDR
             value: "adservice:9555"
```

* Увеличить количество реплик сервиса ad-hoc
командой - kubectl scale replicaset frontend --replicas=3 и проверяем восстановление после удаления - 
kubectl delete pods -l app=frontend | kubectl get pods -l app=frontend -w
```
rudcha@rudcha:~/darudcha_platform/kubernetes-controllers$ kubectl scale replicaset frontend --replicas=3
replicaset.apps/frontend scaled
rudcha@rudcha:~/darudcha_platform/kubernetes-controllers$ k9s 
rudcha@rudcha:~/darudcha_platform/kubernetes-controllers$ kubectl get rs frontend
NAME       DESIRED   CURRENT   READY   AGE
frontend   3         3         3       3m37s
```

* Применим новый манифест с обновленным образом v0.0.2
* Руководствуясь материалами лекции опишите произошедшую ситуацию,
почему обновление ReplicaSet не повлекло обновление запущенных pod?
```
ReplicaSet не умеет обновлять существующие Pod при изменении тэга образа. Для этих целей, нужно использовать Deployment
```
* Написание paymentservice-deployment.yaml
```
rudcha@rudcha:~/darudcha_platform/kubernetes-controllers$ cat paymentservice-deployment.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
 name: paymentservice
 labels:
   app: paymentservice
spec:
 replicas: 3
 selector:
    matchLabels:
      app: paymentservice
 template:
   metadata:
     labels:
       app: paymentservice
   spec:
     containers:
     - name: server
       image: darudcha/paymentservice:v0.0.2
       ports:
         - containerPort: 50051
       env:
         - name: PORT
           value: "50051"
         - name: DISABLE_PROFILER
           value: "1"
         - name: DISABLE_DEBUGGER
           value: "1"
```
* Обновление Deployment

```
kubectl apply -f paymentservice-deployment.yaml | kubectl get pods -l app=paymentservice -w
```
* Probes и проверка обновления приложения при некорректной работе приложения, автоатический откат приложения.
```
NAME READY STATUS RESTARTS AGE
frontend-6bf67c4974-2cns9 0/1 Running 0 10s
1
2
3
4
Events:
 Type Reason Age From Message
 ---- ------ ---- ---- -------
 Warning Unhealthy 5s (x2 over 15s) kubelet Readiness probe failed: HTTP probe
failed with statuscode: 404
```
