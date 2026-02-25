---
name: confluence
description: Confluence 페이지 읽기, 검색, 생성, 수정, 삭제 등 모든 작업을 수행합니다. confluence-cli 를 사용합니다.
triggers:
  - "confluence"
  - "컨플루언스"
  - "confluence 페이지"
  - "confluence page"
  - "confluence 검색"
  - "confluence search"
  - "confluence 생성"
  - "confluence 수정"
  - "confluence 삭제"
---

# Confluence 작업 스킬

`confluence-cli` (https://github.com/pchuri/confluence-cli) 를 사용하여 Confluence 작업을 수행합니다.

---

## 헌법 (CONSTITUTION) - 절대 규칙

<constitution>
### 변경 작업 시 사용자 확인 필수

아래 명령어는 **반드시 실행 전에 AskUserQuestion 도구로 사용자 확인을 받아야 합니다.**
확인 없이 실행하는 것은 금지입니다.

#### 생성 (CREATE)
- `confluence create` — 새 페이지 생성
- `confluence create-child` — 하위 페이지 생성
- `confluence copy-tree` — 페이지 트리 복사
- `confluence comment` — 댓글 작성
- `confluence property-set` — 속성 설정

#### 수정 (UPDATE)
- `confluence update` — 페이지 수정
- `confluence move` — 페이지 이동
- `confluence attachment-upload` — 첨부파일 업로드

#### 삭제 (DELETE)
- `confluence delete` — 페이지 삭제
- `confluence comment-delete` — 댓글 삭제
- `confluence attachment-delete` — 첨부파일 삭제
- `confluence property-delete` — 속성 삭제

### 확인 프로토콜

1. 실행할 **정확한 명령어**를 보여줍니다.
2. 해당 명령이 **무엇을 하는지** 한국어로 설명합니다.
3. **영향 범위** (대상 페이지, 스페이스 등)를 명시합니다.
4. **위험도** (데이터 손실, 되돌리기 가능 여부)를 안내합니다.
5. AskUserQuestion 으로 **명시적 승인**을 받습니다.
6. 승인 후에만 명령을 실행합니다.

### 확인 메시지 형식

```
다음 Confluence 작업을 실행하려고 합니다:

  명령어: confluence <command> <args>
  동작: <설명>
  대상: <페이지 ID / 제목 / 스페이스>
  위험도: <낮음|보통|높음>
  되돌리기: <가능|불가능|부분적>

실행하시겠습니까?
```

### 읽기 전용 명령어 (확인 불필요)

다음 명령어는 데이터를 변경하지 않으므로 확인 없이 실행할 수 있습니다:

- `confluence read` — 페이지 내용 읽기
- `confluence info` — 페이지 메타데이터
- `confluence find` — 제목으로 검색
- `confluence search` — 전체 검색
- `confluence spaces` — 스페이스 목록
- `confluence children` — 하위 페이지 목록
- `confluence comments` — 댓글 목록 조회
- `confluence attachments` — 첨부파일 목록
- `confluence property-list` — 속성 목록
- `confluence property-get` — 속성 조회
- `confluence export` — 로컬 내보내기
- `confluence stats` — 통계
- `confluence edit --output` — 로컬 파일로 내보내기
</constitution>

---

## 사전 조건 확인

모든 작업 전에 `confluence-cli` 설치 여부를 확인합니다:

```bash
which confluence 2>/dev/null || command -v confluence 2>/dev/null
```

미설치인 경우 `/confluence-init` 스킬을 안내합니다.

---

## 명령어 레퍼런스

### 페이지 읽기

```bash
# 페이지 ID 로 읽기
confluence read <pageId>
confluence read <pageId> --format markdown

# URL 로 읽기
confluence read "https://domain.atlassian.net/wiki/viewpage.action?pageId=<pageId>"

# 메타데이터 조회
confluence info <pageId>
```

### 검색

```bash
# 제목으로 검색
confluence find "<title>"
confluence find "<title>" --space <SPACEKEY>

# 텍스트 검색 (키워드 기반)
confluence search "<query>"
confluence search "<query>" --limit 10

# CQL 검색 (반드시 --cql 플래그 필요)
confluence search --cql "space=<SPACEKEY>" --limit 10
confluence search --cql "space=<SPACEKEY> AND type=page" --limit 20
confluence search --cql "title=\"<exact title>\"" --limit 5
confluence search --cql "space=<SPACEKEY> AND title~\"<keyword>\"" --limit 10
confluence search --cql "label=\"<label>\" AND type=page" --limit 10

# 스페이스 목록
confluence spaces

# 하위 페이지
confluence children <pageId>
confluence children <pageId> --recursive --format tree
confluence children <pageId> --recursive --max-depth 3 --show-id --show-url
```

> **주의**: `--cql` 플래그 없이 CQL 문법을 사용하면 텍스트 검색으로 처리되어 의도한 결과가 나오지 않습니다.
> `title=`은 페이지 제목만 검색합니다. 스페이스 이름은 `space=<KEY>`로 검색하세요.

### 페이지 생성 (확인 필수)

```bash
# 새 페이지
confluence create "<title>" <SPACEKEY> --content "<content>"
confluence create "<title>" <SPACEKEY> --file ./content.md --format markdown

# 하위 페이지
confluence create-child "<title>" <parentPageId> --content "<content>"
confluence create-child "<title>" <parentPageId> --file ./content.md --format markdown

# 페이지 트리 복사
confluence copy-tree <sourcePageId> <targetParentId> "<new title>"
confluence copy-tree <sourcePageId> <targetParentId> --dry-run  # 미리보기
```

### 페이지 수정 (확인 필수)

```bash
# 제목 변경
confluence update <pageId> --title "<new title>"

# 내용 변경
confluence update <pageId> --content "<new content>"
confluence update <pageId> --file ./updated.md --format markdown

# 제목 + 내용 변경
confluence update <pageId> --title "<new title>" --content "<new content>"

# 페이지 이동
confluence move <pageId> <newParentPageId>
```

### 페이지 삭제 (확인 필수)

```bash
confluence delete <pageId>
```

### 댓글

```bash
# 댓글 목록 (확인 불필요)
confluence comments <pageId>
confluence comments <pageId> --location inline --format markdown

# 댓글 작성 (확인 필수)
confluence comment <pageId> --content "<comment>"
confluence comment <pageId> --location inline --content "<comment>" --inline-selection "<text>"

# 댓글 삭제 (확인 필수)
confluence comment-delete <commentId>
```

### 첨부파일

```bash
# 목록 조회 (확인 불필요)
confluence attachments <pageId>
confluence attachments <pageId> --pattern "*.png" --download --dest ./downloads

# 업로드 (확인 필수)
confluence attachment-upload <pageId> --file ./report.pdf
confluence attachment-upload <pageId> --file ./a.pdf --file ./b.png --comment "v2"

# 삭제 (확인 필수)
confluence attachment-delete <pageId> <attachmentId>
```

### 속성 (메타데이터)

```bash
# 조회 (확인 불필요)
confluence property-list <pageId>
confluence property-get <pageId> <key>

# 설정 (확인 필수)
confluence property-set <pageId> <key> --value '<json>'

# 삭제 (확인 필수)
confluence property-delete <pageId> <key>
```

### 내보내기

```bash
confluence export <pageId> --dest ./exports
confluence export <pageId> --format html --file content.html
```

---

## 워크플로우 예시

### 페이지 내용 확인 후 수정

1. `confluence read <pageId> --format markdown` 로 현재 내용 확인
2. 로컬에서 내용 편집
3. 사용자 확인 후 `confluence update <pageId> --file ./edited.md --format markdown`

### 검색 후 정보 수집

1. `confluence search "<keyword>"` 로 관련 페이지 검색
2. `confluence read <pageId> --format markdown` 로 각 페이지 확인
3. 필요시 `confluence children <pageId> --recursive --format tree` 로 구조 파악

### 새 문서 작성

1. `confluence spaces` 로 대상 스페이스 확인
2. 로컬에서 마크다운 문서 작성
3. 사용자 확인 후 `confluence create "<title>" <SPACEKEY> --file ./doc.md --format markdown`
