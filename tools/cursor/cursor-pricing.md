---
title: "💡 Cursor Pro 요금제 완벽 정리 (모델 / Auto / 토큰 과금 중심)"
parent: Tools
description: "Cursor의 최신 요금제 구조와 모델별 과금 방식을 개인 사용자(Pro) 기준으로 정리한 문서입니다."
keywords: ["Cursor Pro", "Cursor 요금제", "Cursor 모델", "Cursor Auto", "Cursor 과금", "Cursor Token", "Cursor Pricing"]
---

# 💡 Cursor Pro 요금제 완벽 정리

> **업데이트 기준:** 2025년 10월  
> **출처:** [Cursor Docs: Models](https://cursor.com/docs/models), [Pricing / Auto](https://cursor.com/docs/account/pricing), [Tokens & Pricing](https://cursor.com/learn/tokens-pricing)

---

## 📘 개요

Cursor는 AI 개발용 IDE로, 여러 AI 모델(GPT, Claude 등)을 통합 지원하며  
**사용량 기반(토큰 단위) 과금 구조**를 채택하고 있습니다.  

이 문서는 **개인 사용자(Pro 플랜)** 기준으로  
모델 선택, Auto 모드, 토큰 기반 과금 방식을 이해하기 쉽게 정리했습니다.

---

## ⚙️ 1. Cursor 지원 모델 (Models)

Cursor는 다양한 AI 모델을 선택적으로 지원합니다.

| 구분 | 예시 모델 | 특징 | 비고 |
|------|------------|------|------|
| **고성능 모델** | GPT-4, Claude 3 등 | 응답 품질 높고, 속도 빠름 | 단가 높음 |
| **경량 모델** | GPT-3.5, Claude Instant 등 | 빠르고 저렴함 | 정밀도 낮음 |
| **전용 모델 (Cursor Custom)** | Cursor-code, Cursor-chat | IDE 최적화 | Pro 이상만 사용 가능 |

> 💬 **Tip:**  
> 무거운 모델일수록 정밀하지만 토큰 사용량(=비용)도 빠르게 증가합니다.  
> “짧고 명확한 요청”이 효율적입니다.

---

## ⚡ 2. Auto 모드 (Auto Mode)

> Cursor의 **Auto 모드**는 “가장 적절한 모델”을 자동으로 선택하는 기능입니다.

| 항목 | 설명 |
|------|------|
| **기능** | 요청 내용에 따라 모델을 자동으로 선택 |
| **장점** | 설정 없이 최적 모델을 활용 가능 |
| **주의점** | 사용 모델이 자동으로 바뀌므로 **비용 예측이 어려움** |

📎 **출처:** [Cursor Docs: Auto](https://cursor.com/docs/account/pricing)

<div class="note">
💡 <b>Auto 모드 팁</b>  
Auto 모드는 편리하지만, 요청이 복잡하거나 길면  
GPT-4 같은 고가 모델이 선택되어 크레딧이 빠르게 소모될 수 있습니다.  
따라서 <b>짧은 요청 위주로 Auto 모드</b>를 사용하는 것이 좋습니다.
</div>

---

## 💰 3. 토큰 기반 과금 구조 (Tokens & Pricing)

Cursor는 “토큰(token)” 단위를 기준으로 요금을 계산합니다.

### 🔹 토큰 계산 방식

입력 토큰 + 출력 토큰 = 총 사용 토큰


예시:
- 입력(prompt): 300 tokens  
- 출력(response): 700 tokens  
→ **총 1,000 tokens** 사용  

### 🔹 모델별 단가 차이

| 모델 구분 | 입력 토큰 단가 | 출력 토큰 단가 | 비고 |
|------------|----------------|----------------|------|
| GPT-4 계열 | 높음 | 높음 | 정밀 분석, 코드 품질 우수 |
| GPT-3.5 계열 | 중간 | 중간 | 빠르고 경제적 |
| Claude Instant | 낮음 | 낮음 | 요약/탐색 작업 적합 |

> **참고:** 각 모델의 정확한 단가는 [Tokens & Pricing](https://cursor.com/learn/tokens-pricing) 문서에서 확인 가능합니다.

---

### 🔹 크레딧(credits) 구조

- Pro 사용자는 매월 일정량의 **크레딧**을 제공받습니다.  
- 요청 실행 시마다 크레딧이 **토큰 단가 × 토큰 수** 만큼 차감됩니다.  
- 크레딧이 소진되면 추가 요청이 제한되거나 추가 결제가 필요합니다.  

<div class="highlight">
⚠️ <b>주의:</b>  
크레딧 잔량을 실시간으로 확인하는 습관을 들이세요.  
특히 대형 코드 분석, 문서 생성 등은 토큰 사용량이 급격히 늘어납니다.
</div>

---

## 📊 4. 현재 vs 과거 요금제 비교

| 항목 | 과거 (500회 요청 기반) | 현재 (토큰 기반 과금) |
|------|-------------------------|------------------------|
| **과금 단위** | Fast Request 500회 | 입력 + 출력 토큰 |
| **예측 가능성** | 단순 (횟수 제한) | 요청 크기에 따라 변동 |
| **속도 제한** | Slow Queue 존재 | 고성능 모델 사용 시 자동 조절 |
| **자동완성(AutoComplete)** | 별도, 한도에 포함 안 됨 | 대부분 여전히 별도 처리 |
| **요금 기준** | 요청 횟수 | 모델별 토큰 단가 × 사용량 |

> ✅ **정리 요약:**  
> 현재는 단순 “질문 횟수”가 아니라,  
> **얼마나 많은 연산(토큰)을 썼는가**가 핵심 기준입니다.

---

## 🧠 5. Pro 사용자에게 유용한 팁

<div class="tip">
💡 <b>Pro 사용자 실전 팁</b>
</div>

1. **Auto 모드 남용 주의** — 자동 모델 선택은 편하지만 비용 변동 폭이 큽니다.  
2. **작은 요청으로 나누기** — 한 번에 대량 입력 대신 나눠 요청하세요.  
3. **모델 단가 확인** — 단가가 낮은 모델로 빠른 테스트 후, 필요 시 GPT-4로 전환.  
4. **크레딧 관리** — 월별 사용량을 주기적으로 점검하세요.  
5. **Cursor-code 모델 활용** — IDE 전용 모델로 코드 품질 대비 비용 효율적입니다.

---

## 🪙 6. 요금제 비교 (Free / Pro / Team)

| 항목 | Free | Pro | Team |
|------|------|------|------|
| **가격** | 무료 | 월 20~30 USD | 사용자 수 기준 커스텀 |
| **모델 접근** | 제한적 (GPT-3.5 등) | 모든 모델 (GPT-4, Claude 포함) | 동일 + 협업 기능 |
| **Auto 모드** | 일부 제한 | 전체 가능 | 전체 가능 |
| **크레딧 제공량** | 소량 | 월 단위 고정 제공 | 팀 단위 통합 관리 |
| **협업 기능** | 없음 | 없음 | ✅ 있음 |
| **우선 처리 속도** | 제한적 | 빠름 | 매우 빠름 |

> 📎 자세한 내용은 [Cursor Pricing 페이지](https://cursor.com/docs/account/pricing) 참고.

---

## 🔚 맺음말

Cursor의 과금 체계는 이제 “질문 횟수” 중심이 아니라  
**모델·토큰·연산량 중심의 크레딧 구조**로 전환되었습니다.  

따라서 Pro 사용자라면 다음을 기억하세요:

- ✴️ **짧고 효율적인 요청이 비용을 아낀다.**  
- ⚙️ **Auto 모드는 편리하지만 비용 변동이 크다.**  
- 💵 **크레딧 잔량을 항상 체크하자.**

---

📘 **참고 문서**
- [Cursor Docs: Models](https://cursor.com/docs/models)  
- [Cursor Docs: Pricing / Auto](https://cursor.com/docs/account/pricing)  
- [Cursor Learn: Tokens & Pricing](https://cursor.com/learn/tokens-pricing)
