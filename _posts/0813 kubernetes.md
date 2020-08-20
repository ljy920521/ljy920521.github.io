# 8/13 kubernetes 

---

job (교재 158p)

mynapp-job

```yaml
  1 apiVersion: batch/v1
  2 kind: Job
  3 metadata:
  4   name: mynapp-job
  5 spec:
  6   completions: 3
  7   template:
  8     metadata:
  9       labels:
 10         app: mynapp-job
 11     spec:
 12       restartPolicy: OnFailure
 13       containers:
 14       - name: mynapp
 15         image: busybox
 16         command: ["sleep", "60"]
```

```cmd
vagrant@kube-master1:~/kubernetes$ kubectl get all
NAME                   READY   STATUS      RESTARTS   AGE
pod/mynapp-job-67cbv   0/1     Completed   0          3m43s
pod/mynapp-job-j8m9d   0/1     Completed   0          88s
pod/mynapp-job-l8r94   0/1     Completed   0          2m35s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.233.0.1   <none>        443/TCP   16h

NAME                   COMPLETIONS   DURATION   AGE
job.batch/mynapp-job   3/3           3m23s      3m43s
```



병렬실행

```yaml
  ...
  6   completions: 3
  7   parallelism: 3
```

```cmd
vagrant@kube-master1:~/kubernetes$ kubectl get all
NAME                   READY   STATUS    RESTARTS   AGE
pod/mynapp-job-7t7rv   1/1     Running   0          64s
pod/mynapp-job-9rfr6   1/1     Running   0          64s
pod/mynapp-job-swz6h   1/1     Running   0          64s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.233.0.1   <none>        443/TCP   16h

NAME                   COMPLETIONS   DURATION   AGE
job.batch/mynapp-job   0/3           64s        64s
```



cronjob

```yaml
  1 apiVersion: batch/v1beta1
  2 kind: CronJob
  3 metadata:
  4   name: mynapp-cronjob
  5 spec:
  6   schedule: "*/1 * * * *"
  7   jobTemplate:
  8     spec:
  9       template:
 10         metadata:
 11           labels:
 12             app: mynapp-cronjob
 13         spec:
 14           restartPolicy: OnFailure
 15           containers:
 16           - name: mynapp
 17             image: busybox
 18             command: ["sleep", "60"]
```

```cmd
vagrant@kube-master1:~/kubernetes$ kubectl get all
NAME                                  READY   STATUS              RESTARTS   AGE
pod/mynapp-cronjob-1597284360-2v85s   1/1     Running             0          61s
pod/mynapp-cronjob-1597284420-t2hbm   0/1     ContainerCreating   0          1s
pod/mynapp-job-7t7rv                  0/1     Completed           0          13m
pod/mynapp-job-9rfr6                  0/1     Completed           0          13m
pod/mynapp-job-swz6h                  0/1     Completed           0          13m

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.233.0.1   <none>        443/TCP   17h

NAME                                  COMPLETIONS   DURATION   AGE
job.batch/mynapp-cronjob-1597284360   0/1           61s        61s
job.batch/mynapp-cronjob-1597284420   0/1           1s         1s
job.batch/mynapp-job                  3/3           66s        13m

NAME                           SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
cronjob.batch/mynapp-cronjob   */1 * * * *   False     2        10s             2m7s
```

```cmd
vagrant@kube-master1:~/kubernetes$ kubectl delete cronjobs.batch mynapp-cronjob
cronjob.batch "mynapp-cronjob" deleted
vagrant@kube-master1:~/kubernetes$ kubectl delete jobs.batch mynapp-job
job.batch "mynapp-job" deleted
```



클러스터 내부 서비스

mynapp-svc.yml

```yaml
  1 apiVersion: v1
  2 kind: Service
  3 metadata:
  4   name: mynapp-svc
  5 spec:
  6   ports:
  7   - port: 80
  8     targetPort: 8080
  9   selector:
 10     app: mynapp-rs
```

mynapp-rs.yml

```yaml
  1 apiVersion: apps/v1
  2 kind: ReplicaSet
  3 metadata:
  4   name: mynapp-rs
  5 spec:
  6   replicas: 3
  7   selector:
  8     matchLabels:
  9       app: mynapp-rs
 10 
 11   template:
 12     metadata:
 13       labels:
 14         app: mynapp-rs
 15 
 16     spec:
 17       containers:
 18       - name: mynapp
 19         image: c1t1d0s7/myweb
 20         ports:
 21         - containerPort: 8080
```

```cmd
vagrant@kube-master1:~/kubernetes$ kubectl create -f mynapp-svc.yml
service/mynapp-svc created
vagrant@kube-master1:~/kubernetes$ kubectl get endpoints
NAME         ENDPOINTS            AGE
kubernetes   192.168.56.11:6443   17h
mynapp-svc   <none>               4m30s

vagrant@kube-master1:~/kubernetes$ kubectl get endpoints
NAME         ENDPOINTS                                              AGE
kubernetes   192.168.56.11:6443                                     17h
mynapp-svc   10.233.101.8:8080,10.233.103.6:8080,10.233.76.7:8080   24m
(ENDPOINTS 생성까지 시간 조금 걸림)

vagrant@kube-master1:~/kubernetes$ kubectl get pods -o wide
NAME              READY   STATUS    RESTARTS   AGE   IP             NODE         NOMINATED NODE   READINESS GATES
mynapp-rs-b6h4j   1/1     Running   0          19m   10.233.76.7    kube-node3   <none>           <none>
mynapp-rs-dl5nc   1/1     Running   0          19m   10.233.103.6   kube-node2   <none>           <none>
mynapp-rs-fxvp8   1/1     Running   0          19m   10.233.101.8   kube-node1   <none>           <none>
```



생성한 서비스 접근 테스트

``` 
vagrant@kube-master1:~/kubernetes$ kubectl run nettool -it --image=c1t1d0s7/network-multitool --generator=run-pod/v1 --rm=true bash
If you don't see a command prompt, try pressing enter.
bash-5.0# ip a s
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: tunl0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN group default qlen 1000
    link/ipip 0.0.0.0 brd 0.0.0.0
4: eth0@if16: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 06:68:4e:76:3d:5e brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.233.76.8/32 scope global eth0
       valid_lft forever preferred_lft forever
```

- bash 나오면 서비스도 종료됨

  

mynapp-svc-affinity.yml

```yaml
  1 apiVersion: v1
  2 kind: Service
  3 metadata:
  4   name: mynapp-svc-affinity
  5 spec:
  6   sessionAffinity: ClientIP
  7   ports:
  8   - port: 80
  9     targetPort: 8080
 10   selector:
 11     app: mynapp-rs
~                         
```



affinity

```yaml
vagrant@kube-master1:~/kubernetes$ kubectl create -f mynapp-svc-affinity.yml 
service/mynapp-svc-affinity created

vagrant@kube-master1:~/kubernetes$ kubectl get service
NAME                  TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
kubernetes            ClusterIP   10.233.0.1     <none>        443/TCP   18h
mynapp-svc            ClusterIP   10.233.28.6    <none>        80/TCP    45m
mynapp-svc-affinity   ClusterIP   10.233.51.77   <none>        80/TCP    7s

vagrant@kube-master1:~/kubernetes$ kubectl get service mynapp-svc-affinity -o yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: "2020-08-13T03:06:46Z"
  name: mynapp-svc-affinity
  namespace: default
  resourceVersion: "23404"
  selfLink: /api/v1/namespaces/default/services/mynapp-svc-affinity
  uid: 84883a45-c587-4472-9b9a-d9a0e41d9734
spec:
  clusterIP: 10.233.51.77
  ports:
  - port: 80
    protocol: TCP
    targetPort: 8080
  selector:
    app: mynapp-rs
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 10800
  type: ClusterIP
status:
  loadBalancer: {}
  
  
# Test
vagrant@kube-master1:~/kubernetes$ kubectl run nettool -it --image=c1t1d0s7/network-multitool --generator=run-pod/v1 --rm=true bash
If you dont see a command prompt, try pressing enter.
bash-5.0# curl http://10.233.51.77
Message: Hello World!
Hostname: mynapp-rs-b6h4j
Platform: linux
Uptime: 10329
IP: 10.233.76.7
DNS: 169.254.25.10
bash-5.0# 
```



멀티포트

```yaml
  1 apiVersion: v1
  2 kind: Service
  3 metadata:
  4   name: mynapp-svc-mutliport
  5 spec:
  6   ports:
  7   - name: mynapp-http
  8     port: 80
  9     targetPort: 8080
 10   - name: mynapp-https
 11     port: 443
 12     targetPort: 8443
 13     # 실제로는 https 서비스 안하고있어서 확인은 안됨(Connection refused 출력될 것임)
 14   selector:
 15     app: mynapp-rs
```

```cmd
vagrant@kube-master1:~/kubernetes$ kubectl create -f mynapp-svc-mutilport.yml service/mynapp-svc-mutliport created
vagrant@kube-master1:~/kubernetes$ kubectl get service
NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
kubernetes             ClusterIP   10.233.0.1      <none>        443/TCP          18h
mynapp-svc             ClusterIP   10.233.28.6     <none>        80/TCP           54m
mynapp-svc-affinity    ClusterIP   10.233.51.77    <none>        80/TCP           8m53s
mynapp-svc-mutliport   ClusterIP   10.233.43.218   <none>        80/TCP,443/TCP   6s
vagrant@kube-master1:~/kubernetes$ kubectl get endpoints
NAME                   ENDPOINTS                                                          AGE
kubernetes             192.168.56.11:6443                                                 18h
mynapp-svc             10.233.101.8:8080,10.233.103.6:8080,10.233.76.7:8080               54m
mynapp-svc-affinity    10.233.101.8:8080,10.233.103.6:8080,10.233.76.7:8080               8m58s
mynapp-svc-mutliport   10.233.101.8:8443,10.233.103.6:8443,10.233.76.7:8443 + 3 more...   11s
```



포트 이름 부여

mynapp-rs-named-port.yml

mynapp-svc-named-port.yml

```cmd
vagrant@kube-master1:~/kubernetes$ kubectl create -f mynapp-rs-named-port.yml -f mynapp-svc-named-port.yml 
replicaset.apps/mynapp-rs-named-port created
service/mynapp-svc-named-port created

vagrant@kube-master1:~/kubernetes$ kubectl get service
NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
mynapp-svc-named-port   ClusterIP   10.233.23.213   <none>        80/TCP           6s

vagrant@kube-master1:~/kubernetes$ kubectl get endpoints
NAME                    ENDPOINTS                                                          AGE
mynapp-svc-named-port   10.233.101.9:8080,10.233.103.7:8080,10.233.76.11:8080              21s
```

- targetPort에 포트번호가 아닌 이름을 지정했고, endpoint 확인하면 8080포트를 참조함을 볼 수 있다.



포트 변경 및 확인

```cmd
vagrant@kube-master1:~/kubernetes$ kubectl edit replicasets.apps mynapp-rs-named-port
...
 30         ports:
 31         - containerPort: 1234
 32           name: mynapp-http
 33           protocol: TCP
 ...
 replicaset.apps/mynapp-rs-named-port edited
 
 vagrant@kube-master1:~/kubernetes$ kubectl delete pods -l app=mynapp-rs-named-port
pod "mynapp-rs-named-port-ht9n8" deleted
pod "mynapp-rs-named-port-t57x7" deleted
pod "mynapp-rs-named-port-thqv7" deleted

vagrant@kube-master1:~/kubernetes$ kubectl get pods -l app=mynapp-rs-named-port
NAME                         READY   STATUS    RESTARTS   AGE
mynapp-rs-named-port-86qzb   1/1     Running   0          65s
mynapp-rs-named-port-f9m7d   1/1     Running   0          65s
mynapp-rs-named-port-wcpxh   1/1     Running   0          65s
vagrant@kube-master1:~/kubernetes$ kubectl get endpoints
NAME                    ENDPOINTS                                                          AGE
kubernetes              192.168.56.11:6443                                                 18h
mynapp-svc              10.233.101.8:8080,10.233.103.6:8080,10.233.76.7:8080               72m
mynapp-svc-affinity     10.233.101.8:8080,10.233.103.6:8080,10.233.76.7:8080               26m
mynapp-svc-mutliport    10.233.101.8:8443,10.233.103.6:8443,10.233.76.7:8443 + 3 more...   17m
mynapp-svc-named-port   10.233.101.10:1234,10.233.103.8:1234,10.233.76.12:1234             7m26s
```



서비스 탐색

```
vagrant@kube-master1:~/kubernetes$ kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE
mynapp-rs-b6h4j              1/1     Running   0          140m
mynapp-rs-dl5nc              1/1     Running   0          140m
mynapp-rs-fxvp8              1/1     Running   0          140m
mynapp-rs-named-port-86qzb   1/1     Running   0          76m
mynapp-rs-named-port-f9m7d   1/1     Running   0          76m
mynapp-rs-named-port-wcpxh   1/1     Running   0          76m
vagrant@kube-master1:~/kubernetes$ 
vagrant@kube-master1:~/kubernetes$ 
vagrant@kube-master1:~/kubernetes$ kubectl exec mynapp-rs-b6h4j env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=mynapp-rs-b6h4j
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT_443_TCP_PORT=443
MYNAPP_SVC_PORT=tcp://10.233.28.6:80
KUBERNETES_SERVICE_PORT=443
KUBERNETES_PORT_443_TCP=tcp://10.233.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
MYNAPP_SVC_SERVICE_HOST=10.233.28.6
MYNAPP_SVC_PORT_80_TCP_PROTO=tcp
KUBERNETES_SERVICE_HOST=10.233.0.1
KUBERNETES_PORT=tcp://10.233.0.1:443
KUBERNETES_PORT_443_TCP_ADDR=10.233.0.1
MYNAPP_SVC_SERVICE_PORT=80
MYNAPP_SVC_PORT_80_TCP=tcp://10.233.28.6:80
MYNAPP_SVC_PORT_80_TCP_PORT=80
MYNAPP_SVC_PORT_80_TCP_ADDR=10.233.28.6
NODE_VERSION=14.3.0
YARN_VERSION=1.22.4
HOME=/root

vagrant@kube-master1:~/kubernetes$ kubectl get all --namespace kube-system -l k8s-app=kube-dns 
NAME                           READY   STATUS              RESTARTS   AGE
pod/coredns-58687784f9-tm8gs   0/1     ContainerCreating   0          19h

NAME              TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
service/coredns   ClusterIP   10.233.0.3   <none>        53/UDP,53/TCP,9153/TCP   19h

NAME                          DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/nodelocaldns   4         4         3       4            3           <none>          19h

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/coredns   0/1     1            0           19h

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/coredns-58687784f9   1         1         0       19h

```



bash 창에서 ctrl+r 입력하면 전에 사용했던 명령어 자동완성으로 찾아줌.

metalLB 설치

```
vagrant@kube-master1:~/kubespray$ vi contrib/metallb/roles/provision/defaults/main.yml
  1 ---
  2 metallb:
  3   ip_range: "192.168.56.200-192.168.56.205"
  ...

vagrant@kube-master1:~/kubespray$ vi inventory/mycluster/group_vars/k8s-cluster/k8s-cluster.yml
  ...
  106 kube_proxy_strict_arp: true
  ...
  
vagrant@kube-master1:~/kubespray/contrib/metallb$ ansible-playbook -i ~/kubespray/inventory/mycluster/inventory.ini ~/kubespray/contrib/metallb/metallb.yml --become
```

```cmd
vagrant@kube-master1:~/kubespray/contrib/metallb$ kubectl get namespaces
NAME              STATUS   AGE
...
metallb-system    Active   64s

vagrant@kube-master1:~/kubespray/contrib/metallb$ kubectl get all --namespace metallb-system 
controller                  speaker                     speaker-zxpmw
controller-c89789d9d        speaker-p5kkk               
controller-c89789d9d-xt9ls  speaker-p7wk2               
vagrant@kube-master1:~/kubespray/contrib/metallb$ kubectl get all --namespace metallb-system 
NAME                             READY   STATUS    RESTARTS   AGE
pod/controller-c89789d9d-xt9ls   1/1     Running   0          82s
pod/speaker-p5kkk                1/1     Running   0          82s
pod/speaker-p7wk2                1/1     Running   0          82s
pod/speaker-zxpmw                1/1     Running   0          82s

NAME                     DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/speaker   3         3         3       3            3           <none>          82s

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/controller   1/1     1            1           82s

NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/controller-c89789d9d   1         1         1       82s
```

```
vagrant@kube-master1:~/kubernetes$ vi mynapp-svc-ext-lb.yml
  1 apiVersion: v1
  2 kind: Service
  3 metadata:
  4   name: mynapp-svc-ext-lb
  5 spec:
  6   type: LoadBalancer
  7   ports:
  8   - port: 80
  9     targetPort: 8080
 10   selector:
 11     app: mynapp-rs
 
 student@CCCR15:~/vagrant$ curl http://192.168.56.200
Message: Hello World!
Hostname: mynapp-rs-rcv8r
Platform: linux
Uptime: 21010
IP: 10.233.76.14
DNS: 169.254.25.10
```

인그레스 생성

```
vagrant@kube-master1:~/kubernetes$ vi mynapp-ing.yml
  1 apiVersion: networking.k8s.io/v1beta1
  2 kind: Ingress
  3 metadata:
  4   name: mynapp-ing
  5 spec:
  6   rules:
  7   - host: mynapp.example.com
  8     http:
  9       paths:
 10       - path: /
 11         backend:
 12           serviceName: mynapp-svc-ext-np
 13           servicePort: 80
```





### 실습

1. ```
   1. 다음과 같은 작업을 주기적으로 수행하는 컨트롤러를 작성하시오
   컨트롤러 이름 : cronjob-test1
   작업 수행 주기 : 매 분마다 정시에 동작
   작업을 수행할 파드 레이블 : cronjob-test1
   작업을 수행할 파드 이미지 : busybox
   수행할 작업 : sleep 30
   
   2. 다음과 같은 레플리카셋 컨트롤러를 작성하고, 컨트롤러에 의해 작성된 파드에 접근할 수 있는 서비스를 생성하시오.
   컨트롤러 이름 : rs-test1
   복제 개수 : 2
   템플릿 컨테이너 이미지 : httpd
   템플릿 컨테이너 포트 : 80
   서비스 이름 : svc-test1
   파드 포트 : 80
   서비스 포트 : 8080
   
   3. mynapp-rs.yml 파일의 ReplicaSet을 사용하여 다음과 같이 동작하는 서비스를 작성하시오.
   - 클러스터 내부에서 접근할 수 있는 IP 부여
   - 클러스터 내부의 다른 파드에서 접근 시 동일한 파드에 접근하도록 설정
   
   4. mynapp-rs.yml 파일의 ReplicaSet을 사용하여 다음과 같이 동작하는 서비스를 작성하시오.
   - 각 노드의 31234 포트로 접근 시 파드로 연결
   
   5. mynapp-rs.yml 파일의 ReplicaSet을 사용하여 다음과 같이 동작하는 서비스를 작성하시오.
   - 로드밸런서로부터 외부 IP를 할당받아 서비스 제공
   
   6. mynapp-rs.yml 파일의 ReplicaSet을 사용하여 다음과 같이 동작하도록 구성하시오.
   - NodePort 서비스를 4개 구성하시오 (svc-np1 ~ svc-np4)
   - 각 서비스는 모두 mynapp-rs.yml 설정에 따라 생성되는 파드로 연결되도록 설정하시오 (이름만 다르게)
   - 다음 주소에 따라 각 서비스에 연결되도록 설정하시오 (각 주소는 /etc/hosts/ 파일에 kube-master1의 IP를 직접 작성하여 테스트하시오)
   test1.cccr.local/app1 : svc-np1
   test1.cccr.local/app2 : svc-np2
   test2.cccr.local/app3 : svc-np3
   test2.cccr.local/app4 : svc-np4
   ```