---
title: Proxmox 버전 업그레이드하기 (8 → 9)
description: 
icon: ti-pin
weight: 30
author: ["kmw0410"]
---

{{% notice info %}}
시스템에 따라 업그레이드로 인해 커널 패닉이나 재부팅되는 문제가 발생될 수 있어요. 업데이트 시 주의하세요.

본 문서의 내용은 Proxmox 공식 위키 - [Updgrade from 8 to 9](https://pve.proxmox.com/wiki/Upgrade_from_8_to_9)의 주요 내용만 다르고 있어요. 이 문서에 없는 내용이나 알려진 이슈, 문제 해결은 직접 들어가 확인해 주세요.
{{% /notice %}}

# 소개
Proxmox VE 9.x는 여러 개의 새롭고 주요한 기능들이 포함되어 있어요. 업그레이드를 계획하고 있다면 조심해야 해요. 시작하기 전에 백업을 만들었는지 확인하고, 광범위하게 테스트하세요. 기존 구성에 따라 다운타임을 포함한 여러 수동 작업이 필요할 수 있어요.

**메모**: 유효하고 테스트된
# 새로운 설치
 - **VM과 컨테이너를 외장 스토리지에 백업** ([백업과 복구](https://pve.proxmox.com/wiki/Backup_and_Restore)를 확인하세요)
 - **/etc의 모든 파일을 백업**: /etc/pve에 있는 파일들, 그리고 `/etc/passwd`, `/etc/network/interfaces`, `/etc/resolv.conf` 파일들, 그리고 기본 설치 구성과 다른 모든 설정 파일들이 필요해요.
 - ISO를 통해 최신의 Proxmox VE 9.x 설치 (기존 호스트의 모든 데이터가 지워져요) 백업은 *항상* 업그레이드 프로세스 전에 필요해요. 사전에 테스트 환경에서 백업을 테스트하세요.

시스템이 사용자 지정 되었거나 추가 패키지 또는 기타 타사 저장소/패키지를 사용하는 경우 해당 패키지도 Debian Trixie로 업그레이드되어 호환되는지 확인해야 해요.

일반적으로, Proxmox VE 8.x 시스템을 Proxmox VE 9.x로 업그레이드 하는 방법은 두 가지가 있어요:

 - 새 하드웨어에서 설치 (VM을 백업에서 복구)
 - apt를 통한 기존 환경 업그레이드 (단계별)

# 새로운 설치
 - **VM과 컨테이너를 외장 스토리지에 백업** ([백업과 복구](https://pve.proxmox.com/wiki/Backup_and_Restore)를 확인하세요)
 - **/etc의 모든 파일을 백업**: /etc/pve에 있는 파일들, 그리고 `/etc/passwd`, `/etc/network/interfaces`, `/etc/resolv.conf` 파일들, 그리고 기본 설치 구성과 다른 모든 설정 파일들이 필요해요.
 - ISO를 통해 최신의 Proxmox VE 9.x 설치 (기존 호스트의 모든 데이터가 지워져요)
 - 브라우저의 캐시를 비우거나 강제 리로드(`CTRL` + `SHIFT` + `R`, MacOS는 `⌘` + `Alt` + `R`)
 - 적용 가능한 경우 클러스터 재구축
 - `/etc/pve/storage.cfg` 파일 복원 (이렇게 하면 백업에 사용되는 외부 저장장치에 접근할 수 있어요)
 - `/etc/pve/firewall/`과 `/etc/pve/nodes/<nodes>/host.fw` 방화벽 파일 복원 (적용 가능한 경우)
 - 모든 VM을 백업에서 복원 ([백업과 복구](https://pve.proxmox.com/wiki/Backup_and_Restore)를 확인하세요)

# 중대한 변경 사항
중대한 (API) 변경 사항에 대해서는 [릴리즈 노트](https://pve.proxmox.com/wiki/Roadmap#9.0-known-issues)를 참조하세요.

# 기존 환경 업그레이드
## 사전 요구 사항
 - 모든 노드가 최신 버전의 Proxmox VE 8.4로 업그레이드하세요: 만약 pve-manager 버전이 최소 `8.4.1` 이상이 아니라면, 노드 패키지 구성이 올바른지 확인하세요. (웹 UI -> 레포지토리)
 - 하이퍼컨버저드(Hypr-converged) Ceph: Proxmox VE를 9.0으로 업그레이드 **하기 전에** 모든 Ceph Quincy 또는 Ceph reef 클러스터를 Ceph 19.2 Squid로 업그레이드해야 해요. ([Ceph Quincy → Reef](https://pve.proxmox.com/wiki/Ceph_Quincy_to_Reef), [Ceph Reef → Squid](https://pve.proxmox.com/wiki/Ceph_Reef_to_Squid) 가이드를 따르세요)
 - 동시 설치된 Proxmox Backup Server: [Proxmox Backup Server을 3에서 4로 업그레이드 하는 방법](https://pbs.proxmox.com/wiki/index.php/Upgrade_from_3_to_4)을 확인하세요
 - 노드에 대한 신뢰할 수 있는 접근: 호스트에 독립적인 채널(예:IKVM/IPMI)이나 물리적 접근을 통해 노드에 접근할 수 있어야 해요. SSH만 가능한 경우, 먼저 동일하지만 비운영 머신에서 업그레이드를 시험해 보는 것을 권장하며 만약 연결이 중단되더라도 업그레이드가 계속 진행될 수 있도록, `tmux`나 `screen` 같은 터미널 멀티플렉서 사용을 권장해요.
 - 정상적인 상태의 클러스터
 - 모든 VM과 CT의 유요하고 검증된 백업 (문제가 발생했을 경우를 대비)
 - 루트 마운트 지점의 최소 5GB의 디스크 여유 공간, 이상적으로는 10GB 이상
 - [알려진 업그레이드 이슈](https://pve.proxmox.com/wiki/Upgrade_from_8_to_9#Known_Upgrade_Issues) 확인

## 단계별 행동
클러스터 내 각 Proxmox VE 노드의 명령줄에서 수행해야 해요.

**콘솔 또는 SSH를 통해 작업하세요. SSH 연결이 중단되는 것을 방지 하기 위해 콘솔을 사용하는 것이 바람직해요. GUI에서 제공하는 가상 콘솔을 통해 연결된 상태에서는 업그레이드를 수행하지 말아야 하며 업그레이드 중 연결이 중단될 수 있어요. SSH만 사용 가능한 경우, 연결이 중단되는 문제를 방지하기 위해 터미널 멀티플렉서(예: tmux 또는 screen) 사용을 고려하세요.**

진행하기 전에 모든 VM 및 CT에 대한 유효한 백업이 생성되었는지 확인하는 것을 잊지 마세요.

### pve8to9 체크리스트 스크립트를 지속적으로 사용하세요
최신 Proxmox VE 8.4 패키지에는 `pve8to9`라는 작언 체크리스트 프로그램이 포함되어 있어요. 이 프로그램은 업그레이드 과정 전, 도중 및 후에 발생할 수 있는 잠재적 문제에 대한 힌트와 경고를 제공해요. 다음 명령어를 실행하여 호출할 수 있어요:

```bash
pve8to9
```

**모든** 검사를 활성화한 상태로 실행하려면, 다음을 실행하세요:

```bash
pve8to9 --full
```

업그레이드 전에 반드시 전체 검사를 한 번 이상 실행하세요.

이 스크립트를 오직 문제점을 **확인**하고 보고할 뿐이에요. 기본적으로 시스템에 변경 사항이 적용되지 않으므로, 어떤 문제도 자동으로 해결되지 않아요. Proxmox VE는 매우 다양하게 커스터마이징될 수 있으므로, 특정 설정에서 발생할 수 있는 모든 문제를 스크립트가 인식하지 못할 수 있다는 점을 유념해야 해요!

문제를 해결하려고 시도한 후에는 스크립트를 다시 실행하는 것이 좋아요. 이렇게 하면 수행한 작업이 해당 경고를 실제로 해결했는지 확인할 수 있어요.

### 중요한 가상 머신 및 컨테이너 이동
VM 또는 CT가 업그레이드 하는 동안 계속 구동되어야 하는 경우 업그레이드 대상 노드에서 해당 VM 및 CT를 마이그레이션하세요.

클러스터 업그레이드를 계획 시 유의해야 할 마이그레이션 호환성 규칙:
 - Proxmox VE의 구버전에서 신버전으로 VM 또는 CT를 마이그레이션하는 것은 항상 가능해요.
 - 신버전의 Proxmox VE에서 구버전으로의 마이그레이션은 가능할 수 있지만 일반적으로 지원하지 않아요.

### 구성된 APT 저장소 업데이트
우선, 시스템이 최신의 Proxmox VE 8.4 패키지를 사용하도록 하세요:

```bash
apt update
apt dist-upgrade
pveversion
```

마지막 명령어는 최소 `8.4.1` 또는 이상 버전을 보고해야 해요.

**메모**: 하이퍼컨버저드(Hyper-converged) Ceph 환경에서는 Ceph Squid(버전 19)를 실행 중인지 확인하세요. `ceph --version` 출력을 확인하여 확인할 수 있어요. Ceph Squid를 실행 중이지 않은 경우, [Ceph Reef → Squid 업그레이드 가이드](https://pve.proxmox.com/wiki/Ceph_Reef_to_Squid)를 참고하여 먼저 업그레이드를 완료하세요. Ceph Squid로 업그레이드하기 전에는 어떤 단계도 진행하지 마세요.

**데비안 기본 저장소를 Trixie로 업데이트하기**

모든 데비안과 Proxmox VE의 저장소 항목들을 Trixie로 업데이트하세요.

```bash
sed -i 's/bookworm/trixie/g' /etc/apt/sources.list
sed -i 's/bookworm/trixie/g' /etc/apt/sources.list.d/pve-enterprise.list
```

데비안 Bookworm 전용 저장소가 남아 있지 않은지 확인하세요. 남아 있는 경우 해당 저장소 행의 시작 부분에 `#` 기호를 추가하여 주석 처리할 수 있어요. `/etc/apt/sources.list`와 `/etc/apt/sources.list.d/pve-enterprise.lit`의 모든 항목들을 확인하고, 올바른 Proxmox VE 9 / 데비안 Trixie 저장소는 [패키지 저장소](https://pve.proxmox.com/wiki/Package_Repositories)를 확인하세요.


**Proxmox VE 9 패키지 저장소 추가하기**

엔터프라이즈 저장소를 사용 중인 경우, Proxmox VE 9 엔터프라이즈 저장소를 새로운 deb822 형식에 맞게 추가할 수 있어요. 다음 명령어를 실행하여 관련된 `pve-enterprise.sources` 파일을 생성하세요:

```bash
cat > /etc/apt/sources.list.d/pve-enterprise.sources << EOF
Types: deb
URIs: https://enterprise.proxmox.com/debian/pve
Suites: trixie
Components: pve-enterprise
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
EOF
```

위와 같이 새 엔터프라이즈 저장소를 추가한 후, `apt`가 이를 올바르게 인식하는지 확인해야 해요. 먼저 `apt update`를 실행한 다음 `apt policy`를 실행하여 확인할 수 있어요. 오류가 표시되지 않고 `apt policy`가 원하는 저장소만 표시하는지 확인하세요. 그런 다음 기존 `/etc/apt/sources.list.d/pve-enterprise.list` 파일을 제거할 수 있어요. 기존 저장소가 확실히 제거되었는지 확인하기 위해 `apt update`와 `apt policy`를 실행하세요.

비구독(no-subscription) 저장소를 사용 중인 경우, [패키지 저장소](https://pve.proxmox.com/wiki/Package_Repositories)를 확인하세요. Proxmox VE 9 비구독 저장소를 다음 명령어로 추가할 수 있어요:

{{% notice info %}}
빠른 업데이트를 위해 비구독 한정으로 ROKFOSS 분산미러를 이용할 수 있어요.

엔터프라이즈는 안정성을 위해 원래 저장소를 유지하는 것이 좋아요.
{{% /notice %}}

```bash
cat > /etc/apt/sources.list.d/proxmox.sources << EOF
Types: deb
URIs: https://http.krfoss.org/proxmox/debian/pve
Suites: trixie
Components: pve-no-subscription
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
EOF
```

엔터프라이즈 저장소와 마찬가지로, `apt update` 명령어 실행 후 `apt policy` 명령어로 apt가 저장소를 올바르게 인식하는지 확인해야 해요. 그런 다음 `/etc/apt/sources.list`, `/etc/apt/sources-list.d/pve-install-repo.list`를 추가한 다른 `.list` 파일에서 기존 Proxmox VE 8 비구독 저장소를 제거하세요. 기존 저장소가 제거되었는지 확인하기 위해 `apt update`와 `apt policy`를 다시 실행하세요.

**Ceph 패키지 저장소 업데이트하기**

**메모**: 하이퍼컨버저드(Hyper-converged) Ceph 환경에서만 해당돼요. 확실하지 않은 경우 해당 노드의 웹 UI에서 Ceph 패널과 구성된 저장소를 확인하세요.

ceph.com 저장소를 proxmox.com ceph 저장소로 모두 교체하세요.

**메모**: 지금 시점에서 Proxmox VE에 직접 설치된 하이퍼컨버저드(Hyper-converged) Ceph 클러스터는 **Ceph 19.2 Squid**를 실행해야 하며, 그렇지 않으면 데비안 13 Trixie에서 Proxmox VE 9로 업그레이드하기 전에 Ceph를 먼저 업그레이드해야 해요. 각 노드의 Proxmox VE 웹 UI에서 Ceph 패널을 통해 현재 Ceph 버전을 확인할 수 있어요.

Proxmox VE 8부터는 Ceph용 엔터프라이즈 저장소도 제공되며, 이는 상용 환경 구축에 최적의 선택이에요. 아래 명령어를 실행하여 새로운 deb822 형식의 Trixie 기반 Ceph 엔터프라이즈 저장소를 추가하세요:

```bash
cat > /etc/apt/sources.list.d/ceph.sources << EOF
Types: deb
URIs: https://enterprise.proxmox.com/debian/ceph-squid
Suites: trixie
Components: enterprise
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
EOF
```

먼저 `apt update`를 실행한 후 `apt policy`를 실행하여 `apt`가 올바르게 인식하는지 확인하세요. 오류가 없어야 하며, `apt policy` 출력에 새 저장소가 정상적으로 표시되어야 해요. 그런 다음 기존 `/etc/apt/sources.list.d/ceph.list` 파일을 제거할 수 있어요. `apt update`를 실행한 후 다시 `apt policy`를 실행하여 파일이 제대로 제거되었는지 확인할 수 있어요.

업데이트 중 401 에러로 실패할 경우, ceph에 새로운 접근 권한이 부여되도록 구독을 먼저 새로 고쳐야 할 수 있어요. 웹 UI나 `pvesubscription update --force` 명령어로 수행하세요.

아무 구독이 없는 경우 `비구독(no-subscription)` 저장소를 사용할 수 있으며, 다음 명령어를 따라 새로운 deb822 형식으로 추가하세요:

{{% notice info %}}
빠른 업데이트를 위해 비구독 한정으로 ROKFOSS 분산미러를 이용할 수 있어요.

엔터프라이즈는 안정성을 위해 원래 저장소를 유지하는 것이 좋아요.
{{% /notice %}}

```bash
cat > /etc/apt/sources.list.d/ceph.sources << EOF
Types: deb
URIs: https://http.krfoss.org/proxmox/debian/ceph-squid
Suites: trixie
Components: no-subscription
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
EOF
```

엔터프라이즈 저장소와 마찬가지로, `apt update` 명령어 실행 후 `apt policy` 명령어로 apt가 저장소를 올바르게 인식하는지 확인해야 해요. 그런 다음 기존 `/etc/apt/sources.list.d/ceph.list` 파일을 제거할 수 있어요.

백포트 저장소 라인이 있다면 이를 제거하세요. 백포트 저장소의 패키지가 설치된 상태에서는 업그레이드가 테스트되지 않아요.

**패키지 목록 새로고침하기**

저장소의 패키지 목록을 업데이트하고 오류가 보고되지 않은지 확인하세요:

```bash
apt update
```

### 시스템을 데비안 Trixie와 Proxmox VE 9.0으로 업그레이드하기
이 단계 완료에 소요되는 시간은 시스템의 성능, 특히 루트 파일시스템의 [IOPS](https://ko.wikipedia.org/wiki/%EC%95%84%EC%9D%B4%EC%98%B5%EC%8A%A4) 및 대역폭에 크게 좌우돼요. 느린 하드 디스크의 경우 최대 60분 이상 걸릴 수 있는 반면, SSD 저장 장치를 갖춘 고성능 서버에서는 dist-upgrade가 5분 이내로 완료될 수 있어요.

업그레이드된 패키지의 초기 세트를 얻으려면 이 단계부터 시작하세요:

```bash
apt dist-upgrade
```

위 단계에서, 기본 구성이 해당 패키지에 의해 업데이트된 경우 구성 파일 변경 및 일부 서비스 재시작을 승인하라는 메시지가 표시돼요.

apt-listchanges 출력이 표시될 수도 있으며, 이 경우 "q"를 눌러 종료하세요. 기본 키보드 선택을 묻는 메시지가 나타나면 화살표 키로 해당 키보드를 선택한 후 엔터 키를 누르세요.

서비스 재시작 관련 질문이 있을 경우, 업그레이드 후 재부팅시 모든 서비스가 깨끗하게 재시작되므로 확실하지 않다면 기본값을 사용해야 해요.

### 결과 확인 & 업데이트된 커널로 재부팅하기

dist-upgrade 커맨드가 성공적으로 끝났다면, `pve8to9` 체크 스크립트 및 시스템을 재부팅하여 새로운 Proxmox VE 커널을 사용하는지 다시 확인하세요.

Proxmox VE 8의 옵트인 패키지를 통해 이전에 6.14 커널을 사용한 적이 있더라도 재부팅해야 해요. 업데이트된 커널은 최신 Proxmox VE 9 컴파일러 및 ABI 버전으로 (재)빌드되었으므로, 시스템의 나머지 부분과의 최상의 호환성을 보장하기 위해 재부팅이 필요해요.

# Proxmox VE 업그레이드를 완료하고 난 뒤
브라우저 캐시를 캐시를 비우거나 웹 UI를 강제 리로드하세요. (`CTRL` + `SHIFT` + `R`, macOS의 경우 `⌘` + `Alt` + `R`)

---

## 업그레이드 도중 문제가 발생하였나요?
공식 위키에서 다음 항목들을 확인해 보세요:
 - [체크리스트 문제](https://pve.proxmox.com/wiki/Upgrade_from_8_to_9#Checklist_issues)
 - [알려진 업그레이드 문제](https://pve.proxmox.com/wiki/Upgrade_from_8_to_9#Known_Upgrade_Issues)
 - [문제 해결](https://pve.proxmox.com/wiki/Upgrade_from_8_to_9#Troubleshooting)