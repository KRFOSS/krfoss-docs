---
title: 공식 미러로 등록하기
description: 구축한 미러를 공식 미러로 등록하여 많은 사용자들에게 알릴 수 있어요.
icon:
weight:
author: ["xenix4845",]
---

미러 서버를 운영하고 있다면 공식 미러로 등록해보는 것을 고려해보세요. 공식 미러로의 등록은 지역 사용자들에게 기여할 수 있고, 해당 FOSS 프로젝트에 기여할 수 있게 됩니다.

그러나 등록하기 전 다음의 조건을 만족하고 있는지 스스로 점검하세요.

- 미러를 단순 이력서(포트폴리오), 학습용으로 단기간 운영할 계획은 아닌지
- 미러를 1년 이상 유지할 계획이 있고 지속적으로 운영할 의지가 충분한지
- 서버 네트워크가 200Mbps 이상의 속도를 낼 수 있는지(100메가 미러는 운영자와 사용자 모두를 위해 받아줄 수 없어요)
- 미러 서버가 라즈베리파이 같은 저사양 기기가 아닌 4코어, 8GB RAM 이상인지
    - 단순히 말해서 여러분의 서버가 AMD 라이젠의 경우 2세대, 인텔의 경우 최소 N100
- 서버 업타임이 최근 7일 이상 합계 12시간 이상 불량한적은 없었는지.
    - 다만 전시상황, 테러에 의한 피해, 지역/건물 정전으로 인핸 전기공급 문제, 압수수색 등 법집행기관의 법집행, ISP에 의한 네트워크 장애, 장비의 수명문제 또는 불량, 화재, 정부명령, 날씨로 인한 문제는 피할 수 없으니 제외해요.
    - 추가로 북한의 국경과 매우 가까운 국경지역에서는 미러를 운영하지 마세요. 
- 미러를 운영하고 유지보수하는데에 필요한 정말 최소한의 리눅스 지식을 갖고 있는지. 


{{% notice info %}}
가장 먼저 국내 미러라면 [ROKFOSS PROJECT 미러 등록폼](https://form.krfoss.org)에서 미러를 등록해보는 것을 고려해보세요. 프로젝트 공식 미러로 등록하게 되면 가까운 지역 사용자들에게 기여할 수 있으며, 대한민국 미러 네트워크 대역폭에도 큰 기여를 할 수 있게 됩니다.
{{% /notice %}}


# ROKFOSS 프로젝트에 등록하는 방법

위에 설명되었듯이 ROKFOSS 프로젝트에 등록하여 지역과 프로젝트 등 많은 곳에 기여하고자 한다면 [ROKFOSS PROJECT 미러 등록폼](https://form.krfoss.org)에서 등록을 진행하거나, [공식 디스코드 채널](https://chat.krfoss.org)에서 등록하는 방법을 고려해볼 수 있습니다. 가장 빠른 것은 공식 디스코드에서 진행하는 것입니다!

프로젝트에 공식 미러로 등록하게 된다면 분산미러를 통해 트래픽을 분산할 수 있고 지역 사용자들에게 기여할 수 있습니다. 

분산미러에 대해서 더 알아보고 싶다면 [ROKFOSS 분산미러 웹사이트](https://http.krfoss.org)를 방문해주세요!

# 우분투 공식 미러로 등록하기

우분투 공식 미러로 등록하고자 한다면 https://launchpad.net 에서 등록을 진행할 수 있습니다. 간단한 회원가입을 진행하면 [https://launchpad.net/ubuntu/+newmirror](https://launchpad.net/ubuntu/+newmirror)에서 미러를 등록하세요. 며칠 내로 우분투 크롤러가 방문하여 여러분의 미러를 확인하고 문제가 없다면 자동으로 공식 미러로 등록이 될거에요.

# 칼리리눅스 공식 미러로 등록하기

칼리 리눅스 공식 미러로 등록하는 것은 국내 칼리리눅스 사용자들에게 크게 기여할 수 있는 매우 좋은 방법입니다. 그렇지만 칼리 리눅스는 다른 FOSS 프로젝트들과 달리 특별한 방식으로 미러 시스템을 운영하고 있어요. 이 시스템이 요구하는 조건을 맞추고 있는지 [칼리리눅스 미러 구축하기 문서](https://kali.krfoss.org/community/setting-up-a-kali-linux-mirror/)를 확인해주세요.

만약 기준을 모두 충족하고 있다면 즉시 칼리팀에 연락하여 미러 등록을 진행하거나 프로젝트 디스코드에 참여하여 도움을 받을 수 있어요. 다만 직접 연락하게 되는 경우 미러 구성에서 문제가 있다면 많은 시간을 날릴 수 있으니 프로젝트 디스코드에 참여하여 도움을 받아보는 걸 권장해요.

# OPNsense 공식 미러로 등록하기

OPNsense는 미러를 운영하고 있다면 즉시 OPNsense([project@opnsense.org](project@opnsense.org))에 연락하여 등록하실 수 있어요. 현재 등록된 미러 리스트를 확인하려면 [https://opnsense.org/download/](https://opnsense.org/download/)를 방문해보세요. 모든 미러가 ROKFOSS 프로젝트에 참여하고 있으며 프로젝트에서 등록과 관리를 진행하고 있어요. 

# 아치리눅스 공식 미러로 등록하기

### 깃랩 가입하기
아치리눅스의 경우 자체 깃랩을 운영하고 있어 계정 등록이 필요해요. 우선 [accountsupport@archlinux.org](mailto:accountsupport@archlinux.org)에 아래 예시를 참고하여 메일을 보내고 등록까지 기다리세요.

```
[제목] Request for account creation

Hello,
I would like request for creation of a user account with the "MYUSERNAME" on gitlab
Please let me know if you need any additional information
Thank you.
```

위와같이 보내면 깃랩 운영진에게 계정 생성이 완료되었다는 말과 함께 링크가 담긴 메일을 받게돼요. 해당 링크로 들어가세요.

링크로 들어가게 되면 기본적인 정보를 입력하게 되고 마지막 항목에 **What is the output of: LC_ALL=C pacman -V|tail -n3|base32|head -1 ?** 라는 질문이 있는데 이 경우 아치리눅스 실행이 필요해요. WSL이나 VM, 네이티브 환경에서 다음 명령어를 입력 후 나오는 결과를 붙여넣으세요:

```bash
LC_ALL=C pacman -V|tail -n3|base32|head -1
```

이제 가입이 완료되었어요. 자유롭게 둘러보세요.

### 미러 신청하기
깃랩 가입이 완료되었다면, 이제 공식 미러 신청을 할 수 있어요. 신청 전 다음 조건을 확인해 보세요:

1. 최소 100GB 이상의 디스크 확보 필요 ([관련 문서](https://docs.krfoss.org/mirror/%EB%A6%AC%EB%88%85%EC%8A%A4%20%EB%AF%B8%EB%9F%AC%20%EC%84%9C%EB%B2%84%20%EA%B5%AC%EC%B6%95%ED%95%98%EA%B8%B0/)에서 용량을 확인해 보세요)
2. RSYNC의 경우 rchlinux.org가 아닌 티어 1 미러에서 동기화 필요 (전체 동기화를 진행하여야 하며 일부 동기화 불가)
3. FTP 미러는 지원하지 않으며 HTTP, HTTPS, RSYNC만 지원 (RSYNC의 경우 선택)

위 조건이 모두 충족된다면 이제 미러를 신청할 차례에요. [아치리눅스 미러 이슈](https://gitlab.archlinux.org/archlinux/arch-mirrors/-/issues)로 이동 후 **New issue**를 누르세요.

다음 신청 양식에 맞게 작성하고 이슈를 등록하세요, 처음 신청하는 경우 티어 2로 작성하면 돼요:

```
[제목] New Tier 1(2) Mirror (South Korea)

- Geographical location of the mirror: South Korea (또는 상세 위치 (예: Busan, South Korea))
- URL of the mirror: 
http://미러도메인/archlinux
https://미러도메인/archlinux
rsync://미러도메인/archlinux (선택)
- Bandwidth of the mirror: 현재 사용중인 인터넷 회선 (예: 1Gbps)
- An administrative contact email: 연락받을 메일주소
- Upstream sync: rsync://티어1-미러주소/archlinux
```

올바르게 작성하였고, 미러가 정상 운영 중이라면 최대 7일 내로 답변을 받게되며 정상 등록이 되었다면 https://archlinux.org/mirrors/미러주소 에서 확인할 수 있어요.

---

추가적으로 여러분이 알고 있는 공식 등록 경로가 있다면 이곳에 기여해보는 것을 고려해주세요!