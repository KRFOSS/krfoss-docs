---
title: SSH 포트 변경하기
description: 
icon:
weight:
author: ["kmw0410"]
---

{{% notice info %}}
이 가이드는 `openssh-server` 패키지가 설치된 서버에서만 작동해요.
{{% /notice %}}

우선 SSH를 사용하여 SSH 포트를 변경하려는 서버에 접속하세요. (GUI의 경우 터미널을 열어도 돼요)

```bash
ssh 유저이름@IP주소
```

그런 다음 `/etc/ssh/sshd_config` 파일을 나노(nano) 에디터나 vi 에디터로 여세요.

```bash
# nano 에디터의 경우:
sudo nano /etc/resolv.conf

# vi(vim) 에디터의 경우:
sudo vi /etc/resolv.conf
```

열게 되면 다음과 같은 내용이 보이게 돼요.

```plaintext
# This is the sshd server system-wide configuration file.  See
# sshd_config(5) for more information.

# This sshd was compiled with PATH=/usr/local/bin:/usr/bin:/bin:/usr/games

# The strategy used for options in the default sshd_config shipped with
# OpenSSH is to specify options with their default value where
# possible, but leave them commented.  Uncommented options override the
# default value.

Include /etc/ssh/sshd_config.d/*.conf

#Port 22
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::

...
```

이제 `#Port 22` 부분의 주석(#)을 없애고 22를 원하는 포트로 변경 후 다음 명령어를 입력하여 ssh 서비스를 재시작하세요. (**SSH 클라이언트로 접속한 경우 연결이 끊겨요**)

```bash
systemctl restart sshd
```

변경이 완료 되었으면 SSH 접속 명령어에 `-p`를 붙여 변경된 포트로 접속이 되는지 확인하세요.

```bash
ssh 유저이름@IP주소 -p 포트번호
```