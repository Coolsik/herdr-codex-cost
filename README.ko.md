# herdr-codex-cost

[English](README.md) | 한국어

현재 Herdr pane에서 실행 중인 Codex 세션과 그 하위 에이전트의 누적 API 환산 비용을 계산해 Agent 사이드바에 표시하는 Herdr 플러그인입니다. `tmux-statusline`의 `codex-session-usage` 계산 방식과 모델 단가표를 기반으로 하며, 실제 ChatGPT/Codex 구독 청구액이 아니라 공개 API Standard 요금으로 환산한 추정치입니다.

## 요구 사항

- Herdr 0.7.0 이상
- `python3`, `jq`
- Codex integration (`herdr integration install codex`)

## 설치 및 표시 설정

```bash
herdr plugin install Coolsik/herdr-codex-cost
herdr integration install codex
herdr plugin action invoke dev.herdr-codex-cost.setup-hooks
```

로컬 개발 시에는 `herdr plugin install` 대신 `herdr plugin link "$PWD"`를
사용합니다. hook 설치 후 이미 실행 중인 Codex 프로세스는 재시작해야 합니다.
Codex가 새 hook을 처음 읽을 때 실행 승인을 요청할 수 있습니다.

`~/.config/herdr/config.toml`의 Codex sidebar row에 `$cost`를 추가합니다. 예:

```toml
[ui.sidebar.agents.rows_by_agent]
codex = [
  ["state_icon", "agent", "$cost"],
  ["workspace", "tab"],
]
```

그 다음 Herdr를 재시작하거나 설정을 reload합니다.

## 비용 갱신

다음 Codex 실행 경계에서 비용이 자동으로 갱신됩니다.

- tool 실행 직전 (`PreToolUse`)
- 하위 에이전트 종료 (`SubagentStop`)
- Herdr가 에이전트 상태 변화를 감지했을 때(최종 `working` → `done` 포함)

hook 기반 갱신은 pane별로 최대 5초에 한 번만 실행됩니다. 5초 안에 tool이
여러 번 호출돼도 세션 로그를 다시 읽지 않습니다. 메인 작업의 최종 비용은
Herdr 상태 변경 이벤트가 갱신합니다.

현재 pane을 즉시 수동 갱신할 수도 있습니다.

```bash
herdr plugin action invoke dev.herdr-codex-cost.refresh
```

플러그인을 삭제하기 전에는 이 플러그인이 추가한 Codex hook만 먼저 제거합니다.

```bash
herdr plugin action invoke dev.herdr-codex-cost.remove-hooks
herdr plugin uninstall dev.herdr-codex-cost
```

표시값의 `~`는 계산 완료, `!!`는 미등록 모델·손상된 로그·누락된 하위 세션 때문에 일부 비용을 제외했음을 뜻합니다. 비공개 단가인 `codex-auto-review` 추정분은 `~$1.23(+$0.45)`처럼 분리합니다.

## 테스트

```bash
./test/codex-cost-test
```

## 배포

GitHub 공개 저장소에 올린 뒤 repository topic `herdr-plugin`을 추가하면 Herdr marketplace가 자동으로 색인합니다. 사용자는 `herdr plugin install owner/repo`로 설치할 수 있습니다.
