---
title: RPP(krfs-rport) 사용법
description: 
icon:
weight:
author: ["KRFOSS"]
---

## 사용 전 반드시 읽으십시오

이 도구는 반드시 패키지 관리자를 통해서 설치해야 합니다. 

데비안 계열
``` bash
curl -fsSL http://pkg.krfoss.org/debian/script/install.sh | bash
```

RHEL 계열
``` bash
curl -fsSL http://pkg.krfoss.org/rhel/script/install.sh | bash
```

Arch 계열

``` bash
curl -fsSL http://pkg.krfoss.org/arch/script/install.sh | bash
```

Synology DSM (실험적)
1. 시놀로지 패키지센터에서 수동으로 커뮤니티 패키지센터 주소를 추가해야 합니다. `https://pkg.krfoss.org/synology` 를 추가하십시오. 이름은 원하는대로 지어도 되지만, 예시로 `ROKFOSS` 으로 짓도록 합니다.
2. 패키지센터 목록이 업데이트 되면 `ROKFOSS RPP`를 검색합니다.
3. 설치를 누르되, 시놀로지 자체 패키지가 아니므로 경고가 뜨지만 `동의(빨간색)`를 눌러 진행합니다. 이것은 모든 커뮤니티 패키지에서 나타나는 경고창이므로 안심하셔도 됩니다.
4. 설치가 끝나면 반드시 아래 스크립트를 `root` 계정에서 실행해야 합니다.
``` bash
curl -fsSL https://pkg.krfoss.org/synology/script/install.sh | sh
```
이것은 RPP 패키지가 root로 실행하도록 하는 것입니다. 그래야 포트를 원활하게 개방할 수 있습니다. 기본적으로 패키지가 실행 상태에 놓이면 TCP/47990을 개방하게 됩니다. 원치 않는 경우 패키지를 중지시켜 포트를 닫을 수 있습니다. 

명심하십시오. 패키지 실행은 RPP가 서버 모드로 동작하도록 지시하는 것입니다. 

패키지 업데이트/재설치를 할 때마다 실행해줘야 합니다. 만약 RPP에 중대한 업데이트가 있는게 아니라면 굳이 업데이트하거나 재설치 하지 마십시오.

---

사용 중인 운영체제에 맞는 스크립트를 실행한 뒤에 예시로 데비안의 경우 `apt install krfs-rport -y`를 실행하십시오. 그래야만 사용할 수 있습니다.

**설치 명령어는 반드시 각 운영체제에서 사용하는 패키지 관리자의 명령어를 사용해야 합니다.**

## RPP 설명

RPP(Reverse Proxy Port)는 ROKFOSS 프로젝트에서 개발된 역방향 포트 프록시 도구입니다. SSH를 통해 다른 서버에 자신의 서버의 포트를 역으로 개방하는 것과 동일한 원리입니다. 

그러나 SSH의 경우 키 생성과 추가 등 번거롭고 복잡한 문제가 있으며 RPP는 이러한 번거로움을 모두 제거하여 서버 관리자가 지정한 비밀번호로만 접속을 관리하도록 하여 키 작업에 대한 번거로움을 없앴습니다.

**이 문서는 최대한 어려운 단어를 사용하지 않고 초보자도 이해하기 쉬운 단어로만 구성되었습니다.**

RPP의 설정파일은 다음의 경로에 위치합니다. `/etc/krfs-rport/config.conf`
``` conf
password=
auto-reconn=false
address=
```

이것은 현재 기본 설정값입니다. 서버 운영자로 RPP를 사용한다면 다음의 값을 사용하게 됩니다.

false가 있는 것은 true로 사용하도록 지시할 수 있습니다.

- password
- address

이 두값은 각각 이러한 역할을 합니다.

### password

클라이언트가 서버에 접속할 때 사용해야 하는 비밀번호입니다. 이것이 빈값으로 저장되면 누구나 비밀번호 없이 포트를 붙일 수 있습니다. 만약 특정 사용자들에게만 개방하고 싶다면 비밀번호를 설정하는 것이 좋습니다.

### address

클라이언트가 서버측이 갖고 있는 ipv4/ipv6 주소를 임의로 지정하지 못하게 하며, 서버 운영자가 지정한 주소에만 포트를 붙일 수 있도록 합니다.

## 클라이언트가 사용하는 값

클라이언트는 `auto-reconn`을 사용합니다. 이것은 서버 운영자에 의해 연결이 강제 종료 되길 요청 받은 경우를 제외하고 일반적으로 연결 불안정 등으로 인해 서버측과 연결이 중단되었을 때 자동으로 다시 연결을 붙이게 합니다. 클라이언트측 사용자가 연결을 중단하지 않는 한 무제한으로 연결을 시도하게 됩니다.

## 각종 옵션

아래는 이 프로그램에서 사용할 수 있는 모든 옵션을 나열하였습니다. 필요에 따라서 사용할 수 있습니다.

``` bash
Usage of krfs-rport:
  -a    -auto 옵션과 동일
  -addr string
        서버 주소 (host:port)
  -alist
        -auto-list 옵션과 동일
  -aoff string
        -auto-off 옵션과 동일
  -auto
        시스템 자동 시작에 등록
  -auto-list
        자동 시작 등록 목록 조회
  -auto-off string
        자동 시작 등록 해제 (원격 포트 또는 'all')
  -autorun
        (내부용) 저장된 세션 실행
  -bg-child
        (내부용) 백그라운드 포워딩 세션 실행
  -bind string
        서버 바인드 IP
  -blocklist
        차단된 IP 목록 조회
  -completion string
        자동완성 스크립트 생성 (bash/zsh)
  -list
        백그라운드 포워딩 목록 조회
  -local int
        로컬 포트
  -off string
        백그라운드 포워딩 종료 (원격 포트 번호 또는 'all')
  -password string
        서버 비밀번호 (미지정 시 설정파일 사용)
  -proto string
        프로토콜 (tcp/udp)
  -remote int
        원격 포트
  -server
        리버스 포트 서버로 실행
  -slist
        서버 활성 포워딩 목록 조회
  -soff string
        서버 포워딩 강제 종료 (로컬 포트 번호 또는 'all')
  -unblock string
        IP 차단 해제 (IP 주소 또는 'all')
```

비대화형 연결을 할 수 있도록 옵션을 제공하고 있으나, 대화형 연결을 주로 사용하는 것이 좋습니다.

### 대화형 연결 방법

``` bash
krfs-rport
```
이것은 시스템이 재시작 되어도 다시 연결하지 않는 일회성 연결입니다.

``` bash
krfs-rport -a
```
이것은 시스템이 재시작되면 자동으로 연결하도록 합니다.

### 재시작 후에도 유지하기

클라이언트측에서는 이 명령을 반드시 실행해야 시스템이 재시작된 이후에도 다시 연결을 시도하게 됩니다.
``` bash
systemctl enable krfs-rport-autorun.service 
```

서버측에서는 이 명령을 반드시 실행해야 수동으로 서버 시작 없이 자동으로 서버를 시작할 수 있습니다.
``` bash
systemctl enable rport.service
```

### 무작위 대입 차단 시스템

이 시스템은 기본적으로 무작위 대입을 차단합니다. 최장 292년까지 차단하며, 차단 기간은 첫 10분으로 시작하여 배로 늘어나게 됩니다.

차단 해제는 위 사용가능한 옵션을 참조하여 진행하실 수 있습니다.

### 사용 중 질문

사용 도중 궁금하신 점이 있다면 [ROKFOSS 디스코드](https://chat.krfoss.org)를 방문하여 주세요.