# 4. 훅(Hooks)

훅은 **Claude Code가 특정 시점에 반드시 실행하는 셸 명령**입니다.
`CLAUDE.md` 의 규칙은 Claude가 지키려고 노력하는 "부탁"이고, 훅은 Claude의 판단과 상관없이 돌아가는 "장치"입니다.

## 주요 시점(이벤트)

| 이벤트 | 언제 | 쓰임 |
|---|---|---|
| `SessionStart` | 세션 시작·재개 | 오늘 날짜·현황을 미리 알려주기 |
| `UserPromptSubmit` | 프롬프트를 보낸 직후 | 입력 검사, 맥락 추가 |
| `PreToolUse` | 도구 실행 직전 | **실행을 막을 수 있음** (관문) |
| `PostToolUse` | 도구 실행 성공 후 | 저장 직후 자동 검사·포맷 |
| `Notification` | 입력 대기 등 알림 | 메신저·데스크톱 알림 |
| `Stop` | 응답을 마칠 때 | 끝내기 전 최종 검사 |
| `PreCompact` | 대화 요약 직전 | 중요한 기록 백업 |

## 예시: push 전에 검사 스크립트 돌리기

`.claude/settings.json`

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "python3 \"$CLAUDE_PROJECT_DIR/tools/prepush_gate.py\"",
            "timeout": 600
          }
        ]
      }
    ]
  }
}
```

- `matcher: "Bash"` — Bash 명령을 실행하기 직전에만 돈다
- 스크립트는 실행하려는 명령을 **JSON으로 표준입력(stdin)** 에서 받는다
- 문제가 없으면 `exit 0`, 막아야 하면 이유를 표준에러(stderr)에 쓰고 **`exit 2`**
- `exit 2` 로 막으면 그 이유가 Claude에게 전달되고, Claude가 고친 뒤 다시 시도한다

## 아이디어

1. **파일을 고치자마자 검사**: `PostToolUse` + `matcher: "Edit|Write"` 로 검사 스크립트 실행
2. **자리 비울 때 알림**: `Notification` 으로 승인 대기가 생기면 메신저로 받기
3. **원본 데이터 보호**: `PreToolUse` 로 원본 폴더에 쓰기나 DB `DELETE`·`UPDATE` 를 아예 막기
4. **완료 전 테스트 강제**: `Stop` 에서 테스트가 실패하면 끝내지 못하게 하기

## 주의할 점

- 훅은 **내 권한으로 아무 셸 명령이나** 실행합니다. 남이 만든 폴더·저장소를 받았다면 그 안의 `.claude/settings.json` 을 먼저 읽어 보세요.
- `Stop` 훅은 검사가 계속 실패하면 끝없이 반복할 수 있습니다. 재시도 횟수 제한을 두세요.
- 무거운 검사는 매번 말고 꼭 필요한 순간(예: push 직전)에만 거세요.
- 현재 설정은 `/hooks` 로 확인합니다.

자세한 내용: [Hooks 가이드(공식)](https://code.claude.com/docs/en/hooks-guide)

[← 이전](03-claude-md-and-skills.md) · [목차](../README.md) · [다음: 검증 루프 →](05-verification-loop.md)
