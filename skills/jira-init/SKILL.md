---
name: jira-init
description: Jira CLI 초기 설정. jira-cli 설치 확인 및 인증 설정을 진행합니다.
user-invocable: true
when_to_use: "jira init, jira 설정, jira 세팅, jira setup, 지라 설정, 지라 초기화"
---

# Jira CLI 초기 설정

이 스킬은 `jira-cli` (ankitpokhrel/jira-cli) 도구의 설치 및 인증 설정을 수행합니다.

## 절차

### 1단계: jira-cli 설치 확인

```bash
which jira 2>/dev/null || command -v jira 2>/dev/null
```

위 명령어로 `jira` CLI가 설치되어 있는지 확인합니다.

**미설치인 경우:**

사용자에게 다음 안내를 제공합니다:

```
jira-cli 가 설치되어 있지 않습니다. 설치하시겠습니까?

설치 명령어 (택 1):
  macOS:  brew install ankitpokhrel/tap/jira-cli
  Linux:  brew install ankitpokhrel/tap/jira-cli
          또는 go install github.com/ankitpokhrel/jira-cli/cmd/jira@latest (Go 1.21+ 필요)

※ Homebrew 또는 Go 가 필요합니다.
```

AskUserQuestion 도구로 설치 동의 및 설치 방법 선택을 받은 후 실행합니다.

### 2단계: 기존 설정 확인

```bash
jira me 2>&1
```

정상 응답이 오면 이미 설정이 완료된 상태입니다. 사용자에게 현재 설정 상태를 알려줍니다.

### 3단계: 인증 설정

기존 설정이 없거나 재설정이 필요한 경우, AskUserQuestion 도구로 다음 정보를 수집합니다:

1. **Jira 서버 URL** (예: `https://company.atlassian.net`)
2. **이메일** (Atlassian 계정 이메일)
3. **API 토큰** (Atlassian API Token — Confluence 와 동일한 토큰 사용 가능)

> **참고**: API 토큰은 https://id.atlassian.com/manage/api-tokens 에서 생성합니다.
> Confluence에서 사용 중인 동일한 Atlassian API 토큰을 Jira에서도 사용할 수 있습니다.

수집한 정보로 config 파일을 자동 생성합니다:

```bash
mkdir -p ~/.config/.jira
cat > ~/.config/.jira/.config.yml << 'JIRA_CONFIG'
installation: cloud
server: <server-url>
login: <email>
board:
  type: ""
  name: ""
project:
  key: ""
  type: ""
epic:
  name: customfield_10011
  link: customfield_10014
JIRA_CONFIG
```

그리고 API 토큰을 환경변수로 설정하도록 안내합니다:

```bash
# shell profile (~/.bashrc, ~/.zshrc, ~/.config/fish/config.fish 등)에 추가
export JIRA_API_TOKEN="<api-token>"
```

현재 세션에서 바로 사용하려면:

```bash
export JIRA_API_TOKEN="<api-token>"
```

### 4단계: 설정 검증

```bash
jira project list
```

프로젝트 목록이 정상적으로 반환되면 설정 완료를 알립니다.

## 환경변수 대안

CLI 설정 대신 환경변수를 사용할 수도 있음을 안내합니다:

```bash
export JIRA_API_TOKEN="your-api-token"
```

config 파일 위치: `~/.config/.jira/.config.yml`

## 주의사항

- API 토큰은 절대 코드에 하드코딩하지 않습니다.
- `.env` 파일에 저장하는 경우 `.gitignore` 에 반드시 추가합니다.
- `jira init` 은 interactive 모드만 지원하므로, 위 절차에서는 config 파일 직접 생성 방식을 사용합니다.
- Confluence 와 동일한 Atlassian API 토큰을 사용할 수 있습니다.
