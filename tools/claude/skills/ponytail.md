---
title: Ponytail
parent: Skills
nav_order: 1
description: AI 에이전트가 불필요한 코드를 만들지 않도록 도와주는 스킬, Ponytail 사용법 정리
keywords: Ponytail, Claude Code, AI Agent Skill
---

# Ponytail
> [DietrichGebert/ponytail](https://github.com/dietrichgebert/ponytail)

AI 에이전트가 코드를 짤 때 과도하게 짜지 않고, 꼭 필요한 만큼만 짜도록 유도해주는 스킬이다.
불필요한 라이브러리 설치나 과한 추상화를 줄여준다.

## 설치

Claude Code에서 아래 두 명령어를 **순서대로, 별도의 프롬프트로** 입력한다.

```bash
/plugin marketplace add DietrichGebert/ponytail
```

```bash
/plugin install ponytail@ponytail
```

## 사용법

| 명령 | 기능 |
|---|---|
| `/ponytail` | 강도 설정 (lite / full / ultra / off) |
| `/ponytail-review` | 현재 diff에서 과하게 짠 부분 검토 |
| `/ponytail-audit` | 저장소 전체 감사 |
| `/ponytail-help` | 도움말 |

기본값은 `full` 모드.

## 트러블슈팅: 자동 활성화가 안 될 때

Claude Code 플러그인은 내부적으로 Node.js 라이프사이클 훅을 실행하므로 `node`가 PATH에 잡혀 있어야 한다.
`node`가 없어도 `/ponytail-review` 같은 수동 명령어는 정상 동작하지만, 매 프롬프트마다 자동으로 켜지는 활성화 기능만 조용히 꺼진다 (에러 없이 그냥 동작 안 함).

nvm이나 Nix로 Node를 설치했다면 특히 주의가 필요하다. Claude Code가 훅을 실행하는 건 **비대화형(non-interactive) 셸**인데, nvm 설정은 보통 `.bashrc`/`.zshrc` 같은 대화형 셸 설정 파일에서만 로드되기 때문에 비대화형 셸의 PATH에는 `node`가 안 잡힐 수 있다.
