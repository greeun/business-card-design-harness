# 명함 디자인 하네스 (Business Card Design Harness)

> 독창적이고 인쇄 규격에 맞는 **anti-slop HTML 명함 시안**(기본 20개)과
> 한 페이지 **비주얼 컴패니언 갤러리**를 생성 — **Planner → Generator → Evaluator** 3-에이전트
> 하네스 + 필수 휴먼 비주얼 체크포인트로 구동.

**언어:** [English](./README.md) · 한국어(이 문서)

[Claude Code](https://claude.com/claude-code) 스킬. 짧은 브리프(브랜드·연락처·톤·팔레트)를 받아
실제 온라인 명함 인쇄 서비스 규격에 맞춘 단일 파일 HTML 명함 20개를 만든다 — **각각 서로 다른
레이아웃 아키타입**으로. 시각 미감은 모델이 자체 통과시키지 않는다. 무엇이든 "완료"라 부르기 전,
사람이 미감을 직접 승인한다.

---

## 목차

- [왜 만들었나](#왜-만들었나)
- [작동 원리 (하네스)](#작동-원리-하네스)
- [산출물](#산출물)
- [빠른 시작](#빠른-시작)
- [활성화 흐름](#활성화-흐름)
- [품질 모델 (루브릭)](#품질-모델-루브릭)
- [3개의 휴먼 게이트](#3개의-휴먼-게이트)
- [범위 경계 (RGB vs CMYK)](#범위-경계-rgb-vs-cmyk)
- [산출 단계 (HTML → 인쇄 포맷)](#산출-단계-html--인쇄-포맷)
- [스킬 구성 파일](#스킬-구성-파일)
- [베이크인 디자인 레퍼런스 (BCR)](#베이크인-디자인-레퍼런스-bcr)
- [원전 충실도](#원전-충실도)
- [설치](#설치)
- [한계](#한계)
- [크레딧](#크레딧)

---

## 왜 만들었나

일반 LLM에 "명함 20개 디자인"을 시키면 보통 **한 레이아웃을 20가지 색으로** 바꾼 결과가 나온다:
전부 중앙 정렬, 기본 Arial, 그라데이션 + 드롭섀도, 클립아트 전화/이메일 아이콘. 이게 *슬롭(slop)* —
디자이너라면 절대 내보내지 않을 템플릿 결과물이다. 원인은 두 가지다:

1. **자기평가 편향.** 생성과 평가를 한 세션이 같이 하면, 자기 결과물을 자신 있게 칭찬한다.
   생성과 평가는 **별도 프로세스**여야 한다.
2. **감각 한계.** LLM은 타이포 균형이나 여백을 실제로 *볼 수 없다*. "좋아 보인다"는 추측일 뿐이다.
   시각 미감은 **사람 눈이 루프 안에** 있어야 한다.

이 스킬은 [*Harness Design for Long-Running Application Development*](https://www.anthropic.com/engineering/harness-design-long-running-apps)
(Anthropic, 2026) 패턴을 이식해 둘 다 막는다: GAN 스타일 **역할 분리**(파일로만 통신하는 별도 `Agent`
호출) + Evaluator가 우회할 수 없는 **필수 휴먼 비주얼 체크포인트**.

---

## 작동 원리 (하네스)

오케스트레이터가 세 역할을 **별도 `Agent` 호출**로 디스패치한다. 서로의 추론은 못 보고 **파일로만**
통신한다.

```
                    ┌─────────────────────────────────────────────┐
  브리프 ──▶ 인테이크 │ 브랜드 · 연락처 필드 · 톤 · 팔레트 · 프리셋 · 수량 │
                    └───────────────┬─────────────────────────────┘
                                    │  (+ 라이브 리서치 동의 게이트)
                                    ▼
                 ┌──────────┐   spec.md    ┌────────────┐
                 │ PLANNER  │ ───────────▶ │ GENERATOR  │  베이크인 레퍼런스(BCR) 소비
                 │ "무엇을"  │              │ "어떻게"    │  20개 distinct 아키타입, mm 단위
                 └──────────┘              └─────┬──────┘  cards/*.html + gallery.html + card-spec.json
                       ▲                         │ READY_FOR_QA
                       │ sprint_contract.md      ▼
                       │ (협상)            ┌───────────────────────────┐
                       │                   │ ★ 휴먼 비주얼 체크포인트    │  gallery.html 열고 승인
                       │                   └─────────────┬─────────────┘
                       │                                 ▼
                 ┌──────────┐   critique.md       ┌────────────┐
                 │GENERATOR │ ◀───── FAIL ──────── │ EVALUATOR  │  8개 adversarial probe + 루브릭
                 │ (REFINE/ │                      │ "진짜인가"  │  미감 자체 통과 금지
                 │  PIVOT)  │ ─────── PASS ──────▶  └────────────┘
                 └──────────┘                            │
                                                         ▼
                                              산출물 폴더 패키징
                                          (선택: Stage-2 포맷 변환)
```

- **Planner** — 각 카드가 *무엇을* 담아야 하는지(필드, 3-tier 위계, 프리셋, 다양성 기준) 결정하고
  `spec.md`에 동결. CSS는 지시하지 않는다.
- **Generator** — *어떻게*를 결정. 레퍼런스 카탈로그에서 20개 distinct 아키타입을 골라 mm 단위
  HTML/CSS를 쓰고, 갤러리를 만들고, 핸드오프 전 자가검증.
- **Evaluator** — 적대자. 8개 이진 probe + 5축 루브릭. 휴먼 승인 없이는 시각 미감 축을 **통과시킬 수
  없다**.

Tier = **Full**: 스프린트 계약 협상 + 스프린트별 Evaluator + 파일 기반 핸드오프.

---

## 산출물

실행 결과는 단일 **산출물 폴더**로 패키징된다:

```
<deliverable>/
├─ README.md          ← 팀 핸드오프: 구조 · 검토법 · 인쇄 책임 고지 · 연락처 맵 · 상태
├─ gallery.html       ← 검토 진입점: 전체 카드 타일 + bleed/trim/safe 오버레이 토글
├─ card-spec.json     ← 프리셋 + 카드별 아키타입 맵 + 연락처 필드 맵 + sRGB/CMYK 고지
├─ cards/             ← card-01.html … card-NN.html (단일 파일, 업로드/변환 대상)
├─ docs/              ← spec.md, generator_report.md, critique.md
├─ preview/           ← 스크린샷 (갤러리 + 대표 카드)
└─ _archive/          ← 폐기/대체된 컨셉 (라이브 세트에서 분리 보관)
```

각 카드는 **단일 자족 HTML 파일**(인라인 CSS + 웹폰트), *서로 다른* 레이아웃 아키타입, mm 단위 기하,
CSS에 `trim` / `bleed` / `safe` 정의. 갤러리는 중립 회색 바탕에 카드를 타일링하고 전역 오버레이 토글을
제공한다: **green = bleed, magenta = trim, cyan = safe** (`@media print`에서 숨김).

---

## 빠른 시작

Claude Code 스킬이다. [설치](#설치) 후 작업을 그냥 서술하면 된다:

```
명함 디자인 만들어줘 — Withwiz, 김도윤 Founder & CEO, 브랜드 블루 #2D5BFF
```

**트리거 문구**

- **KO:** `명함 디자인`, `명함 시안`, `명함 시안 20개`, `명함 HTML 만들어줘`,
  `온라인 명함 업로드용 시안`, `비주얼 컴패니언 검토`, `명함 갤러리 생성`, `명함 디자인 하네스`
- **EN:** `business card`, `business card design`, `card design`, `business card mockups`,
  `HTML business card`, `name card design`, `business card gallery`, `design business cards for print`

스킬은 먼저 인테이크를 돌린다 — 브랜드, **모든** 연락처 필드, 톤/업종, 팔레트, 프리셋, 수량을 받기
전에는 생성하지 않는다. (빈 입력은 플레이스홀더 카드를 만들고, 이는 placeholder sweep에서 FAIL.)

---

## 활성화 흐름

| # | 단계 | 게이트 |
|---|------|--------|
| 1 | **인테이크** — 브랜드, 모든 연락처 필드, 톤, 팔레트(hex), 프리셋, 수량 | |
| 2 | **라이브 리서치 게이트** — 선택적 최신 트렌드 조사; 불가 시 정적 레퍼런스로 폴백 | 🔵 사용자 동의 |
| 3 | **Planner** → `spec.md` (Generator가 읽는 유일한 파일) | |
| 4 | **스프린트 계약 협상** — Generator가 관찰가능 체크 제안, Evaluator가 강화·승인 | |
| 5 | **Generator** → 20개 `card-NN.html` + `gallery.html` + `card-spec.json`, 자가검증 | |
| 6 | **갤러리 렌더** — 오케스트레이터 렌더; 개별 카드 스크린샷으로 sanity 체크 | |
| 7 | **★ 휴먼 비주얼 체크포인트** — `gallery.html` 열고 오버레이 토글하며 승인 | 🔴 **필수** |
| 8 | **Evaluator** → `critique.md` (PASS/FAIL): 8 probe + 루브릭; 미감 자체 통과 불가 | |
| 9 | **반복** (FAIL 시) — Generator의 Strategic Decision(REFINE / PIVOT / ESCALATE). 캡 5–15, 기본 8 | |
| 10 | **패키징** — 산출물 폴더 조립 (cards/ docs/ preview/ _archive/ + README) | |
| 11 | **최종 디자인 선택** — N개는 *컨셉*; 사용자가 인쇄할 디자인 선택 (+ 선택적 서브변형 비교 루프) | 🔵 사용자 선택 |
| 12 | **Stage-2 변환** (선택) — 선택된 디자인만 PDF/JPG로 변환 | 🔵 사용자 선택 |
| 13 | **업로드 확정 게이트** — 명시 승인 시에만 "업로드 준비 완료" 선언 | 🔴 사용자 동의 |

---

## 품질 모델 (루브릭)

Evaluator는 모든 배치를 **5축, 1–5점**으로 채점한다. 일반 LLM이 기본적으로 가장 약한 두 축에
**2× 가중치**를 준다:

| # | 기준 | 가중 | 무엇을 검사 |
|---|------|:----:|-------------|
| **C1** | 타이포 위계 | **2×** | 이름/직함/연락처 진짜 3-tier 위계; 카드당 단일 정렬 시스템; 폰트 ≤2 패밀리; 이름 커닝 |
| **C2** | 여백 균형 | **2×** | 콘텐츠가 trim 밖; 의도적 시각 무게; 단일 보호 포컬 포인트; 의도적 비대칭(중앙 충전 ❌) |
| C3 | 브랜드 색 일관성 | 1× | 1–2 브랜드 + 1–2 뉴트럴 일관 적용; 소형 텍스트 대비 ≥ 4.5:1; 브랜드색은 *절제된* 액센트 |
| C4 | 디자인 다양성 / anti-slop | 1× | 20개 distinct 아키타입; archetype+alignment+color 조합 **중복 0**; 색만 바꾼 변주 **금지**; 슬롭 텔 없음 |
| C5 | 규격 정확 | 1× | CSS에 trim/bleed/safe; 모든 텍스트 safe 내; 모든 카드에 모든 필드; 20/20 렌더; `mm` 단위(`px` ❌) |

**Verdict 로직(이진):** 전 기준 ≥ 4 **and** 전 probe clean **and** 휴먼 체크포인트 APPROVED →
**PASS**. 2× 기준 < 4, 또는 1× 기준 < 3, 또는 Definition-of-Done 미검증 항목 → **FAIL**. 모델은
휴먼 승인 없이 C1/C2/C4 미적 판단을 **자체 통과시킬 수 없다**.

왜 2×? C3/C4/C5는 기계적·규칙 순회 가능 — Claude가 기본적으로 적절히 한다. 압력 없이 무너지는 건
*진짜* 타이포 위계(모든 줄을 같은 무게로 평탄화, 정렬 혼용)와 *의도적* 여백(공간 균등 충전, 보이드를
낭비로 취급)이다. 이 두 축이 명함이 **디자인된** 것처럼 보일지, **템플릿** 같을지를 가른다.

---

## 3개의 휴먼 게이트

스킬은 사람이 책임져야 하는 지점에서 완전 자율을 거부한다:

1. 🔵 **라이브 리서치 동의** (2단계) — 웹을 칠지 말지.
2. 🔴 **휴먼 비주얼 체크포인트** (7단계) — *필수*. `gallery.html`을 실제 브라우저에서 열어 미감을
   검토하고 승인. 사용자 승인 기록 없이 미적 축이 ≥ 4로 채점되면 Evaluator probe 6이 **전체 verdict를
   FAIL**시킨다.
3. 🔴 **업로드 확정** (13단계) — 명시적 OK 없이 "업로드 준비 완료"를 선언하지 않는다.

---

## 범위 경계 (RGB vs CMYK)

비주얼 컴패니언은 **RGB 발광 화면**이다. 인쇄된 명함은 **종이에 흡수된 CMYK 잉크** — 화면에서 쨍한
블루/그린은 인쇄에서 탁해지고, 종이 질에 따라 또 달라진다. 갤러리는 인쇄 색을 **검증할 수 없다**.

그래서 스킬은 범위를 지킨다: **sRGB 권고**만 표시하고, **CMYK 변환·물리 교정쇄(교정쇄)·인쇄 색
정확도는 인쇄소/사람 책임**임을 명시한다. 카드가 인쇄 색 정확하다고 주장하지 않는다.

---

## 산출 단계 (HTML → 인쇄 포맷)

- **Stage 1 (기본 산출물): HTML.** `cards/*.html` + `gallery.html` — 검토 / 핸드오프 / 원본 진실.
  스킬은 Stage 1을 완료하고 **멈춘다**. 기본적으로 포맷 변환은 하지 않는다.
- **Stage 2 (선택, 디자인 승인 후 — 사용자 선택):** KR 온라인 명함 서비스
  (오프린트미 · 비즈하우스 · 레드프린팅 …)는 **HTML이 아닌 PDF / AI / JPG**를 받는다. 인쇄소 입고엔
  변환이 필요하다. 스킬은 셋 중 사용자가 고른 것만 제공:
  1. **인쇄용 PDF** — 카드별 앞/뒤 분리 페이지, trim+bleed, 가이드 제거 (인쇄소 입고).
  2. **고해상 JPG** — 카드별·면별 (프리뷰 / SNS / 서비스 업로드).
  3. **통합 PDF** — 전체 카드 1파일 (팀 / 인쇄소 검토).

  Stage 2는 휴먼 체크포인트 승인 전에는 절대 실행되지 않는다.

---

## 스킬 구성 파일

| 파일 | 줄수 | 역할 |
|------|----:|------|
| `SKILL.md` | 277 | 오케스트레이터: 활성화 흐름, 반복 지혜, 원칙, 튜닝 루프, V1/V2 가이던스 |
| `references/business-card-design-reference.md` | 457 | **베이크인 정적 리서치(BCR)** — 규격, 크래프트, 22개 아키타입 카탈로그, 슬롭 텔, HTML/CSS 패턴 |
| `references/generator-prompt.md` | 215 | Generator 역할 프롬프트 (카드 + 갤러리 + card-spec.json 작성) |
| `references/evaluator-prompt.md` | 190 | Evaluator 역할 프롬프트 — 8개 adversarial probe(휴먼 게이트 probe 포함) |
| `references/planner-prompt.md` | 123 | Planner 역할 프롬프트 (`spec.md` 작성) |
| `references/sprint-playbook.md` | 114 | Full tier 스프린트 분해 + 스프린트 계약 협상 |
| `references/rubric.md` | 88 | 5 기준, 2× 가중, verdict 로직, 이중 렌즈 캘리브레이션 |
| `references/evaluator-calibration.md` | 82 | 기준 C1–C5별 few-shot 1 / 3 / 5 점수 앵커 |

모든 역할 프롬프트는 **자족적**이다 — 프롬프트 파일만 받고 디스패치된 서브에이전트가 필요한 모든
것을 갖는다.

---

## 베이크인 디자인 레퍼런스 (BCR)

`references/business-card-design-reference.md` (457줄)는 Generator가 카드를 쓰기 전 소비하는 큐레이션
리서치다. 웹 의존 없이 결과물을 슬롭 바닥에서 끌어올리는 핵심:

| § | 섹션 | 핵심 |
|---|------|------|
| 1 | 표준 규격 | 지역별 정확 치수; bleed / trim / safe; CSS의 mm 규칙; 300dpi 맥락 |
| 2 | KR 온라인 명함 서비스 | 레드프린팅 / 오프린트미 / 비즈하우스 … 업로드 스펙 (90×50mm trim, 2mm bleed, 3mm safe) |
| 3 | 글로벌 주요 서비스 | MOO / Vistaprint / Canva / Adobe Express 디자인 에토스를 취향 앵커로 |
| 4 | 디자인 원칙 — 크래프트 | 타이포 위계 비율, 여백, 브랜드 색 일관성, WCAG 대비 |
| **5** | **레이아웃 아키타입 카탈로그** | **22개 명명 아키타입** + 다양성 레시피 — 20개 distinct 카드의 직접 공급원 |
| 6 | Anti-slop 가이드 | "X 대신 Y" 쌍; 슬롭 텔 블록리스트 |
| 7 | 본받을 디자이너 / 스튜디오 | Pentagram 등; "좋음"의 기준점 |
| 8 | HTML/CSS 구현 | 인쇄 규격 mm 카드 패턴, 토글 가능한 bleed/trim/safe 오버레이, 검토 갤러리 |

22개 아키타입 카탈로그(§5)가 20개 시안을 진짜 *다르게* 만든다 — 색만 바꾼 변주는 금지되고,
Evaluator는 `card-spec.json`에서 다양성을 재계산해 거짓말하는 Generator를 잡아낸다.

---

## 원전 충실도

모든 구성 요소는 원전 아티클의 원칙에 매핑되고 감사로 검증된다(35개 체크리스트 행 + 8개 adversarial
probe, 출시 스킬에서 전부 PASS):

- **GAN 스타일 역할 분리** — Planner / Generator / Evaluator는 별도 `Agent` 호출; **파일 통신만**.
- **few-shot 앵커 루브릭** — 기준별 1/3/5 구체 예시, "좋아 보임" ❌.
- **반복 범위 5–15, 기본 8** — 단일 낮은 캡 ❌; 늦은 돌파는 실재(아티클의 "Dutch Art Museum" 도약은
  iteration ~10에 도래).
- **재시도 시 Strategic Decision** — REFINE / PIVOT / ESCALATE; 피벗엔 critique 증거 필요, 컨텍스트
  리셋 망각 ❌.
- **컨텍스트 리셋 ≠ 컴팩션** — 컨텍스트 불안은 신규 세션 + `handoff.md`로 해소, 컴팩션으로는 ❌.
- **감각 한계 휴먼 체크포인트** — Evaluator가 우회 못 하는 게이트.
- **운영형 Evaluator 튜닝 루프** — 번호 (a)–(d): 로그 읽기 → 발산 탐지 → 반례 추가 → 재실행·확인.
- **모든 구성 요소는 가정을 담는다** — 모델 업그레이드 시 하나씩 제거(급진적 단순화는 실패);
  휴먼 체크포인트와 루브릭은 절대 제거 안 함.

모델 클래스 표(Sonnet 4.5 / Opus 4.5 / Opus 4.6)가 Full tier를 Simplified로 줄여도 되는 시점을 알려준다.

### 실제 실행으로 검증됨

샘플 실행(Withwiz, AI 스타트업; KR-standard 프리셋; 20개)에서 **20/20 distinct 아키타입,
20/20 unique archetype+alignment+color 조합, portrait 2개 + one-bold-move 7개, 모든 카드에 6개 연락처
필드 전부, 플레이스홀더 0**을 산출 — 동결된 `card-spec.json` 스키마로 검증.

---

## 설치

이 스킬은 Claude Code 스킬 저장소에 있다. Claude Code가 사용하게 하려면:

```bash
# business-card-design-harness/ 를 담은 저장소 루트에서
ln -s "$(pwd)/business-card-design-harness" ~/.claude/skills/business-card-design-harness
```

이후 Claude Code를 시작(또는 재시작)하고 위 트리거 문구 중 하나를 쓰면 된다. API 키·외부 서비스 불필요
— 디자인 레퍼런스가 내장돼 있고, 라이브 웹 리서치는 선택·동의제다.

**선택** — 배포용 패키징:

```bash
python ~/.claude/skills/skill-creator/scripts/package_skill.py ./business-card-design-harness
```

---

## 한계

- **인쇄 색은 범위 밖.** RGB 화면 ≠ CMYK 인쇄; 최종 색은 인쇄소 책임([범위 경계](#범위-경계-rgb-vs-cmyk)).
- **모델은 자기 미감을 판정하지 않는다.** 사람이 비주얼 체크포인트를 완료해야 하고, 그 없이는 실행이
  끝나지 않는다. 버그가 아니라 기능이다.
- **HTML은 중간 산물.** KR 인쇄 서비스는 PDF/AI/JPG를 원함 — 실제 주문엔 Stage-2 변환이 필요하고,
  이는 opt-in이다.
- **headless 전체 페이지 갤러리 스크린샷은 화면 밖 카드를 덜 렌더한다**(lazy-paint 캡처 아티팩트).
  *실제* 브라우저에서 검토하고, 개별 카드를 스크린샷해 sanity 체크.
- **20개 카드는 컨셉이지 20개 최종본이 아니다.** 인쇄할 디자인은 사용자가 선택한다.

---

## 크레딧

- 패턴: Anthropic — [*Harness Design for Long-Running Application Development*](https://www.anthropic.com/engineering/harness-design-long-running-apps)
  (Prithvi Rajasekaran, 2026) 및 *Building Effective Agents* — *"가능한 가장 단순한 해법을 찾고,
  필요할 때만 복잡도를 올려라."*
- **skill-wizard-harness** 메타 하네스로 제작(위자드가 동일한 Planner → Generator → Evaluator 패턴을
  스킬 제작에 적용).
- [Claude Code](https://claude.com/claude-code)용.
