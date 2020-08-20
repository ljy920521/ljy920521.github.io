# 8/12 kubernetes 

---



mkdir ~/.kube

sudo cp ~root/.kube/config ~/.kube

sudo chown vagrant:vagrant ~/.kube/config

kubectl

** 명령어 목록이 보이지 않을 경우, kubectl completion bash | sudo tee /etc/bash/_completion.d/kubectl

exec bash

kubectl cluster-info

kubectl get nodes



#master1

vi ~./vimrc (교재 104p)

```
syntax on
autocmd FileType yaml setlocal ts=2 sts=2 sw=2 expandtab autoident nu
# YAML 파일을 효율적으로 작성하기 위한 사용자정의 에디터를 생성한다.
# ts : tabsize, expandtab=, nu: set nu
```



kubectl explain pod

kubectl explain pod.metadata



### 래플리케이션 컨트롤러(파드) 생성

kubectl run myweb-1st-app --image=c1t1d0s7/myweb --port=8080 --generator=run/v1

...

replicationcontroller/myweb-1st-app created	# 정상 생성 확인

kubectl get pods

kubectl get replicationcontroller

kubectl expose replicationcontroller myweb-1st-app --type=LoadBalancer --name myweb-svc

kubectl get services

curl 'SERVICE-IP(192.168.56.11):PORTS'



** 파드스케일링









## 실습

파드

1. 다음과 같이 동작하는 파드 오브젝트를 생성하는 yaml 파일을 작성하시오.
   - pod 이름 : exercise-pod
   - pod 레이블 : app=httpd, env=dev
   - 컨테이너 이미지 : httpd
   - 컨테이너 이름: web
   - 컨테이너 포트 : 80
2. 1번 파일을 사용하여 파드를 생성하고, 다음과 같이 레이블을 변경하시오.
   - env=production
3. 1번 파일에 다음과 같은 annotation 설정을 추가하시오.
   - creator : HongKilDong