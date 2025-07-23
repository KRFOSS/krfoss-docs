---
title: RF-DNS
description: ROKFOSS DNS
icon:
weight:
author: ["DevNergis"]
---

{{% notice info %}}
RF-DNS는 현재 베타 릴리즈 입니다.

DNS가 불안정할 수 있으며 정상 작동하지 않는 기능이 있을 수 있습니다.

DoQ(DNS over QUIC)은 매우 불안정합니다, 실제로 사용하지 않을 것을 권장합니다.
{{% /notice %}}

## RF-DNS는 무엇인가요?

RF-DNS 는 ROKFOSS 프로젝트에서 제공하는 공개 DNS 입니다.

현재 분산 DoT(DNS over TLS)와 DoH(DNS over HTTPS)를 개발중 입니다.

{{% notice info %}}
[`DNS`란?](https://www.cloudflare.com/ko-kr/learning/dns/what-is-dns/)
{{% /notice %}}

## RF-DNS의 특징

### CF-Hello

**CF-Hello**기능을 이용해 모두 접속자의 주변 서버(ICN)로 연결해요.

Cloudflare를 사용하는 웹사이트는 더 빠르게 로드되고, 지연시간이 줄어듭니다.

### DNS Pass-through

**DNS Pass-through**기능을 이용하면 사용자가 원래 사용하던 DNS `ex) AdGuard DNS` 등을 그대로 사용하면서 **CF-Hello**기능을 이용할수 있어요.

현재 `DNS over HTTPS` 에서만 지원해요.

`https://dns.dev.c01.kr/원래_사용하던_DNS의_DoH의_FQDN`

ex) `https://dns.dev.c01.kr/dns.adguard-dns.com`

## 지원하는 프로토콜

Plain: `152.70.36.246`

DNS over HTTPS: `https://dns.dev.c01.kr/dns-query`

DNS over TLS: `dns.dev.c01.kr`

DNS over QUIC: `dns.dev.c01.kr`
