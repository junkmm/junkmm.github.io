---
layout: post
title: LDAP 서버 구성 및 gitlab 연동하기
categories: infra
tags: [infra, ldap, openldap, gitlab]
---
![1-1](/assets/images/infra/1-1.png)
# LDAP 서버 구성하기
## LDAP 인증이란?
LDAP(Lightweight Directory Access Protocol)은 사용자가 조직, 구성원 등에 대한 데이터를 찾는데 도움이 되는 프로토콜이다.

LDAP은 클라이언트-서버 모델을 기반으로 한다. LDAP는 디렉토리를 제공하고 클라이언트는 디렉토리 서비스를 사용하여 항목에 액세스한다.
## LDAP 서버가 디렉토리를 구성하는 방법
![1-2](/assets/images/infra/1-2.png)

각 항목은 객체라고도 불리며 객체는 고유한 'DN(Distinguish Name)'을 갖는다. dn은 항목을 고유하게 식별하는 이름과 항목을 트리의 루트까지 추적하는 경로로 구성된다.
예를 들어 tomas의 dn은 다음과 같다.
```
dn: cn=tomas,ou=group-a,dc=junkmm,dc=io
```
LDAP은 사용자, 그룹, 장치, 응용 프로그램 등과 같은 객체를 저장하며 또한 LDAP 디렉터리 서비스는 보안, 인증 및 권한 부여와 같은 인프라 서비스를 제공하는 데 사용된다.

이번 포스트에서는 LDAP 서버를 구성하고 Gitlab과 연동하여 LDAP의 사용자를 통해 Gitlab에 로그인하는 기능을 구현하고자 한다.

# OpenLDAP과 Gitlab 연동하기
LDAP을 구성하여 계정을 생성하고 Gitlab과 연동하여 LDAP계정을 통해 Gitlab 로그인 하는 설정을 진행해보자.

## OpenLDAP 설정하기
### STEP 1. OpeneLDAP 설치
아래 명령어를 통해 OpenLDAP을 설치한다.
```
apt-get update
apt-get install slapd ldap-utils
```
패키지 설치 중 아래와 같이 LDAP 관리자 계정(admin)의 비밀번호 설정 화면에서 비밀번호를 입력하고 OK 버튼을 클릭한다.
![1-3](/assets/images/infra/1-3.png)
### STEP 2. OpenLDAP 환경 설정하기
아래 명령어를 입력하여 LDAP의 환경 설정을 진행한다.
```
dpkg-reconfigure slapd
```
![1-4](/assets/images/infra/1-4.png)
*no* 선택
![1-5](/assets/images/infra/1-5.png)
사용할 DNS Domain을 입력한다. 그림과 같이 *junkmm.io*를 입력하게 되면 'dc=junkmm,dc=io'라는 LDAP 디렉토리의 base DN을 생성하게 된다.
![1-6](/assets/images/infra/1-6.png)