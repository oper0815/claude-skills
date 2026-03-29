---
name: github-explainer
description: GitHub 링크(URL)를 받으면 해당 리소스를 한국어로 설명해주는 스킬. 사용자가 github.com으로 시작하는 링크를 공유하거나, "이 GitHub 링크 설명해줘", "이 레포 뭐야?", "이 PR 뭔 내용이야?", "이 이슈 봐줘" 같은 말을 하면 반드시 이 스킬을 사용해야 한다. GitHub 링크가 포함된 메시지라면 항상 이 스킬을 먼저 실행할 것.
---

# GitHub 링크 설명 스킬

사용자가 GitHub URL을 공유하면, 해당 리소스가 무엇인지 한국어로 명확하게 설명한다.

## URL 타입 식별

GitHub URL 패턴을 보고 타입을 결정한다:

| URL 패턴 | 타입 |
|----------|------|
| `github.com/{owner}/{repo}` | 레포지토리 |
| `github.com/{owner}/{repo}/pull/{number}` | Pull Request |
| `github.com/{owner}/{repo}/issues/{number}` | Issue |
| `github.com/{owner}/{repo}/blob/{branch}/{path}` | 파일 |
| `github.com/{owner}/{repo}/commit/{sha}` | 커밋 |
| `github.com/{owner}/{repo}/releases/tag/{tag}` | 릴리즈 |
| `github.com/{owner}/{repo}/tree/{branch}` | 레포지토리 (특정 브랜치) |

## GitHub API 호출 방법

WebFetch 도구로 아래 API를 호출한다. 공개 레포는 인증 없이 사용 가능.

### 레포지토리 (3회 호출)

```
GET https://api.github.com/repos/{owner}/{repo}
GET https://api.github.com/repos/{owner}/{repo}/readme
GET https://api.github.com/repos/{owner}/{repo}/contents/
```

- README는 Base64로 인코딩되어 반환된다. `content` 필드를 디코딩해서 읽는다.
- `contents/`는 루트 디렉토리의 파일/폴더 목록을 반환한다. 각 항목의 `name`, `type`(file/dir) 필드를 사용한다.

### Pull Request (2회 호출)

```
GET https://api.github.com/repos/{owner}/{repo}/pulls/{number}
GET https://api.github.com/repos/{owner}/{repo}/pulls/{number}/files
```

### Issue (2회 호출)

```
GET https://api.github.com/repos/{owner}/{repo}/issues/{number}
GET https://api.github.com/repos/{owner}/{repo}/issues/{number}/comments
```

댓글이 있으면(comments > 0) 주요 댓글 내용도 요약에 반영한다.

### 파일

```
GET https://api.github.com/repos/{owner}/{repo}/contents/{path}?ref={branch}
```

### 커밋

```
GET https://api.github.com/repos/{owner}/{repo}/commits/{sha}
```

### 릴리즈

```
GET https://api.github.com/repos/{owner}/{repo}/releases/tags/{tag}
```

## 설명 형식

타입별로 아래 구조를 따라 한국어로 설명한다. 요약은 충분히 길고 구체적으로 작성한다 — 단순히 description 필드만 반복하지 말고, README와 코드 구조에서 얻은 정보를 바탕으로 프로젝트를 실제로 이해한 사람처럼 설명한다.

### 레포지토리

```
## 📦 {레포 이름}

**한 줄 요약**: {description 필드}

**주요 정보**
- 언어: {language}
- ⭐ 스타: {stargazers_count}
- 🍴 포크: {forks_count}
- 마지막 업데이트: {updated_at 날짜}
- 라이선스: {license.name}

**레포 구조**
{루트 디렉토리 항목을 폴더는 📁, 파일은 📄로 표시. 각 항목 옆에 역할 한 줄 설명. 10개 이내}

**프로젝트 설명**
{README를 깊이 읽고 6~8문장으로 설명:
  - 이 프로젝트가 해결하는 문제와 배경
  - 핵심 동작 방식과 주요 개념
  - 사용 방법 (설치, 실행 핵심 단계)
  - 주목할 만한 설계 결정이나 특징
  - 어떤 사람에게 유용한지}
```

### Pull Request

```
## 🔀 PR #{number}: {title}

**상태**: {state} — 열림 / 병합됨 / 닫힘
**작성자**: {user.login}
**대상 브랜치**: {head.label} → {base.label}
**규모**: +{additions}줄 / -{deletions}줄 / {changed_files}개 파일

**변경 내용 요약**
{body와 변경 파일 목록을 읽고 5~7문장으로 설명:
  - 이 PR이 수정하거나 추가하는 것
  - 왜 이 변경이 필요했는지 (버그, 기능 요청, 리팩토링 등)
  - 구체적으로 어떤 파일/로직을 어떻게 바꿨는지
  - 관련 issue 번호나 맥락}

**변경 파일**
{파일 목록, 파일당 변경 성격 한 줄 설명. 최대 7개}
```

### Issue

```
## 🐛 Issue #{number}: {title}

**상태**: {state} — 열림 / 닫힘
**작성자**: {user.login}
**레이블**: {labels 또는 "없음"}
**댓글**: {comments}개

**내용 요약**
{body와 주요 댓글을 읽고 5~7문장으로 설명:
  - 어떤 문제나 요청인지
  - 재현 방법이나 구체적인 상황 (버그인 경우)
  - 논의된 해결책이나 방향 (댓글이 있으면)
  - 현재 처리 상태}
```

### 파일

```
## 📄 {파일명}

**경로**: {path}
**크기**: {size} bytes
**언어**: {파일 확장자 기준}

**파일 설명**
{파일 내용을 읽고 5~7문장으로 설명:
  - 이 파일이 무엇을 하는지
  - 레포에서 어떤 역할을 담당하는지
  - 주요 함수/클래스/로직
  - 다른 파일과의 관계}
```

### 커밋

```
## 💾 커밋 {sha 앞 7자리}

**작성자**: {commit.author.name}
**날짜**: {commit.author.date}
**메시지**: {commit.message}
**규모**: +{stats.additions}줄 / -{stats.deletions}줄

**변경된 파일**
{파일 목록, 파일당 변경 성격 한 줄 설명. 최대 7개}

**커밋 설명**
{변경 파일과 diff를 바탕으로 이 커밋이 실제로 무슨 일을 하는지 4~5문장 설명}
```

### 릴리즈

```
## 🚀 {tag_name}: {name}

**배포일**: {published_at}
**작성자**: {author.login}

**변경사항 요약**
{body를 읽고 5~7문장으로 요약:
  - 새로운 기능
  - 버그 수정
  - 주요 변경 및 breaking change
  - 업그레이드 시 주의사항}
```

## 주의사항

- API 응답이 404이면 비공개 레포이거나 존재하지 않는 리소스임을 알린다.
- API rate limit (시간당 60회)에 걸리면 사용자에게 안내한다.
- README나 body가 영어라도 **요약은 항상 한국어**로 작성한다.
- 설명은 충분히 길고 실질적이어야 한다. 한두 줄로 끝내지 말고 실제로 깊이 읽은 사람처럼 구체적으로 설명한다.
