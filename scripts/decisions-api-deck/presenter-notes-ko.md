# Decisions API 발표 노트 (키워드)

- 덱: `decisions-api-deck.html` (18장, EN / 한국어: 오른쪽 위 버튼 또는 L 키, 노트 패널: NOTES 버튼 또는 N 키)
- 같은 노트가 덱 안의 노트 패널에도 들어 있음 (이 파일과 함께 갱신)
- 엔진 정보 기준: 2026-11-09 GitHub 릴리스 확인 (SGLang v0.5.23, vLLM v0.32.0 / v0.33.0rc3, Ollama v0.40.15)
- (Q) = 질문 나오면

---

## 1. Decisions API.
- SystemOne API(`/v1/systemone`, TypeSafe Jev) = Decisions API(`/v1/decisions`, OpenAI)
- 텍스트 생성 X → 선택지 중 하나 + 확률
- 결정마다 confidence score

## 2. LLM의 동작 방식
- 다음 토큰 확률 분포, vocabulary 전체
- forward pass 1회 = 분포 1개
- chat / completion / decision = 분포 활용 방식의 차이
- 숫자는 설명용 예시
- (Q) logit 계산 경로: hidden state h × 출력 행렬(unembedding, LM head) → logit(토큰당 1개) → ÷ temperature → softmax
  - `logit(i) = W[i] · h`, 행 하나 = 토큰 하나
  - 판단은 h를 만드는 층에서, 출력 행렬은 점수만

## 3. 토큰 샘플링. N회 반복.
- prefill 1회 + decode N회
- 출력 토큰 비용, 답 길이만큼 지연, parsing 필요

## 4. 선택지를 제공하고, 분포를 단 한 번만 계산.
- state + 질문 + 선택지 미리 정의
- prefill 후 한 번 읽기, decode 0회
- 결과: 형식이 정해진 값 + 확률
- Jev 70–500 ms / OpenAI 입력 토큰만 과금 / self-hosted: SGLang, vLLM, Ollama

## 5. 같은 모델, 다른 활용.
- 출력: 자유 텍스트 vs label + 확률
- 비용: input만 (OpenAI $0.10 / 1M)
- 실패: 선택지 밖 답 불가
- 못 하는 것: 추론 설명, 긴 답변, 목록 밖 답

## 6. 추론은 생성으로, 선택은 결정으로
- 대체 X, 역할 분담: 추론·설명은 생성, 목록 중 선택은 결정
- 생성 +: 자유 형식, 이유 설명, 여러 단계 추론(수학·논리), 목록에 없는 답
- 생성 −: 출력 토큰 비용·지연, 형식 오류·parsing, "90% 확신" = 생성된 텍스트일 뿐(보정 X)
- 결정 +: forward pass 1회(약 100 ms), input만 과금, 항상 목록 안 답, 선택지별 확률 → 임계값
- 결정 −: 이유 설명 X(감사), 선언된 선택지만 → "해당 없음" 추가, 계산이 필요한 답에 약함

## 7. 저렴하고 빠른 결정, 수용 또는 위임
- cascade: decision model 먼저 → confidence ≥ τ 수용 / 미만 위임(LLM 또는 사람)
- τ = 내 라벨 데이터로 맞춤, 엔진·모델·버전마다 confidence 다름 → 그대로 옮기지 않기
- JEV-as-a-Judge (arXiv:2609.26550): τ = 0.9, 1,610쌍
  - 68.5% 수용 / 31.5% 위임, 위임분에 오류 85%
  - 정확도 93.4% vs GPT-6 단독 92.5%, 비용 41%
- JevAgentHarness (arXiv:2610.21412): 결정 지점별 임계값, 비용 −14.2%, 500개 과제 결과 변화 없음
- (Q) 임계값 없으면? → −19점

## 8. 정해진 목록에서 고르는 모든 곳에
- 공통점: 답이 미리 정한 목록 안에 있음
- 분류·배정: 티켓·장애 → 팀 / Jev Router(OpenRouter, 2026-09-25) 요청마다 모델·추론 강도
- 분류: Xbox Research 피드백 1만+ 건, GPT-5 수준, 80–100배 빠름 (Microsoft Foundry Blog, 2026-10-09)
- agent 제어: tool·인자·계속/중단, tool 선택 27%, 비용 −14.2% (arXiv:2610.21412)
- agent 메모리: Jev-Mem(arXiv:2609.23986) 분류·연결·라우팅, 구축 6.6배, 질의 −37%
- 평가: JEV-as-a-Judge 판정당 277배 저렴, 텍스트에서 바로 읽히는 판정은 근접
- 추출: edgextract 그래프 연결마다 yes/no, 불확실 → 사람 검토

## 9. 결정을 내리는 다섯 단계
- Render → Check labels → Forward pass(prefill, state 캐시) → Keep labels → Softmax
- 예: logit A 8.1 / B 3.9 / C 2.0 → 0.983 / 0.015 / 0.002
- SGLang·vLLM은 logprob을 읽음, 라벨끼리 재정규화하면 동일
- (Q) one-token-label 검사: prompt vs prompt+label 토큰화 비교, 새 토큰 정확히 1개, 모든 엔진·매 요청
  - 라벨 = 엔진이 붙인 A/B/C, 0–9, yes/no (사용자 선택지 이름 X)

## 10. 질문 유형 3가지
- 같은 메커니즘, p 읽는 법 3가지
- choice = argmax / score = Σ i·pᵢ (정해진 단계에 대한 기댓값, 값 하나) / noul = p(yes)
- 질문마다 라벨·p 별도 (A/B/C, 0/1/2, yes/no)

## 11. 질문을 객관식 프롬프트로 변환
- SGLang 실제 렌더링
- 빈 `<think></think>` = thinking 강제 off
- assistant 첫 토큰 자리에서 분포 읽기
- (Q) state = 판단 대상 입력 데이터 ≈ LLM/agent의 context
  - 지시 X, 판단할 사실만
  - state 1개 → 질문 여러 개, 캐시 재사용
  - prompt injection: 확률은 흔들려도 선택지 밖 답은 불가

## 12. 기존 모델에서 추가 학습
- Jev(RLCD) 비공개 ↔ 공개 레시피 3개
- Nimble: contrastive pairs, 작은 차이로 답이 뒤바뀌는 예시 쌍(승인자 Mira ↔ Noah), 2,676개, Qwen3.5-9B LoRA
- Kev: fitted temperature, 본 적 없는(held-out) 데이터로 확률 보정
- s1: proper-scoring loss, '얼마나 확신해야 하는지'까지 학습, ECE 0.083 vs Jev 0.081
- (Q) 보정 temperature vs sampling temperature: 계산은 같음(logit ÷ T) / 정하는 사람·목적이 다름 / 가중치·답 불변, 확률만 조정
- (Q) s1: soft label(기존 모델 분포 + 정답 라벨), 과신 벌점, 선택지 순서 섞기, 약 10만 개, 정확도 0.808 vs Jev 0.835

## 13. SGLang
- prefill only, 새 토큰 0개
- 기존 `/v1/score` 재사용
- chat template 있는 모든 모델 + 전용 decision model(pplx-decider, Clef, Kev)
- `A:`…`Z:` → `AA:`…, 최대 255개
- thinking 강제 off
- confidence `(n·max(q) − 1)/(n − 1)`, label_mass 별도 필드
- `/v1/systemone` + 독자 규격 `/v1/decisions`
- (Q) chat template: Jinja만, 항상 추론하는 template 거절, 가짜 답변으로 답 위치 확인
- (Q) label_mass: 재정규화 전 라벨 확률의 합(예: 0.90), 낮으면 "어느 선택지도 아님" 경고
- (Q) `/v1/score` vs decision API: 저수준 기능 vs 제품 계약
  - 직접 해야 하는 것: 프롬프트, 토큰 ID, 검사, 답 변환
  - 이식성: 토큰 ID(모델 종속) vs 이름(모든 엔진)
  - 전용 head 모델(pplx-decider, Clef)은 토큰 API로 표현 불가
  - 비유: raw completion vs structured output
- (Q) `/v1/decisions`(SGLang): options/levels/yes_no, confidence 없음, OpenAI 클라이언트 비호환

## 14. vLLM
- greedy decode 1회, 토큰 버림
- temperature 적용 전 logprob, label 토큰만
- 특정 모델: Qwen3, Qwen3.5, DiffusionGemma, Clef Omni, LiquidAI d1-3B
- 최대 26개
- `enable_thinking: false`
- `p_top × label_mass`, `argmax_is_label`
- v0.32.0 `/v1/systemone`, v0.33.0rc3 OpenAI 호환 `/v1/decisions`

## 15. Ollama
- 모델이 학습한 프롬프트 재현
- state + 질문 = JSON 하나 (Nimble 형식)
- decision model만: Nimble, Tev1, Clef, Kev, pplx-decider, d1, d3, AutoTrust
- llama.cpp 자체 `/v1/systemone`(v0.40.4~), 프롬프트 형식·temperature는 모델 파일(GGUF)에 (이전: +100 logit bias)
- MLX: 선택지 토큰의 logit만 계산(출력 행렬의 행 몇 개), label_mass 없음
- confidence 모델별: 대부분 TypeSafe 공식, 일부 엔트로피, d1은 p_max
- 버전 고정: v0.40.4에서 공식 변경, 같은 분포에 0.922 → 0.978

## 16. 실전 예시
- `POST /v1/systemone`, Ollama + nimble
- state: 티켓 1개 / 질문: team(choice), refund(noul), urgency(score)
- (Q) 질문은 누가?
  - 대개 개발자가 코드에 고정(함수 시그니처처럼), state만 바뀜, 질문별 임계값
  - agent: 질문 틀 고정, 선택지는 실행 중 채움(사용 가능한 tool 목록)
  - JevAgentHarness: 결정 지점 8개, 지점별 임계값, tool 선택 27%, 비용 −14.2%, 결과 변화 없음 / 임계값 없으면 −19점
  - LLM 즉석 질문: 임계값 없음 → 위험

## 17. 입력 841 토큰, 출력 4 토큰
- team = 'billing' 0.985 (argmax)
- refund 0.997 = 환불을 요청했을 확률
- urgency score 0.815 = 기댓값, 단계 선택 X
- urgency confidence 0.046 = 거의 균등 → 0.815는 사실상 추측, 소수점 ≠ 신뢰
- 주의: v0.40.4 이전 기록, confidence는 예전 엔트로피 공식 값

## 18. Completion이 아닌 Decision
- 같은 LLM, forward pass 1회, 출력 토큰 비용 X
- 그대로(SGLang·vLLM chat model) or 추가 학습(Nimble·Kev·s1)
- 확률 ≠ 정답 보장
- 마무리: 버전 고정, 임계값은 내 데이터로, Jev도 독립 감사에서 과신
