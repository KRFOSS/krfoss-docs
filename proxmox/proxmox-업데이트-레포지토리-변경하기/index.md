---
title: Proxmox 업데이트 레포지토리 변경하기
description: 
icon: ti-pin
weight: 30
author: ["kmw0410"]
---

Proxmox에서 노드를 선택하고 좌측 메뉴에서 **Shell**을 여세요. (노드 이름은 설치시 지정한 호스트네임을 뜻해요)

![](./1.png)

그런 다음 아래 명령어를 입력하여 국내 미러로 변경하세요:
```
curl -sSL https://http.krfoss.org/pack/prx2.sh | bash
```

변경이 완료되었다면 Proxmox 관련 서비스가 재시작되며 업데이트를 국내 미러에서 받을 수 있어요.