---
layout: post

title: kubeadm으로 고가용성 클러스터 생성 - 스택 컨트롤 플레인 -1

categories: Kubernetes

tags: [Kubernetes, etcd]
---
## 목표

3대의 마스터노드와 2대의 워커노드를 사용한 고가용성 쿠버네티스 클러스터 구성하고자 한다. 고가용성 클러스터를 구성하는 방식은 아래와 같이 두 가지가 있다. 이번 실습은 `스택 컨트롤 플레인` 방식으로 고가용성 클러스터를 구성해본다.

* `스택 컨트롤 플레인` - etcd 노드가 컨트롤 플레인 노드와 함께 배치되는 스택형 컨트롤 플레인 노드 사용
* `외부 etcd 클러스터 사용` - etcd가 컨트롤 플레인과 별도의 노드에서 실행되는 외부 etcd 노드 사용

## 구성하기

![1-1](/assets/images/kubernetes/040301.png)

## Step 1. kube-api용 로드밸런서 구성

### 1. HAProxy 설치

* 패키지 관리자를 업데이트
  ```
  sudo apt-get update
  ```
* HAProxy를 설치
  ```
  sudo apt-get install haproxy
  ```

### 2. HAProxy 구성

* HAProxy 구성 파일 편집

  ```
  sudo vi /etc/haproxy/haproxy.cfg
  ```
* 아래와 같이 내용 변경

  ```
  global
  	log /dev/log	local0
  	log /dev/log	local1 notice
  	chroot /var/lib/haproxy
  	stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
  	stats timeout 30s
  	user haproxy
  	group haproxy
  	daemon

  # --- 생략 ---

  defaults
  	log	global
  	mode	tcp
  	option	tcplog
  	option	dontlognull
          timeout connect 5000
          timeout client  50000
          timeout server  50000

  listen kubernetes-api
          bind 192.168.219.130:6443
          mode tcp
          option tcplog
          balance roundrobin
          server control-plane-1 192.168.219.131:6443 check
          server control-plane-2 192.168.219.132:6443 check
          server control-plane-3 192.168.219.133:6443 check
  ```

  * **`listen kubernetes-api`** : HAProxy에서 제공할 서비스 이름
  * **`bind <로드밸런서 IP>:6443`** : 로드밸런서 IP와 API 서버 포트인 6443을 바인딩
  * **`server k8s-master-01 <컨트롤플레인 1번 노드 IP>:6443 check`** : API 서버를 제공할 쿠버네티스 컨트롤 플레인 노드 IP 및 포트

### 3. HAProxy 실행

* HAProxy 실행 및 enable
  ```
  sudo systemctl restart haproxy.service
  sudo systemctl enable haproxy
  ```
* HAProxy정
  ```
  sudo service haproxy status
  ```

## Step 2. 클러스터 구성하기

### 1. 사전 작업

   5대의 노드에 쿠버네티스 join 전까지의 구성을 진행해야 한다.
   (Reference 쿠버네티스 클러스터 구성하기 참고)

### 2. 첫 번째 Control-plane Node 구성

* 아래 명령어를 통해 k8s 클러스터를 활성화 한다.

  ```bash
  sudo kubeadm init --control-plane-endpoint "192.168.219.130:6443" --upload-certs --pod-network-cidr=10.10.0.0/16
  ```

  `control-plane-endpoint` : LoadBalancer의 ip, 컨트롤플레인에 연결하기 위한 주소

  `upload-certs` : 구성 요소간 통신 보안으로 사용할 TLS 생성

  `pod-network-cidr` : Pod에서 사용할 네트워크 CIDR
* 출력 결과를 저장해 둔다.

  ```bash
  You can now join any number of the control-plane node running the following command on each as root:

    kubeadm join 192.168.219.130:6443 --token bqfrdh.mz44h72adpin6lcu \\
  	--discovery-token-ca-cert-hash sha256:215c4ec1ab6e95fdbfb6b3b90d68c2c3fa0a18034ab0a81e13265388689b93fa \\
  	--control-plane --certificate-key ef135814ad769aa7a1fefcf610023ce9f015c6137779090fbb6561e39929e5b9

  Then you can join any number of worker nodes by running the following on each as root:

  kubeadm join 192.168.219.130:6443 --token bqfrdh.mz44h72adpin6lcu \\
  	--discovery-token-ca-cert-hash sha256:215c4ec1ab6e95fdbfb6b3b90d68c2c3fa0a18034ab0a81e13265388689b93fa
  ```
* kubectl 설정

  ```bash
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
  ```
* CNI 구성하기

  ```bash
  curl <https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml> -O
  sed -i -e 's?10.244.0.0/16?10.10.0.0/16?g' kube-flannel.yaml
  kubectl apply -f kube-flannel.yml
  ```

### 3. 나머지 control-plane Node 구성

* 아래 명령어를 통해 k8s 클러스터의 control-plane으로 join
  ```bash
  kubeadm join 192.168.219.130:6443 --token bqfrdh.mz44h72adpin6lcu \\
  	--discovery-token-ca-cert-hash sha256:215c4ec1ab6e95fdbfb6b3b90d68c2c3fa0a18034ab0a81e13265388689b93fa \\
  	--control-plane --certificate-key ef135814ad769aa7a1fefcf610023ce9f015c6137779090fbb6561e39929e5b9
  ```
* kubectl 설정
  ```bash
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
  ```

### 4. Worker Node Join하기

* 아래 명령어를 통해 k8s 클러스터의 worker-node로 join
  ```bash
  kubeadm join 192.168.219.130:6443 --token bqfrdh.mz44h72adpin6lcu \\
  	--discovery-token-ca-cert-hash sha256:215c4ec1ab6e95fdbfb6b3b90d68c2c3fa0a18034ab0a81e13265388689b93fa
  ```
* kubectl 설정
  ```bash
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
  ```

### 5. 클러스터 구성 확인하기

* kubectl get node 명령어를 통해 cluster구성 결과를 확인할 수 있다. 아래 3대의 control-plane과 2대의 worker로 클러스터가 구성된 것을 확인할 수 있다.
  ```bash
  NAME              STATUS   ROLES           AGE     VERSION
  control-plane-1   Ready    control-plane   32m     v1.26.3
  control-plane-2   Ready    control-plane   16m     v1.26.3
  control-plane-3   Ready    control-plane   16m     v1.26.3
  worker-1          Ready    <none>          4m38s   v1.26.3
  worker-2          Ready    <none>          4m35s   v1.26.3
  ```

## Reference

* [https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/)
* [https://velog.io/@chan9708/k8ssettings](https://velog.io/@chan9708/k8ssettings) - 쿠버네티스 클러스터 구성하기
