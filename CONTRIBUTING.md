# Re:Source AI 팀 공통 규칙

> 모든 레포(server, ai, frontend, infra)에서 코드 작업할 때 지키는 규칙.
> 원본은 조직 `.github` 레포의 `CONTRIBUTING.md`이며, 바꿀 때는 PR로 제안하고 팀 합의 후 반영함.
> 레포별 세부 규칙(실행 방법, 폴더 내부 구조)은 각 레포의 README와 문서를 따름.

## 1. 레포 구성과 역할

| 레포 | 담당 | 하는 일 | 하지 않는 일 |
| --- | --- | --- | --- |
| `server` | 서버 담당자 | API, 인증·회원, 기업 정보, DB 스키마·마이그레이션 | 판정·계산 로직, LLM 호출 |
| `ai` | AI 팀 | 에이전트, MCP 도구, 지식 기반 (세부 설계 중) | 회원·인증 처리, 화면 |
| `frontend` | 프론트 팀 | 화면, 3D 시각화, server API 호출 | DB 직접 접근, ai 직접 호출 |
| `infra` | 서버 담당자 | 전체 통합 실행 compose, Azure 리소스 정의, 버전 조합 기록 | 서비스 코드 |
| `.github` | 서버 담당자 | 이 문서, 기본 이슈·PR 템플릿 | 코드 |

### 호출 관계

```
frontend ──HTTP──▶ server ──HTTP/MCP──▶ ai
                     │
                     └──▶ PostgreSQL
```

- frontend는 server만 호출함 (ai를 직접 호출하지 않음)
- 레포 사이의 약속은 **API 형식**뿐임. 다른 레포의 코드를 복사하거나 import하지 않음

### API 형식 변경 규칙

| 변경 | 허용 |
| --- | --- |
| 필드·엔드포인트 **추가** | 자유 |
| 필드 **삭제·이름 변경·타입 변경** | 호출하는 쪽 담당자와 합의 후, 이슈에 기록 |
| server ↔ ai 형식 변경 | ai 레포의 스키마 폴더(`src/resource_ai/api/schemas/`)를 바꾸는 PR은 server 담당자를 리뷰어로 지정하고 승인 후 병합 |

## 2. 로컬 개발 환경

| 목적 | 방법 |
| --- | --- |
| 한 레포만 개발 | 그 레포의 README대로 실행 (server는 `docker compose up -d --build`) |
| 전체를 함께 확인 | 아래처럼 레포를 **나란히** clone하고 infra에서 compose 실행 |

```
workspace/          ← 레포 아님
├─ infra/
├─ server/
├─ ai/
└─ frontend/
```

```bash
cd workspace/infra
docker compose up -d --build
```

- 팀 표준 도구: Docker Desktop, Git
- 레포 안에 다른 레포를 clone하지 않음

## 3. 브랜치 전략

```
main       배포용. develop → main 릴리스 PR로만 병합 (서버 담당자만)
 └ develop 통합 브랜치. 작업 브랜치에서 PR로만 병합 (직접 push 금지)
    ├ feat/      기능 추가·변경
    ├ fix/       버그 수정
    ├ docs/      문서만 변경
    ├ chore/     설정·의존성·CI·빌드
    ├ refactor/  동작 변화 없는 구조 개선
    └ test/      테스트만 추가·수정
hotfix/          main에서 분기한 운영 긴급 수정
```

### 이름 규칙

`<종류>/<이슈 또는 todo 번호>-<짧은-설명>` (영문 소문자, 하이픈)

```
feat/T-010-company-update
fix/42-ms-login-issuer
docs/parse-input-spec        # 번호가 없으면 생략 가능
```

### 커밋 메시지

브랜치 종류와 같은 접두어 + 영어 한 줄 요약. 본문은 한국어여도 됨.

```
feat: add company update API
fix: validate microsoft issuer by tenant id
```

## 4. 작업 흐름과 PR

```bash
git switch develop && git pull
git switch -c feat/T-010-company-update
# 작업, 테스트, 커밋
git push -u origin feat/T-010-company-update
# GitHub에서 develop으로 PR
```

### PR 규칙

1. 대상 브랜치는 항상 **develop**
2. PR 템플릿을 채움 (관련 이슈, 변경 내용, 테스트 결과)
3. **CI 통과 후 작성자가 직접 병합** (리뷰 승인 필수 아님). 단 ai 레포의 API 형식(스키마) 변경은 서버 담당자 리뷰 승인 후 병합
4. 병합 방식은 **Create a merge commit** 하나만 사용함 (작업 PR, 릴리스 PR 모두). 병합 후 작업 브랜치는 자동 삭제
5. PR 하나는 한 가지 목적만 담음. 기능과 리팩터링을 섞지 않음

### 리뷰 기준

- 동작: 요구사항(이슈·spec)대로 동작하는가, 테스트가 있는가
- 경계: 1장의 레포 역할을 넘지 않았는가 (예: frontend가 ai를 직접 호출)
- 형식: API 형식을 삭제·변경했다면 합의 기록이 있는가
- 보안: 비밀값·토큰이 코드나 로그에 없는가

## 5. CI (각 레포 공통 원칙)

| 항목 | 규칙 |
| --- | --- |
| 실행 시점 | develop·main 대상 PR, develop·main push |
| 필수 검사 | 린트·포맷, 테스트 (레포별 추가 검사는 각 레포 README) |
| 실패 시 | 병합 불가. 고친 뒤 같은 PR에 다시 push |
| 브랜치 보호 | develop·main 모두 CI 통과를 필수 조건으로 설정 |

예: server 레포는 린트, 에이전트 문서 동기화 검사, PostgreSQL 마이그레이션 검사, 테스트를 실행함.

## 6. 배포 규칙 (CD)

| 항목 | 규칙 |
| --- | --- |
| 배포 권한 | **서버 담당자만** develop → main 병합 가능 (main push 권한 제한) |
| 배포 단위 | 레포별로 각자 배포 (server: App Service, ai: Azure Functions, frontend: 프론트 팀 결정) |
| 배포 시점 | main 병합 = 배포. main에는 배포할 코드만 들어감 |
| 버전 | main 병합 시 `vX.Y.Z` 태그를 붙임 (기능 추가 Y, 수정 Z) |
| 순서 | server ↔ ai 형식이 바뀐 릴리스는 **ai 먼저, server 나중** |
| 조합 기록 | 함께 동작을 확인한 버전 조합을 infra 레포에 기록 (예: server v0.3.0 + ai v0.5.0) |
| 되돌리기 | 이전 태그를 다시 배포함. main에 force push 하지 않음 |

### 릴리스 절차 (서버 담당자)

1. ai: develop → main PR (`release: YYYY-MM-DD`) → CI 통과 → **Create a merge commit** → 태그 → 배포
2. server: 같은 절차
3. infra: 버전 조합 기록, 통합 실행으로 확인

### 긴급 수정

1. main에서 `hotfix/<설명>` 분기 → main으로 PR → 병합·배포
2. 같은 변경을 develop에도 반영 (main → develop PR)

## 7. 환경변수와 비밀값

1. 비밀값(API 키, 비밀번호, 토큰)은 **코드와 커밋에 넣지 않음**
2. 로컬은 `.env` 파일을 쓰고 `.gitignore`에 포함함. 필요한 변수 목록은 `.env.example`에 값 없이 남김
3. 운영 값은 Azure(App Service 설정, Key Vault)와 GitHub Secrets에만 둠
4. 비밀값이 커밋되면 즉시 서버 담당자에게 알리고 **키를 재발급**함 (커밋 삭제만으로는 부족함)

## 8. AI 코딩 도구 사용

1. 각 레포의 `AGENTS.md`(Claude Code는 `CLAUDE.md`)가 AI 도구의 규칙임. 작업 전에 읽게 함
2. AI가 만든 코드도 사람이 만든 코드와 똑같이 PR·CI·리뷰를 거침
3. AI 도구에게 develop·main 직접 커밋, push, 병합을 맡기지 않음
4. 에이전트 문서(`AGENTS.md`, `.agents/`)는 승인 후에만 바꾸고, 바꾸면 변경 기록을 남김

## 9. 이슈

- 템플릿: 기능(`feat`), 버그(`fix`), 작업(`chore`)
- 레포를 넘나드는 작업은 주가 되는 레포에 이슈를 만들고, 다른 레포 이슈를 링크함
- 담당자(Assignee)를 지정하고 시작함
