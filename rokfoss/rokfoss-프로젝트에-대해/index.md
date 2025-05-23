---
title: ROKFOSS 프로젝트는 무엇인가요?
description: 
icon:
weight:
author: ["KRFOSS","xenix4845"]
---

## 국내 유일의 FOSS 미러 프로젝트
{{% notice info %}}
ROKFOSS Project는 Republic of Korea Free Open Source Software Project의 약자입니다. 번역하면 대한민국 자유 오픈소스 소프트웨어 프로젝트가 됩니다.
{{% /notice %}}

ROKFOSS 프로젝트는 국내 유일의 FOSS 미러 프로젝트입니다! 여태까지 국내에는 아예 없었던 FOSS 미러 커뮤니티를 형성하여 FOSS 미러 운영자들 간의 정보 교류를 원활하고 활발히 할 수 있도록 지원합니다.

지금까지 모든 FOSS 미러 서버들은 커뮤니티 없이 단독으로만 혼자서 운영하는 경우 밖에 없었습니다. 때문에 미러 구축에 대한 도움이라거나 정보를 구하는 게 매우 어려웠습니다. ROKFOSS 프로젝트는 이런 문제를 해결하기 위해 신규 미러 및 기존 미러 운영자들끼리 교류할 수 있는 커뮤니티를 만들었으며 이로 인해 ROKFOSS 프로젝트 참여 미러들은 모두 수준 높은 관리 상태와 새로운 기능들을 모두가 공유하고 손쉽게 도입할 수 있게 되었습니다.

또한 `공개 미러`를 만들고 나서 주소를 공유하지 않는다면 악성봇 말고는 아무도 찾는 사람이 없는 미러가 되고 말 것입니다. 그러나 ROKFOSS 프로젝트는 FOSS 미러라면 누구나 참여비 같은 거 없이 커뮤니티에 참여하여 미러를 목록에 등록하여 많은 사람들에게 손쉽게 알릴 수 있습니다. 또한 분산미러 시스템에서 지원하는 저장소를 제공하고 있다면 트래픽 또한 큰 수고 없이 손쉽게 확보할 수 있게 됩니다.

{{% notice info %}}
`공개 미러`란 불특정 다수의 사람들이 이용할 수 있는 미러 서버를 말합니다. 보통 방화벽 뒤에 포트포워드로 개방하거나 아예 서버가 공인 아이피를 할당 받아 사람들이 바로 접속할 수 있는 서버를 의미합니다.
{{% /notice %}}

## 미러 운영자를 위한 간편 도구

자신의 남는 서버 자원을 활용하여 리눅스 미러 생태계에 기여하고자 하는 마음이 있다면 이제 힘들게 미러 서버를 구축하는 방법을 구글링할 필요가 없습니다. 우리는 예비 미러 운영자를 위해 최고의 미러 [구축 가이드](/mirror/%EB%A6%AC%EB%88%85%EC%8A%A4%20%EB%AF%B8%EB%9F%AC%20%EC%84%9C%EB%B2%84%20%EA%B5%AC%EC%B6%95%ED%95%98%EA%B8%B0/)와 동기화 스크립트 및 프로그램을 제공합니다. 더 이상 미러를 구축하기 위해 스크립트를 어떻게 짜야할지 고민할 필요가 없습니다.

ROKFOSS 프로젝트에서 제공하는 미러 스크립트는 쉘 스크립트를 안다면 손 쉽게 수정할 수 있는 것이 특징입니다. 또한 해당 스크립트를 기반으로 새로운 기능들을 창조해낼 수도 있습니다. 이 경우에는 GPLv3 라이센스가 적용되므로 소스코드를 깃허브에 공개하여야 합니다. ROKFOSS 프로젝트는 여러분이 공개한 소스코드를 관리/감독하지 않을 것이지만 라이센스는 준수하여 주시길 바랍니다.

만약 스크립트 수정이 익숙하지 않거나 어렵다면 ROKFOSS 프로젝트에서 배포하는 Dsync(디싱크)라는 프로그램을 사용해보는 것도 방법이 될 수 있습니다. 디싱크는 단 2줄만으로 저장소별 설정 파일을 구성할 수 있는 것이 특징입니다. 예를 들자면 아래와 같습니다.

``` conf
repo=kali
save=/mirrors/kali
```

여기서 `repo`는 저장소를 뜻하고 지금 이곳에 설정된 `kali`는 칼리 리눅스의 패키지 저장소를 뜻합니다. `save`는 어디에 저장할지를 의미합니다. 지금 이곳에서는 `/mirrors/kali`로 저장한다고 가정하였습니다. 이와 같은 방법으로 간단하게 설정파일을 구성하여 동기화를 진행할 수 있습니다. 더 자세히 알아보려면 [dsync 문서](/rokfoss/dsync/)를 읽어주십시오.

## 누구나 참여가능한 커뮤니티

ROKFOSS 프로젝트는 누구나 자유롭게 가입하고 참여할 수 있는 [공개 커뮤니티](https://chat.krfoss.org)를 개설하여 운영하고 있습니다. 가입 시에 이메일 주소 외에 어떠한 개인정보도 요구하지 않습니다. 

커뮤니티는 24시간 운영되는 서버에서 서비스 되고 있기 때문에 언제든지 접속해서 소통할 수 있습니다. 또한 PC전용 프로그램과 모바일 앱을 제공하고 있어 다운로드 후 설치, 로그인만 하면 언제 어디서든지 전 세계에서 인터넷만 된다면 편리하게 접속할 수 있습니다.

커뮤니티 가입 후에는 많은 사람들과 소통하면서 정보를 교류할 수 있습니다. 자신이 겪었던 경험들을 이곳 [ROKFOSS Docs](https://github.com/KRFOSS/krfoss-docs)에 문서로 남겨서 자신과 동일한 문제를 겪는 유저에게 문서 링크를 보내주어 스스로 문제를 해결할 수 있도록 도움을 줄 수 있습니다. _여러분의 경험을 공유해주세요!_

## 다양한 프로젝트를 진행합니다!

ROKFOSS는 현재 다음과 같은 프로젝트를 진행하고 있습니다!

### FOSS 미러 프로젝트 (분산미러 시스템)

FOSS 미러 프로젝트는 ROKFOSS의 대표 프로젝트로 `분산미러 시스템`을 제공하고 있습니다. `분산미러 시스템`은 기존 미러 구조를 벗어나 사용자들의 트래픽을 IP 주소를 기반으로 대략적인 위치 정보와 ASN 정보를 얻어내서 가장 최적의 미러로 트래픽을 분산시켜주는 서비스입니다. 2025년 5월 기준으로 수많은 개인과 여러 대학교에서 우리의 `분산미러 시스템`을 사용하고 있습니다. 매일 몇건이 아닌 몇천건의 요청을 안정적으로 처리하고 있습니다. 처리되는 요청의 양은 최근 급격하게 증가하고 있으며 시간이 지남에 따라 더욱 증가할 것으로 예상됩니다.

`분산미러 시스템`에 대해서 조금 더 설명을 하자면 이렇습니다.

예를 들어서 이 글을 보고 있는 여러분들이 _서울특별시_ 에 거주하고 있고 인터넷은 KT를 쓰고 있으며 우분투를 운영체제로 사용하고 있다고 가정해보겠습니다. 여러분은 패키지를 업데이트 하기 위해 `http.krfoss.org`로 미러를 설정한 상태에서 
``` bash
apt update
```

명령어를 실행하였습니다. 그럼 여러분의 요청은 `분산미러 시스템`으로 전달되어 처리가 되고 최신 패키지 정보를 받아오게 될 것입니다. 여기까지는 `mirror.krfoss.org`로 고정적으로 연결되어 받아오게 될 것입니다. 그러나 만약 다음과 같은 출력이 나왔다고 가정해보겠습니다.

``` zsh
기존:1 https://mirror.krfoss.org/ubuntu noble InRelease                                   
받기:2 https://mirror.krfoss.org/ubuntu noble-updates InRelease [126 kB]                  
받기:3 https://mirror.krfoss.org/ubuntu noble-security InRelease [126 kB]
받기:4 https://mirror.krfoss.org/ubuntu noble-backports InRelease [126 kB]
받기:5 https://mirror.krfoss.org/ubuntu noble-updates/main amd64 Packages [1067 kB]
받기:6 https://mirror.krfoss.org/ubuntu noble-updates/main amd64 Components [161 kB]
받기:7 https://mirror.krfoss.org/ubuntu noble-updates/restricted amd64 Components [212 B]             
받기:8 https://mirror.krfoss.org/ubuntu noble-updates/universe amd64 Packages [1062 kB]

[중략...]                                    

받기:15 https://mirror.krfoss.org/ubuntu noble-security/main amd64 Components [21.5 kB]
받기:16 https://mirror.krfoss.org/ubuntu noble-security/restricted amd64 Components [212 B]
받기:17 https://mirror.krfoss.org/ubuntu noble-security/universe amd64 Components [52.2 kB]
받기:18 https://mirror.krfoss.org/ubuntu noble-security/multiverse amd64 Components [208 B]
받기:19 https://mirror.krfoss.org/ubuntu noble-backports/main amd64 Components [7060 B]
받기:20 https://mirror.krfoss.org/ubuntu noble-backports/restricted amd64 Components [212 B]
받기:21 https://mirror.krfoss.org/ubuntu noble-backports/universe amd64 Components [16.4 kB]
받기:22 https://mirror.krfoss.org/ubuntu noble-backports/multiverse amd64 Components [212 B]

패키지 목록을 읽는 중입니다... 완료     
의존성 트리를 만드는 중입니다... 완료
상태 정보를 읽는 중입니다... 완료        
1235 패키지를 업그레이드할 수 있습니다. 확인하려면 'apt list --upgradable'를 실행하십시오.
```
이 출력 예시에서는 1235개의 패키지를 업그레이드 할 수 있다고 보고 하였습니다. 그럼 이제 패키지 업그레이드를 위해 `apt full-upgrade -y`를 실행해보겠습니다.

``` zsh
패키지 목록을 읽는 중입니다... 완료
의존성 트리를 만드는 중입니다... 완료
상태 정보를 읽는 중입니다... 완료        
업그레이드를 계산하는 중입니다... 완료

다음 패키지를 업그레이드할 것입니다:
  [예시이므로 생략..]
1235개 업그레이드, 0개 새로 설치, 0개 제거 및 0개 업그레이드 안 함.
2,909 k바이트 아카이브를 받아야 합니다.
이 작업 후 43.0 k바이트의 디스크 공간을 더 사용하게 됩니다.
받기:1 https://mirror.eliv.kr/ubuntu noble-updates/main amd64 gnome-shell-extension-ubuntu-dock all 90ubuntu3 [120 kB]
받기:2 https://mirror.eliv.kr/ubuntu noble-updates/main amd64 grub2-common amd64 2.12-1ubuntu7.3 [670 kB]
받기:3 https://mirror.eliv.kr/ubuntu noble-updates/main amd64 grub-common amd64 2.12-1ubuntu7.3 [2,120 kB]
[예시이므로 생략..]
받기:1200 https://mirror.techlabs.co.kr/ubuntu ......

[...]

내려받기 2,909 k바이트, 소요시간 1초 (5,361 k바이트/초)
패키지를 미리 설정하는 중입니다...
(데이터베이스 읽는중 ...현재 197334개의 파일과 디렉터리가 설치되어 있습니다.)

[예시이므로 생략..]
```

예시이기 때문에 실제와는 조금 다를 수 있지만 위와 같이 `http.krfoss.org`에서 다양한 미러로 넘어가 파일을 받는 것을 알 수 있습니다. 이는 `분산미러 시스템`이 여러분의 접속 정보를 기반으로 최적화된 미러로 요청을 분산하였기 때문입니다. 

보통 서울에 거주하고 KT망을 쓴다면 대부분의 경우 수도권에 미러가 밀집되어 있기 때문에 수도권에 위치한 미러들을 자주 볼 수 있게 될 것입니다. 그러나 미러의 수가 적은 지방에서는 조금 결과가 다를 수 있습니다. 예를 들어서 광주광역시에 거주하고 있다면 가장 많이 보게 되는 미러는 2025.05 기준 [zzuniMirror](https://mirror.zzunipark.com/) 일 것이며 수도권에 있는 미러들이 가끔 보이게 될 것입니다. 

`분산미러 시스템`은 기본적으로 가장 가까운 미러를 우선적으로 사용하도록 설계되었습니다. 

#### 어떠한 상황에서도 안정적인 동작

다양한 조건에서도 동작할 수 있도록 설계되어 있으며 아래와 같은 경우에도 안정적인 동작이 가능합니다. (2025.05.14 기준)

조건1: 
``` 
- 구미에 거주함
- Rocky를 사용함

결과: Keiminem 미러는 Rocky를 지원하지 않지만 가장 가까운 지역인 광주광역시에 zzuniMirror가 Rocky를 지원하므로 해당 미러가 우선 매칭됨.
```

조건2:
``` 
- 제주도에 거주함
- Almalinux를 사용함

결과: 가장 가까운 미러는 서울특별시에 위치한 미러들이며 거리상으로 보았을 때 PROJECT ELIV가 가장 가까우므로 해당 미러가 우선 매칭됨.
```

조건3:
``` 
- 전주시에 거주함
- LG망을 사용함
- Kali Linux를 사용함

결과: LG망에 가장 최적화되고 가까운 미러는 zzuniMirror이므로 해당 미러가 우선 매칭됨.
```

시간이 지남에 따라서 지방에 새로운 미러가 생겨나고 ROKFOSS 프로젝트에 참여하게 될 수도 있습니다. 가장 최신 정보는 [분산미러 사이트](https://http.krfoss.org/)를 확인하시기 바랍니다.

### 칼리 리눅스 문서 한글화 프로젝트

칼리 리눅스를 사용해보면서 궁금한게 생겨 구글을 찾아보다 우연히 칼리 리눅스 공식 문서를 들어가본 경험이 있을 것입니다. 하지만 칼리 리눅스 공식 문서는 국제연합에서 지정한 공용어인 `영어`로 되어 있습니다. 때문에 한국어를 사용하는 우리는 이해에 많은 어려움을 겪을 수 밖에 없으며 영어를 모국어처럼 잘하는 경우가 아니라면 번역기를 필히 사용해야 합니다.

하지만 번역기는 매번 같은 결과를 내놓지 않을 수 있으며 번역해선 안될 부분을 번역해버리거나 원래 의도와 다른 전혀 생뚱맞은 말을 넣는 경우도 있습니다. 이런 경우를 마주하게 된다면 혼란스러울 수 밖에 없습니다. 

ROKFOSS 프로젝트는 이러한 문제를 해결하기 위해 칼리팀과 공식 채널에서 논의하였으며 그 결과 독립적으로 이러한 번역 프로젝트를 진행해도 된다는 답변을 받았고 곧바로 번역 작업을 시작하였습니다. 다양한 사람들이 [칼리 리눅스 문서](https://kali.krfoss.org/)를 번역하기 위해 기여하고 있습니다. 

ROKFOSS Kali Docs는 번역 뿐만 아니라 독자적인 문서도 만들어서 제공하고 있습니다. 여러분도 깃허브 계정만 있다면 칼리 리눅스 문서를 번역하거나 수정, 추가하는 방식으로 기여할 수 있습니다. 기여를 하고자 한다면 [KRFOSS Kali-Docs](https://github.com/KRFOSS/kali-docs)를 참고하십시오.

### ROKFOSS Docs

채팅형 커뮤니티 또는 게시판 형태의 커뮤니티의 문제점은 여러 가지 소식들이 올라오기 때문에 지식 문서와 같은 글들이 쉽게 묻히고 잊혀집니다. 때문에 사람들은 이를 발견하지 못해 같은 질문을 게시판에 계속 올리거나 같은 내용의 지식 문서들을 올릴 수도 있습니다.

ROKFOSS Docs는 이러한 문제점을 해결하기 위해 지식 베이스를 만들었습니다. ROKFOSS Docs에서는 여러분이 찾고자 하는 내용을 손쉽게 찾을 수 있도록 접근성을 매우 중요시하며 설계되었습니다. 또한 일반 커뮤니티처럼 다양한 글들이 올라와 묻히는 경우도 절대 없습니다.

만약 여러분이 Proxmox에 관련한 정보를 찾고 싶다면 메인 화면에서 검색을 하거나 [Proxmox 분류](https://docs.krfoss.org/proxmox/)를 클릭하여 기여자들이 작성한 문서들을 바로바로 읽어볼 수 있습니다. 혹은 여러분이 직접 기여자가 되어 문서를 새롭게 작성하거나 기존 문서들에 새로운 내용을 추가하는 등의 방식으로 기여할 수 있습니다.

기여자가 되고자 한다면 [krfoss-docs](https://github.com/KRFOSS/krfoss-docs)를 확인하십시오. 누구나 깃허브 계정만 있다면 기여할 수 있습니다. 

## ROKFOSS 프로젝트를 도울 수 있습니다

ROKFOSS 프로젝트는 수익을 창출하지 않는 비영리 프로젝트이기 때문에 여러분들의 도움이 필요합니다.

ROKFOSS 프로젝트를 지지하고 지원하고자 한다면 다음과 같은 도움을 줄 수 있습니다.

### 개인

개인자격으로 이 프로젝트를 지지하고 지원할 수 있습니다. 개인자격의 경우 다음과 같은 것들을 할 수 있습니다.

- ROKFOSS 프로젝트에 금전적인 지원을 할 수 있습니다. [xenix4845](https://github.com/xenix4845) 프로필에 있는 스폰서 버튼을 눌러 커피값 보내주기와 같은 지지와 응원을 보낼 수 있습니다. (KRFOSS Github 조직은 스폰서 기능 자체가 아예 없습니다..)
- Github 계정이 있다면 [프로젝트 저장소](https://github.com/KRFOSS)에 기여할 수 있습니다.
- 혹은 프로젝트를 위해 장비를 기여하거나 기부할 수도 있습니다.
- 그 외 다양한 기여

### 기업 및 단체

기업 및 단체라면 더 많은 것들을 할 수 있습니다. ROKFOSS 미러가 더 많은 것들을 미러링하여 제공할 수 있도록 하드웨어(대용량 SSD, HDD)를 제공하거나 이 모든 것들이 포함된 고성능 서버를 지원할 수 있습니다. 

우리 프로젝트는 다음과 같은 지원을 환영합니다!

- 국내에 위치한 보안 옵션이 포함된 고성능 CDN
- 서버 및 네트워크 인프라 지원. (고성능 CPU와 많은 램, 대용량 HDD가 장착된 서버와 2.5G 이상의 QoS 없는 안정적인 네트워크)
- 프로젝트 비용 지원
- 그 외 다양한 지원

우리의 프로젝트를 후원하고자 한다면 공식 메일인 krfoss@krfoss.org로 연락해주시면 최대한 빠르게 회신을 드리도록 하겠습니다.

_그 외에 우리의 프로젝트를 지지하고 응원해주시는 모든 분들께 진심으로 감사드립니다!_ 