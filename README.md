# Claude Atlassian Plugin

Claude Code 에서 Confluence 와 Jira 를 CLI 로 연동하는 플러그인입니다.

## 왜 MCP 가 아닌 CLI 을 사용해야 하는가?

[MCP 와 CLI 비교](docs/confluence-cli-vs-atlassian-mcp.md)

## 의존성

- [@bjlee2024/confluence-cli](https://github.com/bjlee2024/confluence-cli) (`npm install -g @bjlee2024/confluence-cli`)
- [ankitpokhrel/jira-cli](https://github.com/ankitpokhrel/jira-cli) (`brew install ankitpokhrel/tap/jira-cli`)

## 설치

```bash
# 1. CLI 도구 설치

# Confluence CLI
npm install -g @bjlee2024/confluence-cli

# Jira CLI (택 1)
brew install ankitpokhrel/tap/jira-cli          # macOS / Linux (Homebrew)
go install github.com/ankitpokhrel/jira-cli/cmd/jira@latest  # Go 1.21+

# 2. 플러그인 설치 (아래 중 택 1)

# 방법 A: 글로벌 설치 (모든 프로젝트에서 사용)
./install.sh global

# 방법 B: 프로젝트별 설치
./install.sh project
```

## 사용법

### 초기 설정

```
/atlassian-init    # Confluence + Jira CLI 통합 설치 및 인증 설정
```

동일한 Atlassian API 토큰으로 Confluence 와 Jira 를 한 번에 설정합니다.
토큰은 https://id.atlassian.com/manage/api-tokens 에서 생성하세요.

### Confluence 작업

```
/confluence
```

페이지 읽기, 검색, 생성, 수정, 삭제 등 모든 Confluence 작업을 수행합니다.

### Jira 작업

```
/jira
```

이슈 조회, 생성, 수정, 삭제, 스프린트/에픽/보드 관리 등 모든 Jira 작업을 수행합니다.

## 헌법 (Constitution)

이 플러그인은 **생성/수정/삭제 작업 시 반드시 사용자 확인을 요구**합니다.

### Confluence

- `create`, `create-child`, `copy-tree` (생성)
- `update`, `move` (수정)
- `delete`, `comment-delete`, `attachment-delete`, `property-delete` (삭제)
- `comment`, `property-set`, `attachment-upload` (기타 변경)

### Jira

- `issue create`, `issue clone`, `epic create` (생성)
- `issue edit`, `issue assign`, `issue move`, `issue link` (수정)
- `issue delete` (삭제)
- `issue comment add` (댓글)
- `sprint add`, `epic add` (스프린트/에픽 변경)

### 읽기 전용 (확인 없이 실행)

**Confluence**: `read`, `info`, `find`, `search`, `spaces`, `children`, `comments`, `attachments`, `property-list`, `property-get`, `export`, `stats`

**Jira**: `issue list`, `issue view`, `sprint list`, `epic list`, `board list`, `project list`, `open`, `me`


## CLI vs Atlassian MCP 비교

자세한 비교 분석은 [docs/confluence-cli-vs-atlassian-mcp.md](docs/confluence-cli-vs-atlassian-mcp.md) 참고.

| 항목 | CLI (confluence-cli + jira-cli) | Atlassian MCP |
|------|--------------------------------|---------------|
| 속도 | 빠름 (직접 REST API) | 느림 (MCP 오버헤드) |
| 토큰 효율 | 높음 (간결한 텍스트) | 낮음 (JSON 메타데이터) |
| 기능 범위 | 넓음 (23+ 명령어) | 제한적 (9개 도구) |
| IP 제한 | 없음 (로컬 실행) | 차단 가능 (클라우드 서버 IP) |
| 인증 | API Token (수동 설정) | OAuth (자동 갱신) |

### MCP IP 차단 문제

사내 Atlassian 에 IP allowlist 가 설정된 경우, Atlassian MCP (claude.ai) 는 Anthropic 클라우드 서버 IP 가 차단되어 사용 불가.
로컬 CLI 는 사용자 PC 에서 직접 호출하므로 영향 없음. 해결 방법:

1. `admin.atlassian.com` > Security > IP allowlisting 에서 Anthropic IP 추가 (관리자 권한 필요)
2. IP allowlist 비활성화 (보안팀 승인 필요)
3. **로컬 CLI 계속 사용 (가장 현실적, 추가 설정 불필요)**
