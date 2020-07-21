---
title : "linux exam 풀이"

catagories:
   - Blog
tags:
   - Blog
---

```
linux exam

* 편의를 위해 실제 시험문제를 조금 변경했습니다.

* 구성 전 숙지사항

- Repo 주소는 http://yum.cccr.exam.com/repo 이며, yum 서버를 제외한 모든 서버는 해당 
YUM서버의 repo에서만 필요한 패키지를 다운로드 받을 수 있도록 설정하세요.

- 각 서버의 내부 네트워크 구성을 nmcli 명령어로 아래와 같이 설정하세요.
yum서버는 NAT Network NIC + Hostonly NIC, 
그 외 서버는 Host-only NIC만 있도록 구성

내부 네트워크 정적 ip 설정 / Subnet - 192.168.56.0/24
yum : 192.168.56.101 / 외부는 임의로
dns : 192.168.56.102
nfs : 192.168.56.103
web : 192.168.56.104

- nfs 서버에는 iscsi 스토리지 구성을 위한 가상디스크(10G)를 추가하세요.

---------------------------------------------------------------------------------

1. DNS 서버 구성
- 아래 정보를 이용하여 DNS 서버를 구성하고, 모든 서버에서 nslookup 명령어로 질의하여 각 서버들의 
주소를 받아올 수 있도록 하세요.

- Zone 정보(192.168.56.0/24)
192.168.56.101 -> yum.cccr.exam.com
192.168.56.102 -> dns.cccr.exam.com 
192.168.56.103 -> nfs.cccr.exam.com
192.168.56.104 -> web.cccr.exam.com
(192.168.56.104 -> web1 / 192.168.56.104 -> web2, CNAME 사용할 것)

2. NFS 서버 구성

1) /contents 디렉토리를 web서버에 공유 설정
2) /share 디렉토리의 하위 디렉토리들을 해당 네트워크 대역에 공유 설정
   (/share/doc , /share/movie , /share/music 디렉토리는 직접 생성)
3) 디스크(10G) iSCSI target 설정 후 web서버에서만 접근가능하도록 설정

3. WEB 서버 구성

1) /var/www/html 디렉토리에 NFS 서버의 /contents 디렉토리를 마운트하세요.
2) 가상호스트 설정을 이용해서 
	web1 이라는 이름으로 접근 시 /var/www/html 의 파일을 제공합니다.
	web2 이라는 이름으로 접근 시 /srv/www/html 의 파일을 제공합니다.
	(web2 컨텐츠는 간단한 텍스트로 직접 생성하고 SELinux 는 꺼둡니다.)
3) iSCSI 이니시에이터 설정을 합니다.
4) 제공받은 디스크를 파티셔닝해서 3G 파티션으로 /mnt 에 마운트설정,
				    1G 파티션은 swap 설정합니다.

4. YUM 서버 설정

1) autofs 를 이용해서 nfs서버의 /share 디렉토리 밑에 있는 doc,music,movie 디렉토리를 
간접와일드카드맵으로 마운트하세요.
```


# * 구성 전 숙지사항

- virtualbox host-only network DHCP 비활성화(체크 해제)

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-20_14-53-22.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-20_14-53-22.png)

- yum서버 network 설정

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-20_14-56-05.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-20_14-56-05.png)

- dns서버 network 설정

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-20_14-58-51.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-20_14-58-51.png)

- dns서버 repo 설정

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-20_15-07-30.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-20_15-07-30.png)

repo 경로로 이동해서 .repo 파일을 생성(파일이름은 임의 생성)

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-20_15-07-02.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-20_15-07-02.png)

위와 같이 주소를 yum서버로 지정 ([ ]와 name은 임의 입력 가능)

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-20_15-08-19.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-20_15-08-19.png)

yum 캐시 초기화한 후 yum repolist 명령어로 업데이트 시도했으나 호스트를 찾을 수 없다는 에러 발생

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-20_15-13-57.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-20_15-13-57.png)

아직 dns서버를 구성하지 않았으므로 임시로 vi /etc/hosts 파일에 yum서버를 호스트로 추가

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-20_15-56-50.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-20_15-56-50.png)

재시도 시 repo를 읽어 옴.

nfs, web서버도 위와 동일하게 설정해준다.

---

# 1. DNS 서버 구성

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-20_15-58-13.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-20_15-58-13.png)

패키지 설치(yum -y install bind)

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-20_16-09-24.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-20_16-09-24.png)

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-20_16-08-54.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-20_16-08-54.png)

zone 파일 생성

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-20_16-11-17.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-20_16-11-17.png)

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-20_16-11-32.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-20_16-11-32.png)

/etc/named.conf 수정 및 추가

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-20_16-18-37.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-20_16-18-37.png)

zone파일 권한 설정

* 미설정 시 zone파일을 읽어오지 못해서 아래와 같이 에러 출력

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-20_16-18-15.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-20_16-18-15.png)

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-20_16-12-32.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-20_16-12-32.png)

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-20_16-14-35.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-20_16-14-35.png)

named 재시작 및 활성화, 방화벽 해제

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-20_16-15-09.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-20_16-15-09.png)

nslookup 명령어 사용을 위한 패키지 설치(bind-utils)

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-20_16-19-57.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-20_16-19-57.png)

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-20_16-23-49.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-20_16-23-49.png)

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-20_16-26-07.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-20_16-26-07.png)

각 서버 별로 테스트해볼 것

---

# **2. NFS 서버 구성**

**1) /contents 디렉토리를 web서버에 공유 설정**

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-20_16-28-36.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-20_16-28-36.png)

nfs-utils 패키지 설치

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_10-00-25.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_10-00-25.png)

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_11-02-20.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_11-02-20.png)

/etc/exports 설정 후 exportfs -r 명령어로 불러와서 확인

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_10-05-15.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_10-05-15.png)

서비스 시작 및 활성화, 방화벽 해제

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_10-13-17.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_10-13-17.png)

web서버에서 showmount 명령어 사용을 위해선 rpc-bind, mountd 서비스도 해제해야함.

---

**2) /share 디렉토리의 하위 디렉토리들을 해당 네트워크 대역에 공유 설정**

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_10-22-00.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_10-22-00.png)

/share/doc, movie, music 디렉터리 생성

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_17-51-47.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_17-51-47.png)

/etc/exports에 추가 후 exportfs -r로 확인

---

3**) iSCSI 의 target 설정**

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_11-04-48.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_11-04-48.png)

targetcli 설치

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_11-10-50.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_11-10-50.png)

서비스 시작 및 등록, 방화벽 해제

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_11-08-52.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_11-08-52.png)

추가한 디스크(/dev/sdb) 확인

iscsi 구성

- block 생성 → IQN 주소 설정 → ACL 설정 → LUN 설정 → Portal 설정

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_11-13-17.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_11-13-17.png)

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_11-14-16.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_11-14-16.png)

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_11-16-19.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_11-16-19.png)

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_11-17-06.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_11-17-06.png)

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_11-33-51.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_11-33-51.png)

portal설정은 그대로 사용

---

# 3**. WEB 서버 구성**

**1) /var/www/html 디렉토리에 NFS 서버의 /contents 디렉토리를 마운트하세요.**

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_10-09-26.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_10-09-26.png)

nfs-utils 패키지 설치(showmount 명령어 사용 시 필요)

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_10-13-51.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_10-13-51.png)

nfs 공유 디렉토리 확인

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_10-16-34.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_10-16-34.png)

/var/www/html 폴더가 없으므로 httpd 패키지 미리 설치

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_10-17-12.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_10-17-12.png)

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_10-18-36.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_10-18-36.png)

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_10-19-19.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_10-19-19.png)

마운트 및 nfs서버에서 파일 생성 후 web서버에서 확인

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_15-00-42.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_15-00-42.png)

/etc/fstab 등록

---

**2) 가상호스트 설정을 이용해서** 

**web1 이라는 이름으로 접근 시 /var/www/html 의 파일을 제공합니다.**

**web2 이라는 이름으로 접근 시 /srv/www/html 의 파일을 제공합니다.** 

**(web2 컨텐츠는 간단한 텍스트로 직접 생성하고 SELinux 는 꺼둡니다.)**

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_12-27-58.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_12-27-58.png)

미리 설치했던 패키지 활성화 및 방화벽 해제

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_15-35-47.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_15-35-47.png)

SELinux disabled(/etc/sysconfig/selinux)

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_15-45-16.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_15-45-16.png)

/srv/www/html 디렉터리 생성 후 index.html 생성

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_12-31-16.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_12-31-16.png)

/etc/httpd/conf.d/virtualhost.conf 생성

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_15-39-56.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_15-39-56.png)

*** 교재에 빠진 내용**

/etc/httpd/conf/httpd.conf 맨 마지막 줄에 virtualhost.conf 파일을 등록하고, /srv/www/html 디렉토리 접근 권한을 허용해주세요.(355 ~ 360줄 추가)

Directory 부분을 추가하지 않으면 client denied by server configuration~ 에러 발생함.

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_15-42-43.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_15-42-43.png)

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_15-43-06.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_15-43-06.png)

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_15-43-25.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_15-43-25.png)

httpd 서비스 재시작 후 Local PC에서 접속해볼 것

(Local PC의 /etc/hosts 파일에 web서버 등록 필요)

---

**3) iSCSI 이니시에이터 설정을 합니다.**

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_11-37-24.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_11-37-24.png)

이니시에이터 패키지 설치 및 활성화

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_11-38-21.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_11-38-21.png)

nfs서버에 로그인 시도

* 아래와 같은 에러 발생 시 /etc/iscsi/initiatorname.iscsi 변경, 연결정보 삭제 및 관련 서비스 재시작

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_11-42-03.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_11-42-03.png)

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_11-40-57.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_11-40-57.png)

/etc/iscsi/initiatorname.iscsi 수정

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_11-44-42.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_11-44-42.png)

관련 서비스 재시작

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_11-45-52.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_11-45-52.png)

변경 후 다시 디스커버리 → 로그인  성공

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_11-46-39.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_11-46-39.png)

web서버에서 iscsi디스크(/dev/sdb) 할당 확인

---

**4) 제공받은 디스크를 파티셔닝해서 3G 파티션으로 /mnt 에 마운트설정 1G 파티션은 swap 설정합니다.**

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_11-52-36.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_11-52-36.png)

fdisk 명령어를 이용해서 위와 동일하게 파티셔닝 후 쓰기

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_11-57-18.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_11-57-18.png)

파일시스템 생성 

* partprobe : 리부팅 없이 사용 중인 파티션을 재인식하는 명령어

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_12-16-08.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_12-16-08.png)

마운트

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_18-17-43.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_18-17-43.png)

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_15-02-48.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_15-02-48.png)

UUID 확인 후 /etc/fstab 추가

* iscsi 서비스를 통해 제공받은 디바이스는 반드시 default옵션 대신 _netdev 옵션을 사용해야 함. 그렇지 않으면 부팅불가

---

# 4**. yum 서버 구성**

**1) autofs 를 이용해서 nfs서버의 /share 디렉토리 밑에 있는 doc,music,movie 디렉토리를 간접와일드카드맵으로 마운트하세요.**

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_16-10-50.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_16-10-50.png)

패키지 설치, 서비스 활성화

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_16-18-52.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_16-18-52.png)

마스터 맵 생성

- 마스터 맵의 이름은 임의로 사용 가능하지만, 확장자는 반드시 .autofs로 지정해야함. (여기선 share.autofs로 생성함)

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_16-18-27.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_16-18-27.png)

- /etc/auto.master.d/share.autofs 생성 후 입력

    - 첫번째 필드 : 간접 맵 사용 시 이 경로를 이용해서 nfs 공유 디렉토리에 접근하게 되고 미리 해당 

                            경로를 만들면 안됨. autofs 재시작 시 자동 생성

    - 두번째 필드 : 간접 맵파일의 경로

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_16-32-48.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_16-32-48.png)

![/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_16-32-26.png](/assets/images/2020-07-22-linuxexam/Screenshot_from_2020-07-21_16-32-26.png)

간접 맵 생성

![/assets/images/2020-07-22-linuxexamUntitled.png](/assets/images/2020-07-22-linuxexamUntitled.png)

![/assets/images/2020-07-22-linuxexamUntitled%201.png](/assets/images/2020-07-22-linuxexamUntitled%201.png)

서비스 재시작 후 확인하면 /share까지는  마운트 및 접근이 가능하지만 하위 디렉터리는 찾지 못한다. df -ah 명령어로 확인해봐도 /share 까지만 마운트 되어있다.

*** 교재에 빠진 내용**

교재에서는 nfs 서버 쪽 방화벽만 허용시켜주는데,  클라이언트 쪽에서도 방화벽을  허용시켜줘야만 하위 디렉토리까지 마운트 및 접근이 가능하다.

![/assets/images/2020-07-22-linuxexamUntitled%202.png](/assets/images/2020-07-22-linuxexamUntitled%202.png)

방화벽 허용 및 재시작

![/assets/images/2020-07-22-linuxexamUntitled%203.png](/assets/images/2020-07-22-linuxexamUntitled%203.png)

autofs 재시작 후 하위 디렉토리까지 접근 가능 확인

~~교재 똑같이 따라하면 다된다고, 검증까지 하셨다고 했는데...~~