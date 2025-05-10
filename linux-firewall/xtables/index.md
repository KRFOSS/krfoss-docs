---
title: Xtables
description: 
icon:
weight:
author: ["xenix4845",]
---

Xtables는 iptables에서 함께 사용되는 geoip 기반 모듈이에요. 이게 있으면 원하는 국가를 허용하거나 차단할 수 있죠. 이 문서는 `debian/ubuntu`를 기준으로 해요. 

모든 작업은 root로만 이뤄지니 꼭 `sudo -i`으로 계정을 변경해주세요. 

## apt install xtables-addons-common 

apt로 xtables 패키지를 설치해주세요. 명령어는 다음과 같아요.

{{% notice info %}}
시스템의 기본 미러가 kr.archive.ubuntu.com으로 되어 있다면 빠른 작업을 위해 [미러 변경](https://docs.krfoss.org/ubuntu/%EB%AF%B8%EB%9F%AC%EB%B3%80%EA%B2%BD%ED%95%98%EA%B8%B0/)을 추천드려요.

데비안의 경우 다를 수도 있어요. 그러나 변경하고 싶다면 위 문서에 있는 [공용 명령어](https://docs.krfoss.org/ubuntu/%EB%AF%B8%EB%9F%AC%EB%B3%80%EA%B2%BD%ED%95%98%EA%B8%B0/#id-2.-%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-%EC%8B%A4%ED%96%89%ED%95%B4%EC%84%9C-%EC%88%98%EC%A0%95%ED%95%98%EA%B8%B0)를 사용해서 변경할 수 있어요.
{{% /notice %}}

```bash
apt update
apt install xtables-addons-common -y
```

## /usr/share/xt_geoip 

설치가 완료되었다면 이제 국가별 아이피 정보를 담을 폴더를 생성해줘야 해요. 

``` bash
cd /usr/share
mkdir xt_geoip
``` 

생성이 되었다면 이제 각 나라별의 아이피 정보들이 담긴 데이터를 저장해줘야 해요. 그래야만 국가별 방화벽 설정이 가능해요.

``` bash
cd xt_geoip
wget https://docs.krfoss.org/linux-firewall/xtables/xt_geoip.zip
unzip xt_geoip.zip
```

{{% notice info %}}
unzip이 작동하지 않나요? 그럼 아래 명령어로 패키지를 설치해주세요.
```bash
apt install unzip -y
```
{{% /notice %}}

이제 압축을 모두 풀었으니 다음 명령어를 실행해주세요.

``` bash
modprobe xt_geoip
```

아무 경고 없이 명령 수행이 끝났다면 깔끔한 설정을 위해 시스템 재부팅을 해주는 걸 권장해요. `init 6`로 시스템을 재부팅할 수 있어요.

## iptables 설정

재부팅이 끝났나요? 이제 iptables에서 설정해줄 차례네요! 아래와 같은 예제를 활용해주세요. 

``` bash
iptables -I INPUT -m geoip --src-cc KR -j ACCEPT
iptables -I INPUT -m geoip --src-cc CN -j DROP
```
이 명령어는 한국은 허용(`accept`)하지만 중국에서 오는 트래픽들은 모두 응답하지 않고 버려요(`drop`). 국가 코드는 2글자로만 해야 해요. 국가 코드에 대해서 궁금하다면 위키백과의 [국가별 국가 코드 목록](https://ko.wikipedia.org/wiki/%EA%B5%AD%EA%B0%80%EB%B3%84_%EA%B5%AD%EA%B0%80_%EC%BD%94%EB%93%9C_%EB%AA%A9%EB%A1%9D)을 참고해주세요. 

## 규칙 저장하기

설정을 했다면 이제 규칙을 저장해야 나중에 시스템을 재부팅 하더라도 다시 처음부터 입력하지 않고 그대로 사용할 수 있어요.

{{% notice info %}}
아래 명령어를 사용하려면 사전에 이 패키지를 설치해야 해요.
```bash
apt install iptables-persistent -y
```
{{% /notice %}}

``` bash
netfilter-persistent save
netfilter-persistent reload
``` 

## 완료

모든 설정을 완료하였네요! 이제 국가별로 차단/허용 할 수 있게 되었어요. 참고로 우리가 저장했던 국가별 아이피 데이터는 ipv6와 ipv4 모두 지원해요. 

하지만 서버가 클라우드플레어의 프록싱 기능으로 보호 받고 있다면 이 방화벽 설정은 웹에서는 효과가 없을 수 있어요. 이런 경우에는 클라우드플레어의 데시보드에서 규칙을 설정해서 어떤 국가만 접근을 허용/차단할지 설정해주세요. 