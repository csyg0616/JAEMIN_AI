# 디자인 스킬 설치 현황과 재설치 안내

Claude Code 프로젝트 스킬은 `.claude/skills/<이름>/SKILL.md`에 두면 자동으로 인식됩니다.
아래 스킬 파일은 저장소에 커밋되어 있으므로, 새 클라우드 세션에서도 그대로 유지됩니다.

## 설치된 항목

| 항목 | 경로 | 출처 / 커밋 | 종류 |
| --- | --- | --- | --- |
| frontend-design | `.claude/skills/frontend-design/` | anthropics/skills `8a1541c` | 스킬 |
| design-taste-frontend | `.claude/skills/design-taste-frontend/` | Leonxlnx/taste-skill `ce26fc2` (`skills/taste-skill`, v2 experimental) | 스킬 |
| image-to-code | `.claude/skills/image-to-code/` | Leonxlnx/taste-skill `ce26fc2` (`skills/image-to-code-skill`) | 스킬 |
| web-design-guidelines | `.claude/skills/web-design-guidelines/` | vercel-labs/agent-skills `063bee9` (metadata version 1.0.0) | 스킬 |
| playwright-cli | `.claude/skills/playwright-cli/` | `@playwright/cli` 0.1.22 (`playwright-cli install --skills`, microsoft/playwright-cli `b85c7a7`와 동일) | 스킬 |
| awesome-design-md | `references/awesome-design-md/` | voltagent/awesome-design-md `f696123` | **참고 문서 (스킬 아님)** |

스킬 폴더 이름은 각 `SKILL.md`의 `name:` 값(= `npx skills add --skill`에 쓰는 설치 이름)과 같습니다.
라이선스: taste-skill MIT(`LICENSE` 동봉), vercel agent-skills MIT(README 명시, 원본 폴더에 LICENSE 파일 없음), playwright-cli Apache-2.0, awesome-design-md MIT.

### 스킬별 참고 사항

- **image-to-code**: 원래 Codex용으로 작성되어 "디자인 이미지를 먼저 직접 생성"하는 단계를 요구합니다.
  Claude Code에는 이미지 생성 도구가 없으므로, 참고 이미지(스크린샷, 시안)를 직접 제공해서 쓰는 방식이 현실적입니다.
- **web-design-guidelines**: 실행할 때마다
  `https://raw.githubusercontent.com/vercel-labs/web-interface-guidelines/main/command.md`를 가져옵니다.
  네트워크 정책에서 `raw.githubusercontent.com`이 막히면 동작하지 않습니다.
- **playwright-cli**: 스킬 파일만으로는 동작하지 않고, 아래 CLI 설치가 세션마다 필요합니다.

## 새 세션에서 유지되는 것 / 다시 설치해야 하는 것

| 구분 | 대상 |
| --- | --- |
| 유지됨 (저장소에 커밋) | `.claude/skills/*` 5개 스킬, `references/awesome-design-md/`, 이 문서, `.gitignore`의 `.playwright-cli/` 항목 |
| 매 세션 재설치 | `playwright-cli` 전역 CLI (`npm install -g`) |
| 설치 불필요 (클라우드 기본 제공) | Chromium: `/opt/pw-browsers/chromium` |
| 커밋하지 않음 | `node_modules/`, 브라우저 실행 파일, `.playwright-cli/` (스냅샷·로그·스크린샷 출력) |

## Playwright CLI 재설치 (클라우드 세션)

```bash
# 1) CLI 설치 (브라우저 다운로드는 건너뜀)
PLAYWRIGHT_SKIP_BROWSER_DOWNLOAD=1 npm install -g @playwright/cli@0.1.22
playwright-cli --version

# 2) 미리 설치된 Chromium 사용 설정
#    - cdn.playwright.dev 다운로드는 네트워크 정책에서 차단됨
#    - 컨테이너가 root로 실행되므로 Chromium 샌드박스를 꺼야 실행됨
export PLAYWRIGHT_MCP_EXECUTABLE_PATH=/opt/pw-browsers/chromium
export PLAYWRIGHT_MCP_SANDBOX=false

# 3) 확인
playwright-cli open http://localhost:8000/
playwright-cli snapshot
playwright-cli screenshot --filename=/tmp/check.png
playwright-cli close
```

- 샌드박스 해제는 신뢰할 수 있는 로컬 페이지 테스트에만 쓰세요.
- `playwright-cli install --skills`는 스킬 파일이 이미 커밋되어 있으므로 다시 실행하지 않아도 됩니다.
  실행하면 브라우저 다운로드 단계에서 403 오류가 나지만, 스킬 파일 설치 자체는 먼저 완료됩니다.
- `file://` 주소는 기본적으로 차단되므로 `python3 -m http.server` 같은 로컬 서버로 페이지를 띄워 여세요.
- 로컬 PC에서는 2)의 환경 변수 없이 `npm install -g @playwright/cli@latest` 후 바로 사용하면 됩니다.

## 스킬 재설치 / 업데이트

스킬 파일은 커밋되어 있으므로 평소에는 다시 설치할 필요가 없습니다. 최신 버전으로 바꾸고 싶을 때만 아래를 사용하세요.

```bash
# taste-skill (두 스킬만)
npx skills add https://github.com/Leonxlnx/taste-skill --skill "design-taste-frontend"
npx skills add https://github.com/Leonxlnx/taste-skill --skill "image-to-code"

# vercel agent-skills (web-design-guidelines만)
npx skills add vercel-labs/agent-skills --skill "web-design-guidelines"

# playwright-cli 스킬
playwright-cli install --skills
```

`npx skills add`는 설치 위치나 에이전트를 묻거나 추가 폴더를 만들 수 있으므로, 실행 후 `git status`로 변경 내용을 확인하세요.
원본 저장소를 받아 해당 `SKILL.md`를 `.claude/skills/<설치 이름>/`에 직접 복사해도 결과는 같습니다.

awesome-design-md 참고 문서 갱신:

```bash
git clone --depth 1 https://github.com/voltagent/awesome-design-md /tmp/adm
cp -r /tmp/adm/README.md /tmp/adm/LICENSE /tmp/adm/design-md references/awesome-design-md/
```
