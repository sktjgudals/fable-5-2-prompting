# fable-5-2-prompting

Claude Fable 5.2와 Claude Mythos 5.2용 프롬프트를 점검하고 조정하는 **비공식** 에이전트 스킬입니다. Anthropic의 [Claude Fable 5.2 프롬프팅](https://platform.claude.com/docs/ko/build-with-claude/prompt-engineering/prompting-claude-fable-5-2) 가이드(2026-11-12 기준)를 근거로 만들었습니다. Anthropic과 제휴하거나 승인받은 프로젝트가 아닙니다.

[English README](README.md)

## 하는 일

Fable 5.2는 몇 가지 지점에서 이전 모델과 다르게 동작합니다. Fable 5.1이나 Opus 5.5용으로 쓴 지침이 이제는 비용을 높이거나, 부작용을 내거나, 효과가 없을 수 있습니다. 이 스킬은 에이전트에 두 가지 모드를 줍니다.

- **점검.** 에이전트가 시스템 프롬프트, `CLAUDE.md`, 스킬, 에이전트 파일, 메모리를 읽습니다. Fable 5.2에서 문제가 되는 줄마다 이유와 수정안을 보고합니다. 파일을 직접 고치지는 않습니다.
- **증상 조회.** 관찰한 증상을 말하면 가이드의 처방, 넣을 위치, 비용을 알려줍니다.

| 관찰한 증상 | 스킬이 안내하는 것 |
|---|---|
| 작업이 끝나기 전에 멈춤 | "Finishing work" 문구 (출력 토큰 약 30% 증가) |
| 필요 없는 승인을 요청함 | 메모리와 규칙에 쌓인 오래된 주의 사항, 범위가 넓은 승인 규칙 |
| 기존 코드에 덧붙이기만 하고 통합하지 않음 | 코드 소유권 문장 (출력 토큰 70~80% 증가) |
| 긴 턴 동안 조용함 | `thinking.display` 설정, 그다음 알림이나 프롬프트 문구 |
| 서브에이전트를 너무 많이 띄움 | 상시 위임 지침, 서브에이전트 도구 설명 |
| 간단한 요청에도 느림 | 프롬프트 문구가 아니라 `effort` 수준 |
| `stop_reason: "refusal"` | 거부 카테고리, 답변에 추론을 쓰라는 지침 |

## 항상 켜 두는 프롬프트가 아닌 이유

가이드는 관찰한 동작에 대해 한 번에 하나씩 적용하고 결과를 측정하라고 권합니다. 처방마다 토큰 비용이 있습니다. 스킬은 에이전트가 이 규칙을 따르게 하므로, 모든 문구를 프롬프트에 한꺼번에 넣지 않습니다.

## 설치

**Claude Code 플러그인:**

```
/plugin marketplace add sktjgudals/fable-5-2-prompting
/plugin install fable-5-2-prompting@fable-5-2-prompting
```

**수동 설치:** `skills/fable-5-2-prompting/` 폴더를 `~/.claude/skills/`(또는 Agent Skills 형식을 지원하는 하네스의 스킬 폴더)에 복사합니다.

## 사용

```
내 CLAUDE.md와 에이전트 파일을 Fable 5.2 기준으로 점검해줘.
```

```
Fable 5.2 에이전트가 자꾸 "계속할까요?"로 턴을 끝내. 뭘 바꿔야 해?
```

스킬 본문은 영어로 작성되어 있지만, 에이전트는 사용자의 언어로 답합니다.

## 테스트 방법

[tests/README.md](tests/README.md)의 시나리오를 새 에이전트에게 스킬 없이 한 번, 스킬과 함께 한 번 주고 비교했습니다. 스킬이 없을 때 에이전트는 가이드가 경고하는 넓은 승인 규칙을 그대로 두었고, `reasoning_extraction` 거부를 몰랐으며, 처방 문구를 지어내고 비용을 문구 길이로 추정했습니다. 스킬이 있을 때는 가이드의 문구와 측정 비용을 제시했고, 모든 문구를 한꺼번에 넣자는 요청을 거절했습니다.

## 정확성

- 기준은 공식 가이드입니다. 가이드가 바뀌면 이 스킬이 낡을 수 있습니다. 베타 헤더 이름은 사용 전에 가이드에서 확인하세요.
- 수치는 Anthropic의 자체 테스트 결과입니다. 본인 작업에서 직접 측정하세요.
- 가이드와 다른 부분을 발견하면 [CONTRIBUTING.md](CONTRIBUTING.md)를 참고해 알려 주세요. 변경 이력은 [CHANGELOG.md](CHANGELOG.md)에 있습니다.

## 라이선스

이 저장소의 원저작 텍스트는 MIT입니다. 예시 프롬프트 문구는 Anthropic 문서에서 인용했으며 Anthropic의 소유입니다. [LICENSE](LICENSE)를 참조하세요.
