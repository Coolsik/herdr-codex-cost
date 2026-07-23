# herdr-codex-cost

현재 Herdr pane에서 실행 중인 Codex 세션과 그 하위 에이전트의 누적 API 환산 비용을 계산해 Agent 사이드바에 표시하는 Herdr 플러그인입니다. `tmux-statusline`의 `codex-session-usage` 계산 방식과 모델 단가표를 기반으로 하며, 실제 ChatGPT/Codex 구독 청구액이 아니라 공개 API Standard 요금으로 환산한 추정치입니다.

## 요구 사항

- Herdr 0.7.0 이상
- `python3`, `jq`
- Codex integration (`herdr integration install codex`)

## 설치 및 표시 설정

```bash
herdr plugin link "$PWD"
herdr integration install codex
```

`~/.config/herdr/config.toml`의 Codex sidebar row에 `$cost`를 추가합니다. 예:

```toml
[ui.sidebar.agents.rows_by_agent]
codex = [
  ["state_icon", "agent", "$cost"],
  ["workspace", "tab"],
]
```

그 다음 Herdr를 재시작하거나 설정을 reload합니다. 비용은 Codex 상태가 바뀔 때 자동 갱신됩니다. 즉시 갱신하려면 다음 action을 실행할 수 있습니다.

```bash
herdr plugin action invoke dev.herdr-codex-cost.refresh
```

표시값의 `~`는 계산 완료, `!!`는 미등록 모델·손상된 로그·누락된 하위 세션 때문에 일부 비용을 제외했음을 뜻합니다. 비공개 단가인 `codex-auto-review` 추정분은 `~$1.23(+$0.45)`처럼 분리합니다.

## 테스트

```bash
./test/codex-cost-test
```

## 배포

GitHub 공개 저장소에 올린 뒤 repository topic `herdr-plugin`을 추가하면 Herdr marketplace가 자동으로 색인합니다. 사용자는 `herdr plugin install owner/repo`로 설치할 수 있습니다.
