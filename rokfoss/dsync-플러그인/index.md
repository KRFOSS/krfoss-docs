---
title: DSYNC 플러그인 가이드
description: 
icon:
weight:
author: ["KRFOSS","xenix4845"]
---
## DSYNC 플러그인 개발 가이드

## 목차
1. [플러그인 시스템 개요](#플러그인-시스템-개요)
2. [플러그인 권한 제어 (plugins-power)](#플러그인-권한-제어-plugins-power)
3. [플러그인 설치 방법](#플러그인-설치-방법)
4. [플러그인 파일 구조](#플러그인-파일-구조)
5. [플러그인 타입별 예제](#플러그인-타입별-예제)
   - [웹훅 플러그인 (webhook)](#1-웹훅-플러그인-webhook)
   - [실행 플러그인 (exec)](#2-실행-플러그인-exec)
   - [이메일 플러그인 (email)](#3-이메일-플러그인-email)
   - [스크립트 플러그인 (script)](#4-스크립트-플러그인-script)
6. [옵션형 플러그인 (Option-Based Plugins)](#옵션형-플러그인-option-based-plugins)
   - [옵션 대체 시스템 (Option Override System)](#옵션-대체-시스템-option-override-system)
7. [시간 형식화 함수](#시간-형식화-함수)
8. [고급 기능: DSP 함수](#고급-기능-dsp-함수-v20)
9. [플러그인 간 함수 공유](#플러그인-간-함수-공유-cross-plugin-function-sharing)
10. [플러그인 API 참조](#플러그인-api-참조)
11. [유용한 팁](#유용한-팁)

## 플러그인 시스템 개요

DSYNC 플러그인 시스템은 동기화 작업이 완료된 후 사용자 정의 작업을 실행할 수 있게 해줍니다. 마인크래프트 서버 플러그인과 유사한 개념으로, 사용자가 직접 만든 `.dsp` 파일을 `~/dxync/plugins/` 폴더에 넣으면 자동으로 로드됩니다.

미러 서버 유지에 매우 큰 도움이 되는 플러그인 등을 개발하여 사용할 수 있습니다.

### 주요 특징
- ✅ **간단한 문법**: Go와 유사한 key=value 형식
- ✅ **4가지 플러그인 타입**: webhook, email, exec, script
- ✅ **Python/Bash 스크립트 지원**: indentation 보존으로 Python 완벽 지원
- ✅ **외부 설정 파일**: 민감한 정보를 코드와 분리하여 관리
- ✅ **옵션형 플러그인**: 동기화 없이 독립 실행 가능한 CLI 옵션 추가
- ✅ **우선순위 시스템**: 여러 플러그인의 실행 순서 제어
- ✅ **컨텍스트 정보 제공**: 동기화 결과, 통계 등의 정보 활용
- ✅ **시간 형식화 함수**: 타임스탬프를 읽기 쉬운 형식으로 변환
- ✅ **보안 제어**: plugins-power 설정으로 실행 권한 관리

## 플러그인 권한 제어 (plugins-power)

DSYNC는 보안을 위해 플러그인 타입별 실행 권한을 제어하는 시스템을 제공합니다. 이는 `xync.conf`의 `plugins-power` 설정으로 관리됩니다.

### 권한 모드

#### 1. 제한 모드 (plugins-power=false) - 기본값

**허용되는 플러그인 타입:**
- ✅ **webhook** - 웹훅 API 호출 (HTTP POST 요청만 가능)
- ✅ **email** - SMTP 이메일 전송 (메일 전송만 가능)

**차단되는 플러그인 타입:**
- ❌ **exec** - 외부 프로그램 실행 (보안상 차단)
- ❌ **script** - 스크립트 파일 실행 (보안상 차단)

**적용 대상:**
- 프로덕션 서버
- 신뢰할 수 없는 플러그인이 포함된 환경
- 네트워크 통신만 필요한 경우

**예시 설정 (~/dxync/xync.conf):**
```ini
# 제한 모드 (기본값)
plugins-power=false
```

#### 2. 전체 권한 모드 (plugins-power=true)

**모든 플러그인 타입 허용:**
- ✅ **webhook** - 웹훅 API 호출
- ✅ **email** - SMTP 이메일 전송
- ✅ **exec** - 외부 프로그램 실행
- ✅ **script** - 스크립트 파일 실행

**적용 대상:**
- 신뢰할 수 있는 개발 환경
- 로컬 테스트 서버
- 고급 자동화가 필요한 경우

**예시 설정 (~/dxync/xync.conf):**
```ini
# 전체 권한 모드
plugins-power=true
```

### 권한 모드별 사용 가능한 기능

| 플러그인 타입 | plugins-power=false | plugins-power=true | 주요 용도 |
|--------------|--------------------|--------------------|-----------|
| **webhook** | ✅ 허용 | ✅ 허용 | Discord, Slack, Telegram 알림 |
| **email** | ✅ 허용 | ✅ 허용 | SMTP 이메일 알림 |
| **exec** | ❌ 차단 | ✅ 허용 | 백업 스크립트, 외부 프로그램 실행 |
| **script** | ❌ 차단 | ✅ 허용 | 통계 수집, 로그 분석, 자동화 작업 |

### 제한 모드에서 권장하는 플러그인 작성 방법

제한 모드(`plugins-power=false`)에서는 **webhook**과 **email** 타입만 사용할 수 있으므로, 다음과 같은 방식으로 플러그인을 작성하는 것이 권장됩니다.

#### 1. 웹훅을 활용한 외부 시스템 연동

외부 프로그램 실행 대신 웹훅 API를 호출하여 다른 시스템과 연동합니다.

```dsp
// exec 대신 webhook 사용 예제
name="백업 트리거"
version="1.0"
description="webhook으로 백업 시스템 트리거"
type="webhook"
enabled=true
priority=20

// 백업 시스템의 API 엔드포인트 호출
config.url="https://backup-system.example.com/api/trigger"
config.format="custom"

config.json={
{
  "action": "backup",
  "source": "/mirror",
  "timestamp": {{.Timestamp}},
  "repositories": [
    {{range $i, $r := .SyncResults}}
    {{if $i}},{{end}}
    "{{$r.RepoName}}"
    {{end}}
  ]
}
}
```

#### 2. 이메일을 활용한 알림

복잡한 스크립트 대신 이메일로 상세 리포트를 전송합니다.

```dsp
// script 대신 email 사용 예제
name="상세 동기화 리포트"
version="1.0"
description="이메일로 상세 리포트 전송"
type="email"
enabled=true
priority=25

config.html=true
config.smtp_host="smtp.example.com"
config.smtp_port="587"
config.smtp_user="noreply@example.com"
config.smtp_password="password"
config.from="sync@example.com"
config.to="admin@example.com"
config.subject="[DSync] 동기화 리포트 - {{formatTimeKorean .Timestamp}}"

config.template={
<!DOCTYPE html>
<html>
<head><style>
  body { font-family: sans-serif; }
  .success { color: green; }
  .failed { color: red; }
  table { border-collapse: collapse; width: 100%; }
  th, td { border: 1px solid #ddd; padding: 8px; }
</style></head>
<body>
  <h1>동기화 리포트</h1>
  <p>완료 시간: {{formatTimeKorean .Timestamp}}</p>
  <table>
    <tr><th>저장소</th><th>상태</th><th>전송량</th><th>소요시간</th></tr>
    {{range .SyncResults}}
    <tr>
      <td>{{.RepoName}}</td>
      <td class="{{if .Success}}success{{else}}failed{{end}}">
        {{if .Success}}✅ 성공{{else}}❌ 실패{{end}}
      </td>
      <td>{{if index $.Stats .RepoName}}{{formatSize (index $.Stats .RepoName).TotalBytes}}{{else}}N/A{{end}}</td>
      <td>{{if index $.Stats .RepoName}}{{formatDuration (index $.Stats .RepoName).TotalTime}}{{else}}N/A{{end}}</td>
    </tr>
    {{end}}
  </table>
</body>
</html>
}
```

#### 3. DSP 함수를 활용한 고급 로직

스크립트 실행 없이도 DSP 함수로 복잡한 로직을 구현할 수 있습니다.

```dsp
// 복잡한 조건부 로직을 DSP 함수로 구현
name="스마트 알림"
version="2.0"
description="실패율에 따라 다른 웹훅으로 전송"
type="webhook"
enabled=true
priority=15

// 실패율 계산 함수
func getFailureRate(syncResults) {
    total = len(syncResults)
    if total == 0 {
        return 0
    }
    failed = 0
    foreach(syncResults, func(result) {
        if not result.Success {
            failed = failed + 1
        }
    })
    return (failed * 100) / total
}

// 심각도 판단 함수
func getSeverity(syncResults) {
    rate = call("getFailureRate", syncResults)
    if rate >= 50 {
        return "critical"
    }
    if rate > 0 {
        return "warning"
    }
    return "success"
}

// 웹훅 URL 선택 함수
func getWebhookUrl(syncResults) {
    severity = call("getSeverity", syncResults)
    if severity == "critical" {
        return "https://hooks.slack.com/critical-channel"
    }
    if severity == "warning" {
        return "https://hooks.slack.com/warning-channel"
    }
    return "https://hooks.slack.com/general-channel"
}

// 동적으로 URL 선택 (템플릿 내에서 직접 사용 불가, 설정값으로 사용)
config.url="{{call \"getWebhookUrl\" .SyncResults}}"
config.format="slack"

config.template={
📊 동기화 완료
심각도: {{call "getSeverity" .SyncResults}}
실패율: {{call "getFailureRate" .SyncResults}}%
}
```

### 보안 고려사항

#### 왜 exec와 script가 차단되는가?

**보안 위험:**
- ❌ 임의의 시스템 명령 실행 가능
- ❌ 파일 시스템 접근 및 수정 가능
- ❌ 네트워크 공격 가능성
- ❌ 권한 상승 공격 가능성

**제한 모드의 장점:**
- ✅ 플러그인이 할 수 있는 작업을 명확히 제한
- ✅ 네트워크 통신만 허용 (HTTP, SMTP)
- ✅ 신뢰할 수 없는 플러그인도 안전하게 사용
- ✅ 프로덕션 환경에서 권장

#### 전체 권한 모드 사용 시 주의사항

`plugins-power=true`로 설정하면 플러그인이 시스템에 무제한 접근할 수 있습니다.

**주의사항:**
- ⚠️ 신뢰할 수 있는 플러그인만 사용하세요
- ⚠️ 플러그인 코드를 꼼꼼히 검토하세요
- ⚠️ 프로덕션 환경에서는 신중히 사용하세요
- ⚠️ 플러그인이 실행하는 명령어를 확인하세요

### 권한 모드 확인 방법

현재 권한 모드는 다음 명령어로 확인할 수 있습니다:

```bash
# xync.conf 파일 확인
grep "plugins-power" ~/dxync/xync.conf

# 플러그인 로드 시 로그 확인
./dsync 2>&1 | grep "plugins-power"
```

**제한 모드에서 차단된 플러그인의 로그 예시:**
```
플러그인 차단 (보안): 백업 스크립트 (타입: exec) - plugins-power=false로 인해 실행 불가
   이 플러그인을 사용하려면 xync.conf에서 plugins-power=true로 설정하세요.
```

### 권한 모드 변경 방법

#### 제한 모드 → 전체 권한 모드

```bash
# 1. xync.conf 파일 편집
nano ~/dxync/xync.conf

# 2. 다음 줄을 추가 또는 수정
plugins-power=true

# 3. 저장 후 DSYNC 재실행
./dsync
```

#### 전체 권한 모드 → 제한 모드

```bash
# 1. xync.conf 파일 편집
nano ~/dxync/xync.conf

# 2. 다음 줄을 추가 또는 수정
plugins-power=false

# 3. 저장 후 DSYNC 재실행
./dsync
```

### 권장 사용 시나리오

| 시나리오 | 권장 모드 | 이유 |
|---------|----------|------|
| 프로덕션 미러 서버 | `plugins-power=false` | 안정성과 보안 최우선 |
| 개발/테스트 환경 | `plugins-power=true` | 유연한 테스트 가능 |
| 외부 플러그인 사용 | `plugins-power=false` | 신뢰할 수 없는 코드 차단 |
| 자체 개발 플러그인 | `plugins-power=true` | 고급 자동화 작업 가능 |
| 클라우드/공유 서버 | `plugins-power=false` | 다중 사용자 환경 보안 |
| 로컬 개인 서버 | `plugins-power=true` | 제한 없는 사용 |

### FAQ

**Q: plugins-power 설정이 없으면 어떻게 되나요?**
A: 기본값은 `false`입니다. 명시적으로 `true`를 설정하지 않으면 제한 모드로 작동합니다.

**Q: 제한 모드에서 exec 플러그인을 만들면 어떻게 되나요?**
A: 플러그인은 로드되지만 실행되지 않으며, 로그에 차단 메시지가 기록됩니다.

**Q: webhook으로 모든 작업을 대체할 수 있나요?**
A: 대부분의 알림과 외부 시스템 연동은 webhook으로 가능합니다. 단, 로컬 파일 작업이나 복잡한 스크립트는 제한됩니다.

**Q: 이메일 플러그인은 왜 제한 모드에서도 허용되나요?**
A: 이메일은 SMTP 프로토콜을 통한 네트워크 통신만 사용하며, 시스템에 직접 접근하지 않아 안전하기 때문입니다.

**Q: 개발 중인 플러그인을 안전하게 테스트하려면?**
A: 제한 모드에서 webhook/email 타입으로 먼저 작성하고, 필요시 전체 권한 모드로 전환하여 exec/script 타입을 테스트하세요.

## 플러그인 설치 방법

1. **플러그인 디렉토리 확인**
   ```bash
   ls ~/dxync/plugins/
   ```

2. **플러그인 파일 생성**
   - 파일명: `[플러그인명].dsp`
   - 위치: `~/dxync/plugins/`

3. **플러그인 확인**
   ```bash
   ./dsync 또는 dsync -list-plugins
   ```

4. **플러그인 테스트**
   ```bash
   ./dsync 또는 dsync -test-plugin [플러그인명]
   ```

## 플러그인 파일 구조

### 기본 구조
```dsp
// 플러그인 메타데이터
name="플러그인 이름"
version="1.0"
author="작성자"
description="플러그인 설명"
type="webhook"  // webhook, email, exec, script 중 선택
enabled=true    // true 또는 false
priority=100    // 낮은 숫자가 먼저 실행됨

// 플러그인별 설정
config.설정명="값"
```

### 설정 값 타입

#### 단일 라인 설정
```dsp
config.url="https://example.com"
config.timeout=300
config.enabled=true
```

#### 여러 줄 설정 (중괄호 사용)
```dsp
config.template={
여러 줄의
텍스트를
입력할 수 있습니다
}
```

## 플러그인 타입별 예제

### 1. 웹훅 플러그인 (webhook)

웹훅으로 알림을 전송하는 플러그인입니다.

#### Discord 알림 예제
```dsp
// Discord 웹훅 플러그인
name="Discord 동기화 알림"
version="1.0"
author="관리자"
description="동기화 완료 후 Discord 채널에 알림"
type="webhook"
enabled=true
priority=10

// 웹훅 설정
config.url="https://discord.com/api/webhooks/1234567890/abcdefg"
config.format="discord"
config.username="DSYNC 봇"
config.override=false  // true면 xync.conf의 웹훅 설정 무시

// 메시지 템플릿 (선택사항)
config.template={
🔔 **동기화 완료 알림**

📊 동기화 결과:
- 총 {{len .SyncResults}}개 저장소 처리
- ✅ 성공: {{range .SyncResults}}{{if .Success}}+1{{end}}{{end}}개
- ❌ 실패: {{range .SyncResults}}{{if not .Success}}+1{{end}}{{end}}개

⏰ 완료 시간: {{.Timestamp}}
🖥️ 서버: {{.GlobalConfig.version}}

{{range .SyncResults}}
{{if not .Success}}
⚠️ 실패 저장소: {{.RepoName}}
{{end}}
{{end}}
}
```

#### Slack 알림 예제
```dsp
// Slack 웹훅 플러그인
name="Slack 알림"
version="1.0"
author="DevOps팀"
description="동기화 결과를 Slack으로 전송"
type="webhook"
enabled=true
priority=20

config.url="https://hooks.slack.com/services/T00000000/B00000000/XXXXXXXXXXXXXXXXXXXX"
config.format="slack"

// Slack 메시지 형식
config.template={
:bell: *동기화 완료*

• 처리된 저장소: {{len .SyncResults}}개
• 성공/실패: {{.SuccessCount}}/{{.FailureCount}}
• 소요 시간: {{.ElapsedTime}}

자세한 로그는 서버를 확인하세요.
}
```

#### Telegram 알림 예제
```dsp
// Telegram 봇 알림 플러그인
name="Telegram 알림"
version="1.0"
author="관리자"
description="동기화 결과를 Telegram으로 전송"
type="webhook"
enabled=true
priority=15

// Telegram Bot API URL 형식
// https://api.telegram.org/bot{BOT_TOKEN}/sendMessage
config.url="https://api.telegram.org/bot1234567890:ABCdefGHIjklMNOpqrsTUVwxyz/sendMessage"
config.format="custom"

// Telegram API는 특정 JSON 형식이 필요함
config.json={
{
  "chat_id": "-1001234567890",
  "parse_mode": "Markdown",
  "text": "🔔 *DSYNC 동기화 완료*\n\n📊 *동기화 결과:*\n• 총 저장소: {{len .SyncResults}}개\n• ✅ 성공: {{range .SyncResults}}{{if .Success}}+1{{end}}{{end}}개\n• ❌ 실패: {{range .SyncResults}}{{if not .Success}}+1{{end}}{{end}}개\n\n⏰ 완료 시간: {{.Timestamp}}\n\n{{range .SyncResults}}{{if not .Success}}⚠️ 실패: `{{.RepoName}}`\n{{end}}{{end}}"
}
}
```

#### Telegram 고급 알림 예제 (버튼 포함)
```dsp
// Telegram 인라인 키보드 포함 알림
name="Telegram 고급 알림"
version="2.0"
author="관리자"
description="버튼이 있는 Telegram 알림"
type="webhook"
enabled=true
priority=15

// BOT_TOKEN을 실제 봇 토큰으로 교체하세요
config.url="https://api.telegram.org/bot여기에_봇_토큰_입력/sendMessage"
config.format="custom"

// 인라인 키보드 버튼 포함
config.json={
{
  "chat_id": "-1001234567890",
  "parse_mode": "HTML",
  "text": "🔔 <b>DSYNC 동기화 완료</b>\n\n📊 <b>결과:</b>\n• 총 {{len .SyncResults}}개 저장소\n• ✅ 성공: {{range .SyncResults}}{{if .Success}}+1{{end}}{{end}}개\n• ❌ 실패: {{range .SyncResults}}{{if not .Success}}+1{{end}}{{end}}개\n\n{{range .SyncResults}}{{if not .Success}}⚠️ <code>{{.RepoName}}</code> 실패\n{{end}}{{end}}",
  "reply_markup": {
    "inline_keyboard": [
      [
        {"text": "📋 로그 보기", "url": "https://mirror.example.com/logs"},
        {"text": "🔄 재동기화", "callback_data": "resync"}
      ],
      [
        {"text": "📊 통계", "url": "https://mirror.example.com/stats"}
      ]
    ]
  }
}
}
```

#### Telegram 그룹/채널 설정 방법
```dsp
// Telegram 설정 가이드가 포함된 플러그인
name="Telegram 설정 도우미"
version="1.0"
author="가이드"
description="Telegram 봇 설정 방법"
type="webhook"
enabled=false  // 실제 사용 시 true로 변경
priority=100

// ========== Telegram 봇 설정 방법 ==========
//
// 1. 봇 생성:
//    - Telegram에서 @BotFather 검색
//    - /newbot 명령어 입력
//    - 봇 이름과 username 설정
//    - 봇 토큰 받기 (예: 1234567890:ABCdefGHIjklMNOpqrsTUVwxyz)
//
// 2. Chat ID 찾기:
//    a) 개인 채팅: @userinfobot에게 메시지 보내기
//    b) 그룹: 봇을 그룹에 추가 후 https://api.telegram.org/bot봇토큰/getUpdates 접속
//    c) 채널: 봇을 관리자로 추가, 채널 ID는 보통 -100으로 시작
//
// 3. URL 형식:
//    https://api.telegram.org/bot봇토큰/sendMessage
//
// =============================================

config.url="https://api.telegram.org/bot봇토큰입력/sendMessage"
config.format="custom"

config.json={
{
  "chat_id": "채팅ID입력",
  "text": "테스트 메시지"
}
}
```

#### 커스텀 웹훅 예제
```dsp
// 커스텀 웹훅 (자체 API 서버)
name="내부 API 알림"
version="1.0"
author="시스템팀"
description="내부 모니터링 시스템으로 데이터 전송"
type="webhook"
enabled=true
priority=5

config.url="https://monitor.company.local/api/sync-report"
config.format="custom"

// 커스텀 JSON 형식
config.json={
{
  "event": "sync_complete",
  "timestamp": {{.Timestamp}},
  "server": "{{.GlobalConfig.hostname}}",
  "results": [
    {{range $i, $r := .SyncResults}}
    {{if $i}},{{end}}
    {
      "repo": "{{$r.RepoName}}",
      "success": {{$r.Success}},
      "log": "{{$r.LogFile}}"
    }
    {{end}}
  ],
  "statistics": {
    "total": {{len .SyncResults}},
    "success": {{.SuccessCount}},
    "failed": {{.FailureCount}}
  }
}
}
```

### 2. 실행 플러그인 (exec)

외부 프로그램이나 명령을 실행하는 플러그인입니다.

#### 백업 스크립트 실행 예제
```dsp
// 동기화 후 백업 실행
name="자동 백업"
version="1.0"
author="백업팀"
description="동기화 완료 후 증분 백업 수행"
type="exec"
enabled=true
priority=30

config.command="/usr/local/bin/backup.sh"
config.args="--incremental --compress"
config.timeout=3600  // 1시간 타임아웃

// 환경 변수 설정
config.env.BACKUP_TYPE="incremental"
config.env.BACKUP_DEST="/mnt/backup"
config.env.RETENTION_DAYS="30"
```

#### rsync 통계 수집 예제
```dsp
// 통계 데이터베이스 업데이트
name="통계 수집기"
version="2.0"
author="모니터링팀"
description="동기화 통계를 데이터베이스에 저장"
type="exec"
enabled=true
priority=15

config.command="/opt/scripts/update_stats.py"
config.args="--format json"
config.timeout=60

// Python 스크립트가 DSYNC_CONTEXT 환경변수에서 JSON 데이터를 읽음
```

#### 로그 압축 및 아카이브
```dsp
// 오래된 로그 압축
name="로그 관리자"
version="1.0"
author="시스템팀"
description="7일 이상 된 로그를 압축하여 아카이브"
type="exec"
enabled=true
priority=50

config.command="/bin/bash"
config.args="-c 'find /root/dxync/logs -name \"*.log\" -mtime +7 -exec gzip {} \\;'"
config.timeout=300
```

### 3. 이메일 플러그인 (email)

SMTP를 통해 이메일을 전송하는 플러그인입니다. bash 스크립트 없이 간단하게 이메일 알림을 보낼 수 있습니다.

#### 기본 이메일 알림 예제
```dsp
// 이메일 알림 플러그인
name="이메일 알림"
version="1.0"
author="관리자"
description="동기화 완료 후 이메일로 결과 전송"
type="email"
enabled=true
priority=20

// SMTP 서버 설정
config.smtp_host="smtp.example.com"
config.smtp_port="587"               // 기본값: 587
config.smtp_user="your-username"
config.smtp_password="your-password"

// 이메일 설정
config.from="sender@example.com"
config.to="admin@example.com"
config.subject="[DSync] 동기화 완료 - {{formatTimeKorean .Timestamp}}"

// 이메일 본문 템플릿
config.template={
DSync 동기화가 완료되었습니다.

========================================
동기화 결과
========================================

총 저장소: {{len .SyncResults}}개
완료 시간: {{formatTimeKorean .Timestamp}}
서버 버전: {{index .GlobalConfig "version"}}

저장소 상태:
{{range .SyncResults}}{{if .Success}}✓{{else}}✗{{end}} {{.RepoName}}
{{end}}
{{range .SyncResults}}{{if not .Success}}
실패: {{.RepoName}}
로그: {{.LogFile}}
{{end}}{{end}}
========================================
}
```

#### Gmail 사용 예제

Gmail을 사용하려면 앱 비밀번호를 생성해야 합니다.

```dsp
name="Gmail 알림"
version="1.0"
type="email"
enabled=true
priority=20

// Gmail SMTP 설정
config.smtp_host="smtp.gmail.com"
config.smtp_port="587"
config.smtp_user="youremail@gmail.com"
config.smtp_password="your-app-password"  // Gmail 앱 비밀번호

config.from="youremail@gmail.com"
config.to="admin@example.com"
config.subject="[DSync] 동기화 완료 - {{formatTimeKorean .Timestamp}}"

config.template={
동기화 완료

총 {{len .SyncResults}}개 저장소 처리
시간: {{formatTimeKorean .Timestamp}}
}
```

#### 다중 수신자 지원

email 플러그인은 쉼표(`,`) 또는 세미콜론(`;`)으로 구분된 다중 수신자를 지원합니다.

**다중 수신자 예제:**

```dsp
name="다중 수신자 알림"
version="1.0"
type="email"
enabled=true
priority=20

config.smtp_host="smtp.example.com"
config.smtp_port="587"
config.smtp_user="noreply@example.com"
config.smtp_password="password"
config.from="sync@example.com"

// 방법 1: 쉼표로 구분
config.to="admin@example.com, devops@example.com, monitor@example.com"

// 방법 2: 세미콜론으로 구분
// config.to="admin@example.com; devops@example.com; monitor@example.com"

config.subject="[DSync] 동기화 완료"
config.template={
동기화가 완료되었습니다.
총 {{len .SyncResults}}개 저장소 처리
}
```

**팁:**
- 공백은 자동으로 제거되므로 `"email1@example.com, email2@example.com"` 또는 `"email1@example.com,email2@example.com"` 모두 동일하게 작동
- 쉼표와 세미콜론을 혼용할 수 있음: `"a@test.com, b@test.com; c@test.com"`

#### HTML 이메일 예제

HTML 형식의 이메일을 보내려면 `config.html=true`를 설정하고 템플릿에 HTML 코드를 작성합니다.

```dsp
// HTML 이메일 알림 플러그인
name="HTML 이메일 알림"
version="2.0"
author="관리자"
description="동기화 완료 후 HTML 이메일로 결과 전송"
type="email"
enabled=true
priority=20

// HTML 형식 활성화
config.html=true

// SMTP 서버 설정
config.smtp_host="smtp.example.com"
config.smtp_port="587"
config.smtp_user="your-username"
config.smtp_password="your-password"

// 이메일 설정
config.from="sender@example.com"
config.to="admin@example.com"
config.subject="[DSync] 동기화 완료 - {{formatTimeKorean .Timestamp}}"

// HTML 이메일 본문 템플릿
config.template={
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <style>
        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
            background-color: #f5f5f5;
            margin: 0;
            padding: 20px;
        }
        .container {
            max-width: 800px;
            margin: 0 auto;
            background-color: #ffffff;
            border-radius: 8px;
            box-shadow: 0 2px 8px rgba(0,0,0,0.1);
        }
        .header {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            padding: 30px;
            text-align: center;
            border-radius: 8px 8px 0 0;
        }
        .content {
            padding: 30px;
        }
        .status-badge {
            display: inline-block;
            padding: 4px 12px;
            border-radius: 12px;
            font-size: 13px;
            font-weight: 600;
        }
        .status-success {
            background-color: #d4edda;
            color: #155724;
        }
        .status-failed {
            background-color: #f8d7da;
            color: #721c24;
        }
        .repo-card {
            border: 1px solid #e9ecef;
            border-radius: 6px;
            padding: 15px;
            margin: 10px 0;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>DSync 동기화 완료</h1>
        </div>
        <div class="content">
            <p><strong>동기화 시간:</strong> {{formatTimeKorean .Timestamp}}</p>
            <p><strong>총 저장소:</strong> {{len .SyncResults}}개</p>

            <h2>저장소별 상세 정보</h2>
            {{range .SyncResults}}
            <div class="repo-card">
                <strong>{{.RepoName}}</strong>
                {{if .Success}}
                <span class="status-badge status-success">✅ 성공</span>
                {{else}}
                <span class="status-badge status-failed">❌ 실패</span>
                {{end}}
            </div>
            {{end}}
        </div>
    </div>
</body>
</html>
}
```

#### DSP 함수를 사용한 HTML 이메일

DSP 함수를 활용하면 더욱 동적인 HTML 이메일을 만들 수 있습니다.

```dsp
name="고급 HTML 이메일"
version="2.0"
type="email"
enabled=true
priority=20

// 저장소 타입별 이모지 함수
func getRepoEmoji(repoName) {
    if contains(repoName, "ubuntu") {
        return "🐧"
    }
    if contains(repoName, "debian") {
        return "🌀"
    }
    return "📦"
}

config.html=true
config.smtp_host="smtp.example.com"
config.smtp_port="587"
config.smtp_user="user"
config.smtp_password="pass"
config.from="sender@example.com"
config.to="admin@example.com"
config.subject="[DSync] 동기화 완료 - {{formatTimeKorean .Timestamp}}"

config.template={
<!DOCTYPE html>
<html>
<head>
    <style>
        .emoji { font-size: 24px; margin-right: 8px; }
        .stats { background: #f8f9fa; padding: 15px; border-radius: 6px; }
    </style>
</head>
<body>
    <h1>🔔 DSync 동기화 완료</h1>
    <div class="stats">
        <p>📅 시간: {{formatTimeKorean .Timestamp}}</p>
        <p>📊 저장소: {{len .SyncResults}}개</p>
    </div>

    <h2>저장소 목록</h2>
    {{range .SyncResults}}
    <div>
        <span class="emoji">{{call "getRepoEmoji" .RepoName}}</span>
        <strong>{{.RepoName}}</strong>
        {{if .Success}}✅ 성공{{else}}❌ 실패{{end}}
    </div>
    {{end}}

    {{range $repo, $stats := .Stats}}
    <p>
        <strong>{{$repo}}:</strong>
        전송량 {{formatSize $stats.TotalBytes}},
        소요시간 {{formatDuration $stats.TotalTime}}
    </p>
    {{end}}
</body>
</html>
}
```

#### 고급 HTML 이메일 예제 (Gmail 호환, 상세 통계 포함)

실제 운영 환경에서 사용할 수 있는 완전한 HTML 이메일 예제입니다. 저장소별 이모지, 상세 통계, 긴급 중단 감지 등의 기능을 포함합니다.

어디까지나 예시일 뿐이며 버그가 있을 수도 있습니다.

```dsp
// 동기화 로그 이메일 알림 플러그인
name="로그 이메일 알리미"
version="2.1"
author="관리자"
description="저장소 동기화 완료 후 최신 로그를 HTML 이메일로 전송 (Gmail 호환)"
type="email"
enabled=true
priority=15

// HTML 형식 활성화
config.html=true

// 저장소 이름에 따른 이모지 반환
func getRepoEmoji(repoName) {
    if contains(repoName, "omv") {
        return "🔧"
    }
    if contains(repoName, "ubuntu") {
        return "🐧"
    }
    if contains(repoName, "debian") {
        return "🌀"
    }
    return "📦"
}

// SMTP 설정
config.smtp_host="smtp.example.com"
config.smtp_port="587"
config.smtp_user="your-username"
config.smtp_password="your-password"

config.from="sender@example.com"
config.to="admin@example.com, team@example.com"
config.subject="[DSync] 동기화 완료 - {{formatTimeKorean .Timestamp}}"

// Gmail 호환 HTML 이메일 본문 (인라인 스타일 사용)
config.template={
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
</head>
<body style="font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif; line-height: 1.6; color: #333; background-color: #f5f5f5; margin: 0; padding: 20px;">
    <div style="max-width: 800px; margin: 0 auto; background-color: #ffffff; border-radius: 8px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); overflow: hidden;">

        <!-- 헤더 -->
        <div style="background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); color: white; padding: 30px; text-align: center;">
            <div style="font-size: 48px; margin-bottom: 10px;">{{call "getRepoEmoji" "omv"}}</div>
            <h1 style="margin: 0; font-size: 24px; font-weight: 600;">DSync 동기화 완료 알림</h1>
        </div>

        <!-- 본문 -->
        <div style="padding: 30px;">

            <!-- 요약 정보 박스 -->
            <div style="background-color: #f8f9fa; border-left: 4px solid #667eea; padding: 15px 20px; margin: 20px 0; border-radius: 4px;">
                <div style="margin-bottom: 10px;">
                    <span style="font-weight: 600; color: #495057; display: inline-block; min-width: 120px;">📅 동기화 시간:</span>
                    <span style="color: #212529;">{{formatTimeKorean .Timestamp}}</span>
                </div>
                <div>
                    <span style="font-weight: 600; color: #495057; display: inline-block; min-width: 120px;">📊 전체 상태:</span>
                    <span style="color: #212529;">{{len .SyncResults}}개 저장소 동기화 완료</span>
                </div>
            </div>

            <!-- 저장소별 상세 정보 -->
            <h2 style="font-size: 18px; font-weight: 600; color: #495057; margin: 30px 0 15px 0; padding-bottom: 10px; border-bottom: 2px solid #e9ecef;">🔍 저장소별 상세 정보</h2>

            {{range .SyncResults}}
            <div style="background-color: #ffffff; border: 1px solid {{if .Success}}#28a745{{else}}#dc3545{{end}}; border-radius: 6px; padding: 15px; margin: 10px 0; border-left: 4px solid {{if .Success}}#28a745{{else}}#dc3545{{end}};">
                <div style="font-size: 16px; font-weight: 600; color: #212529; margin-bottom: 8px;">
                    {{if eq .RepoName "emergency_shutdown"}}
                    <span style="margin-right: 8px;">🚨</span>
                    긴급 중단 실행
                    {{else}}
                    <span style="margin-right: 8px;">{{call "getRepoEmoji" .RepoName}}</span>
                    {{.RepoName}}
                    {{end}}
                    {{if .Success}}
                    <span style="display: inline-block; padding: 4px 12px; border-radius: 12px; font-size: 13px; font-weight: 600; background-color: #d4edda; color: #155724;">✅ 성공</span>
                    {{else}}
                    <span style="display: inline-block; padding: 4px 12px; border-radius: 12px; font-size: 13px; font-weight: 600; background-color: #f8d7da; color: #721c24;">❌ 강제 종료됨</span>
                    {{end}}
                </div>
                <div style="font-size: 14px; color: #6c757d; margin: 5px 0;">
                    <span style="font-weight: 600; display: inline-block; min-width: 80px;">{{if eq .RepoName "emergency_shutdown"}}상태:{{else}}로그 파일:{{end}}</span>
                    <code style="background-color: #f8f9fa; padding: 2px 6px; border-radius: 3px; font-family: 'Courier New', monospace; font-size: 13px; color: #e83e8c;">{{.LogFile}}</code>
                </div>
                {{if not .Success}}
                <div style="margin-top: 10px; padding: 10px; background-color: #fff3cd; border-left: 3px solid #ffc107; border-radius: 4px;">
                    <span style="color: #856404; font-size: 13px;">⚠️ 이 저장소는 긴급 중단(emoff)에 의해 강제 종료되었습니다.</span>
                </div>
                {{end}}
            </div>
            {{end}}

            <!-- 구분선 -->
            <div style="height: 1px; background-color: #e9ecef; margin: 25px 0;"></div>

            <!-- 전체 동기화 통계 -->
            {{if .Stats}}
            <h2 style="font-size: 18px; font-weight: 600; color: #495057; margin: 30px 0 15px 0; padding-bottom: 10px; border-bottom: 2px solid #e9ecef;">📈 전체 동기화 통계</h2>

            <div style="background-color: #f8f9fa; padding: 15px; border-radius: 6px; border-left: 4px solid #667eea; margin: 10px 0;">
                <div style="font-size: 12px; color: #6c757d; text-transform: uppercase; letter-spacing: 0.5px; margin-bottom: 5px;">총 저장소 수</div>
                <div style="font-size: 18px; font-weight: 600; color: #212529;">{{len .SyncResults}}개</div>
            </div>

            {{range $repo, $stats := .Stats}}
            <div style="background-color: #f8f9fa; padding: 15px; border-radius: 6px; border-left: 4px solid #667eea; margin: 10px 0;">
                <div style="font-size: 12px; color: #6c757d; text-transform: uppercase; letter-spacing: 0.5px; margin-bottom: 5px;">{{$repo}}</div>
                <div style="margin-top: 8px;">
                    <div style="margin: 3px 0; font-size: 14px; color: #6c757d;">
                        <span style="font-weight: 600; display: inline-block; min-width: 100px;">전송 데이터:</span> {{formatSize $stats.TotalBytes}}
                    </div>
                    <div style="margin: 3px 0; font-size: 14px; color: #6c757d;">
                        <span style="font-weight: 600; display: inline-block; min-width: 100px;">소요 시간:</span> {{formatDuration $stats.TotalTime}}
                    </div>
                    <div style="margin: 3px 0; font-size: 14px; color: #6c757d;">
                        <span style="font-weight: 600; display: inline-block; min-width: 100px;">Stage 1:</span> {{formatDuration $stats.Stage1Time}}
                    </div>
                    <div style="margin: 3px 0; font-size: 14px; color: #6c757d;">
                        <span style="font-weight: 600; display: inline-block; min-width: 100px;">Stage 2:</span> {{formatDuration $stats.Stage2Time}}
                    </div>
                </div>
            </div>
            {{end}}
            {{else}}
            <!-- 긴급 중단 시 통계 없음 -->
            <div style="background-color: #fff3cd; border-left: 4px solid #ffc107; padding: 15px 20px; margin: 20px 0; border-radius: 4px;">
                <span style="color: #856404; font-size: 14px;">⚠️ 긴급 중단으로 인해 동기화 통계를 수집할 수 없었습니다.</span>
            </div>
            {{end}}

            <!-- 구분선 -->
            <div style="height: 1px; background-color: #e9ecef; margin: 25px 0;"></div>

            <!-- 참고 사항 -->
            <h2 style="font-size: 18px; font-weight: 600; color: #495057; margin: 30px 0 15px 0; padding-bottom: 10px; border-bottom: 2px solid #e9ecef;">💡 참고 사항</h2>
            <ul style="color: #6c757d; line-height: 1.8;">
                <li>전체 로그는 서버의 <code style="background-color: #f8f9fa; padding: 2px 6px; border-radius: 3px; font-family: 'Courier New', monospace; font-size: 13px; color: #e83e8c;">/root/dsync/logs/</code> 디렉토리에서 확인하세요</li>
                <li>문제가 발생한 경우 로그 파일을 확인해주세요</li>
                <li>DSync 버전: <strong>{{index .GlobalConfig "version"}}</strong></li>
            </ul>
        </div>

        <!-- 푸터 -->
        <div style="background-color: #f8f9fa; padding: 20px; text-align: center; font-size: 13px; color: #6c757d; border-top: 1px solid #e9ecef;">
            <p style="margin: 5px 0;">이 메일은 자동으로 발송되었습니다.</p>
            <p style="margin: 5px 0;">DSync 플러그인 시스템 v2.1</p>
        </div>
    </div>
</body>
</html>
}
```

**이 예제의 주요 기능:**

1. **다중 수신자 지원**: `config.to` 필드에 쉼표로 구분된 여러 이메일 주소 지정
2. **DSP 함수 활용**: `getRepoEmoji()` 함수로 저장소별 이모지 자동 선택
3. **상세 통계**: 전송 데이터, 소요 시간, Stage 1/2 시간 등 표시
4. **긴급 중단 감지**: `emergency_shutdown` 저장소 및 강제 종료 상태 표시
5. **Gmail 호환**: 인라인 스타일로 대부분의 이메일 클라이언트에서 정상 표시
6. **로그 파일 경로**: 각 저장소별 로그 파일 경로 표시

**사용 전 수정 사항:**
- `config.smtp_host`: 실제 SMTP 서버 주소로 변경 (예: smtp.gmail.com)
- `config.smtp_user`: SMTP 인증 사용자명 입력
- `config.smtp_password`: SMTP 인증 비밀번호 입력
- `config.from`: 발신자 이메일 주소 설정
- `config.to`: 수신자 이메일 주소(들) 설정 (쉼표로 구분)

#### 이메일 타입 vs 스크립트 타입 비교

**email 타입 장점:**
- ✅ bash/curl 불필요
- ✅ 간단한 설정
- ✅ 템플릿 문법 사용 가능
- ✅ 시간 형식화 함수 지원
- ✅ 크로스 플랫폼 호환
- ✅ HTML 이메일 지원 (`config.html=true`)

**script 타입이 필요한 경우:**
- 첨부 파일 필요
- 외부 이메일 전송 서비스 사용 (SendGrid, Mailgun 등)
- 매우 복잡한 이메일 로직

#### SMTP 포트 번호 참고

| 포트 | 설명 | 보안 |
|------|------|------|
| 25 | 기본 SMTP (대부분 차단됨) | 없음 |
| 587 | SMTP with STARTTLS (권장) | TLS |
| 465 | SMTPS (SSL) | SSL |

대부분의 현대 메일 서버는 **포트 587**을 사용합니다.

#### 문제 해결

**Q: "이메일 전송 실패: 535 Authentication failed" 오류**
A: SMTP 사용자명/비밀번호를 확인하세요. Gmail의 경우 앱 비밀번호를 사용해야 합니다.

**Q: 이메일이 스팸함으로 이동합니다**
A: SPF, DKIM, DMARC 레코드를 설정하거나, 신뢰할 수 있는 SMTP 서비스를 사용하세요.

**Q: "connection refused" 오류**
A: 방화벽에서 SMTP 포트(587)가 허용되어 있는지 확인하세요.

**Q: 한글이 깨집니다**
A: 템플릿은 자동으로 UTF-8로 인코딩됩니다. 메일 클라이언트 설정을 확인하세요.

**Q: HTML 이메일을 보내고 싶습니다**
A: `config.html=true`를 설정하고 템플릿에 HTML 코드를 작성하세요. 위의 "HTML 이메일 예제"를 참고하세요.

### 4. 스크립트 플러그인 (script)

인라인 스크립트를 실행하는 플러그인입니다.

#### 동기화 리포트 생성 예제
```dsp
// HTML 리포트 생성
name="리포트 생성기"
version="1.0"
author="운영팀"
description="동기화 결과를 HTML 리포트로 생성"
type="script"
enabled=true
priority=40

config.interpreter="/bin/bash"
config.timeout=120

config.script={
#!/bin/bash

# DSYNC_CONTEXT 환경변수에서 JSON 파싱
CONTEXT_JSON="$DSYNC_CONTEXT"
REPORT_DIR="/var/www/html/reports"
REPORT_FILE="$REPORT_DIR/sync_$(date +%Y%m%d_%H%M%S).html"

# 디렉토리 생성
mkdir -p "$REPORT_DIR"

# HTML 리포트 생성
cat > "$REPORT_FILE" << 'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>동기화 리포트</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; }
        .success { color: green; }
        .failure { color: red; }
        table { border-collapse: collapse; width: 100%; }
        th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
        th { background-color: #f2f2f2; }
    </style>
</head>
<body>
    <h1>DSYNC 동기화 리포트</h1>
    <p>생성 시간: $(date)</p>
EOF

# JSON 데이터 파싱 및 테이블 생성 (jq 사용)
echo "$CONTEXT_JSON" | jq -r '
    "<table>
    <tr><th>저장소</th><th>상태</th><th>로그 파일</th></tr>" +
    (.sync_results[] |
    "<tr>
        <td>\(.RepoName)</td>
        <td class=\"\(if .Success then "success" else "failure" end)\">
            \(if .Success then "성공" else "실패" end)
        </td>
        <td>\(.LogFile)</td>
    </tr>") +
    "</table>"
' >> "$REPORT_FILE"

cat >> "$REPORT_FILE" << 'EOF'
</body>
</html>
EOF

echo "리포트 생성 완료: $REPORT_FILE"

# 이메일 전송 (선택사항)
# mail -s "동기화 리포트" admin@company.com < "$REPORT_FILE"
}
```

#### 실패 알림 및 자동 복구 예제
```dsp
// 실패 시 자동 복구 시도
name="자동 복구"
version="1.0"
author="SRE팀"
description="동기화 실패 시 자동으로 재시도"
type="script"
enabled=true
priority=20

config.interpreter="/bin/bash"
config.timeout=600

config.script={
#!/bin/bash

# JSON 파싱
FAILED_REPOS=$(echo "$DSYNC_CONTEXT" | jq -r '.sync_results[] | select(.Success == false) | .RepoName')

if [ -z "$FAILED_REPOS" ]; then
    echo "모든 저장소가 성공적으로 동기화되었습니다."
    exit 0
fi

echo "실패한 저장소 재시도 중..."

for REPO in $FAILED_REPOS; do
    echo "재시도: $REPO"

    # 5분 대기 후 재시도
    sleep 300

    # 단일 저장소 동기화 실행
    /root/daxnet/daxync/client/dxync -sync "$REPO" -nolog

    if [ $? -eq 0 ]; then
        echo "$REPO 재동기화 성공"

        # 성공 알림
        curl -X POST "https://webhook.site/success" \
            -H "Content-Type: application/json" \
            -d "{\"repo\":\"$REPO\",\"status\":\"recovered\"}"
    else
        echo "$REPO 재동기화 실패 - 관리자 개입 필요"

        # 긴급 알림
        curl -X POST "https://webhook.site/critical" \
            -H "Content-Type: application/json" \
            -d "{\"repo\":\"$REPO\",\"status\":\"critical\",\"message\":\"Manual intervention required\"}"
    fi
done
}
```

#### 디스크 용량 체크 및 정리
```dsp
// 디스크 용량 관리
name="용량 관리자"
version="1.0"
author="인프라팀"
description="미러 디스크 용량 체크 및 자동 정리"
type="script"
enabled=true
priority=60

config.interpreter="/bin/bash"
config.timeout=180

config.script={
#!/bin/bash

# 디스크 사용률 체크
USAGE=$(df /mirror | tail -1 | awk '{print $5}' | sed 's/%//')

echo "현재 디스크 사용률: ${USAGE}%"

if [ $USAGE -gt 90 ]; then
    echo "⚠️ 디스크 사용률 위험 수준!"

    # 오래된 로그 삭제
    find /root/dxync/logs -name "*.log" -mtime +30 -delete
    echo "30일 이상 된 로그 삭제 완료"

    # 임시 파일 정리
    find /mirror -name "*.tmp" -o -name ".*~" -delete
    echo "임시 파일 정리 완료"

    # 중복 파일 검사 (선택사항)
    # fdupes -rdN /mirror

    # 정리 후 사용률 재확인
    NEW_USAGE=$(df /mirror | tail -1 | awk '{print $5}' | sed 's/%//')
    echo "정리 후 디스크 사용률: ${NEW_USAGE}%"

    # 알림 전송
    if [ $NEW_USAGE -gt 85 ]; then
        echo "여전히 디스크 공간이 부족합니다. 수동 개입이 필요합니다."
    fi
elif [ $USAGE -gt 80 ]; then
    echo "⚠️ 디스크 사용률 주의 수준: ${USAGE}%"
fi
}
```

#### Python 스크립트 플러그인 예제

DSYNC v25.11.1부터 `config.script={...}` 블록에서 Python을 사용할 수 있습니다. Python의 indentation이 보존되므로 복잡한 로직을 구현할 수 있습니다.

**기본 Python 예제:**

```dsp
// Python 스크립트 플러그인
name="Python 통계 분석"
version="1.0"
author="관리자"
description="Python으로 동기화 통계 분석"
type="script"
enabled=true
priority=30

config.interpreter="/usr/bin/python3"
config.timeout=60

config.script={
import json
import os
from datetime import datetime

# DSYNC 컨텍스트 가져오기
context_json = os.environ.get('DSYNC_CONTEXT', '{}')
context = json.loads(context_json)

# 플러그인 디렉토리 경로
plugin_dir = os.environ.get('DSYNC_PLUGIN_DIR', '/root/dxync/plugins')

# 통계 분석
sync_results = context.get('sync_results', [])
total = len(sync_results)
success = sum(1 for r in sync_results if r.get('Success', False))
failed = total - success

print(f"📊 동기화 통계")
print(f"  총 저장소: {total}개")
print(f"  성공: {success}개")
print(f"  실패: {failed}개")
print(f"  성공률: {(success/total*100) if total > 0 else 0:.1f}%")

# 실패한 저장소 목록
if failed > 0:
	print(f"\n⚠️ 실패한 저장소:")
	for result in sync_results:
		if not result.get('Success', False):
			print(f"  - {result.get('RepoName', 'Unknown')}")
}
```

**외부 설정 파일을 사용하는 Python 플러그인:**

민감한 정보(API 키, 비밀번호 등)를 플러그인 코드와 분리할 수 있습니다.

```dsp
// 외부 설정 파일을 사용하는 Python 플러그인
name="이메일 알림 (외부 설정)"
version="3.0"
author="관리자"
description="외부 설정 파일에서 이메일 계정 정보를 읽어 전송"
type="script"
enabled=true
priority=25

config.interpreter="/usr/bin/python3"
config.timeout=120

config.script={
import smtplib
import json
import os
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
from datetime import datetime

# 플러그인 디렉토리 경로
PLUGIN_DIR = os.environ.get('DSYNC_PLUGIN_DIR', '/root/dxync/plugins')
CONFIG_FILE = os.path.join(PLUGIN_DIR, 'email-config', 'credentials.conf')

def load_config():
	config = {}
	with open(CONFIG_FILE, 'r') as f:
		for line in f:
			line = line.strip()
			if line and not line.startswith('#'):
				if '=' in line:
					key, value = line.split('=', 1)
					config[key.strip()] = value.strip()
	return config

# 설정 로드
config = load_config()

# DSYNC 컨텍스트 가져오기
context_json = os.environ.get('DSYNC_CONTEXT', '{}')
context = json.loads(context_json)

# 이메일 작성
msg = MIMEMultipart()
msg['From'] = config['FROM_EMAIL']
msg['To'] = config['TO_EMAIL']
msg['Subject'] = '[DSync] 동기화 완료'

sync_results = context.get('sync_results', [])
body = f"동기화가 완료되었습니다.\n\n"
body += f"총 저장소: {len(sync_results)}개\n"
body += f"완료 시간: {datetime.fromtimestamp(context.get('timestamp', 0)).strftime('%Y년 %m월 %d일 %H:%M:%S')}\n\n"

# 저장소별 상태
body += "저장소 상태:\n"
for result in sync_results:
	status = "✅ 성공" if result.get('Success', False) else "❌ 실패"
	body += f"  {status} - {result.get('RepoName', 'Unknown')}\n"

msg.attach(MIMEText(body, 'plain'))

# SMTP 전송
with smtplib.SMTP(config['SMTP_HOST'], int(config['SMTP_PORT'])) as server:
	server.starttls()
	server.login(config['SMTP_USER'], config['SMTP_PASSWORD'])
	server.send_message(msg)

print("✅ 이메일 전송 완료")
}
```

**외부 설정 파일 예시 (`~/dxync/plugins/email-config/credentials.conf`):**

```ini
# 이메일 계정 설정
SMTP_HOST=smtp.example.com
SMTP_PORT=587
SMTP_USER=notify@example.com
SMTP_PASSWORD=your-password-here
FROM_EMAIL=notify@example.com
TO_EMAIL=admin@example.com
```

**디렉토리 구조:**

```
~/dxync/plugins/
├── email-notify.dsp          # Python 플러그인
└── email-config/
    └── credentials.conf       # 외부 설정 파일
```

**장점:**
- ✅ 민감한 정보를 Git에 커밋하지 않아도 됨
- ✅ 서버마다 다른 설정 사용 가능
- ✅ 플러그인 코드 재사용성 향상
- ✅ 보안 강화 (설정 파일 권한 관리: `chmod 600`)

**Python 스크립트 작성 시 주의사항:**

1. **Indentation 규칙**: Python 코드의 indentation은 **탭** 또는 **스페이스** 중 하나로 통일해야 합니다. 혼용하면 `IndentationError`가 발생합니다.

2. **원본 Indentation 보존**: `config.script={` 이후의 모든 라인은 원본 indentation이 그대로 보존됩니다. 따라서 첫 번째 컬럼부터 작성해야 합니다.

   ```dsp
   config.script={
   import os  # ✅ 올바름 - 첫 번째 컬럼부터 시작

   def func():
   	print("hello")  # ✅ 올바름 - 탭으로 indentation
   }
   ```

   ```dsp
   config.script={
       import os  # ❌ 잘못됨 - 불필요한 leading space
   }
   ```

3. **플러그인 디렉토리 참조**: 외부 파일 경로는 반드시 `DSYNC_PLUGIN_DIR` 환경 변수를 사용하세요.

   ```python
   # ✅ 올바른 방법
   PLUGIN_DIR = os.environ.get('DSYNC_PLUGIN_DIR', '/root/dxync/plugins')
   CONFIG_FILE = os.path.join(PLUGIN_DIR, 'myconfig', 'settings.conf')

   # ❌ 잘못된 방법 (임시 파일 경로로 잡힘)
   PLUGIN_DIR = os.path.dirname(os.path.abspath(__file__))
   ```

4. **에러 처리**: 외부 설정 파일이 없을 경우를 대비한 에러 처리를 추가하세요.

   ```python
   try:
       config = load_config()
   except FileNotFoundError:
       print(f"❌ 설정 파일을 찾을 수 없습니다: {CONFIG_FILE}")
       sys.exit(1)
   ```

5. **타임아웃 설정**: Python 스크립트가 오래 실행될 수 있다면 `config.timeout` 값을 적절히 설정하세요 (초 단위). 알아두세요. `config.timeout`값은 최대 3600초(1시간)까지만 사용할 수 있습니다. 

**Python vs Bash 선택 기준:**

| 상황 | 권장 언어 | 이유 |
|------|-----------|------|
| JSON 파싱 필요 | Python | 내장 json 모듈 사용 가능 |
| 복잡한 데이터 처리 | Python | 리스트/딕셔너리 조작 용이 |
| HTTP API 호출 | Python | requests 라이브러리 사용 |
| 간단한 명령 실행 | Bash | 간결하고 빠름 |
| 파일 시스템 작업 | Bash | find, grep 등 네이티브 도구 |
| 외부 설정 파일 | Python | 파싱 및 검증 로직 구현 용이 |

---

## 옵션형 플러그인 (Option-Based Plugins)

**v25.11.1부터 지원**

옵션형 플러그인은 동기화 작업 없이 독립적으로 실행할 수 있는 특별한 플러그인입니다. 사용자 정의 CLI 옵션을 추가하여 `dsync -custom-option` 형태로 플러그인을 직접 실행할 수 있습니다.

### 주요 특징

- ✅ **CLI 옵션 추가**: 플러그인이 자체 명령줄 옵션을 등록
- ✅ **독립 실행**: 동기화 없이 플러그인 기능만 실행
- ✅ **깔끔한 출력**: 디버깅 로그 없이 플러그인 출력만 표시
- ✅ **충돌 감지**: 중복된 옵션 플래그 자동 감지 및 에러 표시
- ✅ **자동 help 통합**: `-help` 옵션에 플러그인 옵션 자동 추가

### 옵션형 플러그인 필드

일반 플러그인 필드 외에 다음 필드를 추가합니다:

| 필드 | 타입 | 필수 | 설명 | 예시 |
|------|------|------|------|------|
| `option` | boolean | ✅ | 옵션형 플러그인 활성화 | `option=true` |
| `option_flag` | string | ✅ | CLI 옵션 플래그 (`-`로 시작) | `option_flag="-email"` |
| `option_description` | string | ⚪ | 옵션 설명 (help 출력용) | `option_description="테스트 이메일 전송"` |

### 기본 예제: Hello World
알아두세요. `config.timeout`값은 최대 3600초(1시간)까지만 사용할 수 있습니다. 더 오래 실행되어야 한다면 별도의 프로그램으로 만들어서 호출해야 합니다.
```dsp
name="Hello World"
version="1.0"
author="관리자"
description="Hello World를 출력하는 옵션형 플러그인"
type="script"
enabled=true
priority=10

// 옵션형 플러그인 설정
option=true
option_flag="-hello"
option_description="Hello World 출력"

config.interpreter="/bin/bash"
config.timeout=10

config.script={
#!/bin/bash
echo "=========================================="
echo "  Hello World from DSync Plugin!"
echo "=========================================="
echo ""
echo "현재 시각: $(date '+%Y년 %m월 %d일 %H:%M:%S')"
}
```

**사용 방법:**
```bash
# 플러그인 실행
./dsync -hello

# 출력:
# ==========================================
#   Hello World from DSync Plugin!
# ==========================================
#
# 현재 시각: 2025년 10월 15일 21:51:28
```

### 실전 예제 1: 이메일 테스트 도구

```dsp
name="이메일 테스트"
version="1.0"
author="관리자"
description="이메일 전송 테스트 플러그인"
type="script"
enabled=true

// 옵션형 플러그인 설정
option=true
option_flag="-test-email"
option_description="SMTP 이메일 전송 테스트"

config.interpreter="/usr/bin/python3"
config.timeout=30

config.script={
import smtplib
import os
import sys
from email.mime.text import MIMEText
from datetime import datetime

# 외부 설정 파일 로드
PLUGIN_DIR = os.environ.get('DSYNC_PLUGIN_DIR', '/root/dxync/plugins')
CONFIG_FILE = os.path.join(PLUGIN_DIR, 'email-key', 'login.conf')

def load_config():
    config = {}
    with open(CONFIG_FILE, 'r') as f:
        for line in f:
            line = line.strip()
            if line and not line.startswith('#') and '=' in line:
                key, value = line.split('=', 1)
                config[key.strip()] = value.strip()
    return config

def send_test_email(config):
    msg = MIMEText(f"테스트 이메일 - {datetime.now()}", 'plain', 'utf-8')
    msg['Subject'] = "[DSync] 이메일 테스트"
    msg['From'] = config['FROM_EMAIL']
    msg['To'] = config['TO_EMAIL']

    with smtplib.SMTP(config['SMTP_HOST'], int(config['SMTP_PORT'])) as server:
        server.starttls()
        server.login(config['SMTP_USER'], config['SMTP_PASSWORD'])
        server.send_message(msg)

    print(f"✅ 테스트 이메일 전송 성공!")
    print(f"   발신: {config['FROM_EMAIL']}")
    print(f"   수신: {config['TO_EMAIL']}")

if __name__ == '__main__':
    config = load_config()
    send_test_email(config)
}
```

**사용 방법:**
```bash
./dsync -test-email
```

### 실전 예제 2: 동기화 통계 조회

```dsp
name="동기화 통계 조회"
version="1.0"
author="관리자"
description="최근 동기화 통계를 조회하는 플러그인"
type="script"
enabled=true

option=true
option_flag="-stats"
option_description="최근 동기화 통계 조회"

config.interpreter="/usr/bin/python3"
config.timeout=10

config.script={
import os
import json
from datetime import datetime

PLUGIN_DIR = os.environ.get('DSYNC_PLUGIN_DIR', '/root/dxync/plugins')
STATS_FILE = os.path.join(PLUGIN_DIR, 'stats', 'last_sync.json')

def load_stats():
    if not os.path.exists(STATS_FILE):
        print("❌ 통계 파일을 찾을 수 없습니다.")
        return None

    with open(STATS_FILE, 'r') as f:
        return json.load(f)

def display_stats(stats):
    print("=" * 60)
    print("  최근 동기화 통계")
    print("=" * 60)
    print(f"실행 시각: {datetime.fromtimestamp(stats['timestamp']).strftime('%Y-%m-%d %H:%M:%S')}")
    print(f"성공: {stats['success_count']}개")
    print(f"실패: {stats['fail_count']}개")
    print(f"총 전송량: {stats['total_size_mb']:.2f} MB")
    print("=" * 60)

if __name__ == '__main__':
    stats = load_stats()
    if stats:
        display_stats(stats)
}
```

**사용 방법:**
```bash
./dsync -stats
```

### help 출력 통합

옵션형 플러그인은 자동으로 `-help` 출력에 포함됩니다:

```bash
./dsync -help

# 출력 예시:
# Usage of ./dsync:
#   -hello
#         Hello World 출력 《플러그인 제공》
#   -test-email
#         SMTP 이메일 전송 테스트 《플러그인 제공》
#   -stats
#         최근 동기화 통계 조회 《플러그인 제공》
#   -test
#         테스트 모드 활성화 (실제 동기화 없이 설정과 연결만 확인)
#   ...
```

### 충돌 감지

여러 플러그인이 동일한 `option_flag`를 사용하면 자동으로 감지하고 에러를 표시하고 동작을 중단합니다:

```bash
./dsync -hello

# 출력:
# 오류: 충돌되는 플러그인이 (Hello World, Hello World 2) 있어 옵션 '-hello'는 사용할 수 없습니다.
# 플러그인 파일을 확인하여 하나만 활성화하세요.
```

### 옵션형 vs 일반 플러그인 비교

| 특징 | 일반 플러그인 | 옵션형 플러그인 |
|------|--------------|----------------|
| 실행 시점 | 동기화 완료 후 자동 실행 | 사용자가 옵션으로 직접 실행 |
| 동기화 여부 | 동기화 작업 수행 | 기본적으로 동기화 없음 (필요시 자식 프로세스로 실행 가능) |
| 컨텍스트 정보 | 동기화 결과, 통계 등 제공 | 기본적으로 빈 컨텍스트 (자식 프로세스로 동기화 실행 가능) |
| 출력 | 로그와 함께 출력 | 플러그인 출력만 깔끔하게 표시 |
| 용도 | 동기화 후 알림, 백업 등 | 독립 도구, 테스트, 조회, 디버깅 등 |
| `enabled` 필드 | `false`면 실행 안 됨 | 값과 관계없이 동기화 후 자동 실행 안 됨 |

### 사용 사례

#### 1. 테스트 및 검증 도구
```bash
./dsync -test-email    # 이메일 설정 테스트
./dsync -test-webhook  # 웹훅 연결 테스트
./dsync -validate      # 설정 파일 검증
```

#### 2. 조회 및 모니터링
```bash
./dsync -stats         # 동기화 통계 조회
./dsync -status        # 저장소 상태 확인
./dsync -disk-usage    # 디스크 사용량 조회
```

#### 3. 유틸리티 도구
```bash
./dsync -cleanup       # 오래된 로그 정리
./dsync -backup        # 수동 백업 실행
./dsync -report        # 리포트 생성
```

#### 4. 디버깅 및 로깅
```bash
./dsync -debug         # 동기화 전체 출력을 로그 파일에 기록
./dsync -verbose       # 상세 로그 출력
```

### 실전 예제 3: 디버그 모드 (자식 프로세스 실행)

이 예제는 옵션형 플러그인이 자식 프로세스로 실제 동기화를 실행하고, 모든 출력을 캡처하여 로그 파일에 기록하는 방법을 보여줍니다.

```dsp
name="디버그 모드"
version="1.0"
author="관리자"
description="동기화를 실행하고 모든 출력을 로그 파일에 기록"
type="script"
enabled=false
priority=10

option=true
option_flag="-debug"
option_description="디버그 모드로 동기화 실행 (모든 출력 기록)"

config.interpreter="/usr/bin/python3"
config.timeout=600

config.script={
import subprocess
import sys
import os
from datetime import datetime

def run_debug_sync():
    plugin_dir = os.environ.get('DSYNC_PLUGIN_DIR', '/root/dxync/plugins')
    export_dir = os.path.join(plugin_dir, 'export')
    os.makedirs(export_dir, exist_ok=True)
    log_file = os.path.join(export_dir, 'debug.log')

    print("=" * 60)
    print("  디버그 모드 시작")
    print("=" * 60)
    print(f"실행 시각: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")
    print(f"로그 파일: {log_file}")
    print(f"명령: dsync -sync kali")
    print("=" * 60)
    print()

    # 재귀 방지: 이 플러그인 파일을 임시로 이름 변경
    current_file = os.path.join(plugin_dir, 'debug.dsp')
    temp_file = current_file + '.running'

    try:
        if os.path.exists(current_file):
            os.rename(current_file, temp_file)
            print(f"🔒 플러그인 파일 임시 비활성화: {os.path.basename(temp_file)}")

        # 자식 프로세스로 dsync 실행
        dsync_path = '/root/daxnet/daxync/client/dxync'
        process = subprocess.Popen(
            [dsync_path, '-sync', 'kali'],
            stdout=subprocess.PIPE,
            stderr=subprocess.STDOUT,
            text=True,
            bufsize=1
        )

        # 출력을 실시간으로 캡처하여 콘솔과 파일에 모두 기록
        with open(log_file, 'w', encoding='utf-8') as f:
            f.write("=" * 80 + "\n")
            f.write(f"  DSync 디버그 로그\n")
            f.write("=" * 80 + "\n")
            f.write(f"실행 시각: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}\n")
            f.write(f"명령: dsync -sync kali\n")
            f.write("=" * 80 + "\n\n")

            for line in process.stdout:
                print(line, end='')
                f.write(line)
                f.flush()

            return_code = process.wait()
            f.write("\n" + "=" * 80 + "\n")
            f.write(f"종료 시각: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}\n")
            f.write(f"종료 코드: {return_code}\n")
            f.write("=" * 80 + "\n")

        print()
        print("=" * 60)
        if return_code == 0:
            print("✅ 동기화 성공")
        else:
            print(f"❌ 동기화 실패 (종료 코드: {return_code})")
        print(f"📝 디버그 로그 저장됨: {log_file}")
        print("=" * 60)

        return return_code
    except Exception as e:
        print(f"❌ 오류 발생: {e}")
        return 1
    finally:
        # 플러그인 파일 복원
        if os.path.exists(temp_file):
            os.rename(temp_file, current_file)
            print(f"🔓 플러그인 파일 복원 완료: {os.path.basename(current_file)}")

if __name__ == '__main__':
    sys.exit(run_debug_sync())
}
```

**사용 방법:**
```bash
./dsync -debug

# 출력:
# ============================================================
#   디버그 모드 시작
# ============================================================
# 실행 시각: 2025-10-15 22:30:15
# 로그 파일: /root/dxync/plugins/export/debug.log
# 명령: dsync -sync kali
# ============================================================
#
# 🔒 플러그인 파일 임시 비활성화: debug.dsp.running
# [동기화 출력 내용...]
#
# ============================================================
# ✅ 동기화 성공
# 📝 디버그 로그 저장됨: /root/dxync/plugins/export/debug.log
# ============================================================
# 🔓 플러그인 파일 복원 완료: debug.dsp
```

**핵심 기법:**

1. **재귀 방지**: `debug.dsp` → `debug.dsp.running`으로 임시 변경하여 자식 프로세스에서 이 플러그인이 다시 로드되는 것을 방지

2. **실시간 출력 캡처**: `subprocess.Popen`과 `stdout.readline()`을 사용하여 출력을 실시간으로 읽음

3. **이중 출력**: 콘솔과 파일에 동시에 출력하여 사용자가 진행 상황을 볼 수 있게 함

4. **안전한 복원**: `try-finally` 블록으로 예외가 발생해도 플러그인 파일이 복원되도록 보장

### 주의사항

1. **컨텍스트 정보**:
   - 기본적으로 빈 컨텍스트로 실행됩니다 (`sync_results`, `configs` 등은 빈 배열)
   - 필요시 자식 프로세스로 `dsync -sync <name>`을 실행하여 동기화 수행 가능
   - 자식 프로세스의 출력을 캡처하여 로그 파일 생성이나 분석 가능 (예: debug 플러그인)

2. **플래그 네이밍**:
   - 기존 dsync 옵션과 중복되지 않도록 주의 (`-test`, `-help` 등)
   - 하이픈으로 시작 (예: `-email`, `-stats`)
   - 소문자와 하이픈만 사용 권장

3. **보안**:
   - `plugins-power=true` 설정 필요 (script/exec 타입 사용 시)
   - 민감한 정보는 외부 설정 파일 사용

4. **우선순위 및 실행**:
   - 옵션형 플러그인의 `priority` 값은 무시됨
   - 한 번에 하나의 플러그인만 실행
   - `enabled` 필드 값과 관계없이 동기화 완료 후 자동 실행되지 않음

5. **재귀 방지**:
   - 자식 프로세스로 dsync를 실행하는 경우, 무한 재귀 방지 필요
   - 플러그인 파일을 임시로 `.running` 확장자로 변경하여 자식 프로세스에서 로드되지 않게 함

### 옵션 대체 시스템 (Option Override System)

**v25.11.1부터 지원**

옵션형 플러그인은 dsync의 일부 기본 옵션을 대체할 수 있습니다. 이 기능을 통해 시스템 옵션의 동작을 플러그인으로 커스터마이징할 수 있습니다.

#### 보호된 옵션 (Protected Options)

다음 시스템 중요 옵션들은 **절대 대체할 수 없습니다**:

```
-help, -emoff, -force, -gendsync, -nolog,
-service, -stop-service, -startup, -sync,
-test, -test-plugin
```

플러그인이 보호된 옵션을 사용하려고 하면 다음과 같은 경고 메시지가 표시되고 플러그인이 로드되지 않습니다:

```bash
[ 경고 ] "플러그인명"은 잘못된 구성을 갖고 있습니다. 따라서 이 플러그인은 사용할 수 없습니다.
```

#### 대체 가능한 옵션 (Replaceable Options)

보호된 옵션을 제외한 모든 dsync 옵션은 플러그인으로 대체할 수 있습니다. 대표적인 예시:

- `-example`: 설정 파일 예제 보기
- `-cleanlog`: 오래된 로그 파일 정리
- `-setup`: 간편 설정 마법사
- `-list-plugins`: 플러그인 목록 조회
- `-create-plugin`: 플러그인 생성 도구

#### 네임스페이스 시스템

플러그인이 시스템 옵션을 대체할 때, 마치 **Minecraft의 네임스페이스 시스템**과 같은 방식으로 동작합니다:

**1. 플러그인이 옵션을 대체하지 않을 때:**
```bash
./dxync -cleanlog    # 시스템 기본 기능 실행
```

**2. 플러그인이 `-cleanlog`를 대체할 때:**
```bash
./dxync -cleanlog        # 플러그인 버전 실행 (우선순위)
./dxync -sys:cleanlog    # 시스템 기본 기능 실행
```

**3. `-sys:` 프리픽스:**
- 플러그인이 옵션을 대체할 때만 `-sys:` 버전이 자동으로 등록됩니다
- 플러그인이 없으면 `-sys:` 옵션도 나타나지 않습니다
- 항상 시스템 기본 기능을 강제로 사용하고 싶을 때 사용

#### 실전 예제: 커스텀 로그 정리 플러그인

시스템의 `-cleanlog` 옵션을 플러그인으로 대체하는 예제입니다:

```dsp
name="고급 로그 정리"
version="1.0"
author="관리자"
description="시스템 cleanlog를 대체하는 고급 로그 정리 플러그인"
type="script"
enabled=true
priority=10

// 시스템 옵션 대체
option=true
option_flag="-cleanlog"
option_description="오래된 로그 파일 정리 (고급 기능)"

config.interpreter="/usr/bin/python3"
config.timeout=30

config.script={
import os
import time
from datetime import datetime, timedelta

def clean_logs():
    log_dir = "/root/dxync/logs"
    days_old = 7
    cutoff_time = time.time() - (days_old * 86400)

    print("=" * 60)
    print("  고급 로그 정리 (플러그인 버전)")
    print("=" * 60)
    print(f"대상 디렉토리: {log_dir}")
    print(f"보관 기간: {days_old}일")
    print("=" * 60)

    total_deleted = 0
    total_size = 0

    for filename in os.listdir(log_dir):
        filepath = os.path.join(log_dir, filename)

        if not os.path.isfile(filepath):
            continue

        if not filename.endswith('.log'):
            continue

        file_time = os.path.getmtime(filepath)

        if file_time < cutoff_time:
            file_size = os.path.getsize(filepath)
            total_size += file_size
            total_deleted += 1

            print(f"🗑️  삭제: {filename} ({file_size / 1024:.2f} KB)")
            os.remove(filepath)

    print("=" * 60)
    print(f"✅ 총 {total_deleted}개 파일 삭제 ({total_size / 1024 / 1024:.2f} MB)")
    print("=" * 60)

if __name__ == '__main__':
    clean_logs()
}
```

**사용 방법:**

```bash
# 플러그인 버전 실행 (고급 기능)
./dxync -cleanlog

# 출력:
# ============================================================
#   고급 로그 정리 (플러그인 버전)
# ============================================================
# 대상 디렉토리: /root/dxync/logs
# 보관 기간: 7일
# ============================================================
# 🗑️  삭제: sync_2025-10-01.log (1.23 MB)
# 🗑️  삭제: sync_2025-10-02.log (0.98 MB)
# ============================================================
# ✅ 총 2개 파일 삭제 (2.21 MB)
# ============================================================

# 시스템 기본 기능 강제 실행
./dxync -sys:cleanlog
```

#### help 출력에서의 표시

플러그인이 옵션을 대체하면 help 출력에서 명확히 표시됩니다:

```bash
./dxync -help

# 플러그인이 -cleanlog를 대체하지 않을 때:
#   -cleanlog
#         오래된 로그 파일 정리 (7일 이상 지난 로그)

# 플러그인이 -cleanlog를 대체할 때:
#   -cleanlog
#         오래된 로그 파일 정리 (고급 기능) 《플러그인 제공》
#   -sys:cleanlog
#         오래된 로그 파일 정리 (시스템 기능)
```

#### 동작 원리

1. **플러그인 로드 단계** (main.go:239-250):
   - 모든 옵션형 플러그인 검사
   - 보호된 옵션 사용 시도 시 플러그인 차단 및 경고 출력
   - 유효한 플러그인만 로드

2. **플래그 등록 단계** (main.go:300-336):
   - 플러그인이 옵션을 대체하면:
     - 플러그인 버전: `-옵션명` (플러그인 함수 실행)
     - 시스템 버전: `-sys:옵션명` (시스템 기능 실행)
   - 플러그인이 대체하지 않으면:
     - 일반 옵션: `-옵션명` (시스템 기능 실행)
     - `-sys:` 버전은 등록되지 않음

3. **실행 단계**:
   - 사용자가 `-옵션명` 입력 → 플러그인 있으면 플러그인 실행, 없으면 시스템 실행
   - 사용자가 `-sys:옵션명` 입력 → 항상 시스템 기능 실행

#### 주의사항

1. **보호된 옵션 확인**:
   - 플러그인 개발 전 보호된 옵션 목록 확인 필수
   - 보호된 옵션 사용 시 플러그인 로드 자체가 차단됨

2. **네이밍 일관성**:
   - 시스템 옵션과 동일한 이름 사용 시 의도를 명확히 할 것
   - `option_description`에 "플러그인 버전", "고급 기능" 등 명시 권장

3. **하위 호환성**:
   - 기존 스크립트나 자동화에서 `-옵션명` 사용 시 플러그인으로 교체됨
   - 안전하게 시스템 기능 사용하려면 `-sys:옵션명` 사용

4. **플러그인 우선순위**:
   - 항상 플러그인이 시스템 옵션보다 우선 실행됨
   - 시스템 기능이 필요하면 반드시 `-sys:` 프리픽스 사용

#### 코드 위치 참조

이 기능의 구현은 다음 파일에서 확인할 수 있습니다:

- **main.go:39-51**: 보호된 옵션 목록 정의
- **main.go:191-200**: `isProtectedOption()` 검증 함수
- **main.go:239-250**: 플러그인 로드 시 보호된 옵션 필터링
- **main.go:300-336**: 조건부 플래그 등록

---

## 시간 형식화 함수

DSYNC 플러그인에서 타임스탬프를 읽기 쉬운 형식으로 변환할 수 있는 함수들을 제공합니다.

### 사용 가능한 시간 함수

#### 1. `formatTime` - 커스터마이징 가능한 시간 형식화

템플릿에서 타임스탬프를 원하는 형식으로 변환합니다.

```dsp
// 기본 형식 (ISO-8601)
{{ formatTime .Timestamp }}
→ 2025-10-13 22:05:02

// 한국어 형식
{{ formatTime .Timestamp "2006년 01월 02일 15:04:05" }}
→ 2025년 10월 13일 22:05:02

// 날짜만
{{ formatTime .Timestamp "2006-01-02" }}
→ 2025-10-13

// 시간만
{{ formatTime .Timestamp "15:04:05" }}
→ 22:05:02

// 유닉스 타임스탬프 그대로
{{ formatTime .Timestamp "unix" }}
→ 1760360702
```

#### 2. `formatTimeKorean` - 한국어 형식 (고정)

한국어 형식으로 간편하게 변환합니다.

```dsp
{{ formatTimeKorean .Timestamp }}
→ 2025년 10월 13일 22:05:02
```

#### 3. `now` - 현재 시간

현재 시간을 유닉스 타임스탬프로 반환합니다.

```dsp
{{ now }}
→ 1760360702 (현재 시간)
```

### 시간 형식 지정 방법

Go의 표준 시간 형식을 사용합니다. **2006-01-02 15:04:05**를 기준으로 원하는 형식을 만듭니다.

| 요소 | 형식 코드 | 예시 |
|------|-----------|------|
| 년 (4자리) | `2006` | 2025 |
| 년 (2자리) | `06` | 25 |
| 월 (숫자, 2자리) | `01` | 10 |
| 월 (숫자, 1-2자리) | `1` | 10 |
| 월 (영어) | `Jan` | Oct |
| 일 (2자리) | `02` | 13 |
| 일 (1-2자리) | `2` | 13 |
| 요일 (약자) | `Mon` | Sun |
| 요일 (전체) | `Monday` | Sunday |
| 시 (24시간, 2자리) | `15` | 22 |
| 시 (12시간, 2자리) | `03` | 10 |
| 시 (12시간, 1-2자리) | `3` | 10 |
| 분 | `04` | 05 |
| 초 | `05` | 02 |
| AM/PM | `PM` | PM |

### 실제 사용 예제

#### Discord 웹훅 with 시간 형식

```dsp
name="Discord 시간 형식"
version="1.0"
type="webhook"
enabled=true
priority=10

config.url="https://discord.com/api/webhooks/YOUR_WEBHOOK_URL"
config.format="discord"
config.username="DSYNC Bot"

config.template={
🔔 **동기화 완료**

⏰ 완료 시간: {{ formatTimeKorean .Timestamp }}
📦 저장소: {{ len .SyncResults }}개
🖥️ 서버: {{ index .GlobalConfig "version" }}
}
```

#### Telegram with 커스텀 시간 형식

```dsp
name="Telegram 알림"
version="1.0"
type="webhook"
enabled=true
priority=15

config.url="https://api.telegram.org/botYOUR_BOT_TOKEN/sendMessage"
config.format="custom"

config.json={
{
  "chat_id": "YOUR_CHAT_ID",
  "text": "{{ .message }}"
}
}

config.template={
🔔 DSYNC 동기화 완료

⏰ {{ formatTime .Timestamp "2006년 1월 2일 15:04" }}
📦 {{ len .SyncResults }}개 저장소 처리 완료
}
```

#### 다양한 시간 형식 예제

```dsp
config.template={
**다양한 시간 형식:**

• 기본: {{ formatTime .Timestamp }}
• 한국어: {{ formatTimeKorean .Timestamp }}
• 날짜만: {{ formatTime .Timestamp "2006-01-02" }}
• 시간만: {{ formatTime .Timestamp "15:04:05" }}
• 년/월/일: {{ formatTime .Timestamp "2006/01/02" }}
• 짧은 형식: {{ formatTime .Timestamp "06-01-02 15:04" }}
• 영어: {{ formatTime .Timestamp "Mon Jan 02 15:04:05 2006" }}
• RFC3339: {{ formatTime .Timestamp "2006-01-02T15:04:05Z07:00" }}
• 유닉스: {{ formatTime .Timestamp "unix" }}
}
```

### 시간 형식 비교표

| 원하는 형식 | 형식 코드 | 결과 예시 |
|-------------|-----------|-----------|
| 기본 ISO | `2006-01-02 15:04:05` | 2025-10-13 22:05:02 |
| 한국어 긴 형식 | `2006년 01월 02일 15시 04분 05초` | 2025년 10월 13일 22시 05분 02초 |
| 한국어 짧은 형식 | `2006년 1월 2일 15:04` | 2025년 10월 13일 22:05 |
| 슬래시 구분 | `2006/01/02 15:04` | 2025/10/13 22:05 |
| 영어 형식 | `Mon, Jan 02 2006 15:04:05` | Sun, Oct 13 2025 22:05:02 |
| 날짜만 | `2006-01-02` | 2025-10-13 |
| 시간만 | `15:04:05` | 22:05:02 |
| 12시간 형식 | `2006-01-02 03:04 PM` | 2025-10-13 10:05 PM |

### 기존 script 플러그인 업그레이드

#### 이전 방식 (bash + date 명령어)

```dsp
type="script"
config.interpreter="/bin/bash"

config.script={
#!/bin/bash
TIMESTAMP=$(echo "$DSYNC_CONTEXT" | jq '.timestamp')
FORMATTED_TIME=$(date -d @"$TIMESTAMP" +"%Y년 %m월 %d일 %H:%M:%S")
echo "완료 시간: $FORMATTED_TIME"
}
```

#### 개선된 방식 (webhook + 템플릿)

```dsp
type="webhook"
config.url="YOUR_WEBHOOK_URL"

config.template={
완료 시간: {{ formatTimeKorean .Timestamp }}
}
```

**장점:**
- ✅ 더 간단하고 읽기 쉬움
- ✅ bash/jq 같은 외부 도구 불필요
- ✅ 크로스 플랫폼 호환성
- ✅ 에러 처리 자동화

### 주의사항

1. **타임스탬프 접근**: `.Timestamp`는 자동으로 제공되는 컨텍스트 변수입니다
2. **형식 코드 대소문자**: 대소문자를 정확히 지켜야 합니다 (예: `Mon` ≠ `mon`)
3. **유닉스 형식**: `"unix"` 문자열을 사용하면 타임스탬프를 그대로 반환합니다
4. **기본값**: 형식을 지정하지 않으면 `2006-01-02 15:04:05` 형식이 사용됩니다

### 문제 해결

**Q: 시간이 9시간 빠르게 표시됩니다**
A: 서버의 타임존 설정을 확인하세요. `TZ` 환경변수를 설정할 수 있습니다.

**Q: 한국어가 깨져서 보입니다**
A: 웹훅 플랫폼이 UTF-8을 지원하는지 확인하세요. Discord, Slack은 기본 지원합니다.

**Q: 형식이 적용되지 않습니다**
A: 형식 코드의 대소문자와 공백을 정확히 확인하세요. 예: `2006-01-02` (O), `2006-01-2` (X)

## 고급 기능: DSP 함수 (v2.0)

DSP 플러그인 시스템은 Go와 유사한 문법으로 함수를 정의하고 호출할 수 있는 강력한 기능을 제공합니다. 이를 통해 복잡한 로직을 구현하고 코드를 재사용할 수 있습니다.

### 함수 정의 문법

```dsp
func 함수명(파라미터1, 파라미터2) {
    // 함수 본문
    return 결과값
}
```

### 함수 기능 예제

#### 1. 웹훅 플러그인 with 함수

```dsp
// 고급 Discord 알림 with 함수
name="스마트 Discord 알림"
version="2.0"
author="관리자"
description="함수를 활용한 스마트 Discord 알림"
type="webhook"
enabled=true
priority=10

// 저장소 이름을 포맷팅하는 함수
func formatRepo(name) {
    if contains(name, "ubuntu") {
        return "🐧 " + name
    }
    if contains(name, "centos") {
        return "🎩 " + name
    }
    if contains(name, "debian") {
        return "🌀 " + name
    }
    return "📦 " + name
}

// 상태에 따른 이모지를 반환하는 함수
func getStatusEmoji(success) {
    if success {
        return "✅"
    }
    return "❌"
}

// 파일 크기를 사람이 읽기 쉬운 형식으로 변환
func humanSize(bytes) {
    if bytes > 1073741824 {
        gb = bytes / 1073741824
        return str(gb) + " GB"
    }
    if bytes > 1048576 {
        mb = bytes / 1048576
        return str(mb) + " MB"
    }
    if bytes > 1024 {
        kb = bytes / 1024
        return str(kb) + " KB"
    }
    return str(bytes) + " B"
}

// 동기화 시간을 포맷팅
func formatDuration(seconds) {
    if seconds > 3600 {
        hours = seconds / 3600
        mins = (seconds % 3600) / 60
        return str(hours) + "시간 " + str(mins) + "분"
    }
    if seconds > 60 {
        mins = seconds / 60
        secs = seconds % 60
        return str(mins) + "분 " + str(secs) + "초"
    }
    return str(seconds) + "초"
}

config.url="https://discord.com/api/webhooks/1234567890/abcdefg"
config.format="discord"
config.username="DSYNC 스마트 봇"

// 템플릿에서 함수 호출
config.template={
🔔 **동기화 완료 알림**

📊 **동기화 결과:**
총 {{len .SyncResults}}개 저장소 처리

{{range .SyncResults}}
{{call "getStatusEmoji" .Success}} {{call "formatRepo" .RepoName}}
{{if not .Success}}   ⚠️ 실패 - 로그: {{.LogFile}}{{end}}
{{end}}

📈 **통계 정보:**
{{range $repo, $stats := .Stats}}
• {{$repo}}: {{call "humanSize" $stats.TotalBytes}} / {{call "formatDuration" $stats.TotalTime}}
{{end}}

⏰ 완료 시간: {{.Timestamp}}
}
```

#### 2. 복잡한 조건부 로직을 가진 웹훅

```dsp
// 조건부 알림 플러그인
name="스마트 알림"
version="2.0"
author="DevOps팀"
description="실패율에 따라 다른 채널로 알림"
type="webhook"
enabled=true
priority=5

// 실패율 계산 함수
func getFailureRate() {
    total = len(.SyncResults)
    failed = 0
    foreach(.SyncResults, func(result) {
        if not result.Success {
            failed = failed + 1
        }
    })
    if total == 0 {
        return 0
    }
    return (failed * 100) / total
}

// 심각도 레벨 판단 함수
func getSeverity() {
    rate = call("getFailureRate")
    if rate >= 80 {
        return "critical"
    }
    if rate >= 50 {
        return "major"
    }
    if rate >= 20 {
        return "minor"
    }
    if rate > 0 {
        return "warning"
    }
    return "success"
}

// 웹훅 URL 선택 함수
func getWebhookUrl() {
    severity = call("getSeverity")
    if severity == "critical" {
        return "https://hooks.slack.com/critical-channel"
    }
    if severity == "major" {
        return "https://hooks.slack.com/alerts-channel"
    }
    return "https://hooks.slack.com/general-channel"
}

// 메시지 색상 선택 함수
func getColor() {
    severity = call("getSeverity")
    if severity == "critical" {
        return "#FF0000"
    }
    if severity == "major" {
        return "#FF8800"
    }
    if severity == "minor" {
        return "#FFFF00"
    }
    if severity == "warning" {
        return "#00CCFF"
    }
    return "#00FF00"
}

config.url="{{call \"getWebhookUrl\"}}"
config.format="slack"
config.override=true

config.template={
{
    "attachments": [
        {
            "color": "{{call \"getColor\"}}",
            "title": "동기화 완료 - {{call \"getSeverity\" | upper}}",
            "text": "실패율: {{call \"getFailureRate\"}}%",
            "fields": [
                {
                    "title": "총 저장소",
                    "value": "{{len .SyncResults}}",
                    "short": true
                },
                {
                    "title": "실패",
                    "value": "{{set \"failed\" 0}}{{foreach .SyncResults \"countFailed\"}}{{get \"failed\"}}",
                    "short": true
                }
            ]
        }
    ]
}
}

// 실패 카운트용 헬퍼 함수
func countFailed(result) {
    if not result.Success {
        current = get("failed")
        set("failed", current + 1)
    }
}
```

#### 3. 데이터 집계 함수 예제

```dsp
// 통계 집계 플러그인
name="통계 분석기"
version="2.0"
author="분석팀"
description="동기화 통계를 분석하고 리포트 생성"
type="webhook"
enabled=true
priority=25

// 총 전송 데이터 계산
func getTotalTransferred() {
    total = 0
    foreach(.Stats, func(repo, stats) {
        total = total + stats.TotalBytes
    })
    return total
}

// 평균 동기화 시간 계산
func getAverageTime() {
    totalTime = 0
    count = 0
    foreach(.Stats, func(repo, stats) {
        totalTime = totalTime + stats.TotalTime
        count = count + 1
    })
    if count == 0 {
        return 0
    }
    return totalTime / count
}

// 가장 느린 저장소 찾기
func getSlowestRepo() {
    slowest = ""
    maxTime = 0
    foreach(.Stats, func(repo, stats) {
        if stats.TotalTime > maxTime {
            maxTime = stats.TotalTime
            slowest = repo
        }
    })
    return slowest
}

// 성공률 계산
func getSuccessRate() {
    total = len(.SyncResults)
    success = 0
    foreach(.SyncResults, func(result) {
        if result.Success {
            success = success + 1
        }
    })
    if total == 0 {
        return 100
    }
    return (success * 100) / total
}

config.url="https://webhook.site/statistics"
config.format="custom"

config.json={
{
    "summary": {
        "total_repos": {{len .SyncResults}},
        "success_rate": {{call "getSuccessRate"}},
        "total_transferred": "{{call \"getTotalTransferred\" | call \"formatSize\"}}",
        "average_time": "{{call \"getAverageTime\" | call \"formatDuration\"}}",
        "slowest_repo": "{{call \"getSlowestRepo\"}}"
    },
    "details": [
        {{range $i, $r := .SyncResults}}
        {{if $i}},{{end}}
        {
            "repo": "{{$r.RepoName}}",
            "status": "{{call \"getStatusEmoji\" $r.Success}}",
            "formatted_name": "{{call \"formatRepo\" $r.RepoName}}"
        }
        {{end}}
    ]
}
}
```

### 중요: 템플릿에서 함수 사용하기

**내장 함수는 템플릿에서 직접 사용 가능합니다:**

```dsp
// ✅ 올바른 사용법 (내장 함수)
{{len .SyncResults}}
{{contains .RepoName "ubuntu"}}
{{formatSize $stats.TotalBytes}}
{{formatTimeKorean .Timestamp}}

// ❌ 잘못된 사용법 (call 불필요)
{{call "len" .SyncResults}}
{{call "contains" .RepoName "ubuntu"}}
```

**사용자 정의 함수는 call로 호출:**

```dsp
// 함수 정의
func getRepoEmoji(name) {
    return "🐧"
}

// ✅ 올바른 사용법 (사용자 정의 함수)
{{call "getRepoEmoji" .RepoName}}

// ❌ 잘못된 사용법 (call 필수)
{{getRepoEmoji .RepoName}}
```

### 내장 함수 참조

DSP 런타임은 다음과 같은 내장 함수를 제공합니다 (템플릿에서 직접 사용):

#### 문자열 함수
- `len(value)` - 길이 반환 (문자열 또는 배열)
- `str(value)` - 값을 문자열로 변환
- `contains(str, substr)` - 문자열 포함 여부 확인
- `replace(str, old, new)` - 문자열 치환
- `upper(str)` - 대문자로 변환
- `lower(str)` - 소문자로 변환
- `trim(str)` - 양쪽 공백 제거

#### 숫자 함수
- `int(value)` - 값을 정수로 변환

#### 시간 함수
- `formatTime(timestamp, format)` - 유닉스 타임스탬프를 지정된 형식으로 변환
  - 첫 번째 인자: 유닉스 타임스탬프 (정수)
  - 두 번째 인자: 형식 문자열 (선택, 기본값: "2006-01-02 15:04:05")
  - 예: `formatTime(1760360702, "2006년 01월 02일")` → "2025년 10월 13일"
- `formatTimeKorean(timestamp)` - 한국어 형식으로 변환 (고정 형식)
  - 형식: "2006년 01월 02일 15:04:05"
  - 예: `formatTimeKorean(1760360702)` → "2025년 10월 13일 22:05:02"
- `now()` - 현재 시간을 유닉스 타임스탬프로 반환
  - 예: `now()` → 1760360702

#### 유틸리티 함수
- `formatSize(bytes)` - 바이트를 읽기 쉬운 크기로 변환 (KB, MB, GB)
- `formatDuration(seconds)` - 초를 읽기 쉬운 시간으로 변환

### 템플릿 헬퍼 함수

웹훅 템플릿에서 사용 가능한 헬퍼 함수들:

#### 함수 호출
- `{{call "함수명" 인자1 인자2}}` - DSP 함수 호출

#### 변수 관리
- `{{set "변수명" 값}}` - 변수 설정
- `{{get "변수명"}}` - 변수 값 가져오기

#### 반복문
- `{{foreach 배열 "함수명"}}` - 배열의 각 요소에 대해 함수 실행

#### 조건문
- `{{when 조건 "참값" "거짓값"}}` - 조건부 값 반환

### 함수 사용 팁

1. **함수 이름 규칙**: camelCase 또는 snake_case 사용 권장
2. **변수 스코프**: 함수 내 변수는 로컬 스코프를 가짐
3. **재귀 호출**: 함수는 자기 자신을 호출할 수 있음 (주의 필요)
4. **타입 변환**: 필요시 `str()`, `int()` 함수로 타입 변환
5. **에러 처리**: 함수 실행 중 에러 발생 시 "ERROR: ..." 문자열 반환

### 고급 예제: 조건부 집계

```dsp
// 조건부 집계 함수
func countByCondition(field, value) {
    count = 0
    foreach(.SyncResults, func(result) {
        if result[field] == value {
            count = count + 1
        }
    })
    return count
}

// 사용 예
config.template={
성공: {{call "countByCondition" "Success" true}}개
실패: {{call "countByCondition" "Success" false}}개
}
```

### 함수 디버깅

함수 개발 시 디버깅을 위한 팁:

```dsp
// 디버그 함수
func debug(label, value) {
    // 콘솔에 출력될 내용
    return "[DEBUG " + label + ": " + str(value) + "]"
}

// 사용 예
config.template={
{{call "debug" "Total" (len .SyncResults)}}
{{call "debug" "FailureRate" (call "getFailureRate")}}
}
```

## 플러그인 간 함수 공유 (Cross-Plugin Function Sharing)

DSP v2.0부터 플러그인들이 서로의 함수를 호출할 수 있습니다. Go 언어의 패키지 시스템처럼, 같은 폴더(`~/dxync/plugins/`)에 있는 다른 플러그인에서 정의한 함수를 자유롭게 사용할 수 있습니다.

### 함수 공유 개요

**작동 방식:**
- 모든 플러그인의 DSP 함수는 자동으로 전역 함수 저장소에 등록됩니다
- 함수 검색 순서: **로컬 함수 → 전역 함수 → 내장 함수**
- 모든 플러그인이 로드된 후 함수 호출 가능

**특징:**
- ✅ Go 패키지처럼 자동 공유 (별도 import 불필요)
- ✅ 공통 유틸리티 함수 라이브러리 작성 가능
- ✅ 코드 재사용성 극대화
- ✅ priority로 로드 순서 제어 가능

### 기본 예제: 유틸리티 함수 라이브러리

#### 1. 유틸리티 플러그인 작성 (`utils.dsp`)

```dsp
// 공통 유틸리티 함수 라이브러리
name="유틸리티 함수"
version="1.0"
author="관리자"
description="모든 플러그인이 사용할 수 있는 공통 함수 모음"
type="script"
enabled=true
priority=1  // 가장 먼저 로드되도록 낮은 숫자 설정

// Discord 메시지 포맷팅
func formatDiscordMessage(title, content) {
    return "🔔 **" + title + "**\n" + content
}

// Slack 메시지 포맷팅
func formatSlackMessage(title, content) {
    return ":bell: *" + title + "*\n" + content
}

// 저장소 타입별 이모지
func getRepoEmoji(repoName) {
    if contains(repoName, "ubuntu") {
        return "🐧"
    }
    if contains(repoName, "debian") {
        return "🌀"
    }
    if contains(repoName, "centos") {
        return "🎩"
    }
    return "📦"
}

// 상태 이모지
func getStatusEmoji(isSuccess) {
    if isSuccess {
        return "✅"
    }
    return "❌"
}

// 동기화 결과 요약
func getSyncSummary(syncResults) {
    total = len(syncResults)
    success = 0
    foreach(syncResults, func(result) {
        if result.Success {
            success = success + 1
        }
    })
    return str(success) + "/" + str(total) + " 성공"
}

// 실패 저장소 목록
func getFailedRepos(syncResults) {
    failed = ""
    foreach(syncResults, func(result) {
        if not result.Success {
            if failed != "" {
                failed = failed + ", "
            }
            failed = failed + result.RepoName
        }
    })
    return failed
}

// 가장 느린 저장소 찾기
func getSlowestRepo(stats) {
    slowest = ""
    maxTime = 0
    foreach(stats, func(repo, stat) {
        if stat.TotalTime > maxTime {
            maxTime = stat.TotalTime
            slowest = repo
        }
    })
    return slowest
}

// 더미 스크립트 (실제 실행 없음)
config.interpreter="bash"
config.script={
#!/bin/bash
echo "유틸리티 함수 로드 완료"
}
```

#### 2. 유틸리티 함수 사용 예제 (`discord_with_utils.dsp`)

```dsp
// utils.dsp의 함수들을 사용하는 Discord 알림
name="고급 Discord 알림"
version="2.0"
author="관리자"
description="공유 함수를 활용한 Discord 알림"
type="webhook"
enabled=true
priority=20  // utils.dsp 이후에 로드

config.url="https://discord.com/api/webhooks/YOUR_WEBHOOK_URL"
config.format="discord"
config.username="DSYNC Bot"

// utils.dsp의 함수들을 자유롭게 호출
config.template={
{{call "formatDiscordMessage" "DSync 동기화 완료" "저장소 동기화가 완료되었습니다."}}

**📊 요약:**
{{call "getSyncSummary" .SyncResults}}

**📦 저장소 목록:**
{{range .SyncResults}}
{{call "getStatusEmoji" .Success}} {{call "getRepoEmoji" .RepoName}} **{{.RepoName}}**
{{end}}

**⚠️ 실패 저장소:**
{{$failed := call "getFailedRepos" .SyncResults}}{{if $failed}}{{$failed}}{{else}}없음{{end}}

**🐌 가장 느린 저장소:**
{{call "getSlowestRepo" .Stats}}

⏰ 완료 시간: {{formatTimeKorean .Timestamp}}
}
```

#### 3. Slack 버전 예제 (`slack_with_utils.dsp`)

```dsp
// 같은 유틸리티 함수를 Slack에서도 사용
name="Slack 알림"
version="2.0"
type="webhook"
enabled=true
priority=25

config.url="https://hooks.slack.com/services/YOUR_WEBHOOK_URL"
config.format="slack"

config.template={
{{call "formatSlackMessage" "DSync 동기화 완료" "저장소 동기화가 완료되었습니다."}}

*📊 요약:* {{call "getSyncSummary" .SyncResults}}

*📦 저장소 목록:*
{{range .SyncResults}}
{{call "getStatusEmoji" .Success}} {{call "getRepoEmoji" .RepoName}} {{.RepoName}}
{{end}}

*🐌 가장 느린 저장소:* {{call "getSlowestRepo" .Stats}}
}
```

### 고급 예제: 복잡한 통계 라이브러리

#### 통계 함수 라이브러리 (`stats_lib.dsp`)

```dsp
name="통계 라이브러리"
version="1.0"
description="고급 통계 분석 함수 모음"
type="script"
enabled=true
priority=1

// 성공률 계산 (백분율)
func getSuccessRate(syncResults) {
    total = len(syncResults)
    if total == 0 {
        return 100
    }
    success = 0
    foreach(syncResults, func(result) {
        if result.Success {
            success = success + 1
        }
    })
    return (success * 100) / total
}

// 실패율 계산 (백분율)
func getFailureRate(syncResults) {
    return 100 - call("getSuccessRate", syncResults)
}

// 전체 전송량 계산
func getTotalTransferred(stats) {
    total = 0
    foreach(stats, func(repo, stat) {
        total = total + stat.TotalBytes
    })
    return total
}

// 평균 동기화 시간
func getAverageTime(stats) {
    totalTime = 0
    count = 0
    foreach(stats, func(repo, stat) {
        totalTime = totalTime + stat.TotalTime
        count = count + 1
    })
    if count == 0 {
        return 0
    }
    return totalTime / count
}

// 심각도 레벨 판단
func getSeverityLevel(syncResults) {
    rate = call("getFailureRate", syncResults)
    if rate >= 80 {
        return "CRITICAL"
    }
    if rate >= 50 {
        return "MAJOR"
    }
    if rate >= 20 {
        return "MINOR"
    }
    if rate > 0 {
        return "WARNING"
    }
    return "SUCCESS"
}

// 심각도별 색상 코드
func getSeverityColor(syncResults) {
    level = call("getSeverityLevel", syncResults)
    if level == "CRITICAL" {
        return "#FF0000"
    }
    if level == "MAJOR" {
        return "#FF8800"
    }
    if level == "MINOR" {
        return "#FFFF00"
    }
    if level == "WARNING" {
        return "#00CCFF"
    }
    return "#00FF00"
}

config.interpreter="bash"
config.script={
#!/bin/bash
echo "통계 라이브러리 로드 완료"
}
```

#### 통계 함수 사용 예제 (`advanced_stats_webhook.dsp`)

```dsp
name="고급 통계 알림"
version="2.0"
type="webhook"
enabled=true
priority=30

config.url="https://webhook.site/stats"
config.format="custom"

config.json={
{
    "severity": "{{call \"getSeverityLevel\" .SyncResults}}",
    "color": "{{call \"getSeverityColor\" .SyncResults}}",
    "summary": {
        "success_rate": {{call "getSuccessRate" .SyncResults}},
        "failure_rate": {{call "getFailureRate" .SyncResults}},
        "total_transferred": {{call "getTotalTransferred" .Stats}},
        "average_time": {{call "getAverageTime" .Stats}}
    },
    "formatted": {
        "transferred": "{{formatSize (call \"getTotalTransferred\" .Stats)}}",
        "avg_time": "{{formatDuration (call \"getAverageTime\" .Stats)}}"
    }
}
}
```

### 함수 네임스페이스 (이름 충돌 방지)

여러 플러그인에서 같은 이름의 함수를 정의하면 **나중에 로드된 플러그인의 함수가 우선**됩니다.

#### 이름 충돌 예제

```dsp
// plugin_a.dsp (priority=10)
func formatMessage(msg) {
    return "A: " + msg
}

// plugin_b.dsp (priority=20)
func formatMessage(msg) {
    return "B: " + msg
}

// 결과: plugin_b의 formatMessage가 사용됨 (나중에 로드)
{{call "formatMessage" "test"}} → "B: test"
```

#### 이름 충돌 방지 방법

**방법 1: 접두사 사용**
```dsp
// discord_helpers.dsp
func discord_formatMessage(msg) { ... }
func discord_getColor() { ... }

// slack_helpers.dsp
func slack_formatMessage(msg) { ... }
func slack_getColor() { ... }
```

**방법 2: 명확한 함수명 사용**
```dsp
// 모호한 이름 (충돌 가능)
func format(data) { ... }
func process(value) { ... }

// 명확한 이름 (충돌 방지)
func formatDiscordEmbed(data) { ... }
func processRepoStatistics(value) { ... }
```

### 로드 순서 제어 (priority)

공유 함수 라이브러리는 **낮은 priority 값**으로 설정하여 다른 플러그인보다 먼저 로드되도록 해야 합니다.

```dsp
// 권장 priority 값
priority=1-10   // 라이브러리 플러그인 (먼저 로드)
priority=20-40  // 일반 알림 플러그인
priority=50-70  // 후처리 플러그인
priority=80-100 // 정리 작업 플러그인
```

**예제:**
```dsp
// utils.dsp - 공통 라이브러리
priority=1

// discord.dsp - utils 함수 사용
priority=20

// cleanup.dsp - 마지막 정리 작업
priority=100
```

### 함수 공유 베스트 프랙티스

#### 1. 라이브러리 플러그인 패턴

공통 함수만 모아둔 전용 플러그인을 만들고 `type="script"`로 설정:

```dsp
name="공통 라이브러리"
type="script"
enabled=true
priority=1

// 함수들만 정의
func utilityFunc1() { ... }
func utilityFunc2() { ... }

// 실제로 실행할 내용은 없음
config.interpreter="bash"
config.script={
#!/bin/bash
exit 0
}
```

#### 2. 모듈별 라이브러리 분리

기능별로 라이브러리를 분리:

- `formatting_lib.dsp` - 메시지 포맷팅 함수
- `stats_lib.dsp` - 통계 계산 함수
- `emoji_lib.dsp` - 이모지 관련 함수
- `notification_lib.dsp` - 알림 관련 함수

#### 3. 문서화 주석 추가

```dsp
// 함수 설명을 주석으로 명확히 작성
// getRepoEmoji: 저장소 이름을 받아 적절한 이모지 반환
// @param repoName: 저장소 이름 (string)
// @return: 이모지 문자열 (string)
func getRepoEmoji(repoName) {
    if contains(repoName, "ubuntu") {
        return "🐧"
    }
    return "📦"
}
```

#### 4. 테스트 플러그인 작성

```dsp
name="함수 테스트"
type="script"
enabled=false  // 평소에는 비활성화
priority=99

config.interpreter="bash"
config.script={
#!/bin/bash
echo "=== 공유 함수 테스트 ==="
}

// 템플릿에서 함수 테스트
config.test_template={
formatDiscordMessage: {{call "formatDiscordMessage" "제목" "내용"}}
getRepoEmoji(ubuntu): {{call "getRepoEmoji" "ubuntu-mirror"}}
getSyncSummary: {{call "getSyncSummary" .SyncResults}}
}
```

### 함수 공유 제한사항

1. **순환 의존성 없음**: 플러그인 A가 B를 참조하고 B가 A를 참조하는 경우는 지원되지 않습니다
2. **로컬 우선**: 같은 이름의 함수가 로컬에 있으면 로컬 함수가 우선됩니다
3. **에러 전파**: 공유 함수에서 에러가 발생하면 호출한 플러그인도 실패합니다

### 실제 사용 사례

#### 사례 1: 멀티채널 알림 시스템

```dsp
// common_formatters.dsp (priority=1)
func formatSyncReport(syncResults) {
    summary = call("getSyncSummary", syncResults)
    failed = call("getFailedRepos", syncResults)
    return "📊 " + summary + "\n⚠️ 실패: " + failed
}

// discord_notifier.dsp (priority=20)
config.template={
{{call "formatSyncReport" .SyncResults}}
}

// slack_notifier.dsp (priority=21)
config.template={
{{call "formatSyncReport" .SyncResults}}
}

// telegram_notifier.dsp (priority=22)
config.template={
{{call "formatSyncReport" .SyncResults}}
}
```

#### 사례 2: 조건부 알림 라우팅

```dsp
// routing_lib.dsp (priority=1)
func shouldNotifyCritical(syncResults) {
    rate = call("getFailureRate", syncResults)
    return rate >= 50
}

func shouldNotifyWarning(syncResults) {
    rate = call("getFailureRate", syncResults)
    return rate > 0 && rate < 50
}

// critical_webhook.dsp (priority=20)
// 실패율 50% 이상일 때만 실행
config.url="https://hooks.slack.com/critical"
config.template={
{{if call "shouldNotifyCritical" .SyncResults}}
🚨 긴급: {{call "getFailureRate" .SyncResults}}% 실패
{{end}}
}

// warning_webhook.dsp (priority=21)
// 실패가 있지만 50% 미만일 때 실행
config.url="https://hooks.slack.com/warning"
config.template={
{{if call "shouldNotifyWarning" .SyncResults}}
⚠️ 주의: 일부 실패 발생
{{end}}
}
```

### 디버깅 팁

공유 함수 관련 문제 해결:

```bash
# 플러그인 로드 순서 확인
./dxync -list-plugins | grep priority

# 특정 플러그인 테스트
./dxync -test-plugin "유틸리티 함수"

# 함수 호출 에러 확인 (로그에서)
grep "함수를 찾을 수 없습니다" /root/dxync/logs/*.log
```

**일반적인 문제:**

| 문제 | 원인 | 해결방법 |
|------|------|----------|
| "함수를 찾을 수 없습니다" | 라이브러리 플러그인이 로드되지 않음 | priority 확인, enabled=true 확인 |
| 잘못된 함수 결과 | 이름 충돌 | 함수명에 접두사 추가 |
| 간헐적 에러 | 로드 순서 문제 | priority 값 조정 (라이브러리는 낮은 숫자) |

### 요약

- ✅ **자동 공유**: 모든 플러그인의 함수는 자동으로 전역에서 사용 가능
- ✅ **Go 스타일**: 별도 import 없이 같은 폴더의 함수를 바로 호출
- ✅ **우선순위**: priority로 로드 순서 제어 (라이브러리는 1-10 권장)
- ✅ **검색 순서**: 로컬 함수 → 전역 함수 → 내장 함수
- ✅ **재사용성**: 공통 함수를 한 곳에 작성하고 여러 플러그인에서 활용
- ⚠️ **이름 충돌**: 같은 이름의 함수는 나중에 로드된 것이 우선

## 플러그인 API 참조

### 사용 가능한 컨텍스트 변수

플러그인에서 사용할 수 있는 변수들입니다 (Go template 문법):

#### 기본 변수
- `{{.Version}}` - 플러그인 API 버전
- `{{.Timestamp}}` - 실행 시간 (Unix timestamp)

#### SyncResults (동기화 결과)
- `{{len .SyncResults}}` - 총 저장소 수
- `{{range .SyncResults}}...{{end}}` - 결과 순회
  - `{{.RepoName}}` - 저장소 이름
  - `{{.Success}}` - 성공 여부 (true/false)
  - `{{.LogFile}}` - 로그 파일 경로
  - `{{.InProgress}}` - 진행 중 여부

#### GlobalConfig (전역 설정)
- `{{.GlobalConfig.version}}` - DSYNC 버전
- `{{.GlobalConfig.webhook_url}}` - 웹훅 URL
- `{{.GlobalConfig.sync_processes}}` - 동시 프로세스 수
- `{{.GlobalConfig.status_path}}` - 상태 경로
- `{{.GlobalConfig.web_root}}` - 웹 루트

#### Stats (통계)
- `{{range $key, $value := .Stats}}...{{end}}` - 통계 순회
  - `{{$value.TotalBytes}}` - 전송 바이트
  - `{{$value.TotalTime}}` - 소요 시간
  - `{{$value.Stage1Time}}` - Stage 1 시간
  - `{{$value.Stage2Time}}` - Stage 2 시간

### 환경 변수

exec와 script 타입 플러그인에서 사용 가능:

- `DSYNC_CONTEXT` - 전체 컨텍스트 정보 (JSON 형식)
- `DSYNC_PLUGIN_DIR` - 플러그인 디렉토리 경로 (예: `/root/dxync/plugins`) **[v25.11.1 신규]**

**환경 변수 사용 예제:**

```bash
# Bash에서
PLUGIN_DIR="$DSYNC_PLUGIN_DIR"
CONFIG_FILE="$PLUGIN_DIR/myconfig/settings.conf"

echo "플러그인 디렉토리: $PLUGIN_DIR"
cat "$CONFIG_FILE"
```

```python
# Python에서
import os

plugin_dir = os.environ.get('DSYNC_PLUGIN_DIR', '/root/dxync/plugins')
config_file = os.path.join(plugin_dir, 'myconfig', 'settings.conf')

print(f"플러그인 디렉토리: {plugin_dir}")
with open(config_file, 'r') as f:
    print(f.read())
```

**사용 시나리오:**
- 외부 설정 파일 로드 (이메일 계정, API 키 등)
- 플러그인 간 데이터 파일 공유
- 플러그인별 로그 파일 생성
- 템플릿 파일 로드

### JSON 파싱 예제

```bash
# jq를 사용한 파싱
REPO_COUNT=$(echo "$DSYNC_CONTEXT" | jq '.sync_results | length')
FAILED=$(echo "$DSYNC_CONTEXT" | jq '.sync_results[] | select(.Success == false)')

# Python을 사용한 파싱
python3 << EOF
import json
import os

context = json.loads(os.environ['DSYNC_CONTEXT'])
for result in context['sync_results']:
    if not result['Success']:
        print(f"Failed: {result['RepoName']}")
EOF
```

## 유용한 팁

### 1. 플러그인 디버깅

```dsp
// 디버그 모드 플러그인
name="디버거"
version="1.0"
author="개발자"
description="컨텍스트 정보 출력"
type="script"
enabled=true
priority=1

config.script={
#!/bin/bash
echo "=== DSYNC Context Debug ==="
echo "$DSYNC_CONTEXT" | jq '.'
echo "==========================="
}
```

### 2. 조건부 실행

```dsp
// 실패 시에만 실행
name="실패 알림"
version="1.0"
author="관리자"
description="실패가 있을 때만 알림"
type="script"
enabled=true
priority=10

config.script={
#!/bin/bash
FAILED_COUNT=$(echo "$DSYNC_CONTEXT" | jq '.sync_results[] | select(.Success == false) | .RepoName' | wc -l)

if [ $FAILED_COUNT -gt 0 ]; then
    echo "⚠️ $FAILED_COUNT개 저장소 동기화 실패!"
    # 알림 로직
fi
}
```

### 3. 플러그인 체인

여러 플러그인을 순서대로 실행하려면 priority를 활용:

- priority=10: 로그 수집
- priority=20: 통계 분석
- priority=30: 리포트 생성
- priority=40: 알림 전송
- priority=50: 정리 작업

### 4. 에러 처리

```dsp
config.script={
#!/bin/bash
set -e  # 에러 시 즉시 중단
set -o pipefail  # 파이프라인 에러 감지

# 에러 트랩
trap 'echo "에러 발생: 라인 $LINENO"' ERR

# 안전한 JSON 파싱
if ! RESULT=$(echo "$DSYNC_CONTEXT" | jq '.sync_results' 2>/dev/null); then
    echo "JSON 파싱 실패"
    exit 1
fi
}
```

### 5. 성능 최적화

- 무거운 작업은 높은 priority 값으로 설정 (나중에 실행)
- timeout을 적절히 설정하여 무한 대기 방지 | 알아두세요. `config.timeout`값은 최대 3600초(1시간)까지만 사용할 수 있습니다. 
- 불필요한 플러그인은 `enabled=false`로 비활성화

## 문제 해결

### 플러그인이 로드되지 않을 때
1. 파일 확장자가 `.dsp`인지 확인
2. 파일 위치가 `~/dxync/plugins/`인지 확인
3. `name` 필드가 있는지 확인
4. 문법 오류가 없는지 확인

### 플러그인이 실행되지 않을 때
1. `enabled=true`인지 확인
2. `type`이 올바른지 확인 (webhook/email/exec/script)
3. 필수 설정이 있는지 확인 (예: webhook의 config.url, email의 config.smtp_host)

### 테스트 방법
```bash
# 플러그인 목록 확인
./dsync 또는 dsync -list-plugins

# 특정 플러그인 테스트
./dsync 또는 dsync -test-plugin "플러그인이름"
```

---

## 플러그인 작성 체크리스트

- [ ] 플러그인 이름과 버전 설정
- [ ] 적절한 타입 선택 (webhook/email/exec/script)
- [ ] enabled=true 설정
- [ ] priority 값 설정 (실행 순서)
- [ ] 필요한 config 설정 추가
- [ ] 에러 처리 로직 구현
- [ ] timeout 설정 확인 (email/exec/script 타입)
- [ ] 테스트 실행 (`-test-plugin`)
- [ ] 실제 동기화에서 테스트

---

## 변경 이력

### v25.11.1 (2025-10-15)
- ✨ **Python 스크립트 플러그인 지원**: `config.script={...}` 블록에서 Python indentation 보존
- ✨ **`DSYNC_PLUGIN_DIR` 환경 변수 추가**: 플러그인 디렉토리 경로를 스크립트에서 참조 가능
- ✨ **외부 설정 파일 패턴**: 민감한 정보를 플러그인 코드와 분리하여 관리하는 베스트 프랙티스 추가
- ✨ **옵션형 플러그인 (Option-Based Plugins)**: 동기화 없이 독립 실행 가능한 플러그인 타입 추가
  - CLI 옵션 동적 등록 (`option=true`, `option_flag="-xxx"`)
  - 깔끔한 출력 (디버깅 로그 없이 플러그인 출력만 표시)
  - 자동 help 통합 및 충돌 감지
- 🐛 **DSP 파서 개선**: config block 내 원본 라인 보존으로 템플릿 indentation 유지
- 📚 **문서 업데이트**: Python 스크립트 예제, 외부 설정 파일, 옵션형 플러그인 가이드 추가

---

*이 가이드는 DSYNC v25.11.1 기준으로 작성되었습니다.*