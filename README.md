# .github

Re:Source AI 조직의 공통 설정 레포.

| 파일 | 역할 |
| --- | --- |
| `CONTRIBUTING.md` | 팀 공통 규칙 (레포 역할, 브랜치, 커밋, PR, 배포) |
| `.github/ISSUE_TEMPLATE/` | 기능·버그·작업 이슈 템플릿의 원본. 조직 기본값이 Private 레포에 적용되지 않으므로 각 레포의 `.github/ISSUE_TEMPLATE/`에 복사해 씀 |
| `.github/pull_request_template.md` | 기본 PR 템플릿. 자기 PR 템플릿이 없는 레포에 적용됨 (server 레포는 자체 템플릿 사용) |

규칙을 바꿀 때는 이 레포에 PR로 제안하고 팀 합의 후 병합함.
CONTRIBUTING.md의 작성 원본은 server 레포 `docs/team-conventions.md`이며, 한쪽을 바꾸면 다른 쪽에도 같은 내용을 반영함.

템플릿을 바꾸면 각 레포에도 복사함: `cp -r ../.github/.github/ISSUE_TEMPLATE .github/` (각 레포 루트에서 실행)
