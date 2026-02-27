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
/confluence-init    # Confluence CLI 설치 및 인증 설정
/jira-init          # Jira CLI 설치 및 인증 설정
```

> **참고**: Confluence 와 Jira 는 동일한 Atlassian API 토큰을 사용할 수 있습니다.
> https://id.atlassian.com/manage/api-tokens 에서 토큰을 생성하세요.

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
