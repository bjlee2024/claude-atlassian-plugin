---
name: jira
description: Jira 이슈 조회, 생성, 수정, 삭제, 스프린트/에픽/보드 관리 등 모든 작업을 수행합니다. jira-cli 를 사용합니다.
user-invocable: true
when_to_use: "jira, 지라, jira 이슈, jira issue, jira 검색, jira search, jira 생성, jira 수정, jira 삭제, jira 스프린트, jira 에픽, jira 보드"
---

# Jira 작업 스킬

`jira-cli` (https://github.com/ankitpokhrel/jira-cli) 를 사용하여 Jira 작업을 수행합니다.

---

## 헌법 (CONSTITUTION) - 절대 규칙

<constitution>
### 변경 작업 시 사용자 확인 필수

아래 명령어는 **반드시 실행 전에 AskUserQuestion 도구로 사용자 확인을 받아야 합니다.**
확인 없이 실행하는 것은 금지입니다.

#### 생성 (CREATE)
- `jira issue create` — 이슈 생성
- `jira issue clone` — 이슈 복제
- `jira epic create` — 에픽 생성

#### 수정 (UPDATE)
- `jira issue edit` — 이슈 수정
- `jira issue assign` — 이슈 담당자 변경
- `jira issue move` — 이슈 상태 전환 (트랜지션)
- `jira issue link` — 이슈 링크

#### 삭제 (DELETE)
- `jira issue delete` — 이슈 삭제

#### 댓글 (COMMENT)
- `jira issue comment add` — 댓글 작성

#### 스프린트/에픽 변경 (MODIFY)
- `jira sprint add` — 스프린트에 이슈 추가
- `jira epic add` — 에픽에 이슈 추가

### 확인 프로토콜

1. 실행할 **정확한 명령어**를 보여줍니다.
2. 해당 명령이 **무엇을 하는지** 한국어로 설명합니다.
3. **영향 범위** (대상 이슈, 프로젝트 등)를 명시합니다.
4. **위험도** (데이터 손실, 되돌리기 가능 여부)를 안내합니다.
5. AskUserQuestion 으로 **명시적 승인**을 받습니다.
6. 승인 후에만 명령을 실행합니다.

### 확인 메시지 형식

```
다음 Jira 작업을 실행하려고 합니다:

  명령어: jira <command> <args>
  동작: <설명>
  대상: <이슈 키 / 프로젝트>
  위험도: <낮음|보통|높음>
  되돌리기: <가능|불가능|부분적>

실행하시겠습니까?
```

### 읽기 전용 명령어 (확인 불필요)

다음 명령어는 데이터를 변경하지 않으므로 확인 없이 실행할 수 있습니다:

- `jira issue list` — 이슈 목록
- `jira issue view` — 이슈 상세 조회
- `jira sprint list` — 스프린트 목록
- `jira epic list` — 에픽 목록
- `jira board list` — 보드 목록
- `jira project list` — 프로젝트 목록
- `jira open` — 브라우저에서 열기
- `jira me` — 현재 사용자 정보
</constitution>

---

## 명령어 선택 가이드 (Decision Tree)

> **원칙**: 목록 조회는 전용 명령, 조건부 검색은 JQL(`-q`)을 사용합니다. 데이터 연계 시 `--plain --columns`를 반드시 사용합니다.

| 목적 | 올바른 접근 | 잘못된 접근 |
|------|------------|------------|
| 프로젝트 목록 | `jira project list` | JQL |
| 보드 ID 확인 | `jira board list` | JQL |
| 멤버별 이슈 현황 | `-q "assignee='email' AND project=PROJ"` | `issue list -pPROJ` + grep |
| 스프린트 이슈 | `-q "sprint in openSprints() AND project=PROJ"` | 보드에서 수동 탐색 |
| 상태별 이슈 | `-q "status='In Progress' AND project=PROJ"` | `issue list` + grep |
| 최근 생성 이슈 | `-q "created >= -7d AND project=PROJ ORDER BY created DESC"` | `issue list` 기본 정렬 |
| 이슈 상세 확인 | `jira issue view <KEY>` | JQL (목록용) |
| 현재 사용자 | `jira me` | JQL |

### 판단 흐름도

```
사용자 요청 분석
├── 프로젝트/보드/에픽 목록? → 전용 list 명령
├── 조건부 이슈 검색?
│   ├── 담당자/상태/라벨/스프린트/날짜 조건? → JQL (-q)
│   └── 특정 이슈 키 알고 있음? → `issue view <KEY>`
├── 다른 도구와 연계? → `--plain --columns` 필수
└── 사용자 정보? → `jira me`
```

> **출력 지침**: 다른 명령과 연계하거나 데이터를 정리할 때는
> 반드시 `--plain --columns <필요한컬럼>` 옵션을 사용합니다.
> 기본 출력은 인터랙티브 테이블 포맷으로 파싱이 어렵습니다.

---

## 출력 형식 & 파싱

| 명령 | 출력 형식 | 파싱 참고 |
|------|----------|----------|
| `jira issue list` | 인터랙티브 테이블 (기본) | 파싱 어려움, `--plain` 추가 필수 |
| `jira issue list --plain` | 탭 구분 텍스트 테이블 | `--columns`로 필요한 컬럼만 출력 |
| `jira issue list --plain --columns key,summary,status,assignee` | 지정 컬럼만 탭 구분 | 파이프라인 처리 최적 |
| `jira issue view <KEY>` | 상세 정보 (제목, 설명, 댓글 등) | `--comments N`으로 댓글 수 제한 |
| `jira board list` | 보드 ID, 이름, 타입 | 보드 ID 추출용 |
| `jira sprint list --board <ID>` | 스프린트 ID, 이름, 상태 | `--state active`로 필터 |
| `jira me` | 현재 사용자 이메일/이름 | 이메일로 JQL assignee 매핑 |

---

## 에러 대응

| 에러 | 원인 | 대응 |
|------|------|------|
| `401 Unauthorized` | 토큰 만료 또는 JIRA_API_TOKEN 미설정 | `/atlassian-init` 안내, `export JIRA_API_TOKEN` 확인 |
| `404 Not Found` | 잘못된 이슈 키 또는 삭제된 이슈 | JQL로 재검색 |
| `400 Bad Request` | JQL 문법 오류 | JQL 구문 확인 (따옴표, 필드명, 연산자) |
| `rate limit` / `429` | API 호출 초과 | 간격 두고 재시도 (5초 대기) |
| JQL 검색 결과 0건 | 조건이 너무 좁거나 필드명 오류 | 조건 완화, `project` 제거 후 재검색 |
| `--plain` 없이 파이프 사용 | 인터랙티브 테이블 깨짐 | `--plain --columns` 추가 |
| 프로젝트 키 모름 | 설정 누락 | `jira project list`로 확인 |

---

## 사전 조건 확인

모든 작업 전에 `jira-cli` 설치 여부를 확인합니다:

```bash
which jira 2>/dev/null || command -v jira 2>/dev/null
```

미설치인 경우 `/atlassian-init` 스킬을 안내합니다.

---

## 명령어 레퍼런스

### 이슈 조회/검색

```bash
# 프로젝트 이슈 목록 (기본: 현재 프로젝트)
jira issue list

# 특정 프로젝트 이슈
jira issue list -p<PROJECT>

# JQL 검색
jira issue list -q "project = PROJ AND status = 'In Progress'"
jira issue list -q "assignee = currentUser() AND resolution = Unresolved"
jira issue list -q "sprint in openSprints() AND project = PROJ"
jira issue list -q "created >= -7d AND project = PROJ"

# 이슈 상세 조회
jira issue view <ISSUE-KEY>
jira issue view <ISSUE-KEY> --comments 10

# 출력 형식 지정
jira issue list --plain
jira issue list -q "<JQL>" --plain --columns key,summary,status,assignee
```

### 이슈 생성 (확인 필수)

```bash
# 기본 생성
jira issue create -tTask -s"이슈 제목" -b"이슈 설명" -P<PROJECT>

# 타입별 생성
jira issue create -tBug -s"버그 제목" -P<PROJECT> -lhigh
jira issue create -tStory -s"스토리 제목" -P<PROJECT>

# 커스텀 필드 포함
jira issue create -tTask -s"제목" -P<PROJECT> --custom "customfield_10001=value"

# 이슈 복제
jira issue clone <ISSUE-KEY>
```

### 이슈 수정 (확인 필수)

```bash
# 제목 변경
jira issue edit <ISSUE-KEY> -s"새로운 제목"

# 설명 변경
jira issue edit <ISSUE-KEY> -b"새로운 설명"

# 담당자 변경
jira issue assign <ISSUE-KEY> "<assignee-email>"
jira issue assign <ISSUE-KEY> ""  # 담당자 해제

# 상태 전환 (트랜지션)
jira issue move <ISSUE-KEY> "In Progress"
jira issue move <ISSUE-KEY> "Done"

# 이슈 링크
jira issue link <ISSUE-KEY1> <ISSUE-KEY2> "Blocks"
jira issue link <ISSUE-KEY1> <ISSUE-KEY2> "is blocked by"
```

### 이슈 삭제 (확인 필수)

```bash
jira issue delete <ISSUE-KEY>
```

### 댓글 (확인 필수)

```bash
# 댓글 추가
jira issue comment add <ISSUE-KEY> -b"댓글 내용"
```

### 스프린트

```bash
# 스프린트 목록 (읽기)
jira sprint list --board <BOARD-ID>
jira sprint list --board <BOARD-ID> --state active

# 스프린트에 이슈 추가 (확인 필수)
jira sprint add <SPRINT-ID> <ISSUE-KEY>
```

### 에픽

```bash
# 에픽 목록 (읽기)
jira epic list
jira epic list -p<PROJECT>

# 에픽에 이슈 추가 (확인 필수)
jira epic add <EPIC-KEY> <ISSUE-KEY>

# 에픽 생성 (확인 필수)
jira epic create -n"에픽 이름" -s"에픽 요약" -P<PROJECT>
```

### 보드/프로젝트 (읽기)

```bash
# 보드 목록
jira board list
jira board list -q"<board-name>"

# 프로젝트 목록
jira project list

# 현재 사용자 정보
jira me
```

### 브라우저에서 열기

```bash
jira open <ISSUE-KEY>
```

---

## 워크플로우 예시

### 이슈 조회 후 상태 변경

1. `jira issue list -q "project = PROJ AND status = 'To Do'"` 로 이슈 확인
2. `jira issue view PROJ-123` 로 상세 확인
3. 사용자 확인 후 `jira issue move PROJ-123 "In Progress"`

### 버그 리포트 생성

1. `jira project list` 로 프로젝트 키 확인
2. 이슈 내용 정리
3. 사용자 확인 후 `jira issue create -tBug -s"버그 제목" -b"재현 단계..." -PPROJ -lhigh`

### 스프린트 현황 확인

1. `jira board list` 로 보드 ID 확인
2. `jira sprint list --board <BOARD-ID> --state active` 로 현재 스프린트 확인
3. `jira issue list -q "sprint in openSprints() AND project = PROJ"` 로 스프린트 이슈 목록 조회

### JQL 활용 검색

```bash
# 나에게 할당된 미해결 이슈
jira issue list -q "assignee = currentUser() AND resolution = Unresolved"

# 최근 7일 생성된 이슈
jira issue list -q "created >= -7d AND project = PROJ ORDER BY created DESC"

# 특정 라벨의 이슈
jira issue list -q "labels = 'backend' AND project = PROJ"

# 우선순위별 이슈
jira issue list -q "priority = High AND status != Done AND project = PROJ"
```
