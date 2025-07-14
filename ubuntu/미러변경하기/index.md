---
title: 우분투 미러 변경하는 방법
description: 
icon:
weight:
author: ["xenix4845", "DevNergis"]
---

## 우분투의 기본 미러

우분투는 기본적으로 우분투에서 호스팅하는 자체 미러를 쓰도록 설정되어 있어요. 다음 명령어를 통해서 지금 설정된 미러를 확인해볼까요?
``` bash
cat /etc/apt/sources.list
```

만약 따로 바꾸지 않았다면, `kr.archive.ubuntu.com`이 포함된 걸 확인할 수 있어요. 그냥 겉으로만 보기에는 이거 한국에 있는 서버 아니야? 하실 수 있겠지만 실제 도메인 정보를 `nslookup` 명령어로 확인해보면 아래와 같이 뜨는 것을 확인할 수 있어요.
``` bash
root@server:~# nslookup kr.archive.ubuntu.com
;; Got recursion not available from 150.230.255.179
Server:         150.230.255.179
Address:        150.230.255.179#53

Non-authoritative answer:
Name:   kr.archive.ubuntu.com
Address: 91.189.91.81
;; Got recursion not available from 150.230.255.179
Name:   kr.archive.ubuntu.com
Address: 2620:2d:4002:1::103
Name:   kr.archive.ubuntu.com
Address: 2620:2d:4000:1::102
Name:   kr.archive.ubuntu.com
Address: 2620:2d:4000:1::101
Name:   kr.archive.ubuntu.com
Address: 2620:2d:4002:1::101
Name:   kr.archive.ubuntu.com
Address: 2620:2d:4000:1::103
```

이 아이피들을 실제로 조회해보면 우리나라에 있는 게 아닌 미국, 영국에 있는 아이피라는 걸 알 수 있어요. 때문에 도메인만 kr이 붙은 것이고 실제로는 바다 건너 외국으로 연결하기 때문에 빠르지 않아요. 그래서 빠른 속도로 패키지를 다운 받으려면 **미러를 변경** 해주는 것이 좋아요!

## 미러 변경 방법

미러를 변경하는 방법은 2가지가 있어요. 

### 1. 직접 파일을 수정하기

`vi` 또는 `nano`로 `/etc/apt/sources.list`를 수정하는 방법이에요. 이 경우에는 버전별로 다르기 때문에 주의해야 해요. 

```bash

# 우분투 메인 저장소
deb https://http.krfoss.org/ubuntu noble main restricted universe multiverse
deb-src https://http.krfoss.org/ubuntu noble main restricted universe multiverse

# 업데이트
deb https://http.krfoss.org/ubuntu noble-updates main restricted universe multiverse
deb-src https://http.krfoss.org/ubuntu noble-updates main restricted universe multiverse

# 보안 업데이트
deb https://http.krfoss.org/ubuntu noble-security main restricted universe multiverse
deb-src https://http.krfoss.org/ubuntu noble-security main restricted universe multiverse

# 백포트
deb https://http.krfoss.org/ubuntu noble-backports main restricted universe multiverse
deb-src https://http.krfoss.org/ubuntu noble-backports main restricted universe multiverse
```

#### 1-1. Ubuntu 24.04 LTS (Noble Numbat) 이상일때

Ubuntu 24.04 LTS (Noble Numbat) 이상 부터는 `/etc/apt/sources.list`가 `/etc/apt/sources.list.d/ubuntu.sources`로 이사​🚚​ 갔어요!

자세한 내용은[Ubuntu 24.04 LTS (Noble Numbat) Release Notes](https://discourse.ubuntu.com/t/ubuntu-24-04-lts-noble-numbat-release-notes/39890#p-99950-deb822-sources-management)의 **deb822 sources management**부분을 참고하세요!

`vi`또는 `nano`로 `/etc/apt/sources.list.d/ubuntu.sources`를 수정하는 방법이에요.

```bash
#우분투 메인, 업데이트, 백포트 저장소
Types: deb
URIs: https://http.krfoss.org/ubuntu/
Suites: noble noble-updates noble-backports
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

# 우분투 보안 업데이트 저장소
Types: deb
URIs: https://http.krfoss.org/ubuntu/
Suites: noble-security
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg
```


위와 같이 수정한 뒤에 아래 명령어를 실행해서 정보를 업데이트 해주세요.

```bash
apt update 
```

그럼 정상적으로 미러가 변경되고 파일을 받아오는 것을 볼 수 있어요! 참고로 위 주소는 분산미러 주소이기 때문에 실제로 명령을 실행하면 `mirror.krfoss.org`에서 받아오는 걸 볼 수 있어요. 다만 실제로 패키지 파일들을 받을 때는 여러 미러에서 받아오도록 되어 있어요. 


### 2. 스크립트 실행해서 수정하기

이번엔 명령어 한 줄만으로 바꿀 수 있는 스크립트 방식이에요. 아래 명령어를 입력해서 간편하게 미러 정보를 변경할 수 있어요!

```bash
curl -sSL https://http.krfoss.org/pack/cm.sh | bash
```

스크립트가 자동으로 우분투 버전을 인지하고 그에 맞게 업데이트 해줘요. [ROKFOSS 프로젝트](https://http.krfoss.org/)에서 배포하는 스크립트이며 많은 사람들이 사용하고 있어요! 만약 스크립트 실행 중에 문제가 발생했다면 페이지 하단에 있는 공식 메일로 연락하기를 눌러서 연락해주세요! 혹은 krfoss@krfoss.org로 편한 메일 클라이언트를 사용해서 보내셔도 좋아요!
