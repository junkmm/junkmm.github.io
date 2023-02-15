---
layout: post
categories: Kubernetes
tags: [kubernetes, kubectl]
---
# kubectl 명령어 자동완성 설정하기

## Kubectl 명령어

kubectl 명령어는 쿠버네티스 API를 사용하여 **쿠버네티스 클러스터의 컨트롤 플레인과 통신하기 위한 Command Tool이다.** kubectl은 터미널 창에서 다음의 구문을 사용한다.

```bash
kubectl [command] [TYPE] [NAME] [flags]
```

 현재 쿠버네티스 클러스터에 배포된 Pod를 조회하는 kubectl 명령어는 아래와 같다.

```bash
kubectl get pod
```

## kubectl 명령어 자동완성을 사용하는 이유

자동완성이란 Linux Terminal, Cisco 장비 등에서 Tab키를 사용하여 타이핑을 줄이고 실수를 방지할 수 있는 기능이다. 하지만 kubectl 명령어는 기본적으로 자동완성을 지원하지 않기 때문에, 별도의 설정을 통해 자동완성 기능을 적용한다면 쿠버네티스 운영에 생산성을 높일 수 있다.

## kubectl 명령어 자동완성 적용하기

### bash-completion 설치

**Ubuntu**

```bash
sudo apt install bash-completion
```

**CentOS**

```bash
yum install bash-completion-extras
```

### kubectl 자동 완성 활성화

```bash
echo 'source <(kubectl completion bash)' >> ~/.bashrc
echo 'alias k=kubectl' >> ~/.bashrc # kubectl 명령어를 k 로 사용할 수 있도록 약어 설정
echo 'complete -o default -F __start_kubectl k' >> ~/.bashrc
```

### 셸 현재 세선에서 bash 자동 완성 활성화

```bash
exec bash
```

### 셸 접속 시 bash 자동 완성 활성화

```bash
echo 'source ~/.bashrc' >> /etc/bash.bashrc
```

## kubectl 자동완성 확인하기

자동완성 설정이 올바르게 적용됐는지 확인하기 위해 `k get pod` 명령어를 입력 해보자

```
root@master:~# k get pod
NAME   READY   STATUS    RESTARTS   AGE
web    1/1     Running   0          22h
```

기존 kubectl 대신 k 로 입력해도 정상적으로 pod의 정보를 가져올 수 있다. 또한 k get po 까지 입력하고 Tab키를 눌러보면 자동으로 k get pod 명령어를 완성
