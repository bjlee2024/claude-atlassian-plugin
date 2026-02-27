# Claude Atlassian Plugin

Confluence CLI (`confluence-cli`) + Jira CLI (`jira-cli`) 연동 플러그인입니다.

## 플러그인 구조

```
claude-atlassian-plugin/
├── .claude-plugin/
│   ├── plugin.json          # 플러그인 메타데이터 (v2.0.0)
│   └── marketplace.json     # 마켓플레이스 등록 정보
├── skills/
│   ├── atlassian-init/SKILL.md  # 통합 초기 설정 스킬
│   ├── confluence/SKILL.md      # Confluence 작업 스킬
│   └── jira/SKILL.md            # Jira 작업 스킬
├── hooks/
│   └── hooks.json           # PreToolUse 훅 (변경 작업 차단)
├── docs/
│   └── confluence-cli-vs-atlassian-mcp.md  # CLI vs MCP 비교 분석
├── install.sh               # 설치 스크립트 (global/project)
├── README.md
└── CLAUDE.md
```

## 의존성

| 도구 | 용도 | 설치 |
|------|------|------|
| [confluence-cli](https://github.com/bjlee2024/confluence-cli) | Confluence REST API CLI | `npm install -g @bjlee2024/confluence-cli` |
| [jira-cli](https://github.com/ankitpokhrel/jira-cli) | Jira REST API CLI | `brew install ankitpokhrel/tap/jira-cli` |

## 사용 가능한 스킬

- `/atlassian-init` — Confluence CLI + Jira CLI 통합 설치 및 인증 설정
- `/confluence` — Confluence 페이지 읽기, 검색, 생성, 수정, 삭제
- `/jira` — Jira 이슈 조회, 생성, 수정, 삭제, 스프린트/에픽/보드 관리

## 크로스 툴 워크플로우

Confluence와 Jira를 연계하는 대표 시나리오입니다.

### Confluence 스페이스 멤버 → Jira 티켓 현황

1. `confluence spaces` → 스페이스 키 확인
2. `confluence search --cql "space=<KEY> AND type=page" --limit 50` → 페이지 목록에서 기여자 수집
3. 각 멤버에 대해:
   ```bash
   jira issue list -q "assignee='<email>' AND project=<PROJ> AND resolution=Unresolved" --plain --columns key,summary,status,priority
   ```

### Jira 이슈 → Confluence 문서화

1. `jira issue list -q "<JQL>" --plain --columns key,summary,status,assignee` → 이슈 목록 수집
2. 마크다운으로 정리 (로컬 파일)
3. 사용자 확인 후 `confluence create/update` → Confluence에 게시

### Jira 스프린트 리뷰 → Confluence 기록

1. `jira board list` → 보드 ID 확인
2. `jira sprint list --board <BOARD-ID> --state active` → 스프린트 ID 확인
3. `jira issue list -q "sprint=<SPRINT-ID>" --plain --columns key,summary,status,assignee` → 이슈 목록
4. 마크다운으로 스프린트 리뷰 문서 작성
5. 사용자 확인 후 `confluence create-child` → Confluence에 게시

---

## 헌법 (Constitution) - 절대 규칙

### Confluence 변경 작업 시 사용자 확인 필수

Confluence 데이터를 생성, 수정, 삭제하는 모든 명령은 **반드시 실행 전에 사용자 확인**을 받아야 합니다.

**확인이 필요한 명령어:**

| 분류 | 명령어 |
|------|--------|
| 생성 | `create`, `create-child`, `copy-tree`, `comment`, `property-set` |
| 수정 | `update`, `move`, `attachment-upload` |
| 삭제 | `delete`, `comment-delete`, `attachment-delete`, `property-delete` |

**확인 없이 실행 가능한 명령어 (읽기 전용):**

`read`, `info`, `find`, `search`, `spaces`, `children`, `comments`, `attachments`, `property-list`, `property-get`, `export`, `stats`

### Jira 변경 작업 시 사용자 확인 필수

Jira 데이터를 생성, 수정, 삭제하는 모든 명령은 **반드시 실행 전에 사용자 확인**을 받아야 합니다.

**확인이 필요한 명령어:**

| 분류 | 명령어 |
|------|--------|
| 생성 | `issue create`, `issue clone`, `epic create` |
| 수정 | `issue edit`, `issue assign`, `issue move`, `issue link` |
| 삭제 | `issue delete` |
| 댓글 | `issue comment add` |
| 스프린트/에픽 변경 | `sprint add`, `epic add` |

**확인 없이 실행 가능한 명령어 (읽기 전용):**

`issue list`, `issue view`, `sprint list`, `epic list`, `board list`, `project list`, `open`, `me`

### 확인 프로토콜

1. 실행할 정확한 명령어를 보여줍니다
2. 해당 명령이 무엇을 하는지 설명합니다
3. 영향 범위 (대상 페이지/스페이스 또는 이슈/프로젝트)를 명시합니다
4. 위험도와 되돌리기 가능 여부를 안내합니다
5. AskUserQuestion 으로 명시적 승인을 받습니다
6. 승인 후에만 명령을 실행합니다

### NEVER

- 사용자 확인 없이 생성/수정/삭제 명령 실행 금지
- 이전 승인을 근거로 새로운 변경 작업 자동 승인 금지
- `--yes` 플래그로 확인 프롬프트 우회 금지
- 여러 변경 작업을 한번의 승인으로 일괄 처리 금지
