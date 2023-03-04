---
layout: post
title: LDAP 서버 구성 및 gitlab 연동하기
categories: infra
tags: [infra, ldap, openldap, gitlab]
---
![1-1](/assets/images/infra/1-1.png)

# LDAP 서버 구성하기
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

![1-3](/assets/images/infra/1-3.PNG)
### STEP 2. OpenLDAP 환경 설정하기
아래 명령어를 입력하여 LDAP의 환경 설정을 진행한다.
```
dpkg-reconfigure slapd
```

![1-4](/assets/images/infra/1-4.png)
**no** 선택

![1-5](/assets/images/infra/1-5.png)
사용할 DNS Domain을 입력한다. 그림과 같이 *junkmm.io*를 입력하게 되면 'dc=junkmm,dc=io'라는 LDAP 디렉토리의 base DN을 생성하게 된다.

![1-6](/assets/images/infra/1-6.png)
조직 이름을 입력한다.

![1-7](/assets/images/infra/1-7.png)
관리자 이름을 입력한다.

![1-8](/assets/images/infra/1-8.png)
**no** 선택

slapd가 제거됐을 때 DB 정보를 지우지 않도록 설정

![1-9](/assets/images/infra/1-9.png)
**yes** 선택

기존의 LDAP 서버의 데이터베이스 파일이 남아있을 경우 새로운 데이터베이스 생성 시 기존 파일을 옮기는 설정

### STEP 3. 작동 확인하기
아래 명령을 통해 LDAP의 기본 정보를 확인해보자.
```
slapcat
```
<details>
<summary>콘솔 출력 결과</summary>
<div markdown="1">

dn: dc=junkmm,dc=io
objectClass: top
objectClass: dcObject
objectClass: organization
o: junkmm
dc: junkmm
structuralObjectClass: organization
entryUUID: a6f10e2c-4ee2-103d-876c-af2988c8e03a
creatorsName: cn=admin,dc=junkmm,dc=io
createTimestamp: 20230304141442Z
entryCSN: 20230304141442.474090Z#000000#000#000000
modifiersName: cn=admin,dc=junkmm,dc=io
modifyTimestamp: 20230304141442Z

dn: cn=admin,dc=junkmm,dc=io
objectClass: simpleSecurityObject
objectClass: organizationalRole
cn: admin
description: LDAP administrator
userPassword:: e1NTSEF9UU9zTkx2MUJPVlFUZ1gzUzhpUmJWMzI5d05XRmdrdUk=
structuralObjectClass: organizationalRole
entryUUID: a6f12330-4ee2-103d-876d-af2988c8e03a
creatorsName: cn=admin,dc=junkmm,dc=io
createTimestamp: 20230304141442Z
entryCSN: 20230304141442.474645Z#000000#000#000000
modifiersName: cn=admin,dc=junkmm,dc=io
modifyTimestamp: 20230304141442Z


</div>
</details>

## LDAP Account Manager 설치하기
LDAP Account Manager는 웹(php) 기반의 LDAP 관리 툴이다. 편의상 LAM이라고 부르는 것 같다. LAM을 사용하면 LDAP 디렉토리 내 객체를 관리하는데 있어 편리성을 제공한다.
이 LAM을 통해서 LDAP내 계정을 생성해보도록 하자.
## STEP 1. LAM 설치하기
아래 명령어를 통해 LAM을 설치한다.
```
apt-get install ldap-account-manager
```
## STEP 2. LAM 설정하기
웹 브라우저로 **http://(server-ip)/lam/** 웹페이지에 접속한다.
![1-10](/assets/images/infra/1-10.png)
**LAM Configuration** 클릭

![1-11](/assets/images/infra/1-11.png)
**Edit server progiles** 클릭

![1-12](/assets/images/infra/1-12.png)
Password에 **lam** 입력