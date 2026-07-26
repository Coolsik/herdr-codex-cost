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
```

로컬 개발 시에는 `herdr plugin install` 대신 `herdr plugin link "$PWD"`를
사용합니다.

0.2.x 버전에서 업데이트했다면 기존 cost용 Codex hook을 한 번 제거합니다.

```bash
herdr plugin action invoke dev.herdr-codex-cost.remove-hooks
```

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

Herdr가 Codex pane을 `working`으로 표시하는 동안 5초마다 비용을 갱신합니다.
pane이 `working` 상태를 벗어나면 polling을 중단하고 상태 변경 handler와 polling
worker가 마지막 갱신을 보장합니다. 세션이 빠르게 재시작되면 새 세션에 비용 정보가
생길 때까지 마지막으로 계산한 비용을 유지합니다.

각 pane은 rollout 경로, 파일 크기, 수정 시각으로 구성된 세션 트리 fingerprint를
캐시합니다. polling 중 fingerprint가 그대로면 token event를 다시 파싱하거나
sidebar metadata를 갱신하지 않고 기존 비용을 재사용합니다.

현재 pane을 즉시 수동 갱신할 수도 있습니다.

```bash
herdr plugin action invoke dev.herdr-codex-cost.refresh
```

`$1.23`은 정상적으로 계산된 값이고, `!!$1.23`은 미등록 모델·손상된 로그·누락된 하위 세션 때문에 일부 비용을 제외했음을 뜻합니다. 비공개 단가인 `codex-auto-review` 추정분은 `$1.23(+$0.45)`처럼 분리합니다.

표시값이 `session?`이면 Herdr가 pane과 Codex thread ID를 연결하지 못한
상태입니다. pane별 진단은 `${TMPDIR:-/tmp}/herdr-codex-cost-$UID/error-*.log`에
기록되며, 연결이 복구되면 자동으로 삭제됩니다. 빠른 세션 재시작 중에는 이미
계산된 비용이 있으면 `session?`로 덮어쓰지 않고 마지막 비용을 유지합니다.
Herdr의 세션 연결 정보가 없으면 Codex pane 상태줄의 thread ID prefix를 읽고
rollout의 전체 ID와 작업 디렉터리를 대조합니다. 정확히 하나만 일치할 때만
fallback을 사용하며, 일치하지 않거나 후보가 여러 개면 `session?`를 유지합니다.

## 테스트

```bash
./test/codex-cost-test
./test/codex-hook-test
./test/poll-cost-test
./test/update-cost-test
```

## 배포

GitHub 공개 저장소에 올린 뒤 repository topic `herdr-plugin`을 추가하면 Herdr marketplace가 자동으로 색인합니다. 사용자는 `herdr plugin install owner/repo`로 설치할 수 있습니다.
