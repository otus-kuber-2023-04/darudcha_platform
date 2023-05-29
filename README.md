# darudcha_platform
darudcha Platform repository

## Домашние задания 1

* Разберитесь почему все pod в namespace kube-system восстановились после удаления.

а) kubelet запущен как сервис systemd среди прочих.

```
|docker@minikube:~$ systemctl kubelet status
|Unknown operation kubelet.
|docker@minikube:~$ systemctl status kubelet
|● kubelet.service - kubelet: The Kubernetes Node Agent
|     Loaded: loaded (/lib/systemd/system/kubelet.service; disabled; vendor preset: enabled)
|    Drop-In: /etc/systemd/system/kubelet.service.d
|             └─10-kubeadm.conf
|     Active: active (running) since Wed 2023-05-10 07:34:33 UTC; 18min ago
|       Docs: http://kubernetes.io/docs/
|   Main PID: 1452 (kubelet)
|      Tasks: 16 (limit: 18803)
|     Memory: 123.7M
|     CGroup: /docker/5ddd29970f5880fc95384249a6d9033e28461e917d11ad0563c6e4ffb2767941/system.slice/kubelet.service
|             └─1452 /var/lib/minikube/binaries/v1.26.3/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf 
|--config=/var/lib/kubelet/config.yaml --container-runtime-endpoint=unix:///var/run/cri-dockerd.sock --hostname-override=min
|ikube --kubeconfig=/etc/kubernetes/kubelet.conf --node-ip=192.168.49.2
|
|docker@minikube:~$ systemctl --type=service --state=active
|  UNIT                               LOAD   ACTIVE SUB     DESCRIPTION        
|                                      
|  containerd.service                 loaded active running containerd container runtime                             
|  cri-docker.service                 loaded active running CRI Interface for Docker Application Container Engine    
|  dbus.service                       loaded active running D-Bus System Message Bus                                 
|  docker.service                     loaded active running Docker Application Container Engine                      
|  kmod-static-nodes.service          loaded active exited  Create list of static device nodes for the current kernel
|  kubelet.service                    loaded active running kubelet: The Kubernetes Node Agent                       
|  minikube-automount.service         loaded active exited  minikube automount                                       
|  ssh.service                        loaded active running OpenBSD Secure Shell server                              
|  systemd-journal-flush.service      loaded active exited  Flush Journal to Persistent Storage                      
|  systemd-journald.service           loaded active running Journal Service                                          
|  systemd-remount-fs.service         loaded active exited  Remount Root and Kernel File Systems                     
|  systemd-sysctl.service             loaded active exited  Apply Kernel Variables                                   
|  systemd-sysusers.service           loaded active exited  Create System Users                                      
|  systemd-tmpfiles-setup-dev.service loaded active exited  Create Static Device Nodes in /dev                       
|  systemd-update-utmp.service        loaded active exited  Update UTMP about System Boot/Shutdown                   
```

б) core-dns - в Deployment с параметром replicas. ReplicaSet восстанавливает его работу
kube-proxy - управляется и создается Daemonset.

* Dockerfile
```
cat Dockerfile 
FROM nginx:alpine
RUN  mkdir /app && touch /app/work.html && chown -R 1001:1001 /app \
     && echo '<html>\n\t<body>\n\t\t<h1>Hello World NGINX!</h1>\n\t</body>\n</html>' > /app/work.html \
     && cp /app/work.html /usr/share/nginx/html
COPY /app/work.html /usr/share/nginx/html
EXPOSE 80/tcp
WORKDIR /usr/share/nginx/html
VOLUME /app
CMD ["/usr/sbin/nginx", "-g", "daemon off;"]
RUN chown -R 1001:1001 /usr/share/nginx
RUN chown -R 1001:1001 /var/log/nginx
RUN touch /run/nginx.pid
RUN chown -R 1001:1001 /run/nginx.pid
RUN chown -R 1001:1001 /var/cache/nginx
RUN chown -R 1001:1001 /etc/nginx
USER 1001
```

* Собрал образ, запусил, проверил и запушил - 
```
docker push darudcha/nginx:latest
The push refers to repository [docker.io/darudcha/nginx]
257b596f12fc: Pushed 
82cb2f94f08a: Pushed 
061c1cb798fd: Pushed 
78e681efa2ec: Pushed 
8ed406239a63: Pushed 
d62989859515: Pushed 
5f70bf18a086: Pushed 
b10c6a9d0977: Pushed 
31531248c7cb: Pushed 
f9cb3f1f1d3d: Pushed 
f0fb842dea41: Pushed 
c1cd5c8c68ef: Pushed 
1d54586a1706: Pushed 
1003ff723696: Pushed 
f1417ff83b31: Pushed 
latest: digest: sha256:cee4ccc8db381c45a9386b6b10e1e6dac52192ed10c79a744f4f847a59a44c0a size: 3437
```

* Манифест
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
   containers:
   - name: nginx
     image: darudcha/nginx:latest
     volumeMounts:
     - name: app
       mountPath: /usr/share/nginx/html
     ports:
     - containerPort: 8000
   initContainers:
   - name: index-html
     image: busybox:1.31.0
/#     command: ['sh', '-c', 'wget -O- https://tinyurl.com/otus-k8s-intro | sh']
     command: ['sh', '-c', 'wget -O- https://raw.githubusercontent.com/express42/otus-platform-snippets/master/Module-02/Introduction-to-Kubernetes/wget.sh | sh']
     volumeMounts:
     - name: app
       mountPath: /app
   volumes:
     - name: app
       emptyDir: {}
```

* Проброс портов и тест
```
kubectl port-forward --address 0.0.0.0 pod/web 8000:80

curl http://0.0.0.0:8000/work.html
<html>\n\t<body>\n\t\t<h1>Hello World NGINX!</h1>\n\t</body>\n</html>
```

Init отработал - 
```
2023-05-10T11:49:32.486220135Z Connecting to raw.githubusercontent.com (185.199.111.133:443)                            │
│ 2023-05-10T11:49:32.529996773Z wget: note: TLS certificate validation not implemented                                   │
│ 2023-05-10T11:49:32.697254206Z writing to stdout                                                                        │
│ 2023-05-10T11:49:32.744308199Z -                    100% |********************************| 79785  0:00:00 ETA          │
│ 2023-05-10T11:49:32.744333698Z written to stdout  
```
