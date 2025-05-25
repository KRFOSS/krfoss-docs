---
title: 호스트 이름 변경하기
description: 
icon:
weight:
author: ["kmw0410"]
---

{{% notice info %}}
이 가이드는 대부분의 리눅스에서 바로 따라할 수 있지만, `Devuan, Alpine Linux나 BusyBox 기반 이미지`를 사용하는 OS(환경)에서는 사용이 불가할 수 있어요.
{{% /notice %}}

우선 SSH를 사용하여 호스트 이름을 변경하려는 서버에 접속하세요. (GUI의 경우 터미널을 열어도 돼요)

```bash
ssh 유저이름@IP주소
```

그런 다음 쉘을 확인하여 현재 호스트 이름을 확인해보세요. 기본적으로 `유저이름@호스트이름` 형식으로 되어있어요 (아래 예시에서는 localhost가 호스트 이름이에요)

```plaintext
[root@localhost] $
```

이제 호스트 이름을 변경할 차례에요. 다음 명령어를 입력하여 원하는 이름으로 변경하세요.

```bash
sudo hostnamectl set-hostname 원하는-이름
```

변경 하였으면 **Ctrl + A + D 또는 exit 명령어를 입력**하거나 **SSH 클라이언트(또는 터미널)을 재시작**하여 다시 로그인하고, 변경된 이름으로 바뀌었는지 확인해보세요 (아래 예시에서는 localhost에서 krfoss로 변경했어요)

```plaintext
[root@krfoss] $
```
