---
title: 잠입/스텔스 게임 분석
description: 잠입 게임의 탐지 모델과 소음 전파, AI 인지 상태 노출과 용서 설계를 1차 출처에서 확인 가능한 범위로 분석하여 긴장과 계획의 플레이를 설계하는 스텔스 디자인 리뷰 가이드
---

# 잠입/스텔스 게임 분석

1. **개요**: 잠입 게임의 정의와 핵심 특징, 설계 철학
2. **유형별 분류**: 순수 잠입, 액션 하이브리드, 2D 잠입, 샌드박스 암살, 이머시브 심 잠입, 공포 잠입, 오픈월드 내 잠입
3. **핵심 메커니즘**: 탐지 모델, 소음 전파, 인지 상태 기계와 탐색, 자극 중재, 정보 시각화, 용서 설계, 은신 자원
4. **플레이 경험 설계**: 긴장과 이완, 발각 이후 설계, 실패 처리와 리트라이, 비살상 플레이 지원, 접근성
5. **주요 사례 분석**: Splinter Cell Blacklist, Mark of the Ninja, Thief, Hitman, Third Eye Crime
6. **기획자 가이드라인**: 설계 원칙, 체크리스트, 함정과 해결책
7. **결론 및 미래 전망**: 장르의 진화와 발전 방향

> **출처 및 신뢰도 안내**
> 이 문서의 수치·메커니즘 서술은 아래 공개 자료를 **직접 열람하여 확인한 것만** 인용합니다. 잠입은 이 리뷰 시리즈에서 **구현 수준 1차 출처가 이례적으로 강한** 장르입니다 — 아래 Game AI Pro 챕터 3건은 **자신이 구현한 시스템을 저자 본인이 서술한 기록**이므로 '표명된 의도'가 아니라 '실제 동작'에 가깝습니다.
> - **Martin Walsh(Ubisoft Toronto)** — Game AI Pro 2 ch.28 「Modeling Perception and Awareness in Tom Clancy's Splinter Cell Blacklist」. 시야 형상, 본 레이캐스트 탐지 파이프라인, 소음 경로 계산, 용서 설계의 핵심 근거입니다. **Figure 28.3은 출시 빌드의 디버그 드로잉**이라 구현 증거로서 강합니다.
> - **Brook Miles(Klei Entertainment)** — Game AI Pro ch.32 「How to Catch a Ninja: NPC Awareness in a 2D Stealth Platformer」. 시야 판정의 논리곱, 사운드 메시 A* 가청 판정, 자극 우선순위의 근거입니다.
> - **Rich Welsh** — Game AI Pro 2 ch.27 「Looking for Trouble: Making NPCs Search Realistically」. 탐색 상태 기계의 분기와 포기 조건의 근거입니다. ⚠️ **대상 타이틀이 익명 처리**되어 있어 특정 게임에 귀속할 수 없습니다.
> - **Tom Leonard(Looking Glass Studios)** — GDC 2003 「Building an AI Sensory System: Examining the Design of Thief: The Dark Project」. 정렬된 3D 뷰콘 집합과 감각/지식 모델 분리의 근거입니다.
> - **Damián Isla(Moonshot Games)** — AIIDE 2013 「Third Eye Crime: Building a Stealth Game Around Occupancy Maps」. 점유 맵의 직접 노출과 '발각 이후' 설계의 근거입니다. ⚠️ **1페이지 확장 초록**이며 구현 수치·플레이테스트 데이터가 전무합니다.
> - **Io-Interactive / Klei / Ubisoft Toronto** — GDC 강연 **자막(자동 생성)** 4건. Hitman의 샌드박스 가이드와 사회적 공간 설계, Mark of the Ninja의 정보 시각화 철학, Blacklist의 연출 근거입니다.
>
> **자동 생성 자막의 취급**: 자막은 **개념·논증의 근거로만 사용하고 수치·고유명사·발표자 이름에는 사용하지 않았습니다.** 자동 인식은 수치와 이름에 취약합니다(이 코퍼스에서 실제로 확인된 왜곡 사례는 부록에 기록했습니다). 본문에 등장하는 **수치는 전부 텍스트 원문(PDF)으로 확인한 것**이며 자막에서 온 수치는 하나도 없습니다.
>
> **이 문서가 의도적으로 비워 둔 영역**: **시스템형 대 저작형 샌드박스의 정량 비교**, **비살상 플레이 지원의 구현 근거**, **공포 잠입(Alien: Isolation 등)의 개발사 1차 출처**는 확보하지 못했습니다. 해당 절은 확인된 진술만 귀속 서술하고 나머지는 **수치 없이 정성 서술**로 처리했으며, 그 사실을 각 절과 부록에 명시했습니다.
>
> **출처의 시점**: 확보된 1차 출처는 **1998~2015년에 집중**되어 있습니다(Thief 1998, Mark of the Ninja 2012, Blacklist 2013, Third Eye Crime 2013). 이들은 **특정 출시작에 대한 역사적 구현 기록**이므로 원리로서 노후하지는 않으나, **현행 업계 관행의 대표성은 보장되지 않습니다.**

---

# 1. 개요

## 1-1. 잠입 게임의 정의

잠입(Stealth) 게임은 플레이어가 **적대적 관측자에게 인지되지 않는 상태를 자원으로 관리**하며 공간을 통과하는 장르입니다. 다른 액션 장르가 '적을 제거하는 능력'을 핵심 자원으로 삼는 데 비해, 잠입 장르의 핵심 자원은 **적의 무지(無知)** 입니다. 플레이어의 목표는 대개 적을 이기는 것이 아니라 **적이 자신에 대해 가진 정보 상태를 원하는 값으로 유지하는 것**입니다.

이 정의는 장르의 모든 설계 결정을 규정합니다. 적의 정보 상태가 게임의 승패를 가르는 변수라면, 그 변수는 **플레이어가 읽을 수 있어야** 합니다. 읽을 수 없는 변수를 관리하라고 요구하는 게임은 잠입 게임이 아니라 제비뽑기입니다. 따라서 잠입 장르에서 **AI의 인지 상태를 플레이어에게 보여주는 문제**는 편의 기능이 아니라 **장르의 성립 조건**입니다.

Splinter Cell Blacklist의 인지 모델을 구현한 Martin Walsh는 이 모델들이 성공하기 위해 갖춰야 할 네 가지 특성을 **공정성(fairness), 일관성(consistency), 좋은 피드백(good feedback), 지능(intelligence)** 으로 규정합니다. 그가 일관성을 설명하는 방식은 장르의 성격을 정확히 드러냅니다.

> "As a player, you need to have some idea of what the AI will perceive and how they will react so you as the player can strategize and improve; so the AI's behavior needs to be somewhat predictable. This is in contrast to actual humans who vary in terms of what they perceive and how they react and, as a result, tend to be very unpredictable."
> — Martin Walsh, Game AI Pro 2 ch.28, p.314

즉 잠입 AI는 **실제 인간과 반대 방향으로 설계됩니다.** 실제 인간은 예측 불가능하지만, 잠입 게임의 경비병은 예측 가능해야 합니다. 플레이어가 전략을 세우고 개선하려면 그래야 하기 때문입니다. 같은 절에서 Walsh는 예측 가능성이 반복성과 다르다는 단서를 답니다 — 거리와 타이밍은 매번 비슷해야 하지만 애니메이션과 barks(짧은 음성 클립)는 달라야 하며, 그렇지 않으면 몰입이 매우 빠르게 깨집니다.

그리고 지능에 대한 그의 정의는 장르 전체의 축약입니다.

> "if your opponents feel dumb, it isn't satisfying to beat them. But not being dumb does not necessarily mean being smart; it means always being plausible."
> — Martin Walsh, Game AI Pro 2 ch.28, p.314

**똑똑하지 않은 것과 멍청해 보이는 것은 다르며, 잠입 AI가 목표해야 하는 것은 똑똑함이 아니라 개연성입니다.** 이 문장은 이 문서 전체에서 반복해서 되돌아올 기준점입니다.

### 장르 발전의 주요 이정표

**형성기 (1980년대 후반~1990년대)**:
- 회피가 전투의 대안으로 제시되는 초기 실험들이 등장
- 《Thief: The Dark Project》(1998): **감각 모델을 게임의 중심 시스템으로 승격**. 1인칭 시점에서 빛과 소리를 자원화

**정착기 (1998~2005)**:
- 《Metal Gear Solid》 계보: 시야 원뿔의 UI 노출과 경계도 상태의 대중화
- 《Splinter Cell》 계보: 빛/그림자의 정밀 시뮬레이션과 수직 이동
- 《Hitman》 계보: 변장을 통한 **사회적 잠입**을 물리적 은폐의 대안으로 확립

**분화기 (2007~2013)**:
- 《Splinter Cell: Conviction》(2010): 시야 형상을 원뿔에서 **박스**로 교체
- 《Mark of the Ninja》(2012): 2D 평면에서 **정보 완전 노출** 잠입을 성립시킴
- 《Splinter Cell: Blacklist》(2013): **관(coffin) 모양 박스** 도입, 3중 형상 병용
- 《Third Eye Crime》(2013): **점유 맵을 UI로 직접 노출**, '발각 이후'를 핵심 플레이로

**확산기 (2012~현재)**:
- 《Dishonored》 계보: 이머시브 심의 도구 상호작용과 잠입의 결합
- 《Alien: Isolation》(2014) 등: **공포 잠입**의 성립 — 회피가 유일한 선택지인 구성
- 《Assassin's Creed》·오픈월드 액션 다수: 잠입을 **선택 가능한 접근법 중 하나**로 편입
- 《Hitman》(2016~): 에피소딕 라이브 운영과 **저작형 가이드가 얹힌 시스템형 샌드박스**

## 1-2. 잠입 게임의 핵심 특징

잠입 장르의 특징은 네 가지 근본 구조로 나눌 수 있습니다.

**첫째, 비대칭 정보 게임입니다.** 플레이어는 적의 위치·순찰·시야를 관측할 수 있지만 적은 플레이어를 관측하지 못하는 상태에서 시작합니다. 게임의 진행은 이 정보 비대칭이 **플레이어에게 유리한 상태에서 불리한 상태로 붕괴할 위험**을 관리하는 과정입니다. 이는 대부분의 액션 장르가 '체력'이라는 하나의 스칼라 자원을 관리하는 것과 근본적으로 다릅니다 — 잠입의 자원은 **적의 머릿속에 있는 상태**이며 플레이어의 캐릭터 시트에 없습니다.

**둘째, 계획이 실행에 선행합니다.** Klei의 창업자는 Mark of the Ninja 개발 과정을 회고하며, 수십 개의 프로토타입 시나리오를 테스트한 끝에 **게임의 요점이 기계적으로는 "환경을 관측하고, 정보를 얻고, 공격을 계획하고, 실행하는 것"이며 그중 계획 단계가 가장 중요하다**는 결론에 도달했다고 설명합니다. 그리고 계획이 성립하려면 **예측 가능한 인과**가 필요합니다. 이 이론이 명확해지자 나머지 설계 결정이 스스로 풀렸다는 것이 그의 진술입니다.

**셋째, 관측 대상이 자율적으로 움직입니다.** 퍼즐 게임의 장애물은 정적이거나 결정론적 사이클을 돕니다. 잠입 게임의 장애물은 **플레이어의 행동에 반응해 상태를 바꾸는 에이전트**이며, 이 반응 자체가 플레이어의 도구가 됩니다. Blacklist의 애니메이션 디렉터는 강연에서 스플린터 셀의 본질이 **"AI를 조종하는 것"** 이며 AI가 플레이어가 원하는 대로 움직이게 만드는 데 집착했다고 진술합니다 — 여기를 보게 하고, 저기로 가게 하는 것. 즉 **적의 인지 상태는 회피 대상이자 동시에 조작 대상**입니다.

**넷째, 실패의 의미가 장르 내에서 갈립니다.** 어떤 잠입 게임에서 발각은 곧 실패(리트라이)이고, 어떤 게임에서 발각은 새로운 플레이 국면의 시작입니다. 이 선택이 장르 내 하위 유형을 가르는 가장 강한 축이며, 3장과 4장에서 반복해 다룹니다.

**핵심 특징 요약**

| 축 | 잠입 장르의 처리 | 대비되는 장르 |
|---|---|---|
| **핵심 자원** | 적의 무지(인지 상태) | 체력·탄약(플레이어 시트 위의 스칼라) |
| **AI 설계 목표** | 예측 가능성 + 개연성 | 도전 강도·다양성 |
| **정보 흐름** | 플레이어가 적을 관측, 적은 플레이어를 모름 | 대칭적 교전 |
| **시간 구조** | 관측 → 계획 → 실행 | 반응 → 실행 |
| **장애물의 성격** | 반응하는 에이전트(조작 가능) | 정적/결정론적 패턴 |
| **실패의 의미** | 유형에 따라 리트라이 또는 새 국면 | 대체로 체력 소모 |

## 1-3. 잠입의 설계 철학

**첫째, 플레이어 지각이 시뮬레이션 정확도를 이깁니다 — 단 개연성이 경계를 긋습니다.**

이것이 잠입 설계의 제1원칙이며, 이 문서에서 가장 강한 1차 출처를 가진 명제입니다. Blacklist 팀은 내부 리뷰에서 반복되는 불만에 직면했습니다 — 크리에이티브 디렉터가 아무도 없는 복도를 달리는데 갑자기 어딘가에서 "누구야!"라는 소리가 들리는 것입니다. 게임을 멈추고 프리캠으로 소리를 들은 NPC의 위치를 보여 주면 대개 납득했지만, 다음번에 또 불만을 제기했습니다. Walsh의 판정은 **그가 옳았다**는 것입니다.

> "One important thing we learned here, as with tuning vision, is that it's only important what's plausible from the player's point of view—it really doesn't matter what the NPC should see or hear; it's what the player thinks the NPC can see and hear. This is a really important distinction. It implies that in the case of player perception versus simulation accuracy, player perception should win. This is not an absolute statement, however, and in the solution we will present, we'll show how this is actually limited by plausibility."
> — Martin Walsh, Game AI Pro 2 ch.28, p.321

**"플레이어는 게임을 멈추고 프리캠을 켤 수 없다"** — 이것이 논증의 전부입니다. NPC가 무엇을 들을 수 있어야 하는가는 무의미하고, 플레이어가 NPC가 무엇을 들을 수 있다고 **생각하는가**만이 유의미합니다.

동시에 이 원칙은 절대 명제가 아닙니다. 개연성이 한계를 긋습니다. Walsh는 모퉁이 바로 너머에 NPC가 있는 경우를 듭니다 — 플레이어가 뛰어서 소리를 내면, 보이지 않는 NPC에게 발각되는 것은 여전히 불공정하게 느껴집니다. 그러나 **NPC가 반응하지 않았는데 플레이어가 모퉁이를 돌아 그를 발견한다면, 그가 못 들었다는 것이 우스꽝스럽게 느껴집니다.** 그래서 팀은 이 경우 **플레이어에게 불공정하게 느껴질지라도 개연적인 결과가 나오는 거리를 골라야 했고**, 화면 밖 NPC의 청각을 아예 꺼 버릴 수는 없었습니다.

**둘째, 모델은 충실도가 아니라 튜닝 가능성을 위해 단순화됩니다.**

Thief의 리드 프로그래머 Tom Leonard는 각 뷰콘이 **콘 안의 어느 위치에 대상이 있든 상수 출력을 낸다**고 서술하며, 그 이유를 명시합니다.

> "For simplicity and gameplay tunability, each viewcone is presumed to produce a constant output regardless of where in the viewcone the subject is positioned."
> — Tom Leonard, GDC 2003

이것은 **시각의 충실도 모델이 아니라 명시적인 단순화**입니다. 잠입 설계자가 물리적 정확도를 추구하기 시작하면 튜닝 노브를 잃습니다. 15년 뒤 Blacklist 팀이 도달한 결론도 같습니다 — 그들의 형상은 시각 모델에서 유도된 것이 아니라 **플레이테스트의 산물**입니다: "we arrived at our solution through many hours of playtesting and designer tweaking."

**셋째, 명료함이 정교함을 이깁니다.**

Klei는 Mark of the Ninja 개발 중 복잡한 AI 시스템을 구현하려다 두 가지 문제에 부딪혔습니다 — 구현 자체가 어려웠고, 더 중요하게는 **플레이어가 AI가 무엇을 하는지 이해하게 만드는 것이 어려웠습니다.** 인과가 매우 명확하게 전달되어야 한다는 이론이 확립되자, 필요한 것은 **더 멍청하고 조작 가능한 AI**라는 결론이 명확해졌습니다. 창업자는 이 결정이 프로그래머 수개월분의 작업을 잘라냈다고 진술합니다.

같은 원리가 조명에도 적용됐습니다. 아티스트의 자연스러운 본능은 부분적으로 그림자에 걸친 상태를 표현하는 것이지만, Klei는 **이진 조명**을 택했습니다 — 한 발이라도 빛에 있으면 보이고, 아니면 안 보입니다. **플레이어가 항상 자신이 그림자 안인지 밖인지 알 수 있게** 하기 위해서입니다.

**넷째, 잠입 AI에는 감각 모델과 지식 모델이라는 두 축이 있습니다.**

Damián Isla는 2013년 시점의 장르를 이렇게 진단합니다.

> "Many stealth games feature sophisticated sensory models for vision (including the effect of lightness and darkness, movement speed etc.), and audition, such as in (Leonard 2003), but few feature sophisticated knowledge models for position tracking. Hence once a player has slipped away, there are only two options for a searching NPC: search the environment randomly (perhaps using heuristic information such last known movement vector) or return to its patrols."
> — Damián Isla, AIIDE 2013, p.206

**감각 모델**(무엇을 지각하는가)과 **지식 모델**(지각한 것으로부터 무엇을 아는가)의 분리는 잠입 설계를 문서화할 때 유용한 축입니다. 이 문서의 3-1·3-2는 감각 모델을, 3-3·3-4는 지식 모델을 다룹니다. Isla의 진단은 **2013년 시점의 평가**이며, 그는 이를 "구조적 결함"이 아니라 "limited options"라 표현하고 「Reinventing Stealth」라는 제하에 **긍정적 기회**로 프레이밍합니다.

**다섯째, 흥미로운 구간은 무지도 발각도 아닌 그 사이의 전이입니다.**

Blacklist의 애니메이션 디렉터는 강연에서 전투 전 애니메이션과 전투 애니메이션은 이미 충분히 좋았고, 자신이 집착한 것은 **그 중간의 전이**였다고 진술합니다. 그가 든 비유는 정확합니다 — 동생과 장난칠 때 **재미있는 부분은 동생이 겁먹는 순간이지 동생이 엄마에게 이르는 순간이 아니라는** 것입니다. 잠입 게임의 페이로드는 알람이 아니라 **의심의 램프 구간**입니다.

---

# 2. 유형별 분류

잠입 장르는 두 개의 축으로 분화합니다. **(A) 발각이 실패인가 국면 전환인가**, **(B) 은폐가 물리적인가 사회적인가.** 아래 분류는 이 두 축의 조합으로 읽으면 정리됩니다.

| 유형 | 발각의 의미 | 은폐의 성격 | 대표작 |
|---|---|---|---|
| **순수 잠입** | 대체로 실패에 근접 | 물리적(빛·그림자·소리) | Thief |
| **액션 하이브리드** | 국면 전환(교전 가능) | 물리적 + 도구 | Metal Gear Solid, Splinter Cell |
| **2D 잠입** | 국면 전환(회복 가능) | 물리적(완전 가시 정보) | Mark of the Ninja |
| **샌드박스 암살** | 부분 실패(사회적 지위 상실) | **사회적**(변장·역할) | Hitman |
| **이머시브 심 잠입** | 국면 전환(다중 해법) | 물리적 + 초자연 도구 | Dishonored |
| **공포 잠입** | 대체로 즉사 | 물리적(회피만 가능) | Alien: Isolation, Outlast |
| **오픈월드 내 잠입** | 국면 전환(전투로 폴백) | 선택적 | Assassin's Creed |

## 2-1. 순수 잠입 (Thief 계열)

### 메커니즘

- **빛과 그림자가 1차 자원**: 플레이어의 노출도가 조명 상태의 연속 함수
- **소리가 2차 자원**: 바닥 재질별 발소리, 이동 속도와 소음의 결합
- **전투의 의도적 열등화**: 정면 교전이 가능은 하나 명백히 불리하게 설계
- **1인칭 시점**: 플레이어의 관측 범위가 캐릭터의 관측 범위와 일치 — 어깨 너머로 모퉁이를 볼 수 없음

Thief의 감각 시스템은 이 유형의 원형입니다. Leonard의 서술에 따르면 Thief는 단일 2D 시야 대신 **정렬된 3D 뷰콘 집합**을 사용합니다.

> "Rather than having a single two-dimensional field of view, the Thief senses implement a set of ordered three-dimensional viewcones described as an XY angle, a Z angle, a length, a set of parameters describing both general acuity and sensitivity to types of stimuli (e.g., motion versus light), and relevance given the alertness of the AI."
> — Tom Leonard, GDC 2003

각 뷰콘은 **XY각·Z각·길이·감도(자극 종류별 민감도)·AI 경계도별 관련성**을 갖습니다. 여기서 중요한 것은 마지막 항목입니다 — **뷰콘의 관련성이 AI의 경계 상태에 따라 달라집니다.** 즉 경계 중인 경비병과 무심한 경비병은 같은 시야 형상을 다르게 사용합니다. 그리고 판정은 **대상이 속한 첫 번째 뷰콘만** 평가합니다("only the first view cone the object is in is considered in sense calculations").

### 특징 및 차별점

- **감각 모델의 해상도가 곧 게임의 해상도**: 조명·이동·노출이 0..1 아날로그로 집계되어 최종 감각값을 형성
- **소리가 3D 지오메트리를 통해 전파**: Leonard는 Thief의 사운드 시스템이 렌더링되는 소리와 되지 않는 소리 모두에 **의미론적 데이터를 태깅해 월드의 3D 지오메트리를 통해 전파**시키며, 소리가 AI에 '도착'할 때 **현실 세계에서 와야 할 방향으로부터** 감쇠된 인지값과 함께 도착한다고 서술합니다
- **자유 지식(free knowledge)**: 관심 대상이 더 이상 감각 펄스를 생성하지 않을 때, AI의 상태에 따라 스케일된 지식이 주어집니다 — Leonard는 이것이 **대상이 시야를 벗어났을 때 AI가 추론하는 것처럼 보이게 만들되 플레이어에게 노골적인 치팅으로 보이지 않게** 하기 위한 것이라고 명시합니다

### 설계 고려사항

- 1인칭 시점은 몰입을 극대화하지만 **관측 비용이 매우 높습니다** — 모퉁이 너머를 보려면 실제로 몸을 내밀어야 하고, 이는 노출을 의미합니다. 계획 단계의 정보 수집 자체가 위험이 되므로 난이도가 급격히 상승합니다
- 빛/그림자를 1차 자원으로 삼으면 **레벨의 조명이 곧 레벨 디자인**이 됩니다. 아트 디렉션과 게임플레이 튜닝이 같은 파라미터를 공유하므로 조직적 마찰이 발생합니다
- 전투를 열등하게 만들되 **불가능하게 만들지는 않는** 균형이 필요합니다. 완전히 불가능하면 공포 잠입(2-6)이 되고, 충분히 유효하면 액션 하이브리드(2-2)가 됩니다

### 대표 사례

- 《Thief: The Dark Project》(1998), 《Thief II》(2000)
- 《Thief》(2014)

## 2-2. 액션 하이브리드 (Metal Gear Solid / Splinter Cell 계열)

### 메커니즘

- **잠입과 교전의 양방향 전이**: 발각되어도 게임이 끝나지 않고, 교전에서 다시 잠입으로 복귀 가능
- **탐지 미터**: AI의 인지 진행도를 연속 게이지로 노출 (3-5에서 상술)
- **도구 기반 조작**: 소음 유발 도구, 시야 차단 도구로 AI를 능동 조작
- **테이크다운**: 무지 상태의 적을 즉시 무력화하는 고보상 행동

### 특징 및 차별점

이 유형의 결정적 특징은 **테이크다운의 존재가 다른 모든 파라미터를 구속한다**는 것입니다. Blacklist 팀이 청각 공정성 문제를 해결하려 할 때 가장 단순한 대안 — 오디오 이벤트 거리를 전역 하향 — 을 기각한 이유가 정확히 이것입니다.

> "it would actually break the game since we had 'insta-kill takedowns' if you got the jump on an NPC; so a player could just sprint through the map stabbing everyone in the back."
> — Martin Walsh, Game AI Pro 2 ch.28, p.321

**즉살 테이크다운이 있으면 청각을 관대하게 만들 수 없습니다.** 플레이어가 맵을 질주하며 모두를 등 뒤에서 찌르는 게임이 되기 때문입니다. Walsh는 기각 사유로 두 가지를 들었는데(경비병이 무반응해 보인다는 점과 테이크다운 붕괴), 여기서는 후자만 인용합니다. 이는 **잠입 설계에서 파라미터들이 독립적이지 않다**는 것을 보여 주는 가장 명확한 실례입니다.

Blacklist의 애니메이션 디렉터는 강연에서 **같은 테이크다운 동작이 AI의 인지 상태에 따라 자동으로 다른 변형으로 호출**된다고 진술합니다 — AI가 플레이어를 인지한 상태면 '위협 제압' 성격의 변형이, 인지하지 못한 상태면 '소리 억제' 성격의 변형(대상을 엄폐물 안으로 끌어들여 숨기는)이 재생됩니다. 동일한 입력과 동일한 판정 기준인데 **AI의 인지 상태 하나만이 연출을 가릅니다.** 이는 인지 상태를 UI 미터뿐 아니라 **애니메이션 자체로도 노출하는** 설계입니다.

### 설계 고려사항

- **양방향 전이의 비용을 정직하게 계산해야 합니다.** 발각 후 잠입 복귀를 허용하려면 AI에게 '포기' 조건이 필요하고(3-3), 이는 곧 AI가 플레이어를 잊는 규칙을 설계해야 한다는 뜻입니다
- 교전이 유효하면 **잠입은 선택지가 되고, 선택지는 최적화 대상**이 됩니다. 잠입이 교전보다 명백히 느리고 어렵다면 플레이어는 잠입을 버립니다. 잠입에 별도의 보상 축(점수·평가·해금)이 필요한 이유입니다
- 도구 기반 AI 조작은 **AI가 조작에 반응한다는 것을 플레이어가 신뢰할 수 있을 때만** 성립합니다. 반응이 확률적이면 도구는 도박이 됩니다

### 대표 사례

- 《Metal Gear Solid》 시리즈
- 《Splinter Cell》 시리즈 — Conviction(2010), Blacklist(2013)

## 2-3. 2D 잠입 (Mark of the Ninja)

### 메커니즘

- **평면화된 공간**: 3D의 깊이를 제거해 이동 가능 영역을 완전히 명시
- **정보의 완전 노출**: 시야 원뿔·소음 반경·AI 상태 아이콘을 모두 화면에 렌더링
- **이진 조명**: 빛 안이면 보이고 밖이면 안 보임 — 중간 상태 없음
- **의도적으로 조작 가능한 AI**

### 특징 및 차별점

Klei의 창업자는 강연에서 레벨 지오메트리를 **완전히 평평하게** 유지하기로 한 결정을 설명합니다. 아티스트들은 이전작처럼 플랫폼에 깊이를 주고 싶어 했지만, 핵심은 **플레이어가 "저 천장 부분에 갈 수 있다, 이 벽에 갈 수 있다"를 알게 만드는 것**이었고, 그래서 완전히 평평하게 두었습니다. 그는 이를 **"플레이어에게 쉬울수록 게임에 좋다"** 는 이론을 테스트한 결과라고 설명합니다.

소음 반경 표시를 둘러싼 논쟁은 이 유형의 철학을 압축합니다. 그가 소음 링을 보여 주자고 제안했을 때 팀 내에서 **"게임이 너무 쉬워지지 않나? 플레이어가 거리를 직접 가늠해야 하지 않나?"** 라는 이견이 나왔습니다. 그의 답은 **아니오** 였습니다 — 숨기면 게임이 '뭉개지고(mushy)', **몇 수 앞을 계획할 수 있는지가 불명확해지기** 때문입니다. 명확하게 만들면 플레이어는 **더 멀리 계획할 수 있고 더 높은 수준의 계획**을 할 수 있게 됩니다.

이것이 2D 잠입의 핵심 통찰입니다. **정보를 숨기는 것은 난이도를 만드는 것이 아니라 계획의 지평을 줄이는 것입니다.** 계획이 장르의 페이로드라면, 정보를 숨기는 것은 페이로드를 줄이는 것입니다.

### 설계 고려사항

- 정보를 완전히 노출하면 **난이도를 다른 축에서 만들어야 합니다** — 공간 구성의 복잡도, 자극 간의 상호작용, 시간 압박
- 2D 평면은 **관측 비용을 0에 가깝게** 만듭니다. 이는 계획을 게임의 중심에 놓을 수 있게 하지만, 동시에 '몰래 훔쳐본다'는 판타지의 일부를 포기하는 것입니다
- 이진 조명은 아트 디렉션과 충돌합니다. **명료함을 위해 시각적 자연스러움을 포기하는 결정**을 조직이 지지해야 합니다

### 대표 사례

- 《Mark of the Ninja》(2012)
- 후속 2D 잠입 인디 다수

## 2-4. 샌드박스 암살 (Hitman)

### 메커니즘

- **사회적 잠입**: 물리적 은폐가 아니라 **역할 수행**으로 접근 권한 획득
- **변장 시스템**: 복장이 곧 통행증이며, 일부 NPC만이 변장을 간파
- **다중 해법**: 하나의 목표에 대해 다수의 접근 경로
- **레벨 = 사회적 공간**: 공간마다 다른 행동 규칙이 부여됨

### 특징 및 차별점

이 유형은 다른 모든 유형과 **은폐의 성격 자체가 다릅니다.** 다른 잠입 게임에서 플레이어는 보이지 않으려 하지만, 이 유형에서 플레이어는 **보이되 올바른 것으로 보이려** 합니다. Io-Interactive의 레벨 디자이너는 강연에서 이를 설명하며, 대규모 공개 행사의 백스테이지에 들어갈 수 없지만 **행사의 마스코트가 되면 갈 수 있다**는 예를 듭니다.

이 유형의 설계 도구는 사회학·인류학에서 옵니다. 강연자는 사회적 자본과 **사회적 공간(social space)** 개념, 그리고 **프론트스테이지/백스테이지** 개념을 레벨 디자인에 도입했다고 설명합니다. 핵심 통찰은 **공간이 암묵적 행동 규칙을 운반한다**는 것입니다 — 사람들은 교회에 들어가면서 "이제 교회니까 이렇게 행동해야지"라고 생각하지 않고, **생각 없이 그냥 그렇게 합니다.** 그리고 그것이 매우 강력합니다. 이 관점의 상세는 5-4에서 다룹니다.

### 설계 고려사항

- 사회적 잠입은 **NPC 밀도를 자원으로 전환**합니다. 물리적 잠입에서 NPC는 위협이지만, 사회적 잠입에서 군중은 **은폐물**입니다
- 변장 시스템은 **간파 규칙이 읽히지 않으면 즉시 붕괴**합니다. 어떤 NPC가 왜 나를 간파하는지 플레이어가 예측할 수 없으면, 변장은 도박이 됩니다
- 다중 해법은 **가이드 설계를 강제**합니다 — 5-4에서 상술하듯, 가이드를 넣는 순간 그 가이드의 실패 조건을 설계해야 합니다

### 대표 사례

- 《Hitman: Blood Money》(2006)
- 《Hitman》(2016), 《Hitman 2》(2018)

## 2-5. 이머시브 심 잠입 (Dishonored 계열)

### 메커니즘

- **시스템 상호작용**: 도구·능력·환경이 조합 가능한 규칙으로 구성
- **초자연 이동 능력**: 순간이동·시간 정지 등이 잠입의 이동 어휘를 확장
- **살상/비살상의 이중 경로**: 두 경로 모두 완주 가능하도록 병렬 설계
- **결과 반영**: 플레이어의 살상 정도가 후속 콘텐츠에 반영

### 특징 및 차별점

이머시브 심 잠입은 **잠입을 시스템 조합의 한 결과로 다룹니다.** 순수 잠입이 감각 모델의 해상도로 깊이를 만든다면, 이 유형은 **도구 간 상호작용의 조합 폭**으로 깊이를 만듭니다. 결과적으로 같은 상황에 대한 해법이 설계자가 예상하지 않은 것까지 포함하게 됩니다.

> **⚠️ 이 절의 근거 상태**: **이 유형에 대한 개발사 1차 출처를 확보하지 못했습니다.** 아래 서술은 구조적·정성적 관찰이며 **수치를 싣지 않았습니다.** 이 장르에 관해 널리 참조되는 Dishonored 2의 레벨 디자인 강연은 **URL과 제목만 확인**했고 내용을 확인하지 못했습니다(부록 참조).

### 설계 고려사항

- 조합 폭이 넓어지면 **AI가 대응해야 할 자극의 종류가 폭발**합니다. 3-4의 자극 중재가 이 유형에서 특히 중요해집니다
- 초자연 이동 능력은 **AI가 도달할 수 없는 위치**를 만듭니다. Blacklist가 '도달 불가 지역' 문제를 별도 시스템으로 처리한 것과 같은 대응이 필요합니다(5-1)
- 비살상 경로를 병렬로 지원하려면 **모든 위협에 대해 비살상 해법이 존재**해야 합니다. 하나라도 빠지면 비살상 플레이는 그 지점에서 무너집니다

## 2-6. 공포 잠입 (Alien: Isolation / Outlast 계열)

### 메커니즘

- **전투의 완전 제거 또는 극단적 열등화**: 회피가 유일하게 유효한 선택
- **추적자의 비결정성**: 순찰 패턴이 아니라 반응적 추적
- **자원 희소성**: 은폐 도구·회복 수단의 극단적 제한
- **발각 = 대체로 즉사**

### 특징 및 차별점

공포 잠입은 **1-1에서 규정한 예측 가능성 원칙을 의도적으로 위반**하는 유형입니다. Walsh의 원칙 — AI는 예측 가능해야 하고, 그래야 플레이어가 전략을 세운다 — 을 뒤집어, **예측 불가능성 자체를 정서적 페이로드**로 삼습니다.

이것이 성립하는 이유는 **목표가 다르기 때문**입니다. 순수 잠입의 목표는 계획의 만족감이지만, 공포 잠입의 목표는 통제력 상실의 정서입니다. 계획이 가능하면 공포가 아닙니다. 다만 이 교환에는 대가가 있습니다 — **플레이어가 실력으로 개선할 수 있는 여지가 줄어듭니다.**

> **⚠️ 이 절의 근거 상태**: **공포 잠입에 대한 개발사 1차 출처를 확보하지 못했습니다.** 위 서술은 확보된 다른 출처의 원칙(Walsh의 예측 가능성 요건)을 **대조축으로 삼은 구조적 분석**이며, 특정 게임의 구현이나 개발사의 표명된 의도로 귀속하지 않습니다. **수치를 싣지 않았습니다.**

### 설계 고려사항

- 예측 불가능성을 도입하되 **불공정성과 구분**해야 합니다. 이 구분선은 Walsh의 기준으로 판정 가능합니다 — 플레이어가 사후에 "아, 저래서 걸렸구나"를 이해할 수 있는가
- 즉사를 채택하면 **리트라이 비용이 게임의 리듬을 지배**합니다. 4-3의 세이브/리트라이 설계가 다른 어떤 유형보다 중요해집니다
- 자원 희소성은 **플레이어가 자원을 쓰지 않게** 만듭니다. 아껴 두고 안 쓰는 도구는 없는 도구와 같습니다

## 2-7. 오픈월드 내 잠입 (Assassin's Creed 계열)

### 메커니즘

- **잠입이 선택적 접근법**: 전투로 폴백 가능
- **군중 은폐**: 사회적 잠입의 경량 버전
- **동기화/경계 상태**: 지역 단위의 경계도 관리
- **수직 이동**: 지붕·건물이 별도의 이동 레이어를 형성

### 특징 및 차별점

이 유형에서 잠입은 **장르가 아니라 기능(feature)** 입니다. 게임의 핵심 루프는 다른 곳에 있고, 잠입은 그 루프에 얹힌 선택지입니다. 이는 두 가지를 함의합니다.

**첫째, 잠입 시스템의 깊이가 의도적으로 제한됩니다.** 감각 모델이 정교하면 잠입을 선택하지 않는 플레이어에게는 비용만 발생합니다.

**둘째, 발각의 처벌이 약해야 합니다.** 잠입이 선택지라면, 잠입 실패가 게임을 멈추게 해서는 안 됩니다. 그래서 이 유형은 거의 항상 **발각 후 전투 폴백**을 제공합니다.

### 설계 고려사항

- 전투 폴백이 존재하면 **잠입은 자발적 제약이 됩니다.** 이 경우 잠입에 대한 보상은 외적(점수·업적)이거나 내적(자기 부과 챌린지)이어야 합니다
- 오픈월드의 넓은 공간은 **초크포인트가 희소**합니다. 3-2의 경로 기반 소음 모델이 실내 환경을 전제한다는 점(Walsh가 명시)에 유의해야 합니다
- 지역 단위 경계도는 **개별 NPC의 인지 상태보다 거친 해상도**입니다. 두 층위가 충돌하면 플레이어는 규칙을 읽지 못합니다

---

# 3. 핵심 메커니즘

이 장은 이 문서의 핵심 자산입니다. 잠입 장르는 이 리뷰 시리즈에서 **구현 수준 1차 출처가 가장 강한** 장르이며, 아래 서술의 대부분은 **해당 시스템을 직접 구현한 엔지니어가 자신이 만든 것을 서술한 기록**에 근거합니다.

## 3-1. 탐지 모델: 시야 형상, 본 레이캐스트, 자세

### 메커니즘

**시야 원뿔은 픽션입니다.** 이것이 이 절의 결론이며, 잠입 설계에서 가장 널리 퍼진 오해를 정정하는 지점입니다.

원뿔은 NPC의 정면을 모델링하는 데는 좋지만 다른 많은 측면에서는 부실합니다. Walsh가 드는 두 가지 대표적 실패는 **주변시**와 **원거리 시야**입니다. 주변시가 왜 원뿔로 안 되는지는 자명하지만, 원거리 시야는 조금 더 깊이 들어가야 합니다.

> "the reason a cone doesn't model vision at a distance well is that a cone expands as it moves away from the NPC, which implies that as things get further away from the NPC in the direction he is looking, he tends to perceive, and become aware of, things that are laterally farther away. Up to a certain distance that is plausible, but past that distance, it doesn't provide reasonable behavior. If the player is far enough away and off to one side, we don't want the NPCs to become aware of them at all"
> — Martin Walsh, Game AI Pro 2 ch.28, p.316

**원뿔은 거리에 따라 계속 옆으로 넓어집니다.** 그래서 멀리 있고 옆으로 벗어난 플레이어까지 NPC가 인지하게 됩니다. 일정 거리까지는 개연적이지만 그 이상은 아닙니다.

Splinter Cell 계보는 이 문제를 두 단계로 개선했습니다.

> "This insight caused us to first replace cones with boxes for vision at a distance; this is the solution we used on Splinter Cell Conviction. On Blacklist, we refined it further and replaced our standard boxes with coffin-shaped boxes (Figure 28.3) that expand up to a point like a cone and then start to contract as they continue to move further away from the NPC, which gives us the effect we want."
> — Martin Walsh, Game AI Pro 2 ch.28, p.316

**원뿔 → 박스(Conviction, 2010) → 관(coffin) 모양 박스(Blacklist, 2013)** 의 계보입니다. 관 모양 박스는 **일정 지점까지는 원뿔처럼 확장하다가 그 이후로는 다시 수축**합니다.

**결정적 단서: Blacklist는 원뿔을 폐기하지 않았습니다.** Figure 28.3(출시 빌드의 디버그 드로잉)의 라벨은 **Box / Cone / Peripheral** 세 가지이며, 캡션은 관 모양 박스와 함께 **"NPC의 정면에서 플레이어가 빠르게 탐지되어야 하는 영역을 정의하는 표준 시야 원뿔"** 이 보인다고 명시합니다. 즉 Blacklist는 **원거리 = 관 모양 박스, 정면 근거리 속결 = 표준 원뿔, 주변시 = 별도 형상**의 3중 병용입니다. 그리고 중재 규칙은 다음과 같습니다.

> "The most inclusive shape the player is in defines the range of the detection timer."
> — Figure 28.3 캡션

**플레이어가 속한 가장 포괄적인 형상이 탐지 타이머의 구간을 결정합니다.**

15년 앞선 Thief는 이미 다른 경로로 같은 지점에 도달했습니다 — 단일 2D 시야 대신 **정렬된 3D 뷰콘 집합**을 쓰고, 대상이 속한 첫 번째 뷰콘만 평가하며, 각 뷰콘은 콘 내 위치와 무관하게 **상수 출력**을 냅니다. Leonard가 든 예시 AI는 **뷰콘 5개**를 갖습니다. 두 출처가 15년 간격으로 도달한 공통 결론은 이것입니다 — **하나의 형상으로 시각을 모델링하려는 시도는 실패하며, 해법은 역할이 다른 여러 형상의 집합과 그들 사이의 중재 규칙입니다.**

### 탐지 파이프라인

Blacklist의 탐지는 다음 순서로 진행됩니다.

**1단계 — 본 레이캐스트.**

> "On Blacklist, we raycast to eight different bones on the player's body. Depending on the stance, it takes a certain number of visible bones to kick off detection. In this case, the player stance is 'in cover' that requires more than the two bones that are currently visible so detection has not yet begun."
> — Figure 28.1 캡션, p.315

플레이어 신체의 **본 8개**로 레이캐스트를 쏘고, **자세(stance)가 탐지 개시에 필요한 가시 본 개수를 결정**합니다. 캡션의 예시에서 플레이어의 자세는 '엄폐 중'이며, 현재 보이는 본 2개보다 더 많은 본이 필요하므로 탐지가 아직 시작되지 않았습니다.

이것은 **부분 노출을 이진 판정으로 무너뜨리지 않는** 설계입니다. 단일 지점 레이캐스트는 "보인다/안 보인다"만 답하지만, 8개 본은 **얼마나 보이는가**를 답하고, 자세는 그 값에 대한 **임계값을 캐릭터 상태에 연동**시킵니다. 결과적으로 "웅크리면 덜 보인다"가 시각적 은유가 아니라 실제 판정 규칙이 됩니다.

**2단계 — 타이머.**

> "Once enough bones are visible to the NPC, the detection process starts and a timer kicks off. The full range of the timer is defined by the detection shape the player is in (Figure 28.3), and the actual value used is arrived at by scaling linearly inside that range based on the current distance to the player. The timer is represented by the growing HUD element that provides feedback to the player that defines his window of opportunity to break LOS or kill the NPC to avoid detection. Once the detection HUD is full, the player is detected."
> — Figure 28.2 캡션, p.315

타이머의 **전체 구간은 플레이어가 속한 탐지 형상이 정의**하고, **실제 값은 현재 거리로 그 구간 내에서 선형 스케일**됩니다. 즉 형상이 '범위'를, 거리가 '그 범위 안의 위치'를 정합니다.

**⚠️ 필수 한정 (1): 거리만이 타이머 변수가 아닙니다.** 본문 p.316은 다르게 서술합니다 — "a timer kicks off, which is scaled based on the distance to the player, lightness of the player, NPC state, etc." **거리·플레이어의 밝기·NPC 상태 등**이 함께 들어갑니다. Figure 28.2 캡션은 거리만 언급하지만 이는 캡션의 축약이며, **"거리 단독"으로 서술하면 안 됩니다.**

**3단계 — HUD 노출.** 타이머는 **성장하는 HUD 요소**로 렌더링되어, 플레이어에게 **LOS를 끊거나 NPC를 처치해 탐지를 피할 '기회의 창(window of opportunity)'** 을 명시합니다.

**4단계 — 확정.** HUD가 가득 차면 탐지 확정입니다.

**⚠️ 필수 한정 (2): 탐지는 '가득 참'에서 이진적이지 않습니다.** p.314에 따르면 Splinter Cell을 포함한 일부 게임에는 **제3의 중간 상태**가 있습니다 — NPC가 무언가를 봤다는 것은 알고 조사하러 가는 상태이며, **보통 진행 바의 어떤 퍼센티지 지점**에서 발생합니다. 즉 진행 바는 단순한 이진 게이트가 아니라 **중간 분기점을 가진 연속 축**입니다.

**⚠️ 필수 한정 (3): 챕터 내 표현 불일치.** Figure 28.2는 타이머가 "채워진다"고 하고 p.316은 "when that timer reaches 0 (or the progress bar is full)"이라 씁니다. 이는 저자의 느슨한 서술이며 모순이 아닙니다.

### 특징 및 차별점

Mark of the Ninja의 시야 판정은 **(최소) 3중 논리곱**입니다.

> "Detection by the sight test involves a series of checks including these questions: Is the interest source within one of the agent's vision cones? Is the game object associated with the interest source currently lit by a light source, or does the agent have the night vision flag, which removes this requirement? Is there any collision blocking line of sight between the agent's eye position and that of the interest source?"
> — Brook Miles, Game AI Pro ch.32, p.415

**시야 형상 포함 여부 ∧ 조명(광원에 비춰졌는가, 단 night vision 플래그면 면제) ∧ 시선 차폐 없음.** 그리고 형상은 원뿔에 한정되지 않습니다 — "we have some which are simply a single ray, an entire circle, or a square or trapezoid for specific purposes"(단일 레이, 전체 원, 사각형, 사다리꼴).

**⚠️ "최소 3중"으로 표기해야 하는 이유**: 원문은 "including these questions"라는 **비망라적** 표현을 씁니다. 실제로 최대 반경 사전 필터와 에이전트별 조건이 추가로 존재합니다.

**⚠️ 수치에 관한 경고**: ch.32에는 **각도·거리의 구체 수치가 전무합니다.** 문서 전체에 도(degree) 표기가 0건이며, `RUN_ON_LOUD_RADIUS`·`INTEREST_RADIUS_SUSPECT` 같은 **미해석 심볼 상수**만 있습니다. 이 출처에서 시야각 수치를 인용하는 것은 불가능합니다.

**시야 형상 설계 비교**

| 게임 | 형상 구성 | 중재 규칙 | 성격 |
|---|---|---|---|
| **Thief**(1998) | 정렬된 3D 뷰콘 집합(예시 AI는 5개), XY각·Z각·길이·감도·경계도별 관련성 | **첫 번째** 뷰콘만 평가 | 각 콘은 위치 무관 **상수 출력** — 명시적 단순화 |
| **Splinter Cell: Conviction**(2010) | 원거리를 **박스**로 | (문서화되지 않음) | 원뿔의 측면 확장 문제 1차 대응 |
| **Splinter Cell: Blacklist**(2013) | **관 모양 박스 + 표준 원뿔 + 주변시** 3중 | **가장 포괄적인** 형상이 타이머 구간 결정 | 관 = 확장 후 수축 |
| **Mark of the Ninja**(2012) | 원뿔 + 단일 레이·원·사각형·사다리꼴 | (문서화되지 않음; 논리곱의 한 항) | 조명·차폐와의 **논리곱** |

### 설계 고려사항

- **완벽한 해법은 없다는 것을 설계에 반영해야 합니다.** Walsh는 자신의 해법의 실패를 명시적으로 인정합니다: 모든 변형에는 임계값(시야 형상의 가장자리)이 있고, **"플레이어가 그 임계값 밖 1cm에 있으면 NPC는 영원히 서서 플레이어를 못 보고, 1cm 안에 있으면 길어야 몇 초 안에 탐지됩니다."** 그는 이것이 **모델을 추상화한 방식과 NPC 상태를 플레이어에게 표시하는 피드백에 맞춰야 하는 필요**의 직접적 결과라고 진단합니다. 즉 **HUD 미터를 노출하기로 한 결정 자체가 임계값의 경직성을 강제**합니다
- **형상은 시각 모델에서 유도되지 않습니다.** Blacklist의 해법은 "many hours of playtesting and designer tweaking"의 산물입니다. 물리적 시야를 시뮬레이션하려는 접근은 이 계보의 어느 지점에도 없습니다
- **8개 본은 Blacklist의 값이지 표준이 아닙니다.** 중요한 것은 개수가 아니라 **부분 노출을 연속값으로 다루고 임계값을 자세에 연동한다**는 구조입니다
- 자세별 임계값을 도입하면 **자세가 이동 속도·소음과도 결합**되어야 합니다. 웅크리면 덜 보이지만 느리다는 교환이 없으면 항상 웅크리는 것이 지배 전략이 됩니다

## 3-2. 소음 전파: 반경이 아니라 경로

### 메커니즘

**두 개의 독립된 1차 출처가 동일한 원리에 도달합니다** — 소음 가청 판정은 유클리드 반경 감쇠가 아니라 **공간 위상을 따르는 '경로'** 로 해결해야 합니다. 서로 다른 스튜디오가, 서로 다른 차원(3D와 2D)에서, 서로 다른 구현으로, 같은 결론에 도달했다는 점이 이 원리의 강도를 보증합니다.

**Blacklist: 초크포인트 거리 합산.**

출발점은 자명합니다.

> "Using Euclidian distance is obviously not good enough since that would mean the player would be heard through walls."
> — Martin Walsh, Game AI Pro 2 ch.28, §28.4.1

그렇다면 사운드 엔진을 쓰면 되지 않을까요? 팀은 그렇게 해 봤고, 실패했습니다.

> "Initially, we used our sound engine, but it was not optimized for calculating arbitrary sound distance, and some calls were insanely expensive. A second issue with using the sound engine was that if the audio data weren't built (which happened often), it created all kinds of false detection bugs, which made testing difficult."
> — Martin Walsh, Game AI Pro 2 ch.28, §28.4.1

두 가지 실패입니다 — **성능**(임의 소리 거리 계산에 최적화되어 있지 않아 일부 호출이 극도로 비쌈)과 **개발 프로세스**(오디오 데이터가 빌드되지 않으면 — 자주 있는 일 — 대량의 오탐 버그가 발생해 테스트가 어려워짐). 두 번째 문제는 특히 시사적입니다: **게임플레이 판정을 오디오 파이프라인에 의존시키면 오디오 빌드 상태가 게임플레이 QA를 차단합니다.**

해법은 TEAS(Tactical Environment Awareness System)를 재사용하는 것이었습니다. TEAS는 원래 다른 목적(전투 중 NPC가 벽을 응시하는 문제 해결)으로 만든 **연결성 그래프**로, 월드를 영역(subnavmesh)으로 나누고 그 사이를 **초크 노드**로 연결합니다.

> "To calculate sound distance, we would get the 'area path' from the source (event location) to the destination (NPC location)."
> — Martin Walsh, Game AI Pro 2 ch.28, §28.4.1

식 28.1은 **소스 → A1으로 이어지는 최근접 초크 → A2로 이어지는 초크 → … → 목적지**의 거리 합산입니다. Walsh가 든 워크드 예시: 소리가 영역 A에서 나고 NPC가 영역 D에 있으며 A는 window1로 B에, B는 door2/window2로 C에, C는 door3로 D에 연결되어 있고 window1이 window2보다 door2에 가깝다면, 총 거리는 `dist(source, window1) + dist(window1, door2) + dist(door2, door3) + dist(door3, destination)`입니다.

같은 방 안이라면 유클리드로 폴백합니다("In the trivial case where the source and destination are in the same room, we just used Euclidian distance"). 그리고 개발자의 자기 평가는 이례적으로 솔직합니다.

> "Although this is clearly a rough approximation, in practice, the results were accurate enough that we made the switch and never looked back."
> — Martin Walsh, Game AI Pro 2 ch.28, §28.4.1

**명백한 대략적 근사이지만 실무상 충분히 정확했고, 바꾼 뒤 다시는 뒤돌아보지 않았습니다.**

**Mark of the Ninja: 사운드 메시 A*.**

> "Detection by sound is determined by pathfinding from the interest source's position to the position of the agent's head."
> — Brook Miles, Game AI Pro ch.32, p.416

**주목할 대비**: 시야 판정은 **에이전트의 눈 위치**를 쓰고("the agent's eye position"), 소리 판정은 **에이전트의 머리 위치**를 씁니다. 사소해 보이지만 두 감각이 서로 다른 앵커를 갖는다는 것은 감각을 **각자의 물리에 맞춰 모델링**했다는 신호입니다.

메시는 레벨 익스포트 시점에 생성됩니다.

> "When a designer exports a level from our level editor, one of the steps is to generate a triangle mesh from all of the empty space within the level... We use this 'sound mesh' at runtime to perform A* pathfinding [Millington 06] from the 'sound' interest source to any agent who might potentially hear it."
> — Brook Miles, Game AI Pro ch.32, p.416

레벨의 **모든 빈 공간**에서 삼각형 메시를 생성하고, 런타임에 소리 소스에서 에이전트까지 **A\* 경로 탐색**으로 가청을 판정합니다. 사운드 메시와 맵 메시 모두 **Jonathan Shewchuk의 Triangle 생성기**로 만들었습니다. 그리고 같은 경로 탐색이 **오디오 시스템에서 필터링 양을 결정하는 데도** 사용됩니다 — 즉 게임플레이 판정과 오디오 렌더링이 **같은 위상 정보를 공유**하되, Blacklist와 달리 게임플레이가 오디오 파이프라인에 종속되지는 않습니다.

핵심 귀결은 Figure 32.1의 캡션에 있습니다.

> "Guard B is within the Sound Radius but can't hear the sound because there is no path from the guard to the sound source. Guard C is outside the Sound Radius and can't hear the sound; however, if the player enters Guard C's vision cone, the player will be spotted."
> — Figure 32.1 캡션, p.415

**반경 안에 있어도 경로가 없으면 듣지 못합니다.**

### 특징 및 차별점

**반경의 역할이 재정의됩니다.** 반경은 가청을 결정하지 않습니다. 반경은 **어떤 에이전트를 검사할지 추리는 필터**입니다.

> "For both performance and gameplay reasons, sight and sound interest sources define a maximum radius, and only agents within that radius are tested to determine whether they can detect the interest source."
> — Brook Miles, Game AI Pro ch.32, p.416

**⚠️ 한정: 반경은 성능 전용이 아닙니다.** 원문은 "For both performance **and gameplay** reasons"라고 명시하며, Listing 32.1에는 소스별 튜닝값(`RUN_ON_LOUD_RADIUS` 등)이 존재합니다. 즉 반경은 **성능 필터이자 동시에 디자이너의 튜닝 노브**입니다.

**⚠️ 확대 해석 금지**: ch.32는 A* **'경로 길이'로 감쇠하는지 여부를 명시하지 않습니다.** 명시된 메커니즘은 **경로의 '존재'** 뿐입니다. 경로 길이 감쇠 모델로 확대 해석해서는 안 됩니다.

**두 구현의 대조**

| 축 | Splinter Cell: Blacklist | Mark of the Ninja |
|---|---|---|
| **공간 표현** | TEAS 영역 그래프(subnavmesh + 초크 노드) | 빈 공간의 삼각형 메시(Triangle 생성기) |
| **계산** | 초크포인트 간 거리 **합산**(식 28.1) | 사운드 메시 위 **A\* 경로 탐색** |
| **같은 방/직결** | 유클리드 거리로 **폴백** | (별도 폴백 미문서화) |
| **반경의 역할** | 이벤트마다 반경 + 우선순위 보유 | 후보 에이전트 **사전 필터** + 튜닝 노브 |
| **앵커** | NPC 위치 | 에이전트의 **머리**(시야는 **눈**) |
| **오디오 시스템과의 관계** | 사운드 엔진에서 **이탈**(비용·빌드 의존성) | 같은 경로 탐색을 오디오 **필터링에도 공유** |
| **자기 평가** | "명백한 대략적 근사"이나 충분히 정확 | (자기 평가 없음) |

**주목할 대비 하나 더**: Blacklist의 §28.4는 **"On Conviction, we had two major problems with our auditory events"** 로 시작합니다 — 즉 이것은 **Conviction(2010)에서 겪은 문제를 Blacklist(2013)에서 해결한 계보**입니다. 시야 형상의 원뿔→박스→관 계보와 정확히 같은 패턴이며, **잠입 시스템은 한 타이틀에서 완성되지 않고 시리즈에 걸쳐 반복 개선된다**는 것을 보여 줍니다.

### 설계 고려사항

- **초크포인트 모델은 실내를 전제합니다.** Walsh는 이를 명시합니다 — 이 모델은 **잘 정의된 영역과 초크포인트를 가진 실내 환경에 더 잘 맞고 넓게 열린 공간에는 덜 맞습니다.** 다만 실무상 그들의 실외 환경도 영역으로 분해할 수 있었으나 **영역과 초크를 정의하는 것이 실외에서는 덜 자명했습니다.** 오픈월드에 이 모델을 그대로 적용하려는 시도는 이 한계를 먼저 검토해야 합니다
- **경로 기반 모델의 진짜 이득은 레벨 디자이너에게 갑니다.** 벽·문·창이 게임플레이 규칙이 되면, 레벨 디자이너가 지오메트리를 그리는 행위가 곧 소음 튜닝이 됩니다. 반경 모델에서는 지오메트리와 소음이 분리되어 있어 레벨 디자이너가 소음을 제어할 수단이 없습니다
- **정확도가 목표가 아닙니다.** Blacklist의 초크 합산은 실제 음향과 무관합니다 — 소리는 초크포인트를 경유해 꺾이지 않습니다. 그럼에도 채택된 이유는 **플레이어의 공간 직관과 일치하기 때문**입니다(벽 뒤면 안 들린다). 이는 1-3의 제1원칙의 직접적 사례입니다
- **가청 판정을 오디오 미들웨어에 위임하지 마십시오.** Blacklist의 경험은 명확합니다 — 비용과 빌드 의존성 양쪽에서 실패합니다

## 3-3. AI 인지 상태 기계: 탐색의 분기와 포기

### 메커니즘

미인지 → 탐색 전이에서, **탐색 상태 기계는 자극의 종류로 분기합니다.**

> "Stimuli received by NPCs can be divided into two categories. Hostile stimuli, such as gunfire and explosions, will trigger an aggressive search response. Although the target isn't known, it is assumed to be hostile from the type of stimulus received. Distraction stimuli on the other hand, for example, a bottle being thrown or a prop being knocked over, will trigger a cautious search."
> — Rich Welsh, Game AI Pro 2 ch.27, §27.3.1

**적대적 자극**(총성·폭발)은 공격적 탐색을, **교란 자극**(병 던지기·소품 넘어뜨리기)은 신중한 탐색을 촉발합니다. 핵심은 **표적을 모르는 상태에서도 자극의 종류만으로 적대성을 가정**한다는 것입니다.

두 모드는 **동일한 탐색 단계를 따르되** 세 가지에서 갈립니다.

> "both the cautious and aggressive searches follow the same phases. The differences between the two come from the speed at which the character moves, the animations that they play, and the rate at which they abandon the search"
> — Rich Welsh, Game AI Pro 2 ch.27, §27.4

**표 27.1(원문 그대로)**

| 항목 | Cautious(신중) | Aggressive(공격적) |
|---|---|---|
| **이동 속도** | Walk | Run |
| **애니메이션** | Slow, glancing around | Weapon raised, more alert |
| **탐색 포기** | After initial location searched (phase 1 only) | After all locations searched or time limit reached (phases 1 and 2) |

이 표의 우아함은 **하나의 상태 기계를 세 개의 파라미터로 두 개의 인격으로 분화시킨다**는 데 있습니다. 새 상태를 추가하지 않고 **같은 단계를 다른 속도·다른 자세·다른 인내심으로** 실행하는 것만으로 "경계하는 경비병"과 "의심하는 경비병"이 구분됩니다.

**⚠️ 한정 (1): 1단계 종료는 표에서는 단정형이나 본문은 완화합니다.** §27.4.1.1은 "**Often** with a cautious search, once the initial position of the stimulus has been investigated, there is no further need to progress into the second search phase"라 씁니다. 그리고 §27.4.2는 결정적입니다 — "**this is ultimately a design decision to be made to best suit your game**." **기본 설계 결정으로 서술해야 하며 법칙이 아닙니다.**

**⚠️ 한정 (2): 자극 종류는 두 개의 문서화된 트리거 중 하나일 뿐입니다.** §27.3.2의 **'표적 상실'** 은 다른 경로입니다 — 이 경우 NPC는 사전 표적 지식(증원 병력이나 아군에게서 전달받은 경우 포함)을 통해 **공격적 탐색으로 라우팅**됩니다.

**⚠️ 한정 (3) — 가장 중요**: **이 챕터의 대상 타이틀은 익명입니다.** §27.2는 "a title that I am unable to name"(CryENGINE 기반 TPS)이라고 명시합니다. **특정 게임에 귀속하거나 업계 보편 관행으로 제시해서는 안 됩니다.** 이는 **단일 구현의 설계 의도**입니다. 이 절의 신뢰도를 다른 절보다 낮게(medium) 두는 이유입니다.

### 탐색 단계

**1단계 — 최종 목격 위치.** "When you're searching for something, the most sensible place to start looking is the last place that you saw it." 신중이든 공격적이든 1단계는 최종 알려진 위치를 확인하는 **'좁은' 탐색**입니다.

동시 조사 인원은 게임의 느낌을 크게 바꿉니다.

- **신중 1단계**: 조사하러 가는 NPC를 제한하면 게임의 느낌이 극적으로 달라집니다. **단 1명만 허용하면 플레이어는 무리에서 표적을 떼어내 개별 처리하거나 더 쉽게 지나칠 수 있는 선택지**를 얻습니다. 그리고 NPC 간 창발적 상호작용이 가능해집니다 — 조사하러 가는 NPC가 근처 아군에게 "확인하러 간다"고 말하는 식
- **공격적 1단계**: 여러 NPC가 동시에 조사하면 더 협조적으로 보이지만, **전원이 한 지점에 뭉치는 것을 막기 위한 제한이 필요합니다 — 동시 탐색 2~3명이 잘 작동합니다**

**2단계 — 광역 탐색.** 새 탐색 지점을 생성하고 조사하며, NPC들이 최근 탐색한 지점을 서로 공유합니다. 플레이어가 발각되지 않으면 NPC는 이전 임무로 자연스럽게 복귀해야 합니다.

**성능 노트**: 미탐색 지점에 대한 시야 판정 레이캐스트는 비용이 큽니다. Welsh는 **지점당 1~2초에 한 번**이면 이동 중 수동적 탐색을 반영하기에 충분하다고 서술합니다.

### 특징 및 차별점

**Mark of the Ninja의 대조적 선택 — 관심사 스택을 두지 않기.**

Klei는 정반대 방향의 문제에 부딪혔습니다: 경비병이 시체를 조사하던 중 플레이어에게 주의를 빼앗기고, 표적을 잃은 뒤 돌아서서 시체를 다시 본다면? 방금 전과 똑같이 놀라는 것은 말이 안 되고, **원래의 조사를 마치러 돌아가는 것이 자연스럽습니다.** 그러나 팀의 판정은 다릅니다.

> "this ultimately felt like going a little bit too far down the rabbit hole. We made the decision that each agent would be aware of only one interest at a time and there would be no stack or queue of interests. Once the highest priority interest was dealt with at any given time, we wanted the situation to naturally reset to a default state and not have the guard continuing on to revisit every past source of interest they may have come across during an encounter."
> — Brook Miles, Game AI Pro ch.32, p.419

**한 번에 정확히 하나의 관심사만, 스택도 큐도 없음.** 해결되면 기본 상태로 리셋되며, 경비병이 조우 중 마주친 과거 자극을 모두 되짚는 '토끼굴'을 명시적으로 기각했습니다.

**⚠️ 귀속 주의**: Klei가 밝힌 이유는 **'용서'가 아니라 개연성 + 복잡도 회피**("too far down the rabbit hole")입니다. 이 규칙이 결과적으로 플레이어에게 관대하게 작동하는 것은 **기능적 부수 효과**이며, **Klei의 표명된 의도로 귀속해서는 안 됩니다.**

### 설계 고려사항

- **포기 조건이 곧 잠입의 리듬입니다.** AI가 언제 탐색을 그만두는지가 플레이어의 숨 쉴 구간을 정의합니다. 이 값이 없으면 발각은 영구적 상태 변화가 되고, 게임은 2-6(공포 잠입)이 됩니다
- **탐색 인원 제한은 난이도 조절기가 아니라 플레이 어휘 생성기입니다.** 1명만 조사하러 오게 만들면 "떼어내서 잡기"라는 플레이가 생겨납니다. 전원이 오게 만들면 그 플레이가 사라집니다
- **자극 종류로 분기하는 구조는 플레이어에게 도구를 줍니다.** 병을 던지면 신중한 탐색, 총을 쏘면 공격적 탐색이라는 규칙은 **플레이어가 AI의 탐색 모드를 선택할 수 있다**는 뜻입니다. 이것이 3-4의 자극 중재와 결합하면 잠입의 조작 어휘가 완성됩니다
- ⚠️ **이 절의 수치(2~3명, 1~2초)는 익명 타이틀의 단일 구현 값**이며 저자가 "works well"이라 평가한 것입니다. 검증된 상수가 아닙니다

## 3-4. 자극 중재와 우선순위

### 메커니즘

Mark of the Ninja의 자극 중재는 **단일 정수 우선순위**로 구현됩니다.

> "Interest sources, and by extension interest entries in agents' brains, contain a simple integer value of priority, where higher priority interests can replace lower or equal priority interests, and the agent will change his focus accordingly. If the agent currently holds a high priority interest, lower priorities interests are discarded and never enter the agent's awareness."
> — Brook Miles, Game AI Pro ch.32, §32.6

핵심 규칙 두 가지: **높은 우선순위는 '낮거나 같은' 것을 대체**하며, **이미 높은 우선순위 관심사를 보유한 에이전트는 낮은 것을 폐기해 인지에 아예 진입시키지 않습니다.**

**Listing 32.2 — Mark of the Ninja의 관심사 우선순위 정의**

| 상수 | 값 |
|---|---|
| `INTEREST_PRIORITY_LOWEST` | 0 |
| `INTEREST_PRIORITY_BROKEN` | 1 |
| `INTEREST_PRIORITY_MISSING` | 2 |
| `INTEREST_PRIORITY_SUSPECT` | 4 |
| `INTEREST_PRIORITY_SMOKE` | 4 |
| `INTEREST_PRIORITY_CORPSE` | 4 |
| `INTEREST_PRIORITY_NOISE_QUIET` | 4 |
| `INTEREST_PRIORITY_NOISE_LOUD` | 4 |
| `INTEREST_PRIORITY_BOX` | 5 |
| `INTEREST_PRIORITY_SPIKEMINE` | 5 |
| `INTEREST_PRIORITY_DISTRACTIONFLARE` | 10 |
| `INTEREST_PRIORITY_TERROR` | 20 |

**⚠️ 필수 교정 (1): 이것은 '출시 값'이 아닙니다.** Miles는 이 값들이 **"near the end of the project"** 기준이며 **"they had changed many times during development in response to unanticipated or blatantly unrealistic behavior"** 라고 명시합니다. 한국어로는 **'프로젝트 후반 기준 값'** 으로 표기해야 합니다. 그리고 그가 덧붙이는 실무 교훈이 중요합니다 — **"Making them easy to change and try out new combinations was a big help."** 이 값들의 의의는 값 자체가 아니라 **값을 쉽게 바꿔 볼 수 있게 만든 것이 큰 도움이 되었다**는 데 있습니다.

### 특징 및 차별점: 설계 가정의 반전

이 절의 가장 중요한 내용은 값이 아니라 **팀이 틀렸던 방식**입니다.

> "Initially, we assumed that finding corpses would be one of the highest priority interests in the game, superseded only by seeing a target (the player), but it turned out to be not so simple. If you hear a sound, and while investigating the sound you discover a body, clearly it makes sense to pay attention to this new discovery. But what if the situation were reversed? If you're investigating a body and you hear a sound, should you ignore it? This is potentially very unwise, especially if the noise is footsteps rapidly approaching you from behind."
> — Brook Miles, Game AI Pro ch.32, p.419

팀은 **시체 발견이 (플레이어 목격 다음가는) 최상위 우선순위**일 것이라 가정했습니다. 논증은 자연스럽습니다 — 소리를 조사하던 중 시체를 발견하면 새 발견에 주의를 돌리는 것이 당연합니다. **그런데 반대는?** 시체를 조사하던 중 소리를 들으면 무시해야 할까요? 그것은 매우 현명하지 못합니다 — **특히 그 소리가 뒤에서 빠르게 다가오는 발소리라면.**

해법은 우아합니다.

> "It turns out that corpses, noises, and a variety of other specific interest types should all be treated on a newest-first basis. The last thing you notice is probably the most important thing. Our solution here was simply to make this set of interests all the same priority. However, new interests of equal priority (to your current interest) are treated as more important."
> — Brook Miles, Game AI Pro ch.32, p.419

**시체·소음 등 다수 유형을 전부 우선순위 4로 동률 배치**하고, **동률은 최신 우선으로 해소**합니다. 근거는 한 문장으로 압축됩니다 — **"마지막에 알아챈 것이 아마도 가장 중요한 것이다."**

동시에 Miles는 **진짜로 서열이 있는 것들**을 구분합니다: "Seeing a broken pot is always less interesting than a shadowy figure or a gunshot. Seeing your partner impaled on a spike trap is always more important than the sound of glass breaking in the distance." 깨진 항아리는 **항상** 그림자나 총성보다 덜 흥미롭고, 동료가 함정에 꿰뚫린 것은 **항상** 멀리서 유리 깨지는 소리보다 중요합니다. 즉 우선순위 테이블은 **서열이 실재하는 곳에서는 서열을, 서열이 없는 곳에서는 동률+최신**을 씁니다.

**⚠️ 필수 교정 (2): 이것은 '표명된 의도 vs 실제 동작'의 괴리가 아닙니다.** 개발 중 수정된 **초기 가정**이며, 최종 설계는 최종 의도와 일치합니다. **반복 튜닝의 사례로 써야 하며 intent/behavior 축의 사례로 라벨링해서는 안 됩니다.**

### 설계 고려사항

- **자극 중재를 단일 정수로 구현할 수 있다는 것이 이 사례의 첫 번째 교훈입니다.** 복잡한 가중치 시스템이나 효용 함수 없이, 정수 하나와 두 개의 규칙(높은 것이 낮거나 같은 것을 대체, 동률은 최신 우선)으로 상용 잠입 게임의 자극 중재가 성립했습니다
- **직관은 반복적으로 틀리며, 틀림을 빠르게 발견하는 것이 설계 역량입니다.** 시체 우선순위 가정은 합리적이었고 절반만 맞았습니다. 값을 쉽게 바꿔 볼 수 있게 만든 것이 이를 발견하게 했습니다
- **'낮은 것은 인지에 진입조차 하지 않는다'는 규칙은 성능 최적화가 아니라 행동 설계입니다.** 이 규칙 덕에 경계 중인 경비병은 깨진 항아리에 한눈팔지 않습니다. 이는 개연성(1-1의 네 번째 특성)에 직접 기여합니다
- **동률+최신 규칙은 플레이어에게 도구를 줍니다.** 시체를 조사하는 경비병을 소음으로 끌어낼 수 있다는 뜻이기 때문입니다. 만약 시체가 최상위였다면 그 플레이는 존재하지 않습니다

## 3-5. 정보 시각화: 플레이어에게 AI 상태 보여주기

### 메커니즘

1-1에서 규정했듯, **AI의 인지 상태를 플레이어에게 보여주는 것은 잠입 장르의 성립 조건**입니다. 확보된 출처들은 이 문제에 대해 세 층위의 해법을 보여 줍니다.

**층위 1 — 연속 진행도 미터.** Blacklist의 탐지 타이머는 성장하는 HUD 요소로 렌더링되며, 그 목적은 명시적입니다 — 플레이어에게 **"LOS를 끊거나 NPC를 처치해 탐지를 피할 기회의 창"** 을 정의해 주는 것. 이것이 결정적인 이유는, 미터가 **정보를 주는 것이 아니라 행동 가능한 시간을 주기** 때문입니다. "당신은 발각되었습니다"는 정보이고, "지금 이 시야에서 벗어나면 아직 발각되지 않는다 — 그리고 남은 여유가 이만큼이다"는 **선택지**입니다.

Walsh는 이 추상화의 성격을 정확히 서술합니다. 무언가를 처음 지각할 때 사람은 그것을 알아보기 시작할 뿐입니다 — "저기 뭔가 움직인다"까지만 말할 수 있고, **그것이 정확히 무엇인지는 시간이 걸리며 기대·조명·목격 시간 등 여러 요인에 달렸습니다.** Blacklist를 포함한 다수의 게임은 이 모든 것을 **진행 바 하나로 추상화**하며, 그 바는 아날로그이지만 **NPC 입장에서는 두 개의 이진 상태**("수상한 게 안 보인다" / "저기 적 1번이다")만을 표상합니다.

**즉 진행 바는 NPC의 내부 상태를 표시하는 것이 아니라, 내부 상태가 갖지 않은 연속성을 플레이어에게 발명해 주는 장치입니다.** 이것이 잠입 UI 설계의 핵심 통찰입니다.

**층위 2 — 이산 상태 아이콘.** Klei의 창업자는 강연에서 AI의 감정을 어떻게 전달할지 고민한 과정을 설명합니다. 재능 있는 아티스트들이 얼굴 애니메이션을 만들고 표정을 과장했지만, **가장 결정적이고 가장 쉬운 방법은 그냥 감정 아이콘을 머리 위에 띄우는 것**이었습니다 — 그래야 모두가 "저 사람은 공포 상태다", "저 사람은 조사 상태다"를 압니다.

이 대비는 시사적입니다. **표정은 예술적으로 우월하지만 정보 채널로는 열등합니다.** 잠입 게임에서 AI 상태는 플레이어가 매 순간 참조해야 하는 데이터이며, 데이터는 읽기 쉬워야 합니다.

**층위 3 — 시스템 자체의 렌더링.** Third Eye Crime은 NPC가 실제로 사용하는 점유 맵을 항상 렌더링합니다. 여기서 표시되는 것은 **AI 상태의 근사치가 아니라 AI가 참조하는 자료구조 그 자체**입니다. 상세는 5-5에서 다룹니다.

### 특징 및 차별점

**Mark of the Ninja의 소음 링 논쟁은 이 절 전체의 논증을 압축합니다.** 3-2에서 확인했듯 Mark of the Ninja의 실제 가청 판정은 **사운드 메시 위의 A\* 경로 존재**이지 반경이 아닙니다. 그런데 플레이어에게 표시되는 것은 **링(원)** 입니다. 그리고 Figure 32.1의 캡션은 **반경 안에 있어도 경로가 없으면 못 듣는다**고 명시합니다.

> ℹ️ **아래는 두 출처를 대조한 이 문서의 분석이며, 어느 출처에도 이 형태로 서술되어 있지 않습니다.** 표시되는 링은 실제 가청 범위보다 **넓게** 나타납니다(반경 안이지만 경로가 없는 구역이 링 안에 포함되므로). 즉 플레이어가 링을 보고 판단하면 **실제보다 위험을 과대평가**하게 됩니다. 이는 플레이어에게 **불리한 방향이 아니라 유리한 방향의 오차**입니다 — 링을 믿고 피한 플레이어는 절대 억울하게 걸리지 않습니다. 이 방향성은 1-3의 제1원칙과 정합적이지만, **Klei가 이를 의도했다는 근거는 확보하지 못했습니다.**

**정보 시각화 층위 비교**

| 층위 | 표시 대상 | 대표 사례 | 플레이어가 얻는 것 | 비용 |
|---|---|---|---|---|
| **연속 진행도 미터** | 탐지 타이머 | Splinter Cell: Blacklist | **행동 가능한 시간의 창** | 임계값 경직성(3-1) |
| **이산 상태 아이콘** | AI의 인지 상태 | Mark of the Ninja | 현재 국면의 즉시 판독 | 연출의 미묘함 상실 |
| **감각 형상 렌더링** | 시야 원뿔·소음 링 | Mark of the Ninja | 계획의 지평 확장 | '몰래 훔쳐본다'는 판타지 감소 |
| **지식 모델 렌더링** | 점유 맵(자료구조 자체) | Third Eye Crime | **발각 이후의 플레이 공간** | 서사적 정당화 필요 |
| **애니메이션 상태 노출** | 인지 상태에 따른 동작 변형 | Splinter Cell: Blacklist | 몰입을 깨지 않는 판독 | 애니메이션 조합 폭발 |

### 설계 고려사항

- **정보를 숨기는 것은 난이도가 아니라 계획 지평의 축소입니다.** Klei의 판정을 반복합니다 — 소음 링을 숨기면 게임이 '뭉개지고' 몇 수 앞을 계획할 수 있는지 불명확해집니다. 잠입의 페이로드가 계획이라면, 정보를 숨기는 것은 페이로드를 줄이는 것입니다
- **미터를 노출하면 임계값이 경직됩니다.** Walsh의 자기 진단(3-1)을 상기하십시오 — 1cm 밖에서는 영원히 안 보이고 1cm 안에서는 몇 초 안에 보이는 문제는 **모델 추상화와 HUD 피드백을 일치시켜야 하는 필요의 직접적 결과**입니다. 미터는 공짜가 아닙니다
- **애니메이션도 정보 채널입니다.** Blacklist에서 같은 테이크다운이 AI 인지 상태에 따라 다른 변형으로 자동 호출된다는 것은, **HUD를 보지 않아도 AI 상태를 알 수 있다**는 뜻입니다. 디자인이 최소 HUD를 원할 때 이것이 대안이 됩니다
- **HUD가 없으면 다른 채널을 반드시 확보해야 합니다.** Blacklist는 소리에 대한 HUD 피드백을 고려했으나 **디자인이 최소 HUD를 원해서** 포기했고, 대신 **3계층 barks 시스템**으로 해결했습니다(4-1에서 상술). 채널을 없애는 것이 아니라 **바꾸는** 것입니다

## 3-6. 용서 설계

### 메커니즘

**용서 설계의 제1원칙은 1-3에서 이미 규정했습니다** — 플레이어 지각 > 시뮬레이션 정확도, 단 개연성으로 경계를 둡니다. 이 절은 그 원칙이 어떤 구체적 장치로 구현되었는지를 다룹니다.

**장치 1 — 화면 밖 NPC의 청각 감쇠.**

> "The first thing we did was that NPCs that are offscreen, but far enough away from the player that it was plausible they didn't hear him, have their hearing reduced for certain events by ½. The result was that the game instantly became more fun and our creative director stopped complaining. We ended up applying this to some indirect visual events as well, such as seeing an NPC get shot."
> — Martin Walsh, Game AI Pro 2 ch.28, p.321

조건은 **두 개의 논리곱**입니다 — **화면 밖에 있으면서 동시에 못 들었다고 해도 개연적일 만큼 충분히 멀어야** 합니다. 화면 밖이기만 하면 되는 것이 아닙니다. 그리고 같은 감쇠가 **'NPC가 총 맞는 것을 목격' 같은 일부 간접 시각 이벤트**에도 적용됐습니다.

**⚠️ 한정 3건**: (a) **½ 감쇠는 "certain events"에 한정**됩니다 — 모든 이벤트가 아닙니다. (b) **"게임이 즉시 재밌어졌다"는 주관적 자기 보고**이지 측정치가 아닙니다. (c) 기각된 대안(반경 일괄 하향)의 사유는 **두 가지**였고(즉살 테이크다운 붕괴 + 경비병 무반응) 이 문서는 2-2에서 전자만 인용했습니다.

**장치 2 — 시야 상실 후 의도적 치팅.**

> "A far simpler solution to this problem is to simply allow the NPC to 'know' the position of their target for a few seconds after losing sight of them—the exact number can be tweaked to suit the feel of the game, but around 2–3 s gives a reasonably realistic feel. This is both a cheap and effective way of giving the AI a human sense of 'intuition' and has been used in a number of high-profile titles, including the Halo, Crysis, and Crackdown series of games."
> — Rich Welsh, Game AI Pro 2 ch.27, §27.3.2

이 권고의 논증 구조가 중요합니다. **이것은 '더 나은 방법'이 아니라 '더 단순한 방법'으로 제시됩니다.** 자연스러운 대안 — 최종 목격 속도로 현재 위치를 외삽 — 은 실패합니다. 캐릭터(특히 플레이어)는 완벽한 직선으로 움직이지 않고, 더 결정적으로 **내비게이션 가능 지점 검사를 생략하면 프롭 내부나 월드 밖 좌표를 만들 수 있기** 때문입니다. Welsh는 이를 명시합니다 — 그런 순진한 방법이 **프롭 안이나 월드 밖 위치를 생성하지 않으리라 보장하는 것은 불가능**합니다.

즉 **치팅이 시뮬레이션보다 안전합니다.** 시뮬레이션은 유효하지 않은 상태를 만들 수 있지만, "그냥 알게 해 주기"는 항상 유효한 상태만 만듭니다.

**⚠️ 한정**: **2~3초는 저자의 감각 기반 휴리스틱**이며("can be tweaked to suit the feel") 측정·검증된 값이 아닙니다. Halo·Crysis·Crackdown 사용 진술도 **저자의 진술**이지 해당 개발사의 1차 문서가 아닙니다.

**장치 3 — 재발견 가능한 'suspect' 관심사.**

Mark of the Ninja는 플레이어에게 부착된 시야 관심사를 둡니다.

> "This interest source is detectable from farther away than the agent's ability to see the player as a target, and so draws them in to investigate without immediately seeing and shooting the player. We want this to happen every time the agent detects the player, and so this particular interest source is marked as being rediscoverable"
> — Brook Miles, Game AI Pro ch.32, p.420

**에이전트가 플레이어를 표적으로 획득할 수 있는 거리보다 '더 멀리서' 감지**되므로, 에이전트는 즉시 발견하고 사살하는 대신 **조사하러 다가옵니다.** 그리고 이것이 매번 일어나기를 원했으므로 **재발견 가능(rediscoverable)** 으로 표시되어 발견자 목록을 두지 않습니다.

이 장치의 우아함은 **용서를 별도 시스템이 아니라 감지 반경의 배치로 구현**한다는 데 있습니다. 특별한 예외 규칙이나 확률적 관대함 없이, 두 반경의 순서만으로 "경비병은 당신을 보기 전에 먼저 의심한다"가 성립합니다.

**장치 4 — 조사된 관심사의 소멸.**

> "the interest source associated with the agent's interest record is marked as investigated, which then removes the interest source (but not the game object it was associated with) from the world, never to be seen or heard of again"
> — Brook Miles, Game AI Pro ch.32, p.419

**⚠️ 정밀 독해 필수**: 제거되는 것은 **관심사 소스**이지 **게임 오브젝트**가 아닙니다. 시체와 깨진 조명은 월드에 남고, **'주목 가능성'만 소모**됩니다. 그리고 기본적으로 "the interest source keeps a list of discoverers and doesn't cause itself to be noticed by the same agent twice" — 관심사 소스는 발견자 목록을 유지해 같은 에이전트가 두 번 주목하지 않게 합니다. suspect 소스는 위에서 보았듯 **명시적 예외**입니다.

### 특징 및 차별점

**용서 장치 비교**

| 장치 | 출처 | 조건 | 성격 | 한정 |
|---|---|---|---|---|
| **화면 밖 청각 ½** | Blacklist | 화면 밖 **∧** 개연적일 만큼 멂 | 지각 우선 | "certain events"에만 |
| **간접 시각 이벤트 ½** | Blacklist | 위와 동일 | 지각 우선 | "some" 이벤트에만 |
| **2~3초 치팅** | 익명 TPS(ch.27) | 시야 상실 직후 | **직관의 환상** + 유효 상태 보장 | 감각 기반 휴리스틱 |
| **suspect 반경 > 표적 획득 반경** | Mark of the Ninja | 항상(rediscoverable) | 즉살 대신 조사 유도 | — |
| **관심사 소스 소멸** | Mark of the Ninja | 조사 완료 시 | 반복 제거 | 오브젝트는 남음 |
| **단일 관심사, 스택 없음** | Mark of the Ninja | 항상 | 상태 리셋 보장 | ⚠️ **의도는 개연성/복잡도**이지 용서가 아님 |

### 거울상: 제1원칙은 양방향으로 작동합니다

지금까지의 장치는 모두 **플레이어에게 유리한 방향**의 지각 관리였습니다. 그러나 같은 원칙이 **반대 방향으로도** 작동하며, 이것을 확인해야 원칙의 정체가 드러납니다.

Welsh는 탐색 지점 스코어링을 다루며, 가장 중요한 두 가중치를 **지점과 표적의 추정 위치 간 거리**, **지점과 NPC 현재 위치 간 거리**로 둡니다. 문제는 여러 NPC가 같은 지점 풀에서 뽑을 때 발생합니다 — 각자 **자기 주변에 국소화된 지점을 선호**하게 되어 탐색이 뭉칩니다. 해법은 세 번째 가중치입니다.

> "By adding an extra weight for the distance of the search spot from the player's actual current position, it gives the AI the illusion of human intuition and shapes the search pattern gently in the correct direction. The weighting for distance to target actual location should be quite subtle compared to the other two weights, so as not to make the NPCs all immediately flock to the target. This would both break the illusion of intuition and make the game feel unfairly stacked against the player."
> — Rich Welsh, Game AI Pro 2 ch.27, §27.4.2.3

**AI에게 플레이어의 실제 현재 위치를 알려 주되, 아주 미묘하게만 알려 줍니다.** 목적은 **"인간적 직관의 착각"** 을 부여하고 탐색 패턴을 올바른 방향으로 **부드럽게** 유도하는 것입니다. 그리고 저자가 명시하는 실패 경계가 결정적입니다 — 이 가중치가 강하면 NPC들이 **즉시 표적에게 몰려들어**, **직관의 착각이 깨지는 동시에 게임이 플레이어에게 불공정하게 짜인 느낌**을 줍니다.

**이것이 3-6 전체의 거울상인 이유**: 2~3초 치팅(장치 2)에서 AI는 플레이어에게 **유리하지 않은** 정보를 공짜로 받지만, 그것이 **직관처럼 보이므로** 허용됩니다. 여기서도 AI는 정답을 공짜로 받지만, **미묘하므로** 직관처럼 보입니다. 두 경우 모두 판정 기준은 시뮬레이션의 정확도가 아니라 **플레이어에게 어떻게 보이는가**입니다.

즉 **제1원칙은 관대함 전용이 아닙니다.** "플레이어 지각 > 시뮬레이션 정확도, 단 개연성으로 경계를 둔다"는 명제는 **대칭적**입니다 — 플레이어에게 유리한 쪽으로 기울일 때도(½ 감쇠), 불리한 쪽으로 기울일 때도(실제 위치 가중치), **기울이는 양을 정하는 것은 지각이지 사실이 아닙니다.** 그리고 양쪽 모두 **들키는 순간 실패**합니다: 화면 밖 NPC가 모퉁이 너머에서 못 들으면 우스꽝스럽고, NPC들이 곧장 몰려오면 커닝이 들통납니다.

**설계 실무로 옮기면**: AI에게 정답을 주는 것 자체는 금기가 아닙니다. **금기는 정답을 준 티가 나는 것**입니다. 커닝의 허용량은 시뮬레이션 논쟁이 아니라 **관측 가능성의 함수**로 정해야 합니다.

### 설계 고려사항

- **용서는 관대함이 아니라 지각의 정합성입니다.** ½ 감쇠는 NPC를 약하게 만들려는 것이 아니라, **플레이어가 볼 수 없는 것이 플레이어를 처벌하지 않게** 하려는 것입니다. 이 구분을 놓치면 난이도만 낮추고 문제는 남습니다
- **개연성이 경계를 긋는다는 것을 잊지 마십시오.** Walsh가 화면 밖 NPC의 청각을 아예 끄지 않은 이유입니다 — 모퉁이를 돌았는데 바로 옆의 NPC가 못 들었다면 우스꽝스럽습니다. 그는 **플레이어에게 불공정하게 느껴질지라도 개연적인 결과가 나오는 거리를 골라야 했다**고 명시합니다. **용서 설계는 불공정감을 0으로 만드는 것이 목표가 아닙니다**
- **치팅이 시뮬레이션보다 안전할 수 있습니다.** 이는 잠입에 국한되지 않는 일반 원리입니다 — 정확한 모델은 유효하지 않은 상태를 만들 수 있고, 거친 근사는 만들지 않습니다
- **용서를 반경 배치로 구현하면 예외 규칙이 필요 없습니다.** Mark of the Ninja의 suspect 반경은 이 접근의 모범입니다
- **AI가 커닝하는 방향도 같은 원칙으로 설계하십시오.** 플레이어의 실제 위치를 스코어링에 넣을 때 **강도를 정하는 것은 사실이 아니라 관측 가능성**입니다. ⚠️ 단 "미묘하게"는 **감각 기반 지침**이며 수치가 제시되어 있지 않습니다 — 자기 게임에서 **NPC가 몰려드는 것이 보이기 시작하는 지점**을 실측해 경계를 찾아야 합니다

## 3-7. 은신 자원: 그림자, 엄폐, 소음

### 메커니즘

잠입 게임의 자원은 **플레이어 시트가 아니라 월드에 있습니다.** 이것이 다른 장르와의 근본적 차이입니다 — 체력은 캐릭터에 속하지만 그림자는 레벨에 속합니다.

**자원 1 — 조명.** 두 가지 접근이 있습니다.

- **연속형**(Thief): 노출도가 조명의 함수로 0..1 아날로그 집계. Leonard의 서술에 따르면 최종 감각값은 lighting/movement/exposure를 아날로그로 집계합니다
- **이진형**(Mark of the Ninja): 광원에 비춰졌는가만 판정하며 시야 판정의 논리곱 중 한 항. Klei는 이를 **명료함을 위해 선택**했습니다 — 한 발이라도 빛에 있으면 보임

**자원 2 — 엄폐와 자세.** Blacklist에서 자세는 **탐지 개시에 필요한 가시 본 개수를 결정**합니다(3-1). 즉 엄폐는 이진 은폐가 아니라 **임계값 수정자**입니다.

**자원 3 — 소음.** 소음은 이동 속도와 결합되어 **속도/은신의 교환**을 만듭니다. Blacklist의 즉살 테이크다운 사례(2-2)가 보여 주듯, 이 교환이 무너지면 게임 전체가 무너집니다.

**자원 4 — 시체와 흔적.** Blacklist의 '변경된 오브젝트' 시스템은 이 자원의 반대편입니다. 오브젝트가 이전에 알려진 상태에서 변하면 **수명(lifetime)을 가진 이벤트**를 생성하고, 그 이벤트를 목격한 첫 NPC가 **점유(claim)** 하여 상태와 여러 요인에 따라 가벼운 조사를 수행합니다. Walsh는 이 시스템의 비용 대비 효과를 이렇게 평가합니다 — **"매우 단순했지만 다른 어떤 시스템보다 투자 대비 효과가 컸다"**, 그리고 일부 리뷰가 실제로 **NPC가 열린 문이나 꺼진 전등 스위치를 알아챈다는 점을 언급**했습니다.

### 특징 및 차별점

**Klei의 이진 조명 결정은 아트와 게임플레이의 충돌을 게임플레이 쪽으로 해소한 사례입니다.** 강연에서 아티스트 측 관점이 명시적으로 서술됩니다 — 아티스트라면 빛에 들어갈 때 그림자가 여기까지 걸치고 부분적으로 보이는 상태를 만들고 싶어 합니다. 그러나 팀은 **한 발이 빛에 있으면 보이고 아니면 안 보이는** 것으로 바꿨습니다. 이유는 하나입니다 — **플레이어가 항상 자신이 그림자 안인지 밖인지 알게** 하기 위해서.

이 결정의 함의는 조명을 넘어섭니다. **잠입 게임에서 시각적 자연스러움과 판독 가능성이 충돌하면, 판독 가능성이 이겨야 합니다.** 왜냐하면 조명은 이 장르에서 배경이 아니라 **규칙**이기 때문입니다.

**은신 자원 설계 비교**

| 자원 | 연속형 접근 | 이진형 접근 | 교환 관계 |
|---|---|---|---|
| **조명** | 노출도가 0..1 (Thief) | 빛 안/밖 (Mark of the Ninja) | 몰입 ↔ 계획 가능성 |
| **자세** | 가시 본 임계값 수정 (Blacklist) | — | 은신 ↔ 이동 속도 |
| **소음** | 이동 속도의 함수 | — | 속도 ↔ 은신 |
| **흔적** | 이벤트 수명 + 점유 (Blacklist) | — | 효율 ↔ 증거 |

### 설계 고려사항

- **자원이 월드에 있으면 레벨 디자이너가 밸런스 담당자가 됩니다.** 조명 배치가 곧 난이도이고, 초크포인트 배치가 곧 소음 규칙(3-2)입니다. 이는 조직 구조에 영향을 미칩니다
- **모든 자원은 교환을 가져야 합니다.** 웅크리면 덜 보이지만 느려야 하고, 어둠은 안전하지만 시야를 제한해야 합니다. 교환 없는 자원은 지배 전략이 됩니다
- **흔적 시스템은 비용 대비 효과가 큽니다.** Blacklist의 '변경된 오브젝트'는 매우 단순한 시스템이었으나 리뷰에 언급될 만큼 인상을 남겼습니다. 다만 Walsh는 함정도 명시합니다 — **대사가 구체적일수록 기억에 남고, 기억에 남을수록 반복될 때 몰입이 깨집니다.** 그들의 해법(3계층 barks)은 4-1에서 다룹니다

---

# 4. 플레이 경험 설계

## 4-1. 긴장과 이완

잠입의 정서적 페이로드는 발각도 무지도 아니라 **그 사이의 전이**에 있습니다. Blacklist의 애니메이션 디렉터는 전투 전 애니메이션과 전투 애니메이션 모두 이미 충분히 좋았고 자신이 집착한 것은 **중간의 전이**였다고 진술하며, 동생과 장난칠 때 **재미있는 부분은 동생이 겁먹는 순간이지 엄마에게 이르는 순간이 아니라는** 비유를 듭니다.

그가 강연에서 시연한 **연쇄 반응**은 이 원리의 구현입니다. AI가 문을 두드리는 중에 전구를 터뜨리면 → AI가 부드럽게 반응해 다가가 다른 AI와 소통하고 "전구일 뿐이니 일하러 돌아가라"고 정리합니다 → 다시 교란하면 **AI가 첫 번째를 기억하고 있어 더 강하게 반응**합니다 → 한 명을 창밖으로 끌어내면 의심이 생기고 → 공포가 생기고 → 공포가 소통되어 패닉으로 번지고 → 그들이 뭉쳐 수색을 요청하고 → **그 결과 그들의 등이 자연스럽게 플레이어를 향하게 되어** 플레이어가 그것을 이용할 수 있게 됩니다.

이 연쇄에서 주목할 것은 **각 단계가 이전 단계를 기억한다**는 점과, **긴장의 최고조가 플레이어에게 새로운 기회를 만든다**는 점입니다. 긴장은 처벌이 아니라 **다음 수의 재료**입니다.

**등을 돌린 이유의 설계.** 같은 강연에서 그는 스텔스 게임에서 **모두가 항상 플레이어에게 등을 돌리고 있는 것**에 대한 불만을 표명하며, NPC에게 **등을 돌릴 이유**를 주려 했다고 설명합니다 — 세수하기, 상자 나르기, 전화하며 서성이기(그는 자신이 통화 중에 아무것에도 집중하지 않고 서성이는 습관을 관찰했다고 말합니다). 그리고 NPC를 좀 더 친근하게 보이게 해서 플레이어가 **"저 사람도 집에 아내가 있을지 모르는데 죽여야 하나"** 라고 생각하게 만들고 싶었다고 진술합니다.

**피드백 채널의 대체.** Blacklist는 소리에 대한 HUD 피드백을 고려했으나 **디자인이 최소 HUD를 원해서** 포기했고, 대신 **3계층 barks**로 해결했습니다. 티어 1은 매우 구체적("발소리를 들은 것 같은데"), 티어 2는 더 일반적("저기 누군가 있는 것 같은데"), 티어 3은 완전히 일반적("저기 누가 있어"). 티어 1부터 소진한 뒤 2, 3으로 내려가므로 **플레이어는 처음 몇 번은 필요한 정보를 정확히 얻고**(달리면 발소리가 난다는 것을 학습), 이후에는 **더 큰 풀을 가진 일반 버전**을 받아 반복을 피합니다.

이 설계의 통찰은 **정보 필요량이 시간에 따라 감소한다**는 것입니다. 학습이 끝난 플레이어에게 구체적 정보는 반복일 뿐입니다. Walsh는 함정도 명시합니다 — **대사가 구체적일수록 기억에 남고, 기억에 남을수록 반복될 때 몰입이 깨집니다.**

## 4-2. 발각 이후 설계

**발각을 실패로 처리할 것인가, 새 국면으로 처리할 것인가.** 2장의 분류가 이 축으로 갈렸듯, 이것이 잠입 설계의 가장 큰 단일 결정입니다.

Third Eye Crime은 이 축을 명시적으로 뒤집은 사례입니다.

> "The fun of the game for the player, therefore becomes not avoiding detection but evading the searching NPCs. Indeed one of the distinguishing features of the gameplay of Third Eye Crime is that the fun happens after, not before, the player is detected."
> — Damián Isla, AIIDE 2013, p.206

**재미가 발각 '이전'이 아니라 '이후'에 발생하며, 플레이어의 목표가 탐지 회피가 아니라 탐색 중인 NPC를 따돌리는 것이 됩니다.** 상세는 5-5에서 다룹니다.

**⚠️ 한정**: 이는 **개발자가 표명한 설계 의도**이며 논문에 이를 뒷받침하는 플레이테스트 데이터나 측정치가 전혀 제시되지 않습니다. 또한 **'발각 = 실패가 업계 관행'이라는 대비는 분석가의 해석**입니다 — 논문의 문자적 대비는 '감각 모델은 정교하나 지식 모델이 빈약하다'는 더 좁은 주장입니다.

**발각 이후를 설계한다는 것의 실무적 의미**는 세 가지입니다.

**첫째, AI에게 포기 조건이 있어야 합니다**(3-3). 포기 없는 발각은 국면이 아니라 종료입니다.

**둘째, 상태 리셋이 보장되어야 합니다.** Mark of the Ninja의 단일 관심사 규칙(3-3)이 이를 담당합니다 — 해결되면 기본 상태로 자연스럽게 리셋되며, 과거 자극을 되짚지 않습니다.

**셋째, 도피 공간이 존재해야 합니다.** Io-Interactive의 레벨 디자이너는 이 문제를 사회적 공간의 언어로 답합니다 — 규칙과 기대와 NPC로 채워지지 않은 **안전 공간**, 사람이 많지 않은 복도 같은 곳을 반드시 확보해야 하며, 그래야 플레이어가 **하나의 사회적 맥락에서 벗어나 완전히 새로운 맥락으로 이동**할 수 있습니다. 그러면 **새 맥락의 NPC들이 이전 사건을 모르는 것이 정당해집니다.** 그리고 그는 이 문제에 완벽한 해법은 없으며 계속 나아져야 하는 영역이라고 덧붙입니다.

같은 강연의 **"스위스 치즈"** 원칙 — 항상 빠져나갈 길이 있어야 하고 **막다른 길은 금지** — 이 여기에 결합합니다.

## 4-3. 실패 처리와 리트라이

잠입의 실패 비용은 다른 장르보다 구조적으로 큽니다. 계획이 페이로드라면(1-2), 실패는 **계획 전체의 무효화**이기 때문입니다. 액션 게임에서 실패는 체력 한 칸이지만 잠입에서 실패는 3분간의 관측과 계획입니다.

Blacklist의 탐지 미터가 결정적인 이유가 여기 있습니다 — 미터는 **실패를 이벤트에서 구간으로 바꿉니다.** "기회의 창"이 존재하면 플레이어는 계획이 무너지는 것을 보면서 **구제 행동**을 취할 수 있습니다. 미터가 없으면 계획은 통보 없이 무효화됩니다.

**리트라이 설계의 실무 기준**

| 유형 | 실패의 성격 | 요구되는 리트라이 설계 |
|---|---|---|
| **순수 잠입** | 계획 전체 무효화 | 자유 세이브 또는 촘촘한 체크포인트 |
| **액션 하이브리드** | 국면 전환 | 리트라이보다 **복귀 경로**가 중요 |
| **공포 잠입** | 대체로 즉사 | 체크포인트 간격이 리듬을 지배 |
| **샌드박스 암살** | 사회적 지위 상실 | 국면 내 회복 + 전체 재시도 유도 |

Io-Interactive의 튜토리얼 설계는 실패 비용을 **공간의 서사로 조절한** 사례입니다. 파리의 패션쇼 레벨이 혼돈으로 무너지면 **매우 비싼 것을 부수는 느낌**이 드는 반면, 훈련 시설의 합판으로 만든 요트는 **실험해도 되는 관대한 장소**입니다. 같은 시스템, 같은 AI인데 **픽션이 실패의 정서적 비용을 바꿉니다.**

## 4-4. 비살상 플레이 지원

> **⚠️ 이 절의 근거 상태**: **비살상 플레이 지원에 대한 1차 출처를 확보하지 못했습니다.** 아래는 확보된 자막 근거(귀속 서술)와 구조적 관찰이며 **수치를 싣지 않았습니다.**

비살상 플레이는 **선언으로 성립하지 않습니다.** 모든 위협에 대해 비살상 해법이 존재해야 하고, 그 해법들이 살상 해법과 **경쟁 가능한 비용**을 가져야 합니다.

Blacklist의 접근은 이를 애니메이션 층위에서 구현합니다. 강연에 따르면 살상/비살상은 **플레이어 선택의 축**으로 설계되어, 살상은 카람빗을, 비살상은 맨손(초크 등)을 사용하는 **병렬 애니메이션 세트**를 가집니다. 그리고 결정적으로 **살상이 대체로 더 빠릅니다** — 즉 비살상은 실제 비용을 지불합니다. 강연자는 이를 플레이 스타일과 연결합니다: 빠른 살상은 공격적 플레이어에게, 비살상은 은신형 플레이어에게 맞춰집니다.

조합 폭발도 명시됩니다. 커버 종류 × 커버 방향 × AI 방향 × 살상/비살상 × 전투/은신의 곱으로 애니메이션 개수가 결정되며, 강연자는 각각이 하루에서 이틀씩 걸린다는 점에서 **"빙산의 일각"** 이라고 표현합니다. (⚠️ **자막의 구체 개수는 인용하지 않습니다.**)

**Hitman의 대조적 접근.** Io-Interactive는 **기회(opportunity) 시스템에 "살상 금지 규칙"** 을 두었다고 진술합니다 — 히트맨 게임에서 이상하게 들릴 수 있지만, 플레이어가 **자기 스타일을 선택할 수 있기를** 원했고 원하지 않는 것을 강요하고 싶지 않았기 때문입니다. 일반 규칙으로 **많은 사람을 죽이라고 지시하는 것을 피하려** 했으며, 그 이유는 일부 플레이어가 기회를 따르면서도 **사일런트 어새신을 유지하고 싶어 하기** 때문입니다.

**설계 원칙**: 비살상 경로는 **가이드가 그것을 배제하지 않을 때만** 살아 있습니다. 튜토리얼과 퀘스트 가이드가 살상을 전제하면, 비살상은 시스템에 존재하되 플레이 문화에서는 사라집니다.

## 4-5. 접근성

잠입은 접근성 요구가 높은 장르입니다. **핵심 자원이 시각·청각 채널로만 전달되기 때문**입니다.

- **청각 정보의 시각 대체**: 소음이 게임플레이 규칙(3-2)이라면 청각 장애 플레이어는 규칙을 읽을 수 없습니다. Mark of the Ninja의 소음 링은 접근성 기능으로 설계되지 않았으나 **정확히 이 문제를 해결합니다** — 명료함을 위한 설계가 접근성과 정렬되는 사례입니다
- **탐지 미터의 대체 채널**: Blacklist의 인지 상태별 애니메이션 변형(3-5)은 HUD를 보지 않아도 상태를 알 수 있게 합니다
- **난이도의 지각 파라미터 적용**: 3자 해설에 따르면 Blacklist는 **난이도 설정별로 시야 형상의 크기와 모양을 조정**해 낮은 난이도에서는 사각지대를 늘렸습니다. ⚠️ 이는 **개발자 강연을 분석한 3자 해설**의 서술이며 Ubisoft 공식 문서가 아닙니다
- **가이드 강도의 선택**: Io-Interactive는 **가이드의 여러 레벨**을 제공해, 전체 기회 표시 / 월드 마커 없이 목표만 / 끄기 중에서 플레이어가 고를 수 있게 했다고 진술합니다. 이는 **난이도가 아니라 정보량을 선택하게 하는** 접근성 모델입니다

**핵심 원칙**: 잠입에서 접근성은 난이도 완화가 아니라 **정보 채널의 다중화**입니다. 3-5에서 확인했듯 정보를 숨기는 것은 난이도가 아니라 계획 지평의 축소이므로, **정보 채널을 늘리는 것은 게임을 쉽게 만드는 것이 아니라 게임을 플레이 가능하게 만드는 것**입니다.

---

# 5. 주요 사례 분석

## 5-1. Splinter Cell: Blacklist (2013) — 탐지 파이프라인의 완성형

### 메커니즘

3-1~3-2에서 상술한 파이프라인을 요약하면: **본 8개 레이캐스트 → 자세가 필요 가시 본 개수 결정 → 타이머 개시(구간은 형상이, 값은 거리·밝기·NPC 상태 등이 결정) → HUD 미터로 노출 → 가득 참에서 확정(중간 조사 상태 경유 가능)**. 소음은 TEAS 영역 경로의 초크포인트 거리 합산으로 판정하고 같은 방은 유클리드로 폴백합니다.

### 설계 교훈

**교훈 1 — 시스템은 시리즈에 걸쳐 수렴합니다.** 원뿔 → 박스(Conviction) → 관(Blacklist), 그리고 사운드 엔진(Conviction의 문제) → TEAS 경로(Blacklist의 해법). **두 축 모두 한 타이틀에서 완성되지 않았습니다.** 잠입 시스템의 품질은 반복 횟수의 함수입니다.

**교훈 2 — 완벽을 목표하지 않고 실패를 문서화합니다.** Walsh는 자신의 해법이 1cm 임계값 문제를 갖는다고 명시하고, 그것이 **HUD를 노출하기로 한 결정의 직접적 결과**임을 진단합니다. 그리고 결론을 냅니다 — 완벽한 시각 모델이 있더라도 플레이어는 NPC가 봤어야 했다고 느끼거나 그 반대로 느낄 수 있으며, 이는 **고칠 수 없습니다.** 실제 삶에서도 누군가 무언가를 똑바로 보고 있는데 못 보는 것에 놀라기 때문입니다. **중요한 것은 일관된 모델, 좋은 피드백, 그리고 자기 게임의 플레이어 기대에 맞추는 것**입니다.

**교훈 3 — 인접 시스템의 재사용이 최고의 투자입니다.** TEAS는 원래 전투 중 NPC가 벽을 응시하는 문제를 풀려고 만든 것이었습니다. 그것이 소음 전파의 해법이 되었습니다.

**교훈 4 — 사회적 인지는 값싼 트릭으로 충분합니다.** '사라지는 NPC 문제'(플레이어가 하나씩 제거하는데 남은 NPC들이 동료의 부재를 모르는 것)에 대해, 팀은 직접 모델링(NPC별 목격/청취 이벤트 히스토리)이 **이득에 비해 작업량이 크다**고 판단하고, 훨씬 단순한 규칙을 택했습니다 — **NPC A가 NPC B의 가청 범위 안에 10초간 있었고, B가 소리를 멈춘 뒤(즉 죽은 뒤) 5초가 더 지나면 조사 이벤트를 생성.** 반복과 익스플로잇을 막기 위해 긴 쿨다운을 둡니다.

## 5-2. Mark of the Ninja (2012) — 정보 시각화의 철학

### 메커니즘

시야 판정은 **형상 ∧ 조명 ∧ 차폐 없음**의 논리곱, 가청은 **사운드 메시 A\* 경로 존재**, 자극 중재는 **단일 정수 우선순위(동률 4는 최신 우선)**, 관심사는 **한 번에 하나, 스택 없음**.

### 설계 교훈

**교훈 1 — 실패를 통과해야 이론에 도달합니다.** Klei의 창업자는 개발 초기에 이미 비전이 있었다고 진술합니다 — **진짜 닌자처럼 느껴지는 스텔스 게임, 유리 대포, 그림자 사이를 날아다니고 적은 당신이 거기 있는지조차 모르는.** 이름도 정해져 있었습니다. **문제는 거기에 어떻게 도달하는지 몰랐다는 것**입니다. 명암, 스텔스 킬, 수리검, 소리를 다 만들었는데 전부 **뭉개지고 형편없었습니다.** 너무 나빠서 한때는 **스텔스 게임을 못 만들겠다고 판단하고 두어 달간 '스텔스 요소가 있는 액션 게임'으로 방향을 틀었으며, 그것이 게임을 더 나쁘게 만들었습니다.**

**"비전이 있으면 괜찮다"는 통념에 대한 반례입니다.** 비전은 있었습니다. 없었던 것은 **검증 가능한 가설**이었습니다.

**교훈 2 — 이론이 명확해지면 결정이 스스로 풀립니다.** 팀이 검증 가능한 가설로 전환하고 다수의 시나리오를 만든 뒤 도달한 이론 — **요점은 관측·정보 획득·계획·실행이며 계획이 가장 중요하고, 계획에는 예측 가능한 인과가 필요하다** — 이 확립되자 나머지가 따라왔습니다: 이진 조명(명료하니까), 소음 링 표시(계획 지평이 늘어나니까), 멍청하고 조작 가능한 AI(플레이어가 이해해야 하니까), 평평한 지오메트리(어디로 갈 수 있는지 알아야 하니까).

**교훈 3 — 플레이테스트는 원하는 것을 묻는 게 아니라 가정을 검증하는 것입니다.** 그는 이를 명시합니다 — 데이터를 **가정이 맞았는지 틀렸는지 확인하는 데** 썼지 **무엇을 원하는지 묻는 데** 쓰지 않았습니다. 플레이어들이 실제로 원한다고 말한 것은 **더 세게 때리고 사람을 베는 것**이었습니다. 그것을 들었다면 스텔스 게임은 없었을 것입니다.

**교훈 4 — 정교한 AI를 자르는 것이 설계 결정입니다.** 스마트 AI를 잘라낸 것이 프로그래머 수개월분을 절약했고, **동시에 게임을 더 좋게** 만들었습니다. 잠입에서 AI의 정교함과 게임의 품질은 같은 방향이 아닙니다.

## 5-3. Thief: The Dark Project (1998) — 감각 모델의 원형

### 메커니즘

**정렬된 3D 뷰콘 집합**(XY각·Z각·길이·감도·경계도별 관련성), **첫 번째 콘만 평가**, **콘 내 위치 무관 상수 출력**. 소리는 **3D 지오메트리를 통해 의미론적 태깅과 함께 전파**되며 올바른 방향에서 감쇠된 인지값으로 도착합니다. **자유 지식(free knowledge)** 은 관심 대상이 펄스 생성을 멈출 때 AI 상태에 따라 스케일되어 주어집니다.

### 설계 교훈

**교훈 1 — 15년 앞선 설계가 여전히 최신입니다.** Thief의 뷰콘 집합과 Blacklist의 3중 형상 병용은 **같은 문제에 대한 같은 답**입니다 — 하나의 형상으로는 시각을 모델링할 수 없고, 역할이 다른 여러 형상과 중재 규칙이 필요합니다. 두 출처는 15년 간격이며 서로 인용하지 않습니다.

**교훈 2 — 단순화를 명시적으로 선언하십시오.** Leonard가 **"단순성과 게임플레이 튜닝 가능성을 위해"** 상수 출력을 쓴다고 쓴 것은 문서화의 모범입니다. 이 문장이 없으면 후대의 독자는 이를 충실도 모델의 근사로 오독합니다.

**교훈 3 — 소리를 다른 자극과 통일하십시오.** Thief는 소리를 **공간상의 위치에 대한 인지 관계**로 다른 자극과 통일합니다. 이 통일 덕에 3-4의 자극 중재 같은 구조가 가능해집니다 — 소리와 시각을 이질적 파이프라인으로 두면 중재할 수 없습니다.

**교훈 4 — 치팅을 설계 언어로 명명하십시오.** '자유 지식'이라는 이름은 이것이 **버그가 아니라 기능**임을 팀 전체에 알립니다. 목적도 명시됩니다 — **추론처럼 보이게 하되 노골적 치팅으로 보이지 않게.** 3-6의 2~3초 치팅과 같은 계열이며, Thief가 15년 먼저 도달했습니다.

## 5-4. Hitman — 사회적 공간과 저작형 가이드

### 메커니즘

**사회적 잠입**(변장 = 접근 권한), **레벨 = 사회적 공간의 팔레트**, **기회(opportunity) = 시스템형 샌드박스 위에 얹은 저작형 가이드**.

### 설계 교훈 A — 레벨을 사회적 공간으로

Io-Interactive의 레벨 디자이너는 리뷰를 분석하다 이상한 것을 발견했다고 진술합니다 — **가장 호평받은 레벨의 리뷰가 미션에 대해 거의 말하지 않고, 그 마을에 그냥 있는 느낌, 작은 이야기들, 이상한 캐릭터들, 방문할 만한 좋은 장소들을 묘사**하고 있었습니다. 그래서 질문을 바꿨습니다 — **"어떻게 멋진 레벨을 만드는가"에서 "어떻게 믿을 만한 일상을 설계하는가"로.** 기존 레벨 디자인 도구상자(트레스패스, 센트리)에는 이 질문에 답할 것이 없었습니다.

답은 사회인류학에서 왔습니다. **사회적 공간**과 **프론트스테이지/백스테이지** 개념입니다. 그가 도출한 6분할 팔레트는 다음과 같습니다.

| 공간 | 성격 | 플레이어에게 주는 것 | 설계 노트 |
|---|---|---|---|
| **공적 공간** | 광장·공원. 규칙도 기대도 없음 | 쉬고, 시간을 쓰고, **바보처럼 느끼지 않고 인벤토리를 열 수 있음** | 시작 위치에 적합. **플레이어의 초기 행동 반경을 표상** |
| **공적 목적 공간** | 뒷골목·막다른 길·시장. 소유자는 없으나 목적이 있어야 할 것 같음 | 방향 감각 | **생명감과 흐름**을 만드는 데 좋음 |
| **공적 규칙 공간** | 교회·아이스크림 가게. **강한 사회적 규칙** | 어떻게 행동해야 하는지 **말해 주지 않아도 앎** | 롤플레이에 최적. **트레스패스 규칙을 암묵적으로 전달** |
| **사적 공간** | 그냥 트레스패스. 모호하고 사람이 적음 | **숨 돌릴 틈** | 너무 채우지 말 것 |
| **사적 전문 공간** | 주방 등. 누가 허용되는지 명확 | 사회적 잠입과 롤플레이의 무대 | **주방은 사고가 나기 좋은 곳** |
| **사적 개인 공간** | 표적의 개인 공간 | **백스테이지 경험** — 환경으로 표적을 알려 줌 | **비싸고, 적어야 특별함** |

**결정적 발견**: 시작부터 전부 트레스패스인 레벨(즉 팔레트의 절반만 쓴 레벨)은 평가가 낮았고 비판을 받았습니다. 그가 진단하는 이유는 **영역을 구분하기 어렵고, 모든 것이 뿌옇고, 누가 어디에 허용되는지 알 수 없다**는 것입니다. 대비되는 레벨에서는 **아이스크림 장수는 여기, 신부는 저기**가 자명합니다.

**⚠️ 한정**: 강연자는 메타크리틱 점수를 근거로 들면서도 **"점수가 전부는 아니다"** 라고 명시합니다. 그리고 문제의 레벨을 **"훌륭한 레벨"** 이라고 옹호하며, 배운 것 덕에 이제 같은 종류의 레벨을 다시 만들 수 있다고 말합니다. **이 사례를 '실패'로 라벨링해서는 안 됩니다.**

**메타 교훈**: 이 작업의 가치는 발견 자체가 아니라 **형식화**에 있었습니다. 강연자는 그 레벨을 만든 사람들은 이 개념 없이도 잘 만들었으며, 이 작업은 **회사에 이미 있던 지식을 형식화한 것**이고 **신입 디자이너에게 접근 가능하지 않던 것을 접근 가능하게 만든 것**이라고 진술합니다. 그 결과 공유 용어(요새/거주자/배회자/달팽이집)가 생겼고 **팀이 자기가 무엇을 하는지 성찰하기 시작했습니다.**

### 설계 교훈 B — 저작형 가이드를 시스템형 샌드박스에 얹기

Io-Interactive는 클로즈드 알파에서 예상치 못한 피드백을 받았습니다. 플레이어가 큰 레벨을 어려워할 것은 예상했으나, **레벨이 텅 비었다고 느낀다**는 것은 예상하지 못했습니다. 콘텐츠는 거기 있었습니다 — **팀이 직접 넣었으니까.** 문제는 플레이어가 **빙산의 일각만** 보고 있다는 것이었습니다. 그래서 방향이 정해졌습니다 — **빙산을 비추되 목구멍에 밀어 넣지는 말 것.**

**튜토리얼의 실패와 전환.** 첫 접근은 기능별 **테스트 챔버**였습니다 — 각 방이 하나의 기능을 격리해 가르치고, 서로 영향을 주지 않게. 유저 테스트에서 플레이어는 각 기능을 이해했습니다. **그런데 실제 레벨에서 전혀 나아지지 않았습니다.** 진단은 정확합니다 — **튜토리얼이 게임을 대표하지 않았습니다.** 기능을 단순한 것들로 증류했지만 **다 합치면 복잡도가 폭발하는데 튜토리얼은 그것을 하지 않았습니다.**

전환의 만트라는 군대 격언에서 왔습니다 — **"훈련한 대로 싸운다(you fight as you train)."** 스트레스를 받으면 추론 능력을 잃으므로 **가능한 한 진짜에 가까운 훈련 시나리오**를 만들어야 하고, 게임이 샌드박스라면 **훈련도 샌드박스여야** 합니다. 그리고 팀 내부의 '선형·구획화된 튜토리얼' 사고를 깨기 위해 **이름을 프롤로그로 바꿨습니다.**

**"치팅 없음" 규칙**이 이 만트라의 논리적 귀결입니다 — 훈련장의 주인공은 실제 레벨에서와 **똑같이 실제적**이어야 합니다. 그렇지 않으면 훈련한 대로 싸울 수 없습니다.

**기회 시스템의 설계 원칙**(강연자는 **"이 원칙들을 전부 어겨 봤기 때문에 갖게 된 것"** 이라고 명시합니다):

1. **정보는 결코 진행의 전제조건이 되어서는 안 됩니다.** 실제로 어긴 사례가 있습니다 — 표적이 대화를 엿들었을 때만 전화기를 떨어뜨리게 만든 것. 빠르게 고쳤으며, **일관성이라는 규칙을 위배하기 때문**입니다. 그의 조언은 명확합니다 — **월드의 실제 사물을 쓰되 정보를 쓰지 마십시오**
2. **살해 전에 손을 놓으십시오.** 특정 방식의 살해를 지시하면 플레이어가 다른 것을 하는 순간 **기회를 실패 처리해야** 하며, 그것은 무리한 요구입니다
3. **가이드 없이도 항상 가능해야 합니다.** 가이드가 아무것도 지시하지 않고, **스토리가 아니라 레벨이 지시**하게

**핵심 교훈 1 — 가이드를 넣으면 그 가이드의 실패 조건을 설계해야 합니다.** 이것이 강연의 명시적 결론 중 하나입니다. 표적의 옷과 시체를 강에 빠뜨렸다면? 밝혀 줄 사람을 죽였다면? 그들의 해법은 **우아한 성능 저하**입니다 — 가이드가 스스로 닫히며 "더는 안내할 수 없다"고 말하되, **플레이어가 외우고 있다면 계속 진행할 수 있습니다.**

**핵심 교훈 2 — 샌드박스 가이드는 타협이며, 그래도 괜찮습니다.** 강연자는 이를 **해법이 아니라 타협**이라고 명시적으로 부릅니다. 매번 새 기회 아이디어가 나올 때마다 **샌드박스에 무엇을 하는가, 리플레이 가치에 무엇을 하는가**를 논의하며, 이는 한 번 정하고 끝나는 템플릿이 아니라 **상시적 재조정**입니다. 그리고 그는 이를 긍정합니다 — **타협은 실제로 꽤 훌륭하며, 무엇이 이 게임의 핵심 경험인지 계속 논의하게 만들고, 상시적 조정이 계속 날을 세우게 합니다.**

## 5-5. Third Eye Crime (2013) — 점유 맵의 직접 노출

### 메커니즘

점유 맵은 NPC의 '플레이어 위치에 대한 믿음'을 **단일 좌표 벡터가 아니라 공간에 대한 이산 확률분포**로 모델링합니다. 초록이 서술하는 알고리즘은 세 문장입니다.

> "Occupancy Maps are a simple representation for probabilistic object tracking, in which object positions are represented not as a vector but as a discrete probability distribution over space. Every time-step in which the target is not observed, the target's probability distribution is diffused. Locations that are visible to the observing NPCs have their probability content zeroed out. The result is a system that allows AI to search a space in an emergent, methodical, organic and unscripted way. (Isla 2005)."
> — Damián Isla, AIIDE 2013, p.206

**(1) 타깃이 관측되지 않는 매 타임스텝마다 확률분포를 확산(diffuse)시키고, (2) NPC의 가시 영역에 해당하는 위치는 확률값을 0으로 만들며, (3) 그 결과 AI가 공간을 창발적·체계적·유기적·비스크립트적으로 탐색합니다.**

이것이 1-3에서 규정한 **지식 모델**의 구체적 형태입니다. 감각 모델(3-1)이 "무엇을 지각하는가"를 답한다면, 이 세 규칙은 "지각하지 못한 것에 대해 무엇을 믿는가"를 답합니다. 그리고 3-3의 탐색 상태 기계와 대조하면 성격이 분명해집니다 — **탐색 지점을 스코어링해 고르는 대신, 확률장의 구조가 탐색 순서를 스스로 만들어 냅니다.**

**⚠️ 귀속 주의 (1)**: 초록은 이 알고리즘 서술을 **자신의 선행 문헌(Isla 2005)에 귀속**시킵니다("(Isla 2005)"). 즉 이 세 규칙은 **Third Eye Crime의 구현 기록이 아니라 기법 일반에 대한 요약**입니다.

**⚠️ 귀속 주의 (2)**: 여기서 **'망각'이 별도 타이머가 아니라 확산에 의한 확률 희석으로 자연 발생한다**는 해석, 그리고 **'연속 확률장 대 이산 상태 기계'** 라는 대비는 **원문에 없습니다.** 널리 인용되지만 이 출처가 뒷받침하지 않으므로 이 문서는 채택하지 않았습니다. 확산율·그리드 해상도 등 **수치도 전무**합니다(5-5 말미 참조).

> "Because he is telepathic, Rothko can actually see what is happening in the minds of the NPCs. What this means to the player is that the Occupancy Map which the NPCs use to track them is actually rendered at all times. The Occupancy Map therefore becomes a sort of 'heat map' indicating the threat level of various areas of the map. A bright red glow to the player indicates a dangerous area. Under the hood, that red glow actually represents high probability values in the Occupancy Map, meaning the NPCs are likely to search there for the player."
> — Damián Isla, AIIDE 2013, p.206

**표시되는 것은 AI 상태의 근사치가 아니라 AI가 참조하는 자료구조 그 자체**입니다. 붉은 발광은 점유 맵의 높은 확률값이며, **위협 히트맵**으로 기능합니다. 주인공이 텔레파시 능력자라는 설정이 이 UI 노출의 **서사적 정당화**로 쓰입니다.

### 설계 교훈

**교훈 1 — 서사가 UI를 정당화할 수 있습니다.** 점유 맵을 그냥 그리면 디버그 뷰이지만, 주인공이 텔레파시 능력자면 그것은 **초능력의 시각화**입니다. 같은 픽셀, 다른 의미. 잠입 UI의 노출 수위를 높이고 싶을 때 **픽션이 도구가 됩니다.**

**교훈 2 — 지식 모델을 노출하면 새 플레이 공간이 열립니다.** 감각 모델(시야 원뿔)을 노출하면 플레이어는 **회피**를 계획합니다. 지식 모델(점유 맵)을 노출하면 플레이어는 **기만**을 계획합니다 — NPC가 어디를 뒤질 것 같은지 알기 때문입니다. 이것이 4-2의 '발각 이후' 설계를 가능하게 하는 기술적 조건입니다.

**교훈 3 — 감각/지식 모델의 분리는 유용한 진단 축입니다.** 자기 게임의 잠입 AI가 정교한지 물을 때, 두 축을 나눠 물어야 합니다 — **무엇을 지각하는가**(감각)와 **지각한 것으로부터 무엇을 아는가**(지식). Isla의 2013년 진단은 다수의 게임이 전자에 투자하고 후자에 투자하지 않았다는 것입니다.

### ⚠️ 이 사례의 근거 한계 (필독)

이 절은 이 문서에서 **근거가 가장 약한** 절입니다.

1. **출처는 1페이지 확장 초록입니다.** 그리드 해상도·확산율·색 매핑/임계값 등 **구현 수치가 전무**하므로 이 인용에 **어떤 수치도 붙일 수 없습니다.**
2. **플레이테스트 데이터·측정치·평가 섹션이 전혀 없습니다.** 전부 **표명된 설계 의도**입니다.
3. **이해 상충**: Isla는 자사 상용 제품과 자신의 선행 기법을 동기화하는 중이며, 논문은 게임 출시 **이전**에 쓰였습니다("upcoming... soon to be released on iOS").
4. **'구조적 설계 결함'은 과장입니다.** Isla는 "limited options"라 했고 「Reinventing Stealth」 제하에 **긍정적 기회**로 프레이밍합니다. **'한계/제약'으로 순화하고 2013년 시점 평가로 날짜를 명기해야 합니다.**
5. **우선권 주장은 자기 신고입니다.** 기법의 기원은 로보틱스 문헌이며 이전에도 게임에 쓰였으나 이 게임이 **게임플레이의 핵심 구성요소로 쓴 최초**라는 주장은 논문 내 독립 검증이 없습니다.
6. **구현 세부는 이 논문이 아니라 저자의 선행 문헌(Isla 2005, AI Game Programming Wisdom 3)을 참조해야 합니다.** 이 문서는 그 문헌을 확보하지 못했습니다.

**독립 확인(출시)**: 독립 리뷰 2건이 출시된 게임에서 이 기능을 확인해 줍니다 — 하나는 **"적의 탐색 패턴을 붉은 하이라이트로 지도에 그리는 육감"**, 다른 하나는 발각 시 **"추격자들이 당신이 있을 거라 믿는 장소를 보여 주는 붉은 경로가 무대에 번져 나온다"** 고 서술합니다. 즉 **기능이 출시되었다는 사실 자체는 논문 밖에서 확인됩니다.**

---

# 6. 기획자 가이드라인

## 6-1. 잠입 설계 원칙

**원칙 1 — 플레이어 지각이 시뮬레이션 정확도를 이깁니다. 단 개연성이 경계를 긋습니다.**
NPC가 무엇을 들을 수 있어야 하는가는 무의미합니다. 플레이어가 NPC가 무엇을 들을 수 있다고 **생각하는가**만이 유의미합니다. **플레이어는 게임을 멈추고 프리캠을 켤 수 없기 때문**입니다. 동시에 이것은 절대 명제가 아닙니다 — 개연성이 무너지면 몰입이 무너지므로, **불공정하게 느껴지더라도 개연적인 결과를 택해야 하는 지점**이 존재합니다.

**원칙 2 — 똑똑함이 아니라 개연성을 목표하십시오.**
"멍청하지 않다는 것이 곧 똑똑하다는 뜻은 아니며, 항상 개연적이라는 뜻입니다." 정교한 AI는 잠입에서 자주 **역효과**입니다 — Klei는 스마트 AI를 잘라내 수개월을 절약하면서 **동시에 게임을 더 좋게** 만들었습니다.

**원칙 3 — 정보를 숨기는 것은 난이도가 아니라 계획 지평의 축소입니다.**
잠입의 페이로드는 계획입니다. 소음 반경을 숨기면 게임이 '뭉개지고' 몇 수 앞을 계획할 수 있는지 불명확해집니다. **명확하게 만들면 플레이어는 더 멀리, 더 높은 수준으로 계획합니다.**

**원칙 4 — 하나의 형상으로 시각을 모델링하려 하지 마십시오.**
원뿔은 거리에 따라 옆으로 넓어져 멀리 벗어난 플레이어까지 인지하게 만듭니다. 해법은 **역할이 다른 여러 형상과 중재 규칙**입니다. 15년 간격의 두 출처가 독립적으로 같은 결론에 도달했습니다.

**원칙 5 — 소음은 반경이 아니라 경로입니다.**
유클리드 거리는 벽을 통과합니다. 반경은 **후보를 추리는 필터이자 튜닝 노브**이고, 실제 가청은 **공간 위상을 따르는 경로**가 결정합니다. 두 개의 독립된 1차 출처가 같은 원리에 도달했습니다.

**원칙 6 — 치팅이 시뮬레이션보다 안전할 수 있습니다.**
속도 기반 외삽은 프롭 내부나 월드 밖 좌표를 만들 수 있습니다. "그냥 몇 초간 알게 해 주기"는 항상 유효한 상태만 만듭니다. **거친 근사가 정확한 모델보다 나은 경우가 있습니다.**

**원칙 7 — 파라미터는 독립적이지 않습니다.**
즉살 테이크다운이 있으면 청각을 관대하게 만들 수 없습니다. **하나의 관대함이 다른 곳의 익스플로잇이 됩니다.** 튜닝하기 전에 무엇이 그 값에 의존하는지 목록화하십시오.

**원칙 8 — 값을 쉽게 바꿀 수 있게 만드는 것이 값을 잘 고르는 것보다 중요합니다.**
Klei의 우선순위 상수는 **개발 중 여러 번 바뀌었고**, 저자가 강조하는 것은 값이 아니라 **"바꾸고 새 조합을 시도하기 쉽게 만든 것이 큰 도움이 되었다"** 는 사실입니다.

**원칙 9 — 흥미로운 구간은 전이입니다.**
무지도 발각도 아닌 **그 사이**가 페이로드입니다. 재미있는 부분은 겁먹는 순간이지 엄마에게 이르는 순간이 아닙니다.

**원칙 10 — 훈련한 대로 싸웁니다.**
튜토리얼이 격리된 테스트 챔버라면, 플레이어는 기능을 이해하고도 실제 레벨에서 나아지지 않습니다. **복잡도가 폭발하는 것이 게임의 본질이라면 훈련도 그래야 합니다.**

## 6-2. 시스템 설계 체크리스트

**탐지 모델**
- [ ] 시야 형상이 **여러 개**이고 각각의 **역할**(원거리 / 정면 속결 / 주변시)이 정의되어 있는가? 여러 형상에 동시에 속할 때의 **중재 규칙**이 있는가?
- [ ] 부분 노출을 **연속값**으로 다루는가? **자세가 탐지 임계값을 수정**하며 그 자세에 **속도 비용**이 있는가?
- [ ] 탐지 타이머의 변수가 **거리 단독**이 아니라 조명·NPC 상태 등을 포함하는가? **중간 상태**(봤다는 것은 알고 조사하러 감)가 있는가?
- [ ] 형상 가장자리의 **임계값 경직성**(1cm 문제)을 인지하고 있는가?

**소음 모델**
- [ ] 가청 판정이 **경로 기반**인가? 반경이 **필터·노브**로 쓰이는가, 가청 결정자로 쓰이는가?
- [ ] 같은 방/직결 시 **폴백**이 정의되어 있는가? 게임플레이 판정이 **오디오 미들웨어에 종속**되어 있지 않은가?
- [ ] 실외/개활지에서 이 모델이 성립하는가? (초크포인트 모델은 **실내 전제**)

**인지 상태·탐색·자극 중재**
- [ ] 자극 종류에 따라 탐색 모드가 **분기**하며, 각 모드의 **포기 조건**이 정의되어 있는가?
- [ ] 동시 조사 인원에 **제한**이 있는가? (제한은 난이도가 아니라 **플레이 어휘 생성기**)
- [ ] 표적 상실 시 **몇 초간의 의도적 치팅**이 있는가? AI가 관심사 **스택/큐**를 갖는다면 리셋이 보장되는가?
- [ ] 우선순위가 **단일 정수**이며 **동률 해소 규칙**(최신 우선)이 있는가? 진짜 서열이 있는 것과 없는 것을 **구분**했는가?
- [ ] 낮은 우선순위가 **인지에 진입조차 하지 않는** 규칙이 있는가? 값을 **런타임/데이터로 쉽게 바꿔 볼 수** 있는가?

**정보 시각화**
- [ ] AI 인지 상태가 **연속 진행도**로 노출되며, 그 미터가 **행동 가능한 시간의 창**을 정의하는가?
- [ ] HUD 외의 **대체 채널**(애니메이션·barks·아이콘)이 있는가? 감각 형상(시야·소음)을 **렌더링**하는가? 안 한다면 그 결정의 근거는?
- [ ] 표시된 정보의 오차 방향이 **플레이어에게 유리한 쪽**인가?

**용서 설계**
- [ ] 화면 밖 NPC의 지각이 감쇠되는가? 조건이 **화면 밖 ∧ 개연적 거리**의 논리곱인가?
- [ ] 용서가 **관대함**이 아니라 **지각의 정합성**으로 설계되었는가? **예외 규칙**이 아니라 **반경 배치**로 구현할 수 있는가?
- [ ] 어떤 관대함이 어떤 익스플로잇을 여는지 **목록화**했는가? 개연성이 무너지는 지점에서 **의도적으로 불공정을 감수**하는가?

**발각 이후**
- [ ] 발각이 **실패인가 국면 전환인가**를 명시적으로 결정했는가? 국면 전환이라면 **복귀 경로**가 있는가?
- [ ] **도피 공간**(규칙과 NPC로 채워지지 않은 곳)이 있고 **막다른 길이 없는가**? (스위스 치즈)
- [ ] AI 상태의 **쿨다운/리셋**이 개연적으로 정당화되는가?

**레벨과 가이드**
- [ ] 레벨에 **공적/사적 공간의 팔레트 전체**가 쓰였는가? **규칙 공간**으로 트레스패스를 암묵 전달하는가? **개인 공간**이 존재하되 **희소**한가?
- [ ] 가이드를 넣었다면 그 **실패 조건**을 설계했고, 실패 시 **우아하게 성능 저하**되는가?
- [ ] **정보가 진행의 전제조건**이 되어 있지 않은가? 가이드 없이도 **모든 것이 가능**한가?
- [ ] 튜토리얼이 **진짜 게임과 같은 복잡도**를 갖는가?

## 6-3. 디자인 함정과 해결책

| # | 함정 | 증상 | 근본 원인 | 해결책 |
|---|---|---|---|---|
| **1** | **시야 원뿔 맹신** | 멀리 있고 옆으로 벗어난 플레이어가 발각됨 | 원뿔은 거리에 따라 **측면으로 계속 확장** | 원거리는 **관 모양 박스**(확장 후 수축), 정면 속결은 원뿔, 주변시는 별도 형상. **가장 포괄적 형상**이 타이머 구간 결정 |
| **2** | **유클리드 소음** | 벽 너머로 들림 / 벽 뒤인데 안 들려서 이상함 | 반경은 **공간 위상을 모름** | **경로 기반**으로 전환(초크포인트 합산 또는 사운드 메시 A*). 반경은 **필터·노브**로 강등 |
| **3** | **사운드 엔진 위임** | 판정이 극도로 비쌈 + 오디오 미빌드 시 대량 오탐 | 게임플레이가 **오디오 파이프라인에 종속** | 가청 판정을 **자체 위상 그래프**로 분리 |
| **4** | **보이지 않는 경비병에게 발각** | "이유 없이 걸렸다"는 반복 불만 | **플레이어는 프리캠을 켤 수 없음** | 화면 밖 **∧** 개연적 거리인 NPC의 지각을 **특정 이벤트에 한해** 감쇠 + **피드백 채널**(barks) 강화 |
| **5** | **반경 일괄 하향으로 공정성 해결 시도** | 즉살 테이크다운으로 맵 질주 가능 / 경비병이 무반응해 보임 | **파라미터가 독립적이지 않음** | 전역 하향 금지. **조건부 감쇠**로 국소 해결 |
| **6** | **위치 외삽으로 추적 구현** | NPC가 프롭 안이나 월드 밖으로 이동 시도 | 순진한 외삽은 **유효 지점을 보장 못 함** | **2~3초간 그냥 알게 해 주는 치팅**으로 대체(단, 감각 기반 값이므로 자기 게임에 맞춰 튜닝) |
| **7** | **정교한 AI 추구** | 구현 지연 + 플레이어가 AI를 이해 못 함 | **정교함과 판독성은 같은 방향이 아님** | **멍청하고 조작 가능한 AI**로 전환. 목표는 똑똑함이 아니라 **개연성** |
| **8** | **정보를 숨겨 난이도 확보** | 플레이가 '뭉개짐', 계획이 아니라 시행착오가 됨 | 정보 은닉 = **계획 지평의 축소** | 감각 형상을 **렌더링**. 난이도는 공간 구성·자극 상호작용에서 확보 |
| **9** | **자연스러운 그림자 그라디언트** | 플레이어가 자신이 보이는지 모름 | 아트의 자연스러움과 **판독성이 충돌** | **이진 조명**(한 발이라도 빛에 있으면 보임). 조명은 배경이 아니라 **규칙** |
| **10** | **관심사 스택/큐** | 경비병이 과거 자극을 끝없이 되짚음 | '토끼굴' — 개연성을 좇다 복잡도 폭발 | **한 번에 하나, 스택 없음**. 해결 시 기본 상태로 리셋 |
| **11** | **시체를 최상위 우선순위로** | 시체 조사 중 뒤에서 오는 발소리를 무시함 | 서열이 있다는 **가정이 절반만 맞음** | 시체·소음 등을 **동률**로 두고 **최신 우선**으로 해소. "마지막에 알아챈 것이 가장 중요" |
| **12** | **발각 = 즉시 실패** | 계획 3분이 통보 없이 무효화됨 | 실패가 **이벤트**이지 구간이 아님 | **탐지 미터**로 기회의 창 부여 + AI **포기 조건** 정의 |
| **13** | **막다른 길** | 발각 후 플레이가 즉시 종료됨 | 도피 공간 부재 | **스위스 치즈** — 항상 빠져나갈 길. 규칙과 NPC가 없는 **안전 공간** 확보 |
| **14** | **팔레트 절반만 쓴 레벨** | 영역 구분이 뿌옇고 누가 어디 허용되는지 모름 | 전부 트레스패스 = **사회적 대비 상실** | **공적 규칙 공간**으로 암묵 전달, **개인 공간**을 희소한 보상으로 |
| **15** | **격리된 테스트 챔버 튜토리얼** | 기능은 이해하는데 실제 레벨에서 못 함 | **복잡도의 합성**을 가르치지 않음 | 훈련한 대로 싸움 — **작은 샌드박스**로 훈련. 훈련장에서도 **치팅 없음** |
| **16** | **가이드가 정보를 게이트** | 대화를 들어야만 아이템이 등장 | 일관성 위배 | **정보는 결코 진행의 전제조건이 되면 안 됨.** 월드의 실제 사물을 쓸 것 |
| **17** | **가이드가 살해 방식을 지정** | 플레이어가 딴짓하면 기회 실패 처리 | 저작형 가이드가 시스템을 침범 | **살해 전에 손을 놓을 것.** 가이드는 우아하게 닫히되 플레이는 계속 |
| **18** | **구체적 barks의 반복** | 기억에 남는 대사가 반복되어 몰입 붕괴 | **구체적일수록 기억에 남음** | **3계층 barks** — 구체 → 일반 → 완전 일반으로 하강. 학습 완료 후 정보 필요량은 감소 |
| **19** | **비살상을 선언만 함** | 특정 지점에서 비살상 플레이가 무너짐 | 모든 위협에 비살상 해법이 없음 | 병렬 해법 세트 + **정직한 비용**(살상이 더 빠름). 가이드가 살상을 전제하지 않게 |
| **20** | **사라지는 NPC 문제** | 동료가 전멸했는데 남은 NPC가 태연함 | 사회적 인지 미모델링 | **값싼 트릭으로 충분** — 가청 범위에 N초 함께 있었고 소리가 M초간 끊기면 조사 이벤트 + 긴 쿨다운 |

---

# 7. 결론 및 미래 전망

## 7-1. 잠입의 본질

잠입 장르의 핵심 자원은 **적의 무지**이며, 그 자원은 플레이어의 캐릭터 시트가 아니라 **NPC의 머릿속과 레벨의 지오메트리**에 있습니다. 이 하나의 사실이 장르의 모든 설계 결정을 규정합니다.

- 자원이 NPC 안에 있으므로 **AI 상태를 보여주는 것이 성립 조건**입니다(3-5)
- 자원이 레벨에 있으므로 **레벨 디자이너가 밸런스 담당자**입니다(3-7, 5-4)
- 자원이 관측으로만 파악되므로 **계획이 페이로드**입니다(1-2, 5-2)
- 페이로드가 계획이므로 **정보 은닉은 난이도가 아니라 페이로드의 축소**입니다(3-5)
- AI가 예측 가능해야 계획이 서므로 **잠입 AI는 실제 인간과 반대 방향으로 설계**됩니다(1-1)

그리고 이 모든 것의 위에 제1원칙이 있습니다 — **플레이어 지각 > 시뮬레이션 정확도, 단 개연성으로 경계를 둡니다.**

## 7-2. 확보된 근거가 말하는 것과 말하지 않는 것

이 문서의 근거는 **1998~2015년에 집중**되어 있습니다. 이는 세 가지를 의미합니다.

**첫째, 원리는 노후하지 않았습니다.** 15년 간격의 Thief와 Blacklist가 시야 형상에 대해 독립적으로 같은 결론에 도달했고, 서로 다른 스튜디오의 3D와 2D 구현이 소음 전파에 대해 독립적으로 같은 원리에 도달했습니다. **독립적 수렴은 원리의 강도를 보증합니다.**

**둘째, 대표성은 보장되지 않습니다.** 이들은 **특정 출시작에 대한 역사적 구현 기록**입니다. 2013년 이후 업계가 무엇을 하고 있는지에 대해 이 문서는 답하지 않습니다.

**셋째, 모든 개발사 진술은 '설계 의도'입니다.** 다만 Game AI Pro 챕터들은 **저자가 자신이 구현한 시스템을 서술**한 것이므로 실제 동작 쪽에 가깝고, Blacklist ch.28의 Figure 28.3은 **출시 빌드의 디버그 드로잉**이라 구현 증거로서 강합니다.

## 7-3. 장르의 전망

**전망 1 — 지식 모델이 다음 프론티어입니다.** Isla의 2013년 진단(감각 모델은 정교하나 지식 모델은 빈약)이 여전히 유효한지가 이 장르의 향방을 정합니다. 감각 모델은 이미 수렴했습니다 — 여러 형상 + 중재, 경로 기반 소음. 남은 것은 **NPC가 지각한 것으로부터 무엇을 아는가**입니다.

**전망 2 — '발각 이후'가 설계 공간으로 남아 있습니다.** 발각을 실패가 아니라 국면 전환으로 다루는 설계는 지식 모델을 요구하며, 지식 모델을 노출하면 플레이어는 회피가 아니라 **기만**을 계획하게 됩니다.

**전망 3 — 시스템형과 저작형의 긴장은 해소되지 않습니다.** Io-Interactive의 결론은 이것을 **해법이 아니라 상시적 재조정**으로 규정합니다. 이 긴장을 없애려는 시도보다 **매 결정마다 논의하는 프로세스**를 갖추는 편이 현실적입니다.

**전망 4 — 정보 시각화의 수위는 계속 올라갈 것입니다.** Mark of the Ninja가 감각 형상을 전면 노출했고 Third Eye Crime이 자료구조 자체를 렌더링했습니다. **정보를 숨기는 것이 난이도가 아니라는 인식**이 확산될수록 이 방향은 강화됩니다. 그리고 서사가 노출을 정당화할 수 있습니다.

## 7-4. 기획자를 위한 최종 조언

**첫째, 자기 게임의 잠입이 무엇을 위한 것인지 먼저 답하십시오.** 계획의 만족(순수 잠입)인가, 통제력 상실의 정서(공포 잠입)인가, 선택지의 하나(오픈월드)인가. 이 답이 예측 가능성을 얼마나 줄지를 정하고, 그것이 나머지를 정합니다.

**둘째, 검증 가능한 가설로 전환하십시오.** Klei는 비전이 있었고 이름도 있었지만 **도달하는 법을 몰랐고**, 가정 위에 기능을 쌓다 스텔스를 포기할 뻔했습니다. 전환점은 **"이 상자 뒤에 숨었을 때 경비병이 지나가면 은밀한 느낌이 들 것이다"** 같은 작은 가설을 만들어 테스트한 것이었습니다.

**셋째, 플레이테스트로 가정을 검증하되 원하는 것을 묻지 마십시오.** 플레이어가 원한다고 말한 것은 더 세게 때리는 것이었습니다.

**넷째, 잘라내는 것이 설계입니다.** 스마트 AI를 자른 것이 프로그래머 수개월을 절약하면서 게임을 더 좋게 만들었습니다. **무엇을 자를지 아는 방법은 검증된 이론을 갖는 것**입니다.

**다섯째, 완벽을 목표하지 말고 실패를 문서화하십시오.** Walsh는 자신의 해법의 1cm 문제를 명시하고 그것이 HUD 노출 결정의 직접적 결과임을 진단합니다. **중요한 것은 일관된 모델, 좋은 피드백, 그리고 자기 게임의 플레이어 기대에 맞추는 것**입니다.

---
## 부록: 출처 목록

이 문서의 수치·인용문·메커니즘 서술은 다음 출처를 **직접 열람하여 확인한 것만** 인용했습니다. 각 인용 지점에도 출처와 페이지를 병기했습니다.

### 확인한 출처 (WebFetch 직접 확인)

- **Martin Walsh(Ubisoft Toronto), 「Modeling Perception and Awareness in Tom Clancy's Splinter Cell Blacklist」** (Game AI Pro 2, ch.28, p.313-326) — 이 문서에서 **가장 강한 근거**입니다. 확인한 내용: 성공의 4요소(fairness/consistency/good feedback/intelligence)와 "not being dumb does not necessarily mean being smart; it means always being plausible", **원뿔의 측면 확장 문제**와 원뿔→박스(Conviction)→**관 모양 박스**(Blacklist) 계보, **Figure 28.1**(본 8개 레이캐스트, 자세별 필요 가시 본 개수, '엄폐 중'은 현재 보이는 2개보다 더 필요), **Figure 28.2**(타이머 구간 = 형상, 값 = 거리 선형 스케일, 성장하는 HUD, "window of opportunity to break LOS or kill the NPC"), **Figure 28.3**(라벨 Box/Cone/Peripheral, "The most inclusive shape the player is in defines the range of the detection timer"), **본문 p.316의 완전한 타이머 변수 목록**("distance to the player, lightness of the player, NPC state, etc."), **제3의 중간 상태**(p.314, "usually at some percentage of the progress bar"), **1cm 임계값 자기 진단**, **"many hours of playtesting and designer tweaking"**, TEAS 구조와 **식 28.1의 초크포인트 합산 + 워크드 예시**(A→window1→B→door2→C→door3→D), 사운드 엔진 이탈 사유 2건(비용·오디오 미빌드 시 오탐), **유클리드 폴백**, **"clearly a rough approximation... never looked back"**, **½ 청각 감쇠**와 조건(화면 밖 ∧ 개연적 거리), 간접 시각 이벤트 적용, **기각된 대안**(반경 하향 → 즉살 테이크다운 붕괴 + 무반응), **제1원칙 전문**("player perception should win... limited by plausibility"), **3계층 barks**, **변경된 오브젝트** 시스템과 리뷰 언급, **사라지는 NPC 문제의 10초/5초 해법**: http://www.gameaipro.com/GameAIPro2/GameAIPro2_Chapter28_Modeling_Perception_and_Awareness_in_Tom_Clancy's_Splinter_Cell_Blacklist.pdf
- **Brook Miles(Klei Entertainment), 「How to Catch a Ninja: NPC Awareness in a 2D Stealth Platformer」** (Game AI Pro, ch.32, p.413-421) — 확인한 내용: **시야 판정의 논리곱**("including these questions" — 비망라적임을 확인), **night vision 플래그**, **다른 시야 지오메트리**(단일 레이·원·사각형·사다리꼴), **Figure 32.1 캡션**(Guard B는 반경 안이나 경로 없어 못 들음), **소리 = 머리 / 시야 = 눈**의 앵커 차이, **사운드 메시**(레벨 익스포트 시 빈 공간 삼각형 메시, **Triangle by Jonathan Shewchuk**, 오디오 필터링에도 같은 경로 탐색 공유), **반경의 이중 목적**("For both performance and gameplay reasons"), **Listing 32.2의 12개 상수 전부**(LOWEST=0 … TERROR=20), §32.6의 **우선순위 규칙**과 **"near the end of the project" + "changed many times during development"**, **시체 우선순위 가정의 반전**과 **"The last thing you notice is probably the most important thing"**, 진짜 서열이 있는 것들의 예(깨진 항아리 vs 그림자/총성), **"too far down the rabbit hole"**과 단일 관심사 규칙, **관심사 소스 소멸(게임 오브젝트는 잔존)**, 발견자 목록, **rediscoverable suspect 소스**(표적 획득 거리보다 멀리서 감지): https://www.gameaipro.com/GameAIPro/GameAIPro_Chapter32_How_to_Catch_a_Ninja_NPC_Awareness_in_a_2D_Stealth_Platformer.pdf — ⚠️ **문서 전체에 각도(degree) 표기 0건**이며 반경은 미해석 심볼 상수(`RUN_ON_LOUD_RADIUS` 등)뿐입니다. **이 출처에서 시야각·거리 수치를 인용하는 것은 불가능합니다.**
- **Rich Welsh, 「Looking for Trouble: Making NPCs Search Realistically」** (Game AI Pro 2, ch.27, p.305-312) — 확인한 내용: **적대적/교란 자극 분기**, **표 27.1 원문 전체**(Walk/Run, "Slow, glancing around"/"Weapon raised, more alert", "After initial location searched (phase 1 only)"/"After all locations searched or time limit reached (phases 1 and 2)"), **§27.4의 3대 차이축**(속도·애니메이션·포기율), **§27.4.1.1의 완화**("**Often** with a cautious search..."), **§27.4.2의 "this is ultimately a design decision"**, **외삽의 실패**(프롭 내부·월드 밖 좌표 생성 불가 보장), **2~3초 치팅**("can be tweaked to suit the feel", Halo·Crysis·Crackdown 진술), **신중 1단계 1명 제한의 플레이 효과**(무리에서 떼어내기), **공격적 1단계 2~3명 제한**, **레이캐스트 1~2초 타임슬라이싱**, §27.3.2의 **표적 상실 → 공격적 탐색 라우팅**, **§27.4.2.3 「Selecting the Best Search Spot」의 스코어링 가중치**(가장 중요한 두 가중치 = 표적 추정 위치까지의 거리 + NPC 현재 위치까지의 거리 / 여러 NPC가 같은 풀에서 뽑을 때의 국소화 문제 / **플레이어의 실제 현재 위치까지의 거리에 추가 가중치**를 주어 "인간적 직관의 착각"을 부여하고 패턴을 부드럽게 유도 / **결정적 제약**: 나머지 둘에 비해 **미묘해야** 하며, 강하면 NPC들이 즉시 표적에게 몰려들어 **직관의 착각이 깨지고 게임이 불공정하게 짜인 느낌**을 줌): https://www.gameaipro.com/GameAIPro2/GameAIPro2_Chapter27_Looking_for_Trouble_Making_NPCs_Search_Realistically.pdf — ⚠️ **§27.2가 대상 타이틀을 "a title that I am unable to name"(CryENGINE 기반 TPS)로 익명 처리**합니다. **특정 게임 귀속 및 업계 보편 관행으로의 일반화 금지.** 이 문서가 이 절의 신뢰도를 낮게 둔 이유입니다.
- **Tom Leonard(Looking Glass Studios), 「Building an AI Sensory System: Examining the Design of Thief: The Dark Project」** (GDC 2003 / Game Developer 재게재) — 확인한 내용: **정렬된 3D 뷰콘 집합 전문**(XY각·Z각·길이·acuity/sensitivity·경계도별 relevance), **"only the first view cone the object is in is considered"**, **"For simplicity and gameplay tunability... constant output"**, **예시 AI의 뷰콘 5개**, **소리의 3D 지오메트리 전파와 의미론적 태깅**, **자유 지식(free knowledge)** 과 "the appearance of deduction... without overtly demonstrating cheating to the player": https://www.gamedeveloper.com/programming/building-an-ai-sensory-system-examining-the-design-of-i-thief-the-dark-project-i-
- **Damián Isla(Moonshot Games), 「Third Eye Crime: Building a Stealth Game Around Occupancy Maps」** (AIIDE vol.9, AAAI, 2013, p.206) — 확인한 내용: **점유 맵 알고리즘의 3규칙**(미관측 타임스텝마다 **확산** → NPC 가시 영역은 **확률값 0** → **창발적·체계적·유기적·비스크립트적 탐색**) — ⚠️ **초록 자신이 이 서술을 선행 문헌 Isla 2005에 귀속**시키므로 **Third Eye Crime의 구현 기록이 아니라 기법 일반의 요약**입니다. **텔레파시 설정과 점유 맵 상시 렌더링, 히트맵, 붉은 발광 = 높은 확률값**, **"the fun happens after, not before, the player is detected"**, **감각 모델 대 지식 모델 진단**과 "only two options"(무작위 탐색 or 순찰 복귀), **「Reinventing Stealth」 제하의 긍정적 프레이밍**, **우선권 주장**(로보틱스 기원, "first use as a core component of gameplay"), 참고문헌(Isla 2005 / Leonard 2003 / Moravec 1988): https://cdn.aaai.org/ojs/12663/12663-52-16180-1-2-20201228.pdf (랜딩: https://ojs.aaai.org/index.php/AIIDE/article/view/12663 — HTTP 200 확인) — ⚠️ **1페이지 확장 초록**입니다. **구현 수치·플레이테스트 데이터·평가 섹션이 전무**하며 **게임 출시 이전에 작성**되었습니다("upcoming... soon to be released on iOS"). 상세 한계는 5-5 참조.
- **Game Developer, 「Bringing Balance to Stealth AI in Splinter Cell: Blacklist」** — 확인한 내용: **관 모양 시야**("maintains clarity of vision in front but also prevents them from having strong peripheral vision at longer distances"), **8개 본 레이캐스트**("from its eyes to eight of the bones in Fisher's skeleton"), **화면 밖 청각 약화**("It doesn't make them deaf, but they just don't hear quite as well when the camera isn't looking at them"), **난이도별 형상 조정**("The size and shape of each cone is tweaked for each difficulty setting, allowing for more blind spots on lower difficulties"), **커스텀 커버 포인트로 디자이너가 가시성 수동 조정**: https://www.gamedeveloper.com/design/bringing-balance-to-stealth-ai-in-splinter-cell-blacklist — ⚠️ **이 글은 Ubisoft 공식 문서가 아니라 개발자 강연을 분석한 3자 해설**입니다. 이 문서는 이 출처를 **4-5의 난이도별 형상 조정 1건에만** 사용했고, 그 지점에 3자 해설임을 명기했습니다. **핵심 메커니즘은 전부 ch.28 원문으로 확인**했으며 이 글에 의존하지 않았습니다.
- **GDC Vault, 「Level Design in 'HITMAN': Guiding Players in a Non-Linear Sandbox」** — **세션 메타데이터를 자막이 아닌 경로로 확인**했습니다: 발표자 **Mette Poedenphant Andersen · Jacob Mikkelsen(Io-Interactive)**, 세션 초록(학습 곡선이 "a way too steep"였다는 유저 리서치 결과, 파리 레벨의 규모 이해 실패, "how to think and act like 47"). 이 문서에서 **발표자 이름과 소속은 이 메타데이터에서만** 가져왔습니다(자막에서 인용하지 않음): https://www.gdcvault.com/play/1023872/Level-Design-in-HITMAN-Guiding — ⚠️ **영상·슬라이드는 페이월(Members Only)로 접근하지 못했습니다.**
- **Jonathan Shewchuk, 「Triangle」** (CMU) — Mark of the Ninja의 사운드 메시 생성기로 ch.32가 지목한 도구. **URL 생존만 확인(HTTP 200)** 했고 내용은 이 문서의 서술에 사용하지 않았습니다: http://www.cs.cmu.edu/~quake/triangle.html

### 강연 자막(자동 생성)으로 내용 확인한 출처

아래 4건은 **로컬 자막 파일**로 내용을 확인했습니다. 자막은 **롤링 형식**(각 줄이 앞줄 꼬리를 반복)이므로 **겹침 제거 후** 읽었습니다.

- **[자막] 「Level Design in Hitman: Guiding Players in a Non-Linear Sandbox」**(Io-Interactive) — **5-4의 저작형 가이드 논증의 핵심 재료**입니다. 확인한 내용: 클로즈드 알파에서 **레벨이 "텅 비었다"** 는 예상 밖 피드백과 **"빙산의 일각"** 진단, **테스트 챔버 튜토리얼의 실패**(기능은 이해했으나 실제 레벨에서 나아지지 않음 — "복잡도가 폭발하는데 튜토리얼은 그것을 하지 않았다"), **"you fight as you train"** 만트라와 작은 샌드박스로의 전환, **'튜토리얼'→'프롤로그' 개명**(팀 내 선형·구획화 사고를 깨기 위해), **"치팅 없음"** 규칙(훈련장의 주인공도 실제와 동일), **Groundhog Day 프레이밍**(반복이 주는 통제력·권능감 vs 견딜 수 없는 패턴 인식 → 기회는 **선택적·유연·일관**이어야), **기회 = 퀘스트의 히트맨식 번역**(엿듣기로 드러내기, 이동 대신 침투, 텔레포트 금지), **3대 설계 원칙**("정보는 결코 진행의 전제조건이 되면 안 됨" + 실제로 어긴 전화기 사례 / "살해 전에 손을 놓을 것" / "가이드 없이도 항상 가능해야"), **가이드의 우아한 성능 저하**("가이드가 닫히지만 외우고 있다면 계속 가능"), **가이드 강도의 3단계 선택**, **지속 프롭·리빌 영속화**, **합판 요트 대 파리 패션쇼의 실패 비용 대비**, **종이 프로토타입 + 격주 래피드 프로토타이핑 + 유저 리서치 협업**, **"라이브 게임이면 일찍 라이브로"**, 결론부 **"가이드를 넣는다는 것은 실패 조건을 설계하는 것"** 과 **"샌드박스 가이드는 타협 — 그래도 괜찮다"**(상시 재조정이 날을 세운다), Q&A의 **기회 내 "살상 금지 규칙"**(플레이어가 사일런트 어새신을 유지하고 싶어 하므로).
- **[자막] 「Hitman Levels as Social Spaces: The Social Anthropology of Level Design」**(Io-Interactive) — **5-4의 사회적 공간 논증의 핵심 재료**입니다. 확인한 내용: **리뷰가 미션이 아니라 '그 마을에 있는 느낌'을 묘사**하고 있었다는 발견과 질문의 전환("어떻게 멋진 레벨을 만드는가" → **"어떻게 믿을 만한 일상을 설계하는가"**), 기존 레벨 디자인 도구상자에 이 질문의 답이 없었다는 진단, **사회적 자본·사회적 공간·프론트스테이지/백스테이지**의 도입, **공간이 암묵적 행동 규칙을 운반하며 사람들은 생각 없이 따른다**는 관찰, **6분할 팔레트**(공적 / 공적 목적 / 공적 규칙 / 사적 / 사적 전문 / 사적 개인)와 각각의 용도·비용, **전부 트레스패스인 레벨의 평가 저조**와 그 진단(영역 구분이 뿌옇고 누가 어디 허용되는지 알 수 없음) 및 **"점수가 전부는 아니다" + "훌륭한 레벨"이라는 강연자 자신의 유보**, **4대 목표**(크고 의미 있는 공적 공간 / 기대와 노는 규칙 공간 / 보상이 되는 개인 공간 / 팔레트 전체 사용), **형식화의 메타 가치**(이미 있던 지식을 신입에게 접근 가능하게 만듦, 공유 용어 발생, 팀이 자기 작업을 성찰하기 시작), **스위스 치즈**("항상 빠져나갈 길, 막다른 길은 금지"), **안전 공간을 통한 사회적 맥락 전환**("새 맥락의 NPC가 이전 사건을 모르는 것이 정당해짐")과 **"완벽한 해법은 없다"** 는 유보, **"real이 아니라 believable"**, 큰 공적 공간의 새 문제(압도감)와 다양성 요구.
- **[자막] 「How We Created Mark of the Ninja Without (Totally) Losing Our Minds」**(Klei Entertainment) — **5-2와 3-5의 정보 시각화 철학의 핵심 재료**입니다. 확인한 내용: **비전은 있었으나 도달법을 몰랐다**는 진술(이름까지 정해져 있었음), **가정 위에 쌓은 기능들이 "뭉개지고 형편없었다"**, **스텔스를 포기하고 '스텔스 요소가 있는 액션 게임'으로 두어 달 선회 → 더 나빠짐**, **검증 가능한 가설로의 전환**과 다수의 시나리오 제작, **핵심 이론**(관측 → 정보 획득 → 계획 → 실행, **계획이 가장 중요**, **예측 가능한 인과**가 전제), 이론 확립 후 결정들이 스스로 풀렸다는 진술, **이진 조명**(한 발이 빛에 있으면 보임, 아티스트의 자연스러운 그라디언트 본능을 기각, "플레이어가 항상 그림자 안인지 밖인지 앎"), **소음 링 표시 논쟁**("너무 쉬워지지 않나?" → **아니오, 숨기면 뭉개지고 몇 수 앞을 계획할 수 있는지 불명확해짐 → 명확하면 더 멀리, 더 높은 수준의 계획**), **멍청하고 조작 가능한 AI로의 전환**(복잡한 AI는 구현도 어렵고 플레이어가 이해 못 함; 스마트 AI를 자른 것이 프로그래머 수개월 절약), **감정 아이콘이 얼굴 애니메이션보다 결정적**(공포 상태/조사 상태를 모두가 앎), **완전히 평평한 지오메트리**(어디로 갈 수 있는지 알아야 하므로), **"플레이어에게 쉬울수록 게임에 좋다"**, **플레이테스트로 가정을 검증하되 원하는 것을 묻지 않음**(플레이어가 원한다고 말한 것은 "더 세게 때리고 사람을 베는 것"), **잘라내는 법은 검증된 이론을 갖는 것**.
- **[자막] 「Animating the Spy Fantasy in Splinter Cell: Blacklist」**(Ubisoft Toronto) — **4-1의 긴장·이완과 4-4의 비살상 지원의 재료**입니다. 확인한 내용: **"스플린터 셀은 AI를 조종하는 것"** 이라는 규정과 그에 대한 집착, **인지 상태가 테이크다운 변형을 자동으로 가름**(인지 시 = 위협 제압 / 미인지 시 = 소리 억제 + 엄폐물로 끌어들여 숨기기 — **같은 입력·같은 기준, AI 인지 상태 하나만이 연출을 가름**), **애니메이션 조합의 곱**(커버 종류 × 커버 방향 × AI 방향 × 살상/비살상 × 전투/은신)과 **"빙산의 일각"**, **살상/비살상의 병렬 세트**(살상 = 카람빗, 비살상 = 맨손/초크)와 **살상이 대체로 더 빠름**(플레이 스타일과 연결), **NPC에게 등을 돌릴 이유를 주기**(세수·상자 나르기·통화 중 서성임 — "스텔스 게임은 다들 그냥 등을 돌리고 있다"는 불만), **NPC를 친근하게 만들어 살해를 선택으로 만들기**, **전이 구간이 페이로드**("전투 전도 전투도 이미 좋았고 집착한 것은 중간의 전이" + **"재미있는 부분은 동생이 겁먹는 순간이지 엄마에게 이르는 순간이 아니다"**), **연쇄 반응 시연**(문 두드림 → 전구 터뜨림 → 부드러운 반응·"전구일 뿐이니 일하러 가라" → 재교란 시 **AI가 첫 번째를 기억** → 창밖으로 끌어냄 → 의심 → 공포 → 소통을 통한 패닉 → 수색 요청으로 뭉침 → **등이 자연스럽게 돌아감 → 플레이어가 이용**).

### ⚠️ 자동 생성 자막의 취급 규율

| 층위 | 신뢰도 | 이 문서의 취급 |
|---|---|---|
| **개념·논증·설계 주장** | **신뢰 가능** | 귀속 서술로 채택("Klei의 창업자는 …라고 진술합니다") |
| **수치·통계·비율** | ⚠️ **신뢰 불가** | **일절 인용하지 않음** |
| **고유명사·발표자 이름** | ⚠️ **왜곡됨** | **자막에서 인용하지 않음.** 필요한 경우 GDC Vault 메타데이터 등 **비ASR 경로로만** 확인 |
| **슬라이드의 도표·수치** | **자막에 부재** | 자막은 음성만 담으므로 **확인 불가** |

**이 코퍼스에서 확인된 실제 왜곡 사례**: (1) 다른 세션에서 제목의 **"restrain"이 "risk changing"으로** 인식됨. (2) 1985년 게임이 **"Super Mario World"로** 잘못 표기됨. (3) 이 문서가 읽은 자막에서도 **Klei가 "clay"로, 창업자 이름이 왜곡되어** 나타납니다. 이것이 **자막에서 이름과 수치를 인용하지 않는 규율의 직접적 근거**입니다.

**본문에 자막 수치가 하나도 없다는 것의 의미**: 4-4에서 애니메이션 개수를, 5-2에서 시나리오 개수를 **구체적으로 쓰지 않은 것은 자료가 없어서가 아니라 이 규율 때문**입니다. 그 수치가 필요하면 **강연 슬라이드나 GDC Vault 영상으로 별도 확인해야 합니다.**

**본문 수치의 출처는 전부 텍스트 원문(PDF)입니다**: 본 8개, 우선순위 정수값(0/1/2/4/5/10/20), 뷰콘 5개, 2~3초, 2~3명, 1~2초, ½ 감쇠, 10초/5초 — **전부 Game AI Pro 챕터와 Thief 기사의 원문**이며 자막이 아닙니다. **이 수치들은 위상이 다르므로 사용 가능합니다.**

### 접근 실패 및 미확인 출처

- **GDC Vault 영상·슬라이드** — **페이월(Members Only)** 로 접근하지 못했습니다. 무료 사용자는 최근 2년 콘텐츠의 30%만 접근 가능합니다. ✅ **이 한계는 로컬 강연 자막 4건으로 상당 부분 해소**되었으나, ⚠️ **슬라이드의 도표·수치·다이어그램은 여전히 미확인**입니다. 특히 **Figure 28.3에 상당하는 시각 자료를 강연 슬라이드에서 교차 확인하지 못했습니다**(단, 해당 그림은 챕터 PDF 원문으로 확인했습니다).
- **Walsh, GDC 2014 세션**(gdcvault.com/play/1020195, /play/1020436) — **접근하지 못했습니다.** 이 문서의 Blacklist 서술은 **전부 Game AI Pro 2 ch.28 원문**으로 확인했으므로 이 세션에 의존하지 않습니다.
- **「My GDC talk on Dishonored 2's holistic level design」**(youtube.com/watch?v=NO1lvc3ikXE) — **제목만 확인**했고 **내용을 확인하지 못했습니다.** 이 문서의 **2-5(이머시브 심 잠입)가 1차 출처 없이 정성 서술로 남은 직접적 원인**입니다. 이 영역을 보강하려면 이 출처를 먼저 확보해야 합니다.
- **Isla, D. 2005, 「Probabilistic Target-Tracking and Search Using Occupancy Maps」**(AI Game Programming Wisdom 3) — **확보하지 못했습니다.** 점유 맵의 **구현 세부는 AIIDE 초록이 아니라 이 문헌을 참조해야 합니다.** 5-5의 수치 공백의 원인입니다.
- **Moravec 1988, 「Sensor Fusion in Certainty Grids for Mobile Robots」**(AI Magazine 9(2): 61-74) — 점유 맵의 로보틱스 기원으로 지목된 문헌. **확보하지 않았습니다**(이 문서의 서술에 불필요).

### 출처 이용 시 유의사항

**0. 실무 메모 — PDF 출처는 로컬 추출로 읽어야 합니다.** WebFetch는 **PDF 바이너리를 파싱하지 못하고** 원문 대신 "읽을 수 없다"는 응답을 돌려줍니다. 이 문서의 핵심 근거 4건(Game AI Pro 챕터 3건 + Isla 초록)은 전부 PDF이므로, **파일을 내려받아 로컬에서 텍스트를 추출한 뒤 읽었습니다**(pymupdf 등). **PDF 출처를 WebFetch 결과만으로 "확인 불가"나 "근거 없음"으로 판정하면 거짓 음성이 발생합니다.** 후속 작업자는 이 우회를 먼저 적용하십시오.

**1. 수치의 성격 — 무엇이 측정치이고 무엇이 감각값인가**

| 수치 | 성격 | 귀속처 |
|---|---|---|
| **본 8개** | **구현치** | Blacklist(Figure 28.1). 단 **Blacklist의 값이지 표준이 아님** |
| **우선순위 정수(0~20)** | ⚠️ **'프로젝트 후반 기준 값'.** 출시 값이 아니며 개발 중 여러 번 변경됨 | Mark of the Ninja(Listing 32.2) |
| **뷰콘 5개** | **예시 AI의 값** | Thief(Figure 4 설명) — 표준 아님 |
| **½ 청각 감쇠** | ⚠️ **플레이테스트 기반 튜닝값.** "certain events"에 한정 | Blacklist |
| **2~3초 치팅** | ⚠️ **감각 기반 휴리스틱**("can be tweaked to suit the feel"). 측정·검증되지 않음 | 익명 TPS(ch.27) |
| **2~3명 / 1~2초** | ⚠️ 저자가 "works well"이라 평가한 **단일 구현 값** | 익명 TPS(ch.27) |
| **10초 / 5초**(사라지는 NPC) | **구현치** | Blacklist |
| **관 모양 박스 형상** | ⚠️ **플레이테스트 산물**("many hours of playtesting and designer tweaking") — 시각 모델에서 유도되지 않음 | Blacklist |

**결정적 정합성 검사**: Walsh 자신이 자기 해법의 **1cm 임계값 문제**를 인정하고, 그것이 **HUD를 노출하기로 한 결정의 직접적 결과**라고 진단합니다. 즉 **이 수치들 중 어느 것도 '올바른 값'이 아니며, 전부 특정 게임의 특정 피드백 설계에 맞춰 튜닝된 값**입니다. 자기 게임에 그대로 이식할 수 있는 상수가 아닙니다.

**2. "게임이 즉시 재밌어졌다"는 측정치가 아닙니다.** Blacklist의 ½ 감쇠 효과에 대한 진술("The result was that the game instantly became more fun and our creative director stopped complaining")은 **주관적 자기 보고**입니다. 이 문서는 이를 **효과의 증거가 아니라 팀의 판단 근거**로만 인용했습니다.

**3. 익명 타이틀 문제 (ch.27).** 3-3과 3-6의 일부는 **대상 타이틀이 익명 처리된 단일 구현**에 근거합니다. 이 문서는 해당 절의 신뢰도를 **medium**으로 명시하고 **특정 게임에 귀속하지 않았으며 업계 보편 관행으로 제시하지 않았습니다.**

**4. 시점 민감성 (2026-07-16 기준)**

| 출처 | 시점 | 유의사항 |
|---|---|---|
| **Thief 감각 시스템** | 1998 / 2003 문서화 | 원리는 유효하나 **28년 전 구현** |
| **Mark of the Ninja** | 2012 | Listing 32.2는 **'프로젝트 후반' 값** |
| **Splinter Cell: Blacklist** | 2013 | Figure 28.3은 **출시 빌드 디버그 드로잉** — 구현 증거로 강함 |
| **Third Eye Crime** | 2013 | ⚠️ **게임 출시 이전에 쓰인 초록**. Isla의 장르 진단은 **2013년 시점 평가** |
| **Hitman** | 2016 / 2018 | 에피소딕 라이브 운영 맥락 — 강연자 스스로 **"계속 조정 중"** 이라 명시 |

**⚠️ 전체 대표성**: 확보된 1차 출처는 **1998~2018년**에 분포하며 핵심 근거는 **2012~2013년에 집중**되어 있습니다. **현행 업계 관행의 대표성은 보장되지 않습니다.**

**5. 개발사 진술의 성격.** **모든 개발사 진술은 '설계 의도'이며 독립 감사된 '실제 동작'이 아닙니다.** 단 Game AI Pro 챕터 3건은 **저자가 자신이 구현한 시스템을 서술**한 것이므로 실제 동작 쪽에 가깝습니다. Third Eye Crime의 초록은 **가장 먼 쪽**(출시 전, 이해 상충, 측정 없음)입니다.

**6. 원문에 근거가 없어 채택하지 않은 주장.** 점유 맵과 관련해 널리 인용되는 두 가지 서술 — **'망각'이 별도 타이머가 아니라 확산에 의한 확률 희석으로 자연 발생한다**는 해석과 **'연속 확률장 대 이산 상태 기계'** 라는 대비 — 는 **Isla 초록 원문에 존재하지 않습니다.** 초록이 담은 것은 3규칙(확산 / 가시 영역 0화 / 창발적 탐색)까지이며, 그 이상의 해석은 이 출처가 뒷받침하지 않습니다. ⚠️ 또한 초록은 **3규칙조차 선행 문헌(Isla 2005)에 귀속**시키므로, **Third Eye Crime의 구현 근거로 인용해서는 안 됩니다.** 구현 세부와 수치는 Isla 2005를 확보해야 합니다.

**7. 이 문서가 원문에 없는 것을 표시한 지점.** 3-5의 '소음 링이 실제 가청보다 넓게 표시된다'는 관찰은 **ch.32의 두 서술(반경 필터 + Figure 32.1)과 강연 자막(링 표시)을 대조한 이 문서의 분석**이며, 어느 출처에도 이 형태로 서술되어 있지 않습니다. 해당 지점에 그 사실을 명기했습니다.

---

# 관련 문서 (See Also)

- [AI / NPC 행동 & 적/몬스터 난이도 설계](121-ai_npc_behavior_difficulty.md)
- [레벨/스테이지 구조 & 목표·지형 배치](130-level_structure_layout.md)
- [HUD & 피드백(이펙트·사운드·알림·버튼 반응)](171-hud_feedback_systems.md)
- [장르 분류 가이드라인](032-Genre-Guide.md)
- [기획 요소 리뷰 목록](033-Review.md)
