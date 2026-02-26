# Claude Confluence Plugin

Claude Code 에서 Confluence 를 CLI 로 연동하는 플러그인입니다.

## 왜 MCP 가 아닌 CLI 을 사용해야 하는가?

[MCP 와 CLI 비교](docs/confluence-cli-vs-atlassian-mcp.md)

## 의존성

- [@bjlee2024/confluence-cli](https://github.com/bjlee2024/confluence-cli) (`npm install -g @bjlee2024/confluence-cli`)

## 설치

```bash
# 1. confluence-cli 설치
npm install -g @bjlee2024/confluence-cli

# 2. 플러그인 스킬 등록 (아래 중 택 1)

# 방법 A: 글로벌 설치 (모든 프로젝트에서 사용)
cp skills/*.md ~/.claude/skills/

# 방법 B: 프로젝트별 설치
cp skills/*.md .claude/skills/
```

## 사용법

### 초기 설정

```
/confluence-init
```

`confluence-cli` 미설치 시 자동 설치를 안내하고, `confluence init` 으로 인증 설정을 진행합니다.

### Confluence 작업

```
/confluence
```

페이지 읽기, 검색, 생성, 수정, 삭제 등 모든 Confluence 작업을 수행합니다.

## 헌법 (Constitution)

이 플러그인은 **생성/수정/삭제 작업 시 반드시 사용자 확인을 요구**합니다:

- `create`, `create-child`, `copy-tree` (생성)
- `update`, `move` (수정)
- `delete`, `comment-delete`, `attachment-delete`, `property-delete` (삭제)
- `attachment-upload` (첨부파일 업로드)

읽기 전용 명령어는 확인 없이 실행됩니다:

- `read`, `info`, `find`, `search`, `spaces`, `children`
- `comments`, `attachments`, `property-list`, `property-get`
- `export`, `stats`


## 간헐적 MCP 접속 불가 원인
# Atlassian MCP IP 차단 문제 분석

### 현상

- `confluence-cli` (로컬 CLI): 정상 동작
- Atlassian MCP (claude.ai 내장): IP 차단 오류 발생

### 원인 분석

#### 접속 경로 차이

| 방식 | 접속 IP | 결과 |
|------|---------|------|
| `confluence-cli` | 본인 PC / 회사 네트워크 IP | 허용됨 |
| Atlassian MCP (claude.ai) | Anthropic 클라우드 서버 IP | 차단됨 |

#### 상세 설명

Atlassian MCP는 **Anthropic의 클라우드 서버**에서 Confluence REST API를 호출한다.
회사 Atlassian 조직에 **IP allowlist**가 설정되어 있으면, 허용되지 않은 외부 IP는 API 접근이 차단된다.

로컬 CLI는 사용자의 PC에서 직접 API를 호출하므로, 회사 네트워크 IP가 이미 허용 목록에 포함되어 정상 동작한다.

### 해결 방법

#### 방법 1: Atlassian IP Allowlist에 Anthropic IP 추가 (권장)

**경로:** `admin.atlassian.com` > 조직 선택 > **Security** > **IP allowlisting**

- Anthropic MCP 서버의 IP 대역을 허용 목록에 추가
- Anthropic이 공식적으로 MCP 서버 IP 대역을 공개하는지 확인 필요
- 관리자 권한 필요

**장점:** MCP를 정상적으로 활용 가능
**단점:** 외부 IP 허용에 대한 보안 승인 필요

#### 방법 2: IP Allowlist 비활성화

- 조직의 IP allowlist 정책을 해제하거나 완화
- 보안팀 승인 필수

**장점:** 모든 외부 서비스 연동 가능
**단점:** 보안 수준 저하, 조직 보안 정책 위반 가능성

#### 방법 3: 로컬 CLI 계속 사용 (가장 현실적)

- 현재 `confluence-cli`가 정상 동작하므로 그대로 활용
- Claude Code 내에서 CLI 기반 스킬(`/confluence`)로 Confluence 연동
- IP 제한 변경 없이 기존 환경 유지

**장점:** 추가 설정 불필요, 보안 영향 없음
**단점:** claude.ai 웹에서 직접 MCP 연동 불가

### 비교 요약

| 방법 | 난이도 | 보안 영향 | 관리자 필요 |
|------|--------|-----------|-------------|
| Anthropic IP allowlist 추가 | 중 | 중 | O |
| IP allowlist 해제 | 중 | 높음 | O |
| **로컬 CLI 사용 (현재)** | **없음** | **없음** | **X** |

### 결론

1. 회사 Atlassian 관리자에게 **IP allowlist 설정 확인 및 Anthropic IP 허용 요청**
2. 승인이 어려운 경우, 현재 방식(`confluence-cli` 기반)을 유지하는 것이 가장 현실적
