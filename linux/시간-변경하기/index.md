---
title: 시간 변경하기
description: 
icon:
weight:
author: ["kmw0410"]
---

우선 SSH를 사용하여 시간을 변경하려는 서버에 접속하세요. (GUI의 경우 터미널을 사용해도 돼요)

```bash
ssh 유저이름@IP주소
```

그런 다음 **timedatectl을 입력**하여 `Local time`과 `Time zone`이 올바르게 설정되어 있는지 확인하세요.

```plaintext
               Local time: Sat 2025-05-24 12:43:47 KST
           Universal time: Sat 2025-05-24 03:43:47 UTC
                 RTC time: Sat 2025-05-24 03:43:47
                Time zone: Asia/Seoul (KST, +0900)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```

위 두 개의 항목이 설정되어 있다면 NTP 또는 RTC에 의해 자동으로 설정된 것이지만, 다른 시간대로 되어있다면 아래 명령어 중 하나의 명령어를 입력하세요:

```bash
# NTP(네트워크 타임 프로트콜)를 사용할 경우:
sudo timedatectl set-ntp true

# 직접 시간대를 설정할 경우:
sudo timedatectl set-timezone Asia/Seoul
```

정상적으로 설정 되었다면 이제 **다시 timedatectl을 입력**하여 정상적으로 변경되었는지 확인해 보세요.