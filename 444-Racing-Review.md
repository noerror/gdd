---
title: 레이싱 게임 분석
description: 레이싱 게임의 주행 물리와 타이어 그립 모델, 러버밴딩과 캐치업 AI, 트랙 설계를 분석하여 물리 충실도와 접근성의 균형을 최적화하는 레이싱 디자인 리뷰 가이드
---

# 레이싱 게임 분석

1. **개요**: 레이싱 게임의 정의와 핵심 특징, 설계 철학
2. **유형별 분류**: 시뮬레이션, 심케이드, 아케이드, 카트/파티, 오픈월드 레이싱, 러너/모바일
3. **핵심 메커니즘**: 주행 물리와 타이어 그립, 핸들링 어시스트, 러버밴딩과 캐치업 AI, 트랙 설계, 소모 자원, 드리프트, 틱레이트와 결정론
4. **플레이 경험 설계**: 속도감 연출, 어시스트 사다리, 공정성과 재미, 리플레이·고스트, 접근성
5. **주요 사례 분석**: 대표 게임들의 설계 결정과 교훈
6. **기획자를 위한 디자인 가이드라인**: 설계 원칙, 체크리스트, 함정과 해결책
7. **결론 및 미래 전망**: 장르의 진화와 발전 방향

---

> **이 문서의 출처 원칙**
>
> 이 문서에 등장하는 수치와 주장은 부록의 출처 목록에서 직접 확인한 것만 기재합니다. 개발사가 자사 제품에 대해 표명한 진술은 예외 없이 **귀속 서술**("…라고 밝혔습니다")로 처리하며, 사실 단정과 구분합니다. 확인하지 못한 수치는 정성 서술로 대체했고, 검증에 실패했거나 출처 간 진술이 충돌하는 항목은 부록의 "출처 이용 시 유의사항"에 그 사실을 명시했습니다.

---

# 1. 개요

## 1-1. 레이싱 게임의 정의

레이싱(Racing)은 플레이어가 탈것을 조종하여 **정해진 경로를 다른 경쟁자보다 빠르게 통과하는 것**을 목표로 삼는 장르입니다. 표면적으로는 "먼저 도착하기"라는 단일 승리 조건을 가진 단순한 장르로 보이지만, 실제 설계의 난이도는 이 단순한 목표를 떠받치는 두 개의 층에서 발생합니다.

첫째 층은 **탈것의 거동을 결정하는 물리 모델**입니다. 가속, 제동, 선회는 모두 타이어와 노면 사이에서 발생하는 마찰의 결과이며, 이 마찰을 어떤 해상도로 모사하느냐가 게임의 정체성을 결정합니다. 둘째 층은 **경쟁 상대를 어떻게 배치하고 관리하느냐**입니다. 실제 모터스포츠는 실력 차가 그대로 시간 차로 누적되어 중계 화면에서 경쟁자가 사라지는 일이 흔하지만, 게임은 그 상태를 그대로 두면 재미가 붕괴합니다.

이 두 층의 긴장 관계가 레이싱 장르의 본질입니다. 물리를 정교하게 만들수록 숙련자의 표현력은 올라가지만 진입 장벽도 함께 올라갑니다. 경쟁을 인위적으로 조율할수록 매 순간의 긴장감은 유지되지만 플레이어가 "속았다"고 느낄 위험이 커집니다. **레이싱 게임 설계는 이 두 축 위에서 자기 좌표를 고르는 작업**이며, 어느 한쪽 끝이 정답인 장르가 아닙니다.

### 물리 충실도는 목표가 아니라 선택지입니다

레이싱 장르를 다룰 때 가장 흔한 오해는 "시뮬레이션이 상위 개념이고 아케이드는 그것을 덜어낸 하위 개념"이라는 인식입니다. 이 인식은 실무자의 서술과 어긋납니다.

게임 개발자 리비오 데 라 크루즈(Livio De La Cruz)는 2016년 8월 12일 Game Developer 기고문에서 레이싱 구현을 물리 기반 시뮬레이션과 아케이드 방식의 두 갈래로 나누면서, 두 방식이 **구성 방향 자체가 반대**라고 서술합니다. 아케이드 방식에 대해 그는 "you basically start with nothing, and then build individual mechanics until you end up with the right amount of complexity that you're looking for"라고 기술합니다. 즉 아케이드는 시뮬레이션에서 복잡도를 덜어낸 결과물이 아니라, **아무것도 없는 상태에서 필요한 메커닉만 골라 쌓아 올린 별개의 축조물**이라는 것입니다. 같은 글은 이 둘이 깔끔한 이분법이 아니며 하이브리드 모델이 유효하다는 점도 함께 밝히고 있습니다.

이 관점은 실무적으로 중요합니다. 아케이드 게임을 만들면서 "시뮬레이션을 먼저 만든 뒤 쉽게 만들자"고 접근하면, 결과물은 대개 시뮬레이션의 복잡성은 남고 아케이드의 명료함은 없는 어중간한 물건이 됩니다.

### 장르 발전의 주요 이정표

**여명기 (1970년대 후반 - 1980년대)**:
- 의사 3D 스프라이트 스케일링으로 속도감을 연출하던 시기
- 코스는 분기 없는 단일 경로, 경쟁자는 장애물에 가까운 존재
- 제한 시간 연장이 곧 목표였던 아케이드 과금 구조가 설계를 규정

**3D 전환기 (1990년대)**:
- 폴리곤 렌더링과 실시간 물리의 도입
- 드리프트를 조작 문법으로 승격시킨 아케이드 계보의 확립
- 시뮬레이션 계보의 분기 — 타이어·서스펜션 모델의 도입

**온라인·시뮬레이션 성숙기 (2000년대 - 2010년대)**:
- 랭킹·리플레이·고스트가 표준 기능으로 정착
- 오픈월드 레이싱의 부상 — 트랙이 아니라 세계를 주행하는 구조
- 심케이드의 확립 — 시뮬레이션 외피에 어시스트 사다리를 결합

**데이터·학습 기반기 (2020년대)**:
- 머신러닝 기반 AI 드라이버 연구의 등장 (5-1 참조)
- 라이브 서비스와 시즌제 운영의 결합
- 단일 월드형 트랙 구조의 실험 (5-3 참조)

## 1-2. 레이싱 게임의 핵심 특징

### 접지 한계 위의 조작

레이싱의 조작은 대부분의 액션 장르와 달리 **"입력이 곧 결과가 아닌"** 구조를 가집니다. 핸들을 더 꺾으면 더 도는 것이 아니라, 어느 지점을 넘으면 그립을 잃고 오히려 덜 돕니다. 가속 페달을 더 밟으면 더 빨라지는 것이 아니라, 코너 탈출에서는 스핀합니다. 이 **비선형성**이 레이싱을 숙련의 장르로 만드는 핵심입니다.

소니 AI 연구진이 2022년 2월 9일 Nature에 게재한 논문은 이 과제를 "drivers must execute complex tactical manoeuvres to pass or block opponents while operating their vehicles at their traction limits"라고 규정하며, 주행 과제를 **차량을 접지 한계(traction limit)에서 운용하는 것**으로 정의합니다. 이 정의는 시뮬레이션에만 해당하는 것이 아닙니다. 카트 게임의 드리프트 역시 그립을 의도적으로 잃고 회복하는 행위이며, 접지 한계라는 개념을 조작 문법으로 번역한 결과물입니다.

### 경쟁의 인위적 관리

레이싱은 **경쟁 상대의 성능을 실시간으로 조율하는 것이 장르 표준 기법으로 자리 잡은 드문 장르**입니다. 이 기법을 러버밴딩(rubber-banding)이라 부르며, 3-3에서 상세히 다룹니다.

이는 다른 장르와 비교하면 특이한 위치입니다. 격투 게임에서 상대 캐릭터의 체력을 몰래 조정하면 그것은 부정입니다. 그러나 레이싱에서는 AI 차량의 속도를 플레이어와의 거리에 따라 조정하는 것이 **정상적인 설계 관행**으로 취급됩니다. 이 비대칭은 레이싱의 승리 조건이 "상대를 제압하는 것"이 아니라 "먼저 도착하는 것"이며, 경쟁자가 시야에서 사라지면 게임 자체가 성립하지 않는다는 구조적 특성에서 비롯됩니다.

### 반복 주행과 라인 최적화

레이싱의 숙련은 **같은 구간을 반복 주행하며 최적 라인을 찾아가는 과정**으로 구성됩니다. 이 특성이 리플레이, 고스트, 랭킹 같은 장르 고유의 기능을 낳았습니다. 플레이어는 자신의 과거 주행과 경쟁하며, 이는 다른 플레이어가 없어도 성립하는 자족적 동기 구조입니다.

### 속도감이라는 연출 과제

레이싱은 **실제 속도가 아니라 지각된 속도**를 설계하는 장르입니다. 같은 시속에서도 시야각, 카메라 흔들림, 노면 텍스처의 흐름, 엔진음의 피치, 모션 블러에 따라 체감 속도는 크게 달라집니다. 4-1에서 다룹니다.

### 트랙이라는 고정 콘텐츠

레이싱의 콘텐츠는 대부분 **트랙**이라는 형태로 공급됩니다. 트랙은 절차적 생성과 궁합이 나쁜 콘텐츠입니다. 코너의 배치, 시야의 확보, 추월 지점의 설계는 모두 정밀한 의도를 요구하며, 무작위 생성은 대개 "달릴 수는 있지만 재미없는" 결과를 냅니다. 이 때문에 레이싱은 수작업 트랙 제작 비용이 콘텐츠 볼륨을 직접 제약하는 장르입니다.

## 1-3. 레이싱의 설계 철학

### 철학 1: 충실도는 수단이며 재미가 목적입니다

물리 충실도를 높이는 것이 자동으로 좋은 게임을 만들지 않습니다. 어밸런치 스튜디오(Avalanche Studios)의 해미시 영(Hamish Young)이 GDC 2019에서 발표한 세션 "Vehicle Physics and Tire Dynamics in 'Just Cause 4'"의 공식 세션 설명은, 발표자가 준경험적(semi-empirical) 매직 포뮬러 타이어 모델을 다루면서 **"why it may not be optimal for arcade and 'simcade' handling"**을 검토하고, 준경험적이지 않으면서 파라미터가 현저히 적고 더 플레이어 친화적인 결과를 내는 디자이너 친화적 접근을 **제안(propose)**한다고 기술합니다.

초록은 여기서 멈추지만, 강연 본문은 그 헷지 뒤의 논증을 드러냅니다. 영이 든 대표적 근거는 **언더스티어**입니다 — 실차에서 제동 중 언더스티어는 자연스러운 현상이지만, 플레이어가 T자 교차로에 접근해 코너를 돌지 못하는 것은 오픈월드 게임에서 **받아들일 수 없는 결과**라고 그는 설명합니다. 이 논증의 구조에 주목하십시오. 실측 모델을 배제하는 근거가 **연산 비용이 아니라 설계 목표 자체**입니다. Just Cause 4의 목표 중 하나는 영이 **제한된 인지 부하(limited cognitive load)**라 명시한 것 — 고속 주행 중에도 운전 외의 것을 생각할 수 있어야 한다 — 이었고, 이 목표 아래에서 "정확한 타이어"는 개선이 아니라 **결함**입니다.

**충실도가 목적이 아닐 때, 정확한 모델은 더 나은 도구가 아니라 틀린 도구입니다.** 파라미터 수는 이 판단의 부산물입니다 — 실측 데이터에 피팅하기 위한 계수들은 실측 대상이 존재하지 않는 가상의 탈것 앞에서 의미를 잃고, 디자이너의 반복 속도만 갉아먹습니다. 다만 이것은 **한 팀의 실무자 회고이지 통제된 비교가 아니므로**, 이 문서는 영의 진술을 귀속 서술로 다루고 일반 명제로 승격시키지 않습니다 (5-2, 부록 유의사항 6·7 참조).

### 철학 2: 공정성은 사실이 아니라 지각입니다

러버밴딩 설계의 핵심 통찰은, 문제가 **AI가 도움을 받는다는 사실 자체가 아니라 플레이어가 그것을 알아채는 것**이라는 점입니다.

닉 멜더(Nic Melder)는 Game AI Pro 42장 "A Rubber-Banding System for Gameplay and Race Management"에서 이렇게 기술합니다. "When done correctly, this effect can be unnoticeable to the player while keeping the racing feeling close and exciting. However, when done poorly, it can leave the player feeling cheated, especially if the AI, racing in the same car as they, obviously has more speed."

즉 같은 기법이 잘 적용되면 보이지 않고, 잘못 적용되면 배신감을 남깁니다. **차이는 기법의 유무가 아니라 은폐의 품질입니다.** 이 관점은 3-3 전체를 관통하는 원리이며, 4-3에서 윤리 문제로 다시 다룹니다.

### 철학 3: 물리의 부산물은 버그가 아니라 설계 분기점입니다

물리 시뮬레이션은 설계자가 의도하지 않은 거동을 만들어냅니다. 이때 선택지는 두 가지입니다 — 제거하거나, 콘텐츠로 승격시키거나.

데 라 크루즈는 앞의 기고문에서 모터스톰(MotorStorm)의 사례를 듭니다. 이 게임의 물리는 작은 노면 요철에서도 차량이 접지를 잃기 쉬웠는데, 그는 에볼루션 스튜디오(Evolution Studios)가 이를 "그냥 일어나는 일"로 취급하는 대신 **"designing tracks that explicitly called attention to this mechanic, thus challenging players to master it"** 하는 쪽을 택한 것으로 보인다고 서술합니다. 다만 이는 기고자가 외부에서 추론한 의도이지 스튜디오의 진술이 아니므로, "트랙 설계가 물리를 뒤따를 수 있다"는 설계 교훈으로 읽되 에볼루션의 공식 입장으로 인용해서는 안 됩니다.

### 철학 4: 필수 메커닉은 존재하지 않습니다

레이싱 장르에는 반드시 있어야 하는 단일 메커닉이 없습니다. 데 라 크루즈는 각기 "핵심"으로 여겨지는 메커닉을 생략한 채 출시된 타이틀들을 열거하며 이를 반박합니다. "Not every game needs a tire slip mechanic (see Mario Kart), or a drifting mechanic (see Rocket League), or a boost button (see TrackMania), or a throttle for shifting gears (see F-Zero)."

이 목록의 함의는, **레이싱 설계에서 메커닉은 상속받는 것이 아니라 선택하는 것**이라는 점입니다. 기획 초기에 "레이싱이니까 드리프트는 넣어야지"라는 식의 관성적 결정은 근거가 없습니다.

---

# 2. 게임 유형별 분류

레이싱 하위 장르는 **물리 충실도**와 **접근성**이라는 두 축이 만드는 스펙트럼 위에 분포합니다. 아래 분류는 이 스펙트럼 위의 대표적 좌표들이며, 경계는 연속적입니다.

## 2-0. 스펙트럼 개관

| 유형 | 물리 충실도 | 진입 장벽 | 경쟁 관리 | 대표 좌표 |
|------|------------|----------|----------|----------|
| 시뮬레이션 | 최상 | 최상 | 최소 (실력 그대로 반영) | iRacing, Assetto Corsa |
| 심케이드 | 상 | 중 (어시스트로 조절) | 중 (난이도 슬라이더) | Forza Motorsport, Gran Turismo |
| 아케이드 | 중~하 | 하 | 상 | 리지 레이서, 번아웃 |
| 카트/파티 | 하 | 최하 | 최상 (아이템+캐치업) | 마리오 카트 |
| 오픈월드 레이싱 | 중 | 하~중 | 중~상 | 니드 포 스피드, 포르자 호라이즌 |
| 러너/모바일 | 최하 | 최하 | 해당 없음~상 | 모바일 엔드리스 레이서 |

이 표에서 읽어야 할 것은 **물리 충실도와 경쟁 관리가 대체로 역상관**이라는 점입니다. 물리가 정교할수록 경쟁을 인위적으로 조율할 여지가 줄어듭니다. 파워를 조작하면 물리 시뮬레이션이 그 조작을 드러내기 때문입니다(3-3-4 참조). 반대로 물리가 추상적일수록 캐치업을 숨길 공간이 넓어집니다.

## 2-1. 시뮬레이션 (Simulation)

### 메커니즘

- 타이어의 슬립 앵글·슬립 비율에 따른 비선형 마찰력 산출
- 서스펜션 지오메트리, 하중 이동, 에어로다이내믹 다운포스
- 타이어 온도·공기압의 동적 변화와 그에 따른 그립 변동
- 연료 소모에 따른 질량 변화와 밸런스 변화
- 실제 차량 데이터에 기반한 파워트레인·기어비 모델

### 특징 및 차별점

시뮬레이션 계보의 핵심은 **"플레이어의 실수를 게임이 보정하지 않는다"**는 계약입니다. 코너 진입 속도를 잘못 잡으면 그대로 코스를 이탈하며, 게임은 개입하지 않습니다. 이 비관용성이 곧 이 하위 장르의 상품 가치입니다.

경쟁 관리 측면에서도 시뮬레이션은 러버밴딩을 최소화하거나 배제하는 경향이 있습니다. 대신 **주행 실력 자체로 상대를 매칭하는 방식**(레이팅 시스템, 스플릿 분할)을 택합니다. 이는 러버밴딩의 대안이 존재한다는 중요한 사실을 보여줍니다 — 경쟁을 가깝게 유지하는 방법은 AI를 조작하는 것 외에 **비슷한 실력의 상대를 붙여주는 것**도 있습니다.

### 설계 고려사항

- **입력 장치 전제**: 휠·페달 사용자와 패드 사용자의 물리적 표현력 격차가 큽니다. 패드 사용자에게는 미세한 스티어링 입력이 물리적으로 불가능하므로, 순수 시뮬레이션을 표방하면서도 패드 입력에는 별도의 보간 처리가 필요합니다.
- **학습 곡선의 절벽**: 초심자가 첫 랩을 완주하지 못하는 경험은 이탈로 직결됩니다. 어시스트 사다리(4-2)를 사전에 설계해야 합니다.
- **콘텐츠 검증 비용**: 실차 라이선스와 실제 트랙 스캔은 비용과 일정을 지배합니다.
- **커뮤니티 신뢰**: 이 하위 장르의 사용자는 물리 변경에 극도로 민감하며, 패치 단위의 물리 개정이 커뮤니티 분열을 일으킬 수 있습니다.

### 대표 사례

- iRacing: 레이팅 기반 매칭과 스플릿 분할로 경쟁을 관리
- Assetto Corsa: 모딩 생태계를 통한 콘텐츠 확장
- Gran Turismo: 콘솔 기반 시뮬레이션의 대중화 좌표 (5-1 참조)

## 2-2. 심케이드 (Simcade)

### 메커니즘

- 시뮬레이션 계열의 물리 구조를 유지하되 관용 구간을 확대
- ABS·트랙션 컨트롤·안정성 제어·주행선 표시 등 어시스트 계층
- 어시스트를 끌수록 보상이 증가하는 구조적 유인
- 난이도 슬라이더로 AI 실력을 단계적으로 조정

### 특징 및 차별점

심케이드는 **하나의 물리 위에 여러 개의 난이도 표면**을 얹는 구조입니다. 이 하위 장르의 발명은 "어시스트를 켠 플레이어와 끈 플레이어가 같은 게임을 하고 있다고 느끼게 하는 것"입니다.

시뮬레이션과의 결정적 차이는 **어시스트가 부끄러운 것이 아니라 설계된 사다리의 일부**라는 태도입니다. 잘 설계된 심케이드는 어시스트를 하나씩 끄는 과정 자체를 진행 구조로 취급합니다.

### 설계 고려사항

- **어시스트 간 의존 관계**: 안정성 제어를 끄면 트랙션 컨트롤만으로는 스핀을 막을 수 없는 등, 어시스트는 서로 독립적이지 않습니다. 조합 폭발을 관리해야 합니다.
- **보상 배율의 정직성**: 어시스트를 끄면 보상이 오르는 구조는 유효하지만, 배율이 과하면 숙련자가 아닌 플레이어가 고통을 감수하고 어시스트를 끄게 만들어 경험을 훼손합니다.
- **AI 난이도의 의미 통일**: 난이도 슬라이더가 AI의 실력을 올리는 것인지, 러버밴딩 강도를 조절하는 것인지, 파워를 주는 것인지에 따라 플레이어의 지각이 완전히 달라집니다(3-3 참조).

### 대표 사례

- Forza Motorsport 계열: 어시스트 세분화와 보상 배율의 결합 (5-4 참조)
- Gran Turismo 계열: 시뮬레이션과 대중성 사이의 좌표 설정

## 2-3. 아케이드 (Arcade)

### 메커니즘

- 필요한 메커닉만 선택적으로 축조 (1-1 참조)
- 그립 상실과 회복이 조작 문법으로 승격 (드리프트, 3-6 참조)
- 부스트·니트로 등 명시적 속도 자원
- 충돌이 페널티가 아니라 연출 또는 보상이 되는 경우
- 강한 경쟁 관리로 항상 접전 유지

### 특징 및 차별점

아케이드의 설계 원리는 **"조작의 의도가 즉시 화면에 반영되는 것"**입니다. 시뮬레이션이 입력과 결과 사이에 물리라는 지연층을 두는 반면, 아케이드는 그 층을 얇게 만듭니다.

이 하위 장르는 **속도감 연출의 밀도**로 승부합니다. 실제 속도보다 지각 속도를 우선하며, 카메라·FOV·모션 블러·사운드가 물리만큼 중요한 설계 대상입니다.

### 설계 고려사항

- **부스트 경제**: 부스트를 어떻게 벌게 하느냐가 곧 플레이 스타일을 규정합니다. 충돌로 벌게 하면 난폭 운전을, 드리프트로 벌게 하면 코너 공략을, 슬립스트림으로 벌게 하면 추격을 장려합니다.
- **관용의 일관성**: 벽 충돌 후 복귀가 쉬운 게임에서 특정 구간만 가혹하면 학습이 무너집니다.
- **캐치업의 가시성**: 물리가 추상적일수록 캐치업을 숨기기 쉽지만, 아이템·부스트 같은 명시적 장치와 결합하면 오히려 노출됩니다.

### 대표 사례

- 리지 레이서 계열: 드리프트를 코어 문법으로 확립
- 번아웃 계열: 충돌을 페널티에서 보상으로 전환
- Split/Second: 캐치업을 코어 루프의 전제 조건으로 삼은 사례 (5-5 참조)

## 2-4. 카트/파티 (Kart / Party)

### 메커니즘

- 아이템을 통한 비대칭 간섭
- 순위 기반 아이템 분배 (후위일수록 강한 아이템)
- 드리프트-미니터보 등 숙련 보상 장치
- 다수 동시 주행으로 접전 밀도 확보

### 특징 및 차별점

카트/파티의 핵심은 **실력 차를 결과 차로 번역하지 않으면서도 실력이 의미 있게 만드는 것**이라는 모순된 요구를 동시에 만족시키는 것입니다. 이 모순은 두 개의 분리된 계층으로 해결됩니다 — 아이템 계층은 실력 차를 압축하고, 드리프트 계층은 실력 차를 보상합니다.

주목할 점은 **순위 기반 아이템 분배가 러버밴딩의 명시적 형태**라는 것입니다. 3-3에서 다루는 은폐형 러버밴딩과 달리, 이 방식은 캐치업의 존재를 플레이어에게 공개합니다. 공개된 캐치업은 "속았다"는 감정 대신 "제도"라는 인식을 만듭니다 — 이는 4-3의 핵심 논점입니다.

### 설계 고려사항

- **아이템 분배 곡선**: 후위 보정이 과하면 선두 유지가 불가능해지고, 약하면 역전 기대가 사라집니다.
- **선두의 경험**: 선두 플레이어가 지속적으로 피격당하는 구조는 "이기는 것이 벌"이 되는 역설을 만듭니다.
- **동시 인원과 밀도**: 트랙이 길고 넓을수록 플레이어가 분산되어 경쟁감이 희석됩니다 (5-3 참조).
- **입력 단순화**: 가속·드리프트·아이템의 3버튼 구조가 표준입니다. 추가 입력은 접근성을 직접 훼손합니다.

### 대표 사례

- 마리오 카트 계열: 아이템·드리프트 이중 계층의 표준 (5-3 참조)

## 2-5. 오픈월드 레이싱 (Open-world Racing)

### 메커니즘

- 트랙이 아니라 연속된 세계 위에서의 주행
- 레이스 이벤트가 세계 내 지점으로 배치
- 자유 주행 구간과 경쟁 구간의 교차
- 차량 수집·튜닝·커스터마이즈의 메타 계층

### 특징 및 차별점

오픈월드 레이싱은 **레이싱을 이벤트에서 상태로 전환**합니다. 플레이어는 "레이스를 시작한다"가 아니라 "세계에 있다가 레이스에 진입한다"는 경험을 합니다.

이 구조는 트랙 제작 비용 문제(1-2 참조)에 대한 하나의 답이기도 합니다. 하나의 세계를 만들고 그 위에 경로를 여러 개 그으면 트랙 수가 조합적으로 늘어납니다. 그러나 이 방식은 **경로가 세계의 지형에 종속**되므로, 코너 문법(3-4)을 의도대로 설계하기 어렵다는 대가를 치릅니다.

### 설계 고려사항

- **분산 문제**: 넓은 세계는 플레이어를 흩뜨립니다. 이는 카트/파티에서도 동일하게 발생하는 구조적 문제입니다 (5-3 참조).
- **경로 가독성**: 트랙과 달리 오픈월드는 "어디로 가야 하는지"가 자명하지 않습니다. 내비게이션 UI가 곧 난이도입니다.
- **자유 주행의 목적**: 이벤트 사이의 이동이 지루하면 세계는 로딩 화면이 됩니다.
- **밀도 대 규모**: 세계가 클수록 좋은 것이 아닙니다. 이벤트 밀도가 낮은 대형 맵은 공허감을 만듭니다.

### 대표 사례

- 니드 포 스피드 계열: 도시 기반 오픈월드와 추격 구조
- 포르자 호라이즌 계열: 페스티벌 구조를 통한 이벤트 배치

## 2-6. 러너/모바일 (Runner / Mobile)

### 메커니즘

- 가속이 자동이거나 최소화된 입력
- 차선 변경 수준으로 축약된 조작
- 세션 길이의 극단적 단축
- 물리보다 리듬과 패턴 인식이 주된 과제

### 특징 및 차별점

러너/모바일 계열은 **레이싱의 물리 층을 거의 제거하고 경쟁 층만 남긴** 형태입니다. 엄밀히 말해 이들 중 상당수는 레이싱보다 러너 장르에 가까우며, "먼저 도착한다"는 목표만 공유합니다.

이 계열이 설계 참고 대상으로 유의미한 이유는, **레이싱에서 물리를 제거했을 때 무엇이 남는지**를 보여주기 때문입니다. 남는 것은 속도감 연출, 순위 긴장, 반복 개선입니다. 이 셋이 레이싱의 최소 골격이라는 점을 역으로 증명합니다.

### 설계 고려사항

- **한 손 조작 전제**: 터치 입력의 정밀도 한계가 물리 모델의 상한을 규정합니다.
- **세션 길이**: 1-3분 세션이 표준이며, 이는 트랙 설계를 근본적으로 제약합니다.
- **수익화와 공정성의 충돌**: 성능이 과금과 연동되면 순위 경쟁의 의미가 붕괴합니다.

---

# 3. 핵심 게임 메커니즘

## 3-1. 주행 물리와 타이어 그립 모델

### 메커니즘

레이싱 물리의 중심에는 **타이어**가 있습니다. 차량이 만들어내는 거의 모든 힘 — 가속, 제동, 선회 — 은 타이어와 노면의 접촉면에서 발생하며, 따라서 타이어 모델이 곧 주행 감각입니다.

타이어 모델링에서 가장 널리 알려진 접근은 **매직 포뮬러(Magic Formula)**, 곧 파체이카(Pacejka) 모델입니다. 차량 동역학 문헌은 이를 일관되게 **준경험적(semi-empirical) 모델**로 기술합니다 — 즉 물리 제1원리에서 유도한 것이 아니라, 실제 타이어를 계측한 데이터에 곡선을 피팅하기 위해 고안된 함수 형태이며, B/C/D/E로 통칭되는 계수들이 그 곡선의 형상을 결정합니다.

이 구조는 중요한 함의를 가집니다. **준경험적 모델은 계측 대상이 존재할 때 강력합니다.** 실제 타이어의 데이터가 있으면 그 거동을 높은 정확도로 재현합니다. 그러나 계측 대상이 없을 때 — 가상의 탈것이거나, 목표가 현실 재현이 아니라 특정한 감각일 때 — 이 모델의 파라미터들은 디자이너에게 직관적 의미를 주지 못합니다.

### 특징 및 차별점

어밸런치 스튜디오의 해미시 영은 GDC 2019 세션에서 이 문제를 정면으로 다룹니다. 아래 서술은 **해당 강연의 자동 생성 자막으로 확인한 강연 본문의 논증**을 요약한 것입니다. 자막 기반 확인의 한계는 부록 유의사항 7을 함께 읽으십시오.

#### 핸들링 스펙트럼과 부품의 재조립

영은 핸들링 구현을 하나의 **연속 스펙트럼**으로 배치합니다 — 한쪽 끝에는 타이어 모델 없이 강체(rigid body)에 힘을 직접 가하는 **카트 레이싱** 방식이, 그 위로 아케이드 레이서와 물리 엔진 벤더의 **제약 기반(constraint-based) 차량 SDK**, 심케이드가 있고, 반대쪽 끝에 복잡도가 큰 정밀 타이어 모델의 **드라이빙 시뮬레이션**이 놓입니다. 그는 강연의 목표를 "어느 쪽 끝에서 출발하든 중간의 스위트 스팟으로 이동하는 방법"으로 규정합니다. 중요한 것은 그의 최종 모델이 스펙트럼 위의 **한 점을 고른 결과가 아니라는 점**입니다 — 그는 드리프트 제어를 카트 레이서의 강체 힘 방식에서, 안정성을 물리 SDK 계열의 마찰 클램프에서, 코어 타이어 모델을 시뮬레이션 쪽에서 가져와 조립했다고 정리합니다. **1-1에서 인용한 데 라 크루즈의 "아케이드는 덜어낸 시뮬레이션이 아니다"라는 논지의 구체적 실행 사례**입니다.

#### 준경험적 모델이 아케이드에 맞지 않는 이유

공식 세션 설명이 "may not be optimal"이라는 헷지로 남긴 자리에, 강연 본문은 네 갈래의 구체적 근거를 제시합니다.

**첫째, 수정 저항성.** 영은 준경험적 모델의 제작 경로를 그대로 서술합니다 — 실제 타이어를 계측 장비에 걸고 입력을 변화시키며 결과를 측정한 뒤, 그 결과에 곡선을 피팅해 게임에서 재현합니다. 정확도는 피팅 정밀도와 실행 주파수를 올리면 개선됩니다. 그러나 그는 **타이어가 하는 일을 수정하려 들면 상당히 어려워진다**고 지적합니다. 이 모델은 계측 결과의 재현을 위해 만들어졌으므로 **재현이 아닌 목표를 주면 구조적으로 저항합니다.**

**둘째, 시뮬레이션 주파수.** Just Cause 4는 토네이도를 비롯한 온갖 연산이 동시에 도는 대형 오픈월드였고, 물리 시뮬레이션은 전용 드라이빙 게임보다 **현저히 낮은 고정 주파수**로 돌았습니다. 영은 이 주파수에서 실측 타이어 모델을 돌리면 **특히 하중 이동(weight transfer)에서 피드백이 형편없어지고 차량 거동이 예측 불가능해진다**고 설명합니다. **지탱할 시간 해상도를 주지 못할 모델은 정확한 것이 아니라 불안정한 것입니다.**

**셋째, 언더스티어의 수용 불가능성.** 가장 설계적인 근거입니다. 영은 드라이빙 시뮬레이터에서 언더스티어가 큰 문제이며, 제동하면서 언더스티어가 나 **코너를 못 도는 것은 오픈월드 게임에서 받아들일 수 없다**고 말합니다. 그가 든 상황은 평범합니다 — T자 교차로에 진입해 그 코너를 돌아야 한다. 반면 **오버스티어는 상대적으로 덜 문제**라고 덧붙입니다. 즉 실측 타이어의 정확한 거동 중 일부는 이 게임의 맥락에서 **재현할 가치가 없는 정도가 아니라 재현해서는 안 되는 것**입니다.

**넷째, 보정 비용.** 오버스티어를 물리적으로 올바르게 교정하려면 복잡한 트랙션 컨트롤과 스태빌리티 컨트롤이 필요하며 제대로 만들기 어렵다고 그는 지적합니다. **정확한 모델을 고른 대가가 정확한 보정 시스템을 추가로 만드는 비용으로 되돌아옵니다.**

이 네 근거를 관통하는 구조가 있습니다. 실측 모델을 배제하는 이유가 **연산 비용이나 제작 비용이 아니라 설계 목표 자체**라는 것입니다. 영이 명시한 설계 목표 중 하나는 **제한된 인지 부하(limited cognitive load)** — 고속 주행 중에도 플레이어가 운전 외의 다른 일을 생각할 수 있어야 한다는 것이었습니다. 이 목표 아래에서 "정확한 타이어"는 개선이 아니라 **결함**입니다.

#### 슬립 앵글·슬립 레이시오는 버리지 않습니다

흔한 오해를 정리해야 합니다. 파라미터 축소형 모델은 시뮬레이션의 **입력 개념을 버리는 것이 아닙니다.** 영의 모델은 시뮬레이션과 **거의 같은 입력 파라미터**를 씁니다 — 슬립 레이시오(slip ratio), 슬립 앵글(slip angle), 캠버(camber), 휠 로드(wheel load). 슬립 레이시오는 바퀴의 회전 속도와 전진 속도의 관계로, 완전히 접지해 구르면 0, 바퀴가 잠겨 미끄러지면 음수(제동력), 노면보다 빨리 돌면 양수(가속력)가 됩니다. 슬립 앵글은 바퀴가 향한 방향과 실제 진행 방향 사이의 각도입니다.

**바뀌는 것은 입력이 아니라 입력을 힘으로 바꾸는 함수입니다.** 매직 포뮬러는 그 자리에 계측 피팅된 수식을 두지만, 영은 그 자리를 **디자이너가 그리는 곡선**으로 대체합니다. 이것이 "준경험적이지 않다"는 말의 실질입니다. 휠 로드도 같은 논리로 다뤄집니다 — 시뮬레이션에서는 고주파 서스펜션이 휠 로드에 풍부한 동적 디테일을 주지만, 낮은 주파수에서는 그 디테일을 얻을 수 없고 **애초에 하중 이동이 과해지는 것도 바람직하지 않으므로**, 영은 정지 상태의 서스펜션 힘과 동역학이 반영된 힘을 **블렌딩**해 안정적인 휠 로드를 만든다고 설명합니다.

#### 마찰 클램프 — 자유를 사는 장치

이 강연의 기술적 중심은 **마찰 클램프(friction clamp)**이며, 공식 초록의 병렬 나열이 감춘 **인과의 핵심**이 여기 있습니다.

문제는 이렇습니다. 아케이드 감각을 위해 그립을 실제보다 높이면 모델이 **발산(diverge)**합니다. 어떤 속도의 바퀴에 큰 마찰력을 걸면 결과 속도가 **반대 방향으로 더 커지고**, 그러면 더 큰 힘이 생기고, 다시 뒤집히며 진동이 증폭됩니다. 영은 **마찰이 저항하려던 속도의 부호를 뒤집는 것**이 이런 모델이 불안정해지는 핵심 원인이라고 지적합니다.

마찰 클램프는 **마찰이 마찰답게만 행동하도록 강제하는 상한**입니다. 영은 이 원리가 물리 엔진 벤더의 제약 기반 차량 모델이 내부적으로 하는 일과 본질적으로 같으며, 속도 제곱 항이 있어 상한이 없는 공력(aerodynamics)에도 같은 장치가 유효하다고 설명합니다. 그의 모델은 세 방향에 클램프를 겁니다 — 바퀴가 앞으로 미끄러지는 **종방향**, 바퀴가 노면 대비 반대로 돌아버리는 것을 막는 **각방향**, 그리고 관성까지 고려해야 하는 **횡방향**. 그리고 결정적인 문장이 나옵니다. 클램프를 갖춰 두면 **마찰을 원하는 만큼 얼마든지 올려도 절대 깨지지 않으며 디자이너는 완전히 자유롭게 굴 수 있다**고 그는 말합니다.

이것이 이 강연의 실질적 논증입니다. **목적은 파라미터를 줄이는 것이 아니라 안정성을 물리적 사실성에서 분리해 내는 것입니다.** 클램프가 물리적 타당성의 하한을 구조적으로 보장하므로, 그 위에 얹는 곡선은 **더 이상 현실을 재현할 의무가 없어집니다.** 곧 **파라미터 감소는 원인이 아니라 결과입니다** — 안정성을 다른 장치가 책임지면 곡선은 감각만 표현하면 되고, 감각만 표현하는 곡선에는 계측 피팅용 계수가 필요 없습니다.

다만 SDK의 제약 기반 솔버와는 차이를 둡니다. 영은 반복 솔버가 클램프 대비 힘을 스케일 다운해 항상 안전 범위에 머무르게 하지만, 그 대가로 **휠 스핀으로 그립을 잃는 것 같은 효과를 전혀 모델링하지 못한다**고 지적합니다. 그래서 그의 모델은 클램프를 유지하되 **유효 질량(useful mass)**을 도입합니다 — 바퀴가 여럿인 차량에서 각 바퀴의 클램프에 차량 전체 질량을 쓸 수는 없으므로, **해당 바퀴의 휠 로드를 전체 휠 로드 합으로 나눈 비율**만큼을 그 바퀴가 대표하게 합니다. 그는 이를 완벽하지는 않지만 매우 잘 작동하는 **휴리스틱**이라고 분명히 밝힙니다.

#### 곡선을 그리다 — 파라미터가 줄어드는 실제 경로

클램프가 안정성을 책임지면 힘 곡선은 **스플라인 툴에서 직접 그리는 대상**이 됩니다. 영의 모델은 0에서 출발해 피크를 지나 포화(saturation)에 이르는 **3단계 형상**을 기본으로 하며 곡선의 점 개수에는 제한이 없습니다. 종방향은 가속과 제동에 각각 곡선을 두고, 옆으로 미끄러지는 중이면 곡선 형상을 줄여 휠 스핀이 더 나게 합니다(도넛 기동이 여기서 나옵니다). 횡방향은 슬립 앵글에 대한 힘 곡선을 슬립 구간별로 여러 개 둡니다. 영은 종·횡 곡선의 관계가 다소 반직관적이지만 **매직 포뮬러 같은 복잡한 수식을 수정하는 것보다는 여전히 단순하다**고 말합니다.

**드리프트가 이 구조에서 창발합니다.** 핸드브레이크를 당기면 슬립 앵글이 커져 횡그립이 낮은 곡선으로 진입하고, 그 상태에서 가속하면 휠 스핀이 나며 횡력이 다시 낮아집니다. 영의 표현으로는 **아래 그래프와 위 그래프가 중간 구간을 건너뛰면서** 거동이 바뀝니다. 드리프트는 별도 구현한 특수 상태가 아니라 **곡선 배치의 결과**입니다 (3-6 참조).

여기에 이 강연에서 가장 실무적인 통찰이 붙습니다. **곡선의 높이가 아니라 기울기가 그립감입니다.** 영은 곡선을 올라가는 동안에는 접지감이 들고, 평평해지면 미끄러지는 느낌이 들며, 떨어지면 미끄러짐이 가속되는 느낌이 든다고 설명합니다. 즉 **곡선의 기하학이 곧 감각의 어휘**이며, 디자이너는 수식 계수가 아니라 그래프 모양으로 감각을 조형합니다. 이것이 "디자이너 친화적"이라는 말의 구체적 내용입니다.

파라미터 감소는 여기서 완성됩니다. 영은 모든 곡선을 **정규화(normalize)**했고 그 결과 게임 전체가 **소수의 타이어 세트**만으로 돌아갔다고 밝힙니다. 개별 탈것에서 실제로 저작한 값은 **앞바퀴와 뒷바퀴의 그립 값**이었고, 튜닝의 실체는 **앞뒤 그립의 균형을 잡는 일**이었습니다. 최종 힘은 그립 값에 정규화된 곡선, 휠 로드, 타임 스텝을 곱해 임펄스로 만듭니다. 그는 이 결과를 "꽤 단순한 식"이라고 표현합니다. 다양한 탈것을 소수의 타이어 세트와 바퀴당 그립 값 하나로 다루는 이 구조가, 세션 설명이 말한 "a wide range of vehicles"에 대한 **직접적인 답**입니다. 실제로 비행기 등 비(非)자동차 탈것에는 선형 램프와 포화만 쓰고 휠 스핀 효과를 아예 뺐다고 그는 밝히는데, **클램프가 물리적 타당성을 보장하므로 그래도 괜찮았다는 것**이 그 근거입니다.

#### 거짓 그립의 청구서 — 피치·롤의 국소화

그립을 실제보다 올리면 대가가 따릅니다. 영은 그립만 올리면 차가 **휘청거리고 롤이 과해진다(wallow and roll)**고 지적합니다. 통상적 대응은 서스펜션을 단단하게 하고 안티롤을 넣는 것이지만, 그러면 **서스펜션 값이 현실적 범위를 벗어납니다.** 그의 해법은 문제를 발생 지점에 가두는 것입니다 — 서스펜션을 고치는 대신 **힘을 적용하는 방식**을 고칩니다. 접촉점의 임펄스를 선형 성분과 각 성분으로 분해한 뒤 **롤과 피치 성분에만 축소 계수를 곱합니다.** 그 결과 서스펜션 셋업은 현실적으로 유지되면서 과잉 그립의 부작용만 사라집니다.

이 조치의 함의는 클램프의 그것과 같습니다. **거짓말을 시스템 전체에 퍼뜨리지 않고 한 지점에 가둡니다.** 그립을 위해 서스펜션을 비현실적으로 만들면 그 비현실은 충돌·착지·요철 등 모든 곳으로 번지지만, 힘 적용 단계에서 롤·피치만 눌러 두면 나머지 물리는 정직하게 남습니다. **아케이드 물리의 품질은 거짓말의 양이 아니라 거짓말의 국소성이 결정합니다.**

| 축 | 준경험적 모델 (매직 포뮬러 계열) | 영이 제안한 파라미터 축소형 모델 |
|-----|--------------------------------|-------------------|
| 입력 파라미터 | 슬립 레이시오·슬립 앵글·캠버·휠 로드 | **동일** — 입력이 아니라 힘 함수를 교체 |
| 힘 함수의 정체 | 계측 데이터에 피팅된 수식 | 디자이너가 스플라인 툴에서 그리는 곡선 |
| 안정성의 출처 | 모델의 물리적 정확성과 높은 실행 주파수 | **마찰 클램프** — 곡선의 사실성과 분리되어 보장 |
| 파라미터의 의미 | 피팅용 계수 — 개별 의미가 직관적이지 않음 | 곡선의 기울기 = 그립감 (높이가 아님) |
| 탈것당 저작 비용 | 탈것마다 계측 데이터 필요 | 정규화 후 앞·뒤 그립 값 |
| 적합 영역 | 현실 재현이 목표인 시뮬레이션 | 아케이드·심케이드, 낮은 시뮬레이션 주파수 |
| 위험 | 아케이드 목표에서 튜닝 비용 과다 | 클램프가 부실하면 고그립에서 발산 |

### 노면 이산화(discretization) 문제와 다접지점 모델

타이어 모델의 함수 형태만큼 중요한 것이 **타이어가 노면을 어떻게 샘플링하는가**입니다. 이 논점에 대한 구체적인 1차 자료가 존재합니다.

Turn 10 스튜디오는 2023년 7월 17일 자사 발표문에서 다음과 같이 밝혔습니다.

> "Our new tire physics has 8 points of contact with the track surface running at 360 cycles per second. This allows tires to find grip on any type of uneven surface. The result is better drivability everywhere and realistic behavior on uneven surfaces like curbs."

그리고 이전 세대의 구성과 그 문제를 이렇게 밝혔습니다.

> "Previous Forza Motorsport games had only a single point of tire contact on the surface that moved at 60 cycles per second. This was problematic for curbs and rumble strips which could cause inconsistencies and unrealistic changes in suspension forces. This could also lead to tires briefly losing contact with the ground as they rolled over bumps resulting in unrealistic loss of traction."

**이 진술에서 설계적으로 중요한 것은 수치가 아니라 "왜"입니다.** Turn 10이 밝힌 인과를 풀면 다음과 같습니다.

- 타이어를 **접지점 하나로 근사**하면 실제 접지면이 면적을 가진다는 사실이 모델에서 사라집니다. 그 결과 연석이나 럼블스트립처럼 **노면의 공간 주파수가 접지면 크기에 근접하는 지형**에서 단일 점 샘플은 노면을 대표하지 못하며, Turn 10은 이것이 서스펜션 힘의 불일치와 비현실적 변화를 낳았다고 밝혔습니다.
- 더 나아가 요철을 넘을 때 **그 하나의 점이 순간적으로 노면을 놓치면 타이어 전체가 접지를 잃은 것으로 계산**되어 비현실적인 트랙션 상실이 발생했다고 밝혔습니다.
- 다접지점은 그 대응책입니다 — 여러 점으로 샘플링하면 일부 점이 요철 위에 있어도 나머지가 그립을 찾습니다.

즉 이것은 **노면 이산화 문제**입니다. 연속적인 노면을 유한한 샘플로 근사할 때 샘플이 부족하면 지형의 고주파 성분이 물리에 에일리어싱으로 나타납니다. **접지점 수는 공간 해상도, 연산 주기는 시간 해상도**이며, 두 해상도가 모두 부족할 때 증상이 가장 심하게 드러나는 지점이 바로 연석입니다.

이 논점은 3-1의 앞부분과 층위가 다릅니다. 매직 포뮬러 대 파라미터 축소형 모델의 대립이 **"주어진 접지 상태에서 힘을 어떻게 계산할 것인가"**의 문제라면, 이산화는 **"접지 상태 자체를 어떻게 관측할 것인가"**의 문제입니다. **함수를 아무리 정교하게 골라도 입력이 되는 노면 샘플이 부실하면 결과는 부실합니다.**

위 진술은 게임 출시(2023-10-10) 약 3개월 전에 게재된 개발사 자체 홍보 자료의 진술이며, 같은 발표문이 함께 제시한 "충실도 향상 배수"는 별도의 비판적 독법을 요구합니다 — **3-7을 반드시 함께 읽으십시오.**

### 설계 고려사항

- **모델 선택은 목표의 함수입니다**: "가장 정확한 모델"을 고르는 것이 아니라 "우리 게임의 감각을 가장 적은 비용으로 만들 수 있는 모델"을 고르는 것입니다.
- **힘 계산과 노면 관측을 분리해서 검토하십시오**: 주행감이 나쁠 때 원인이 타이어 함수에 있는지 노면 샘플링에 있는지 먼저 가르십시오. 연석·요철에서만 증상이 나타난다면 후자일 가능성이 큽니다.
- **탈것의 다양성이 모델을 규정합니다**: 승용차만 다루는 게임과 오토바이·트럭·특수 차량을 함께 다루는 게임은 요구가 다릅니다. 탈것마다 계측 데이터를 전제하는 모델은 이 요구에 구조적으로 불리하며, 영의 팀이 곡선 정규화로 도달한 좌표가 그 대안입니다.
- **파라미터 수는 조직의 문제이기도 합니다**: 파라미터가 많으면 물리 프로그래머만 튜닝할 수 있고, 적으면 디자이너가 직접 반복할 수 있습니다. 이는 기술 선택이 아니라 조직 설계입니다.
- **안정성을 곡선이 아니라 별도 장치에 맡기십시오**: 단순화된 모델은 정상 주행 영역에서는 잘 작동하다가 고속 스핀, 공중 낙하, 벽면 접촉 같은 극한에서 발산할 수 있습니다. 극한 영역 검증은 필수이되, **더 나은 대응은 곡선의 사실성에 안정성을 의존시키지 않는 것**입니다 — 마찰 클램프처럼 하한을 구조적으로 보장하는 장치가 있으면 극한의 안전성과 곡선의 자유를 동시에 얻습니다.

### 대표 사례

- Just Cause 4: 다양한 탈것을 단일 핸들링 체계로 수용하기 위해 타이어 동역학을 설계 지렛대로 사용 (5-2 참조)
- Gran Turismo: 접지 한계 운용을 과제로 정의하는 시뮬레이션 좌표 (5-1 참조)

## 3-2. 핸들링 어시스트 (Handling Assists)

### 메커니즘

어시스트는 플레이어의 입력과 물리 사이에 삽입되는 **보정 계층**입니다. 주요 유형은 다음과 같습니다.

- **ABS (잠김 방지 제동)**: 제동 시 타이어 잠김을 방지하여 조향 능력을 유지
- **TCS (트랙션 컨트롤)**: 구동력이 그립을 초과할 때 출력을 제한하여 휠스핀 방지
- **안정성 제어**: 차량의 요(yaw) 거동을 감지하여 스핀 방향으로의 회전을 억제
- **주행선 표시**: 최적 라인과 제동 지점을 시각적으로 제시
- **자동 제동 / 자동 변속**: 조작 요소 자체를 제거
- **되감기(Rewind)**: 실수를 시간 축에서 취소

여기에 더해, 거의 모든 레이싱 게임에 존재하지만 어시스트로 인식되지 않는 보정이 하나 있습니다 — **속도 의존적 조향각 제한**입니다.

데 라 크루즈는 앞의 기고문에서 이를 다음과 같이 기술합니다. "Almost every racing game improves car handling by limiting the front wheels' maximum steering angle based on how fast the car is travelling. For example, at low speeds, you may allow players to steer their wheels by at most ten degrees, but then at high speeds, you may instead limit them to at most two degrees."

**주의**: 여기 언급된 10도·2도는 기고자가 개념 설명을 위해 든 예시 값이며, 출시된 특정 타이틀에서 계측된 수치가 아닙니다. 이 문서는 이를 실제 튜닝 값으로 인용하지 않으며, **"저속에서는 넓게, 고속에서는 좁게 조향각을 제한한다"는 원리**만 취합니다.

이 보정이 중요한 이유는, 이것이 **어시스트를 모두 끈 "순수" 모드에서도 대개 작동한다**는 점입니다. 아날로그 스틱의 최대 변위를 고속에서 그대로 조향각에 매핑하면 차량은 즉시 스핀합니다. 즉 **완전히 보정되지 않은 레이싱 게임은 사실상 존재하지 않습니다.** 어시스트의 유무가 아니라 정도의 문제입니다.

### 특징 및 차별점

어시스트 설계에서 자주 놓치는 점은, 어시스트가 **난이도 장치일 뿐 아니라 입력 장치 격차의 보정 장치**라는 것입니다. 휠 사용자는 900도 회전 범위에서 미세한 조향이 가능하지만, 패드 사용자는 몇 밀리미터의 스틱 변위로 같은 표현을 해야 합니다. 속도 의존 조향각 제한은 본질적으로 이 격차에 대한 대응입니다.

### 설계 고려사항

- **어시스트는 기본값이 곧 메시지입니다**: 초기 상태에서 어떤 어시스트가 켜져 있는지가 게임의 정체성을 선언합니다.
- **끄는 경험을 설계하십시오**: 어시스트를 끄면 갑자기 게임이 불가능해지는 구조는 사다리가 아니라 절벽입니다. 각 어시스트는 단계적으로 약해질 수 있어야 합니다.
- **보이지 않는 어시스트를 문서화하십시오**: 속도 의존 조향각 제한처럼 UI에 노출되지 않는 보정은 팀 내부에서 반드시 명시적으로 관리해야 합니다. 그렇지 않으면 "어시스트 전부 껐는데 왜 이렇게 잘 도느냐"는 커뮤니티 논쟁에 대응할 수 없습니다.
- **되감기는 러버밴딩의 사촌입니다**: 되감기는 실수의 대가를 제거하지만, 러버밴딩과 달리 **플레이어가 명시적으로 요청**합니다. 이 차이가 지각된 공정성을 가릅니다 (4-3 참조).

## 3-3. 러버밴딩과 캐치업 AI (Rubber-banding & Catch-up AI)

이 절은 이 문서에서 가장 중요한 절이며, 닉 멜더의 Game AI Pro 42장 "A Rubber-Banding System for Gameplay and Race Management"(Part VI. Racing, 507-512쪽)를 1차 출처로 삼습니다. 이 챕터는 러버밴딩을 **완결된 시스템 아키텍처**로 기술하는 드문 공개 자료입니다.

### 3-3-1. 정의와 목적

멜더는 러버밴딩을 다음과 같이 정의합니다. "Rubber-banding is a technique used in racing games to keep the AI drivers near to the players in order to maintain the excitement in races. In simple terms, when an AI-controlled vehicle gets too far in front of the player, it will slow down to allow the player to catch up and, similarly, AI-controlled vehicles behind the player will gain a boost to their speed to help them catch up to the player."

주목할 점은 이 정의가 **양방향**이라는 것입니다. 러버밴딩은 뒤처진 AI를 밀어주는 것만이 아니라, **앞선 AI를 늦추는 것**을 포함합니다.

데이브 마크(Dave Mark)는 2010년 6월 2일 Game Developer 기고문 "Rubber-banding as a Design Requirement"에서 이 양방향성을 플레이어 관점에서 표현합니다. "If you are doing well, the competition starts doing well. If you are sucking badly, so do they." 즉 플레이어가 잘하면 경쟁자도 잘하게 되고, 못하면 경쟁자도 못하게 됩니다.

이 두 진술을 합치면 러버밴딩의 본질이 드러납니다 — **러버밴딩은 난이도를 낮추는 장치가 아니라 난이도를 플레이어에게 고정시키는 장치**입니다.

### 3-3-2. 구간 구조 (Region Architecture)

멜더의 시스템은 플레이어와의 **거리(distance from player)**를 입력으로 받아 **러버밴딩 배수(multiplier)**를 출력하는 함수로 구성됩니다. 이 함수는 다섯 개의 구간(region a~e)으로 나뉩니다.

| 구간 | 위치 | 동작 | 명칭 |
|------|------|------|------|
| region c | 플레이어 인접 | **러버밴딩 완전 비활성화** | 데드존 |
| region d | 플레이어 앞쪽 | 거리에 따라 선형적으로 감속 효과 증가 | 전방 밴딩 (forward banding) |
| region e | 플레이어 앞쪽 먼 거리 | 감속 효과가 최대치로 유지 | 전방 밴딩 포화 |
| region b | 플레이어 뒤쪽 | 거리에 따라 선형적으로 가속 효과 증가 | 역방향 밴딩 (reverse banding) |
| region a | 플레이어 뒤쪽 먼 거리 | 가속 효과가 최대치로 유지 | 역방향 밴딩 포화 |

멜더는 각 구간을 이렇게 설명합니다. "Within region c, rubber-banding is disabled. In practice this means any vehicles around the player will behave normally so the player will not see any obvious differences between his vehicle and the AI's vehicle when in close competition."

또한 그는 선형 관계가 유일한 선택지가 아님을 명시합니다. "It should be noted that a linear relationship is not the only option; the use of a sigmoid or other function would also work."

**설계상 가장 중요한 구간은 region c, 곧 데드존입니다.** 이는 러버밴딩 시스템의 핵심 발명입니다 — 플레이어가 AI를 가장 자세히 관찰할 수 있는 순간(근접 경쟁 중)에 시스템을 완전히 꺼버림으로써, 러버밴딩의 존재를 관측 불가능하게 만듭니다.

### 3-3-3. 두 가지 조작 방법: 파워 대 드라이버 스킬

멜더는 AI를 빠르거나 느리게 만드는 두 가지 방법을 제시합니다.

**방법 1 — 파워 조작**: "Method one modifies the power that the cars have. Cars in front of the player will have reduced amounts of power, reducing acceleration, and top speed. Similarly, vehicles behind the player will gain power, and so will have better acceleration and will possibly also have a higher top speed."

**방법 2 — 드라이버 스킬 조작**: "Method two modifies the driver's skill level. In order to make cars in front of the player slow down, their driver skill is lowered. Depending upon how difficulty is implemented, this can cause the AI to brake earlier than normal, to slow down more for corners, and to accelerate out of corners less vigorously."

그리고 이 챕터의 **핵심 설계 규칙**이 등장합니다.

> "As a rule, it is better to modify the driver skill to slow down or speed up the vehicles, as the drivers are still in the same cars and so no 'cheating' is happening. However, the only way to increase the speed of the cars, when the driver skill is already at maximum, is to add extra power to the cars. A similar situation occurs at the lower end where the driver skill is already at a minimum; further slowdown can only be achieved by reducing their power."

이 규칙의 논리 구조를 풀면 다음과 같습니다.

1. **드라이버 스킬 조작이 우선입니다.** 이유는 명시적입니다 — AI가 여전히 같은 차에 타고 있으므로 **"부정(cheating)이 일어나지 않기" 때문**입니다. AI가 코너에서 일찍 브레이크를 밟아 느려지는 것은 물리적으로 정당한 결과이며, 플레이어가 관찰해도 "저 AI가 실수했다"로 읽힙니다.
2. **파워 조작은 폴백입니다.** 드라이버 스킬이 이미 최대치에 도달했는데도 더 빠르게 해야 한다면, 남은 수단은 파워를 더하는 것뿐입니다. 최소치에서도 마찬가지입니다.
3. 멜더는 그림 42.2에서 이 순서를 시각화하며, **파워 배수는 난이도(스킬) 배수가 최소 또는 최대에 도달한 뒤에야 변하기 시작한다**고 캡션에 명시합니다.
4. 실무적 결론은 조합입니다. "In practice, the best approach is to use a combination of the two methods, as they each have distinct advantages and disadvantages."

**이것이 러버밴딩 설계의 가장 이식성 높은 원리입니다.** 캐치업을 구현할 때 "무엇을 조작할 것인가"의 우선순위는 **플레이어가 그 조작을 물리적으로 정당한 사건으로 해석할 수 있는가**로 결정됩니다. 스킬 조작은 AI의 판단을 바꾸므로 정당해 보이고, 파워 조작은 AI의 능력을 바꾸므로 부정해 보입니다.

### 3-3-4. 파워 기반 러버밴딩의 실패 모드

멜더는 파워 조작이 왜 폴백에 머물러야 하는지를 구체적 실패 모드로 열거합니다. 그는 **"The severity of these problems is largely a factor of how large the power changes are"**라고 전제합니다 — 즉 파워 변화량이 클수록 문제가 심각해집니다.

| 실패 모드 | 원문 근거 | 증상 | 멜더가 제시한 완화책 |
|----------|----------|------|-------------------|
| 그립 초과로 인한 제어 불능 | "The AI is unable to control the cars due to the additional power overwhelming the available tire grip, in particular spinning the car when exiting a corner." | 추가된 파워가 가용 타이어 그립을 압도하여 **코너 탈출 시 AI 차량이 스핀** | "setting up the cars to drive with the extra power enabled at all times" (추가 파워가 항상 켜진 상태를 전제로 차량을 셋업) 또는 "extra grip can be added to make it harder to break traction" (그립을 추가) |
| 저출력으로 인한 주행 불능 | "Underpowered cars are unable to accelerate up hills or change gears." | 파워가 깎인 차량이 **언덕을 오르지 못하거나 기어 변속을 하지 못함** | (챕터는 3-3-5의 비활성화 조건으로 대응) |
| 물리 연동 오디오의 이상 동작 | "Audio issues if the audio is directly controlled by the car's physics. ... there is a much greater likelihood that the car will be able to hit the limiter when in the top gear, causing audio to play the limiter effect constantly." | 파워가 추가된 차량이 최고 기어에서 **레브 리미터를 계속 때려 리미터 효과음이 끊임없이 재생** | (오디오가 물리에 직결된 구조 자체의 재검토) |

이 표는 러버밴딩 설계에서 가장 실무적으로 유용한 부분입니다. 특히 **첫 번째와 세 번째 실패 모드는 물리 충실도가 높을수록 심해집니다.** 물리가 정교하면 파워 조작이라는 거짓말을 물리 엔진이 스스로 폭로합니다 — 그립을 넘어선 파워는 스핀으로, 리미터를 넘어선 회전수는 소리로 드러납니다.

이것이 2-0 표에서 언급한 **"물리 충실도와 경쟁 관리의 역상관"의 기술적 근거**입니다. 멜더 자신이 이 인과를 명시합니다. "Depending on the depth of the underlying physics simulation, power-based application can lead to a number of issues."

### 3-3-5. 러버밴딩을 반드시 꺼야 하는 상황

멜더는 러버밴딩을 **전면 비활성화해야 하는 세 가지 상황**을 명명합니다. "There are a number of situations when you do not want power modifiers to be applied to a vehicle. If any of these are present, then all rubber-banding effects should be disabled."

**1. 레이스 시작 (Race start)**

이 항목은 단순한 예외가 아니라 **부호가 반대로 뒤집히는 사례**이기에 특히 중요합니다.

멜더는 문제를 이렇게 규정합니다. "One of the biggest problem areas with racing AI is racing into the first corner, where multiple vehicles need to slow down and navigate the corner simultaneously. Ideally, you want cars to be spread out and be in single file by the time they reach the first corner to avoid pileups, crashes, and blockages."

그리고 러버밴딩이 이 목표를 정면으로 거스른다고 지적합니다. "Obviously, forward rubber-banding will cause them to bunch together more, which is the antithesis of what we want to happen. In fact, giving the vehicles in front of the player extra power to help spread them out is actually desirable (i.e., applying reverse banding to the cars in front)."

즉 **레이스 시작 시에는 앞선 차량에게 오히려 추가 파워를 주어 흩뜨리는 것(역방향 밴딩)이 바람직합니다.** 평시의 러버밴딩이 "앞선 차를 늦춘다"였다면, 스타트에서는 "앞선 차를 밀어준다"가 됩니다. 목적이 접전 유지에서 **첫 코너 파일업 방지**로 바뀌기 때문입니다.

**2. 저속 (Low speed)**

"If a vehicle is far in front of the player and its power has been reduced, it will be unable to accelerate effectively. From a standing start (e.g., after a crash) the car may not actually have enough power to get the car to start moving."

정지 상태에서 재출발할 때 — 예컨대 충돌 직후 — 깎인 파워로는 **차가 아예 움직이지 못할 수 있습니다.** 이는 3-3-4의 두 번째 실패 모드가 극단적으로 발현된 경우입니다.

**3. 랩 뒤처짐 (Being lapped)**

"On a circuit, it is possible that an AI may lap the player. In this case the AI will be so far in front of the player (in terms of track distance) that it will be racing with maximum negative rubber-banding being applied (i.e., on lowest driver skill and lowest power). This has the effect that the car will be driving very slowly around the track in a very uncompetitive manner. In this case, it is better to disable rubber-banding on that vehicle so that it is not obvious to the player that rubber-banding is occurring."

AI가 플레이어를 한 바퀴 추월하면 트랙 거리상 극단적으로 앞서 있게 되어 최대 음의 러버밴딩을 받게 되고, 그 결과 **터무니없이 느리게 기어다니며 러버밴딩의 존재를 폭로합니다.** 해법은 그 차량의 러버밴딩을 끄는 것입니다.

이 세 상황의 공통 구조를 읽어야 합니다. **러버밴딩은 "플레이어와의 거리"라는 단일 입력만 보기 때문에, 거리 외의 맥락이 지배적인 상황에서 반드시 오작동합니다.** 따라서 러버밴딩 시스템에는 반드시 **맥락 기반 무효화 계층**이 필요합니다.

### 3-3-6. 뭉침(Bunching) 문제와 대응

멜더는 전방 밴딩의 부작용을 지적합니다. "A side effect of forward banding, especially when using large changes in driver skill/power over short distances, is that the AI vehicles can become bunched together. If the player quickly catches up with them, they will still be in a bunch even though they will have minimal rubber-banding affecting them."

왜 문제인가에 대한 그의 설명은 플레이어 경험 관점입니다. "This is undesirable as it can enable the player to overtake many vehicles at once. As a player, part of the fun with racing games is battling against the AI to gain position; being able to overtake many opponents at once is detrimental to the player's experience."

즉 **한 번에 여러 대를 추월하는 것은 이득이 아니라 재미의 손실**입니다. 추월 하나하나가 레이싱의 재미 단위인데, 뭉친 무리를 한꺼번에 통과하면 그 단위가 소멸합니다.

멜더가 제시한 두 가지 안티-뭉침 기법은 다음과 같습니다.

**기법 1 — 선두 차량 거리를 최대 거리로 사용**: "by using the greater of the maximum defined distance or the distance of the furthest vehicle from the player, the effect of the rubber-banding can be implemented over a greater distance. This does not eliminate the bunching, but slows it down because of the greater distance that the effect is scaled over."

고정된 최대 거리 대신, **정의된 최대 거리와 최원거리 차량까지의 거리 중 큰 값**을 쓰면 효과가 더 긴 거리에 걸쳐 분산됩니다. 멜더는 이것이 뭉침을 **제거하지는 못하고 늦출 뿐**이라고 정직하게 한정합니다.

**기법 2 — 평균 위치 사용**: "By taking the average position of all the vehicles in front of the player, past a threshold distance, and then affecting all those vehicles the same amount (based upon their average position), the entire pack will reduce their speeds simultaneously. Because multiple vehicles are being affected in the same way simultaneously, they maintain the same spacing while being affected by the rubber-banding."

임계 거리를 넘은 앞쪽 차량들의 **평균 위치를 기준으로 전원에게 동일한 효과**를 적용하면, 무리 전체가 동시에 감속하므로 **상호 간격이 보존됩니다.** 이것이 더 근본적인 해법입니다 — 뭉침의 원인이 "차량마다 다른 배수를 받는 것"이므로, 같은 배수를 주면 뭉치지 않습니다.

### 3-3-7. 파라미터 선택

멜더는 파라미터 선택을 난이도와 지각 양쪽에 영향을 주는 결정으로 규정합니다. "It is important that sensible parameters are chosen when setting up the rubber-banding system, as this can have a large effect on both the difficulty of the game and the perception that the player will have towards the AI."

**핵심 튜닝 결정은 데드존(region c)의 폭입니다.**

> "In general, you want to make sure that there is no rubber-banding being applied when close to the player, otherwise power advantages can become visible. It is therefore essential to choose the width of the dead zone in region c so that the AI has no undue, visible advantage, but also so that the AI can still catch up to within the player's rear view."

이 문장은 **양방향 실패 조건을 정의합니다**.

| 데드존 폭 | 결과 |
|----------|------|
| 너무 좁음 | 근접 경쟁 중에도 러버밴딩이 작동하여 **AI의 파워 우위가 눈에 보임** → 부정 지각 |
| 너무 넓음 | AI가 **플레이어의 백미러 안까지 따라붙지 못함** → 캐치업 기능 자체가 무력화 |

즉 데드존은 "은폐"와 "기능" 사이의 교환입니다. 멜더가 제시한 기준선은 명확합니다 — **AI가 부당하고 가시적인 우위를 갖지 않으면서도, 플레이어의 후방 시야 안까지는 따라붙을 수 있어야 합니다.**

멜더는 러버밴딩을 난이도 밸런싱 수단으로도 활용할 수 있다고 기술합니다.

- **전방 밴딩의 최소 거리를 음수로 설정**: "By having a negative minimum distance for the forward banding, this will allow the player to more easily overtake the AI as, when side by side, the AI will always have less power than the player." — 나란히 달릴 때 AI가 항상 플레이어보다 파워가 낮아져 추월이 쉬워집니다.
- **최대 거리 조정**: "by adjusting the maximum distance that the rubber-banding operates over, the AI will become more spread out, reducing the number of overtaking opportunities compared with a smaller maximum distance." — 최대 거리를 늘리면 AI가 더 흩어져 추월 기회가 줄어듭니다.

### 3-3-8. 분할 화면 멀티플레이어

인간 플레이어가 둘 이상일 때 누구를 기준으로 삼을지가 문제가 됩니다. 멜더의 해법은 명료합니다.

> "A good solution is to treat the rearmost player car as the reference for reverse banding, and the foremost player car as reference for the forward banding, with any AI between the player cars having no rubber-banding applied."

- **역방향 밴딩(뒤에서 밀어주기)의 기준**: 가장 뒤에 있는 플레이어
- **전방 밴딩(앞에서 늦추기)의 기준**: 가장 앞에 있는 플레이어
- **플레이어들 사이에 있는 AI**: 러버밴딩 미적용

이 설계의 논리는 **플레이어 전원을 포함하는 구간 전체를 데드존으로 확장**하는 것과 같습니다. 어느 플레이어도 자기 근처의 AI가 조작되는 것을 보지 못합니다.

### 3-3-9. 특수 사례 — 1위 러버밴딩

멜더는 선두 차량이 구조적으로 도주하는 문제를 지적합니다. "The fastest times will always be generated when there is only one vehicle on track, as it is possible to drive the perfect racing line without needing to overtake other vehicles. Essentially, this is the situation that the vehicle in first place is in and so a large gap will normally form between the first vehicle and the rest of the pack."

**선두는 트랙에 혼자 있는 것과 같으므로 완벽한 라인을 탈 수 있고, 따라서 자연히 격차가 벌어집니다.** 이는 러버밴딩의 결함이 아니라 레이싱의 물리적 구조에서 나오는 현상입니다.

해법은 러버밴딩을 한 겹 더 얹는 것입니다. "This can be reduced by having additional rubber-banding that only applies to the vehicle in first place. This works in the same way as previously described, but the distance used is between the first and second place instead of the AI and the player."

즉 **1위 차량에만 적용되는 별도의 러버밴딩**을 두되, 기준 거리를 "AI와 플레이어 사이"가 아니라 **"1위와 2위 사이"**로 바꿉니다.

### 3-3-10. 레이스 페이스 시스템 — 장거리 레이스로의 확장

멜더는 러버밴딩 아키텍처가 더 일반적인 시스템으로 확장 가능하다고 논증합니다. "The purpose of the rubber-banding system is to slow down or speed up the AI vehicles around the player. Essentially, it makes an AI faster or slower, based upon some predetermined criteria. By adding to these criteria, we can expand the rubber-banding implementation into a system that can adjust the race pace over long races."

핵심 통찰은 **러버밴딩이 "거리"라는 하나의 기준을 쓰는 특수 사례일 뿐이며, 기준을 추가하면 레이스 페이스 관리 도구가 된다**는 것입니다.

그가 제시하는 페이싱 프로파일은 다음과 같습니다.

> "Across a 50-lap race, a racer will typically race hard for the first 10 laps, then relax (slow down) for the next 30 laps before racing hard for the last 10 laps. If pit stops are involved, then it may be desirable for the AI to race hard on the lap before the pit stop and the lap after, before going back to its relaxed state of driving."

| 구간 (50랩 기준) | AI 주행 모드 |
|-----------------|-------------|
| 1-10랩 | 하드 (전력 주행) |
| 11-40랩 | 릴랙스 (감속) |
| 41-50랩 | 하드 (전력 주행) |
| 피트스톱 직전 랩 | 하드 |
| 피트스톱 직후 랩 | 하드 |
| 그 외 | 릴랙스 상태로 복귀 |

**이 50랩·10/30/10 프로파일은 이 문서에서 인용하는 몇 안 되는 구체 수치이며, 멜더의 챕터 본문에 명시된 값입니다.** 다만 그는 이를 "typically"라는 표현으로 일반적 경향을 기술하는 데 사용하며, 특정 타이틀의 튜닝 값으로 제시하지 않습니다. 멜더는 레이스 페이스 시스템 구현에 대한 추가 아이디어를 다른 자료([Jimenez 09] E. Jimenez, "The Pure Advantage: Advanced Racing Game AI", 2009)에서 찾을 수 있다고 안내합니다.

### 3-3-11. 결론 — 멜더의 요약

챕터의 결론은 시스템 전체를 압축합니다.

> "This article described a rubber-banding system that works by varying both the driver skill and the vehicle power to keep the AI competitors around the player. Only manipulating the driver skill will allow the AI to stay around the player without appearing to cheat. However, in the situations where the AI just can't go fast or slow enough, power-based rubber-banding can be used."

### 설계 고려사항 (종합)

- **거리 단일 입력의 한계를 인정하십시오**: 러버밴딩은 거리만 봅니다. 스타트, 저속, 랩 뒤처짐은 거리가 오도하는 상황이며, 반드시 별도의 무효화 규칙이 필요합니다.
- **데드존을 먼저 정하십시오**: 다른 파라미터보다 데드존 폭이 지각을 지배합니다.
- **스킬 우선, 파워는 폴백**: 이 순서를 뒤집으면 물리 엔진이 당신의 거짓말을 폭로합니다.
- **오디오·물리 결합을 점검하십시오**: 파워 조작의 가장 은밀한 노출 경로는 소리입니다.
- **뭉침은 추월의 재미를 파괴합니다**: 평균 위치 기법으로 간격을 보존하십시오.
- **1위 문제는 별도 시스템입니다**: 플레이어 기준 러버밴딩만으로는 선두 도주를 막지 못합니다.

## 3-4. 트랙 설계 (Track Design)

### 메커니즘

트랙은 레이싱의 레벨 디자인이며, 그 문법은 **코너**를 단위로 구성됩니다.

- **코너의 유형**: 저속 헤어핀, 중속 스윕, 고속 시케인, 복합 코너
- **에이펙스(apex)**: 코너 안쪽의 정점 — 최적 라인이 접하는 지점
- **진입/탈출 배치**: 코너 탈출 후 직선이 길수록 탈출 속도의 가치가 커짐
- **시야(sightline)**: 플레이어가 코너 너머를 언제 볼 수 있는가
- **고저차**: 하중 이동과 시야 차단을 동시에 만드는 장치
- **추월 지점**: 제동 구간이 긴 곳에 자연 발생

### 특징 및 차별점

트랙 설계의 근본 원리는 **코너 하나가 아니라 코너의 연쇄가 난이도를 만든다**는 것입니다. 코너 탈출 속도는 다음 직선의 전 구간 속도를 결정하므로, 긴 직선 앞의 코너는 그 트랙에서 가장 중요한 코너가 됩니다. 이 구조가 "느린 코너를 잘 도는 것이 빠른 구간을 만든다"는 레이싱의 역설적 스킬을 낳습니다.

**시야는 난이도의 은밀한 축입니다.** 블라인드 코너(진입 전에 출구가 보이지 않는 코너)는 물리적으로 어렵지 않아도 첫 주행에서 반드시 실패하게 만듭니다. 이는 학습을 강제하는 장치이며, 반복 주행 전제의 게임에서는 유효하지만 일회성 진행이 중요한 게임에서는 부당하게 느껴집니다.

### 설계 고려사항

- **물리를 트랙에 반영하십시오**: 1-3의 모터스톰 사례처럼, 물리가 만들어내는 특징적 거동을 트랙이 조명하면 그 거동은 버그에서 스킬로 승격됩니다. 반대로 물리 특성과 무관한 트랙은 어떤 물리 엔진에서도 똑같이 밋밋합니다.
- **추월 지점을 의도적으로 배치하십시오**: 추월이 불가능한 트랙은 러버밴딩이 아무리 잘 작동해도 재미가 없습니다. 3-3-6에서 봤듯 추월은 레이싱 재미의 기본 단위입니다.
- **첫 코너를 특별 취급하십시오**: 3-3-5에서 멜더가 지적했듯, 첫 코너는 다수 차량이 동시에 감속·선회해야 하는 구조적 병목입니다. 그리드에서 첫 코너까지의 거리는 파일업 방지를 위한 튜닝 대상입니다.
- **오픈월드에서는 코너 문법이 지형에 종속됩니다**: 2-5에서 지적한 대로, 세계 위에 경로를 긋는 방식은 트랙 설계의 정밀도를 희생합니다.
- **랩 구조 대 포인트투포인트**: 순환 트랙은 반복 학습과 랩타임 비교에 유리하고, 지점 간 코스는 서사적 진행감에 유리합니다.

## 3-5. 데미지·타이어·연료 (소모 자원)

### 메커니즘

- **데미지 모델**: 시각적 손상, 성능 저하, 부품 파손, 리타이어
- **타이어 마모**: 랩 수에 따른 그립 저하, 컴파운드 선택(소프트/미디엄/하드)
- **연료**: 소모에 따른 질량 감소 → 후반부 랩타임 향상
- **피트스톱**: 위 자원들을 회복하되 시간을 지불하는 교환

### 특징 및 차별점

소모 자원은 레이싱에 **시간 축의 전략**을 도입합니다. 이것이 없으면 레이싱은 "매 랩 최선을 다한다"는 단조로운 최적화가 되지만, 타이어와 연료가 있으면 "지금 아껴서 나중에 쓴다"는 자원 배분 문제가 됩니다.

주목할 점은 이 시스템이 **3-3-10의 레이스 페이스 시스템과 직접 연동**된다는 것입니다. 멜더가 제시한 페이싱 프로파일에서 AI가 초반 10랩을 전력 주행한 뒤 30랩을 여유롭게 달리는 것은, 실제 모터스포츠에서 타이어와 연료를 관리하는 행동과 정확히 대응합니다. 즉 **레이스 페이스 시스템은 소모 자원 시뮬레이션의 저비용 대체재이기도 합니다** — 자원을 실제로 시뮬레이션하지 않고도, AI가 자원을 관리하는 것처럼 보이게 만들 수 있습니다.

### 설계 고려사항

- **소모는 관측 가능해야 합니다**: 타이어가 닳는데 플레이어가 그것을 느끼지 못하면 시스템은 없는 것과 같습니다. 그립 저하는 조작감으로, 연료는 UI로 전달되어야 합니다.
- **데미지는 회복 불가능한 페널티가 되기 쉽습니다**: 초반 사고로 성능이 영구 저하되면 남은 레이스가 무의미해집니다. 러버밴딩이 이를 보정하려 들면 3-3-4의 실패 모드가 발현됩니다.
- **AI도 같은 규칙을 받아야 합니다**: 플레이어만 타이어가 닳고 AI는 닳지 않는 구조는 3-3-3에서 멜더가 경계한 "부정" 지각의 전형입니다.
- **피트스톱은 순위를 뒤섞는 강력한 장치입니다**: 이는 러버밴딩 없이 접전을 만드는 방법이기도 합니다.

## 3-6. 드리프트 (Drift)

### 메커니즘

드리프트는 **후륜(또는 전체) 접지를 의도적으로 상실시킨 상태에서 차량을 제어하는 행위**입니다. 게임에서 드리프트는 크게 두 계보로 나뉩니다.

- **물리 창발형**: 하중 이동과 슬립 앵글의 결과로 자연히 발생. 별도 코드가 없음
- **모드 전환형**: 특정 입력(버튼, 조향+제동)으로 진입하는 별개 상태. 물리와 무관하게 규칙으로 정의

### 특징 및 차별점

이 두 계보의 차이는 기술이 아니라 **설계 계약**입니다. 물리 창발형은 "당신이 물리를 이해한 만큼 표현할 수 있다"고 말하고, 모드 전환형은 "이 버튼을 누르면 이 일이 일어난다"고 말합니다.

모드 전환형 드리프트의 대표적 확장은 **드리프트 중 축적되는 부스트**입니다. 이 구조는 드리프트를 "빠른 코너링 기법"이 아니라 "속도 자원 획득 행위"로 재정의합니다. 그 결과 최적 플레이가 "코너를 빠르게 도는 것"이 아니라 "가능한 한 오래 드리프트하는 것"이 되며, 이는 실제 레이싱 물리와 정반대 방향입니다. 이는 결함이 아니라 의도된 설계입니다 — 2-4에서 지적했듯 드리프트 계층은 실력 차를 보상하는 장치이므로, 학습 가능하고 명시적인 규칙일수록 목적에 부합합니다.

1-3에서 인용했듯, 데 라 크루즈는 드리프트가 필수 메커닉이 아님을 로켓 리그를 예로 들어 지적합니다. 드리프트를 넣을지는 **선택**입니다.

### 설계 고려사항

- **드리프트가 최적 라인이면 그것은 레이싱이 아니라 드리프트 게임입니다**: 드리프트 보상이 과하면 모든 코너를 드리프트로 통과하는 것이 지배 전략이 됩니다.
- **진입과 이탈의 대칭성**: 드리프트 진입은 쉽고 이탈이 어려우면 플레이어는 드리프트를 회피합니다.
- **시각·청각 피드백이 곧 조작감입니다**: 드리프트 상태의 판정은 코드에 있지만 플레이어에게는 타이어 연기와 스키드음으로만 존재합니다.

## 3-7. 물리 틱레이트와 결정론

### 메커니즘

- **물리 틱레이트**: 물리 시뮬레이션이 초당 몇 회 갱신되는가
- **렌더 프레임레이트와의 분리**: 물리는 고정 스텝, 렌더는 가변 스텝이 일반적
- **결정론(determinism)**: 같은 입력이 같은 결과를 내는가
- **리플레이 구현 방식**: 입력 재생(결정론 필요) 대 상태 기록(결정론 불필요)

### 특징 및 차별점

물리 틱레이트는 레이싱에서 특히 민감합니다. 타이어의 그립 상실과 회복은 짧은 시간 규모에서 일어나므로, 틱레이트가 낮으면 접지 한계 부근의 거동이 계단식으로 튀거나 아예 표현되지 않습니다. 이 때문에 시뮬레이션 계열은 렌더 프레임레이트보다 훨씬 높은 물리 틱레이트를 사용하는 것이 일반적입니다.

**결정론은 리플레이·고스트 기능의 구현 방식을 규정합니다.** 물리가 결정론적이면 리플레이를 입력 시퀀스만으로 저장할 수 있어 용량이 극적으로 작아집니다. 결정론적이지 않으면 차량의 위치·자세를 주기적으로 기록해야 하며, 용량은 커지지만 물리 변경에 강건합니다.

### 해상도 수치를 읽는 법 — "48배 충실도"라는 사례

3-1에서 인용한 Turn 10의 발표문은 타이어 물리의 해상도 변화를 구체적 수치로 밝힌 드문 1차 자료입니다. 정리하면 다음과 같습니다.

| 항목 | 이전 세대 (Turn 10 진술) | Forza Motorsport (2023) (Turn 10 진술) | 배수 |
|------|------------------------|--------------------------------------|------|
| 접지점 수 (공간 해상도) | 1개 | 8개 | 8배 |
| 연산 주기 (시간 해상도) | 초당 60사이클 | 초당 360사이클 | 6배 |

같은 발표문은 이어서 **"This 48x improvement in tire fidelity results in a more fun and rewarding driving experience"**라고 밝히며, 그 직전 문장에서 이번 물리 도약이 "bigger than Forza Motorsport 5, 6 and 7 combined"라고 표현합니다.

**이 수치는 다음과 같이 읽어야 합니다.**

**첫째, 해상도 수치 자체는 검증 가능한 기술 사양입니다.** 접지점 8개와 초당 360사이클은 구현의 사실이며, 3-1에서 봤듯 그것이 해결하려는 문제(연석에서의 이산화 오차)도 명확히 서술되어 있습니다. 이 부분은 인용해도 좋습니다.

**둘째, "48배 충실도"는 다른 종류의 진술입니다.** 발표문은 **48이라는 수가 어떻게 도출되었는지 설명하지 않습니다.** 다만 발표문이 스스로 밝힌 두 배수 — 접지점 8배와 연산 주기 6배 — 의 곱이 정확히 48이라는 산술적 사실은 확인됩니다. 이 문서는 이 대응이 우연인지 도출 근거인지 **출처가 확정해 주지 않는다**는 점을 명시합니다.

**셋째, 그리고 가장 중요하게 — 해상도의 곱은 충실도의 측정치가 아닙니다.** 이것이 이 절의 핵심 원리입니다. **해상도 증가는 충실도의 필요조건일 뿐 충분조건이 아닙니다** — 잘못된 타이어 모델을 8개의 점에서 초당 360번 계산해도 그것은 여전히 잘못된 모델이며, 더 자주 더 여러 곳에서 틀릴 뿐입니다. **"충실도"는 실측 대비 오차로 정의되는 양**이지 연산량으로 정의되는 양이 아니고, 해상도를 48배로 올렸을 때 오차가 48분의 1로 줄어든다는 보장은 없습니다(수치 해석에서 이런 선형 관계는 오히려 예외입니다). 따라서 **"48배 충실도 향상"은 마케팅 프레이밍으로 취급하고, 인용한다면 반드시 "연산 해상도 기준"임을 병기**하십시오.

**이 사례가 주는 일반 교훈**: 공급자가 제시하는 "N배 향상" 수치는 대개 **측정하기 쉬운 양(연산량, 해상도, 데이터 크기)의 비율**이지 소비자가 관심 있는 양(정확도, 재미, 주행감)의 비율이 아닙니다. 두 양은 상관은 있으나 동일하지 않으며, **상관을 동일성으로 제시하는 것이 이런 수치의 전형적 수사입니다**(5-4 참조).

### 설계 고려사항

- **틱레이트는 물리 모델의 요구에서 역산하십시오**: "높을수록 좋다"가 아니라 "우리 타이어 모델이 안정적으로 수렴하는 최소 스텝"이 기준입니다.
- **결정론은 초기에 결정해야 합니다**: 후반에 결정론을 도입하는 것은 사실상 물리 재작성입니다.
- **부동소수점 비결정성**: 플랫폼 간 결정론은 컴파일러·CPU 아키텍처 수준의 문제이며, 크로스 플랫폼 고스트 공유를 계획한다면 초기에 검증해야 합니다.
- **틱레이트와 러버밴딩의 상호작용**: 3-3-4에서 봤듯 파워 조작은 물리를 통해 노출됩니다. 물리 해상도가 높을수록 이 노출이 정밀해집니다.

---

# 4. 플레이 경험 설계

## 4-1. 속도감 연출

### 메커니즘

- **시야각(FOV) 동적 변화**: 고속에서 FOV를 넓혀 주변 흐름을 가속
- **카메라 위치와 흔들림**: 노면에 가까울수록 속도감 증가
- **모션 블러 / 스피드 라인**: 시각적 흐름 강조
- **엔진음의 피치와 레이어**: 회전수에 연동된 사운드
- **노면 텍스처와 주변물 배치**: 시각적 참조점의 밀도
- **화면 흔들림과 색수차**: 극한 속도의 연출적 과장

### 특징 및 차별점

**속도감은 실제 속도와 독립적으로 설계 가능한 변수입니다.** 같은 시속 200km에서도 위 요소들의 조합에 따라 체감은 완전히 달라집니다 — 즉 **밸런싱을 위해 실제 속도를 낮춰야 할 때, 연출을 강화하여 체감을 보존할 수 있습니다.** 특히 **참조점의 밀도**가 결정적입니다. 사막 한복판을 200km로 달리는 것보다 터널을 100km로 달리는 것이 빠르게 느껴지며, 이는 트랙 설계(3-4)가 속도감 연출의 일부라는 뜻입니다.

### 설계 고려사항

- **속도감과 조작성은 상충합니다**: FOV를 넓히고 카메라를 흔들면 속도감은 오르지만 코너 진입 지점 판단은 어려워집니다.
- **연출 강도를 플레이어에게 넘기십시오**: 모션 블러와 화면 흔들림은 멀미의 주된 원인입니다 (4-5 참조).
- **부스트 순간의 연출 예산을 따로 잡으십시오**: 부스트는 속도감 연출을 몰아 쓰는 순간이며, 평시와 대비될수록 효과적입니다.
- **사운드가 절반입니다**: 엔진음을 끄고 플레이해 보면 속도감의 상당 부분이 청각에서 온다는 것을 알 수 있습니다.

## 4-2. 진입 장벽과 어시스트 사다리

### 메커니즘

어시스트 사다리는 **플레이어가 시간이 지남에 따라 보정을 하나씩 걷어내며 물리에 가까워지는 경로**입니다.

전형적인 단계는 다음과 같습니다.

| 단계 | 상태 | 대상 |
|------|------|------|
| 0단계 | 자동 가속 + 자동 제동 + 주행선 + 전 어시스트 | 첫 완주가 목표인 플레이어 |
| 1단계 | 수동 가속 + 자동 제동 + 주행선 + 안정성 제어 | 조작을 배우기 시작한 플레이어 |
| 2단계 | 수동 제동 + 주행선(코너만) + ABS/TCS | 라인을 배우는 플레이어 |
| 3단계 | 주행선 제거 + ABS/TCS | 트랙을 외운 플레이어 |
| 4단계 | 전 어시스트 해제 + 수동 변속 | 숙련자 |

### 특징 및 차별점

이 사다리 설계의 핵심은 **각 단계가 그 자체로 완결된 게임이어야 한다**는 것입니다. 0단계의 플레이어가 "나는 진짜 게임을 하고 있지 않다"고 느끼면 사다리는 실패합니다.

3-2에서 지적했듯, **완전히 보정되지 않은 레이싱 게임은 존재하지 않습니다.** 속도 의존 조향각 제한은 4단계에서도 대개 작동합니다. 이 사실을 팀이 인지하는 것이 중요합니다 — 사다리의 최상단조차 "순수 물리"가 아니라 설계된 지점입니다.

### 설계 고려사항

- **사다리를 오르는 유인을 설계하십시오**: 보상 배율이 표준적 해법이나, 과하면 강제가 됩니다 (2-2 참조).
- **되돌아갈 수 있어야 합니다**: 어시스트를 껐다가 다시 켜는 것이 수치스럽게 느껴지면 플레이어는 시도 자체를 하지 않습니다.
- **단계 간 간격이 균일해야 합니다**: 특정 단계에서 난이도가 급증하면 그 지점이 이탈 지점이 됩니다.
- **AI 난이도와 어시스트를 분리하십시오**: 이 둘을 하나의 슬라이더로 묶으면 "어시스트는 끄고 싶지만 AI는 쉽게 하고 싶은" 플레이어를 배제합니다.

## 4-3. 공정성 대 재미 — 러버밴딩의 윤리

### 문제의 구조

러버밴딩은 **플레이어에게 알리지 않고 상대의 성능을 조작하는 시스템**입니다. 이를 윤리적으로 정당화할 수 있는가는 레이싱 설계의 오래된 논점입니다.

멜더의 챕터는 이 문제를 도덕이 아니라 **품질**의 문제로 다룹니다. 1-3에서 인용했듯 그는 "when done correctly, this effect can be unnoticeable to the player"라고 쓰며, 실패했을 때 "leave the player feeling cheated"라고 씁니다. 이 관점에서 러버밴딩의 죄는 존재가 아니라 **노출**입니다.

이 입장은 3-3-3의 규칙과 일관됩니다 — 드라이버 스킬 조작이 파워 조작보다 나은 이유가 "부정이 아니어서"가 아니라 **"부정이 일어나지 않기 때문(no 'cheating' is happening)"**, 즉 AI가 같은 차에 타고 있다는 사실 때문이라는 논리입니다. 조작은 여전히 일어나지만, 그 조작이 물리 법칙 안에 머무릅니다.

### 세 가지 입장

| 입장 | 내용 | 장점 | 위험 |
|------|------|------|------|
| 은폐형 | 러버밴딩을 쓰되 데드존·스킬 우선으로 감춤 (멜더의 접근) | 몰입 유지, 접전 보장 | 발각 시 신뢰 붕괴 |
| 공개형 | 캐치업을 명시적 규칙으로 노출 (순위 기반 아이템 분배) | "제도"로 인식되어 배신감 없음 | 선두 경험 훼손, 실력 무의미화 우려 |
| 배제형 | 러버밴딩 없이 실력 기반 매칭·레이팅으로 접전 확보 | 결과의 정당성 | 매칭 풀 필요, 싱글플레이에 적용 곤란 |

**이 세 입장은 우열이 아니라 장르 좌표의 함수입니다.** 2-0 표에서 보듯 카트/파티는 공개형이 자연스럽고, 시뮬레이션은 배제형이 자연스러우며, 심케이드와 아케이드는 은폐형의 주된 영역입니다.

### 캐치업이 코어 루프의 전제인 경우

데이브 마크는 앞의 기고문에서 러버밴딩이 난이도 조율을 넘어 **메커닉의 전제 조건**이 되는 경우를 지적합니다. Split/Second의 파워 플레이는 앞선 차량을 향해 환경 파괴를 발동시키는 메커닉인데, **뒤에 있을 때만 사용할 수 있습니다.** 마크는 이렇게 씁니다. "being in the lead isn't a particularly fun experience when you can't trigger the game's main selling point."

즉 이 게임에서 플레이어가 선두를 오래 유지하면 **게임의 핵심 판매 포인트를 쓸 수 없게 됩니다.** 따라서 러버밴딩은 난이도 장치가 아니라 코어 루프가 작동하기 위한 필수 조건입니다.

**이 사례가 중요한 이유는 러버밴딩의 정당화 논리를 하나 더 제공하기 때문입니다.** 마리오 카트형 모델에서 러버밴딩의 목적은 "무리를 가깝게 유지하는 것"이지만, Split/Second형 모델에서는 "플레이어를 메커닉이 작동하는 위치에 두는 것"입니다. 후자에서 러버밴딩을 제거하면 게임이 더 공정해지는 것이 아니라 **작동을 멈춥니다.**

**단, 강도의 문제는 남습니다.** 마크의 기고문은 당대 Thunderbolt 리뷰를 인용하는데, 그 리뷰어는 "you'll often find the following pack extremely close behind, often catching up six second gaps within two"라고 서술합니다 — 6초 격차를 2초 만에 좁힌다는 것입니다. **이 수치는 리뷰어의 주관적 인상을 블로그 글이 간접 인용한 것이며, 개발사의 텔레메트리나 공개된 튜닝 값이 아닙니다.** 이 문서는 이를 검증된 수치가 아니라 **"캐치업이 과하다고 지각된 사례"**로만 인용합니다. 그럼에도 교훈은 유효합니다 — **코어 루프가 캐치업을 요구하더라도, 캐치업이 지각 가능해지면 여전히 비판받습니다.**

### 장르를 넘는 패턴

마크는 이 패턴이 레이싱에 국한되지 않는다고 일반화합니다. "AI in shooters is really cut from the same template of the rubber-banding in Mario Kart then. 'Do well, and then lose.'"

이 일반화는 러버밴딩을 **동적 난이도 조정(DDA)의 한 사례**로 위치시킵니다. 레이싱의 러버밴딩 논의가 다른 장르에 이식 가능한 이유가 여기 있습니다. 관련 설계는 [AI / NPC 행동 & 적/몬스터 난이도 설계](121-ai_npc_behavior_difficulty.md)를 참조하십시오.

### 설계 고려사항

- **입장을 명시적으로 선택하고 문서화하십시오**: 팀이 은폐형인지 공개형인지 합의하지 않으면 시스템이 중간에서 표류합니다.
- **은폐형을 택했다면 노출 경로를 전수 조사하십시오**: 3-3-4의 오디오 사례처럼, 노출은 예상치 못한 채널로 일어납니다.
- **커뮤니티는 결국 알아냅니다**: 리플레이와 텔레메트리가 있는 게임에서 러버밴딩은 데이터로 측정됩니다. 은폐를 전략의 전부로 삼지 마십시오.
- **캐치업의 필요성을 먼저 물으십시오**: 트랙 설계(추월 지점), 피트스톱, 아이템, 매칭으로 접전을 만들 수 있다면 러버밴딩은 필요 없을 수 있습니다.

## 4-4. 리플레이·고스트

### 메커니즘

- **리플레이**: 주행 전체를 재생. 카메라 자유 조작
- **고스트**: 과거 주행을 반투명 차량으로 동시 표시
- **개인 최고 고스트 / 랭킹 고스트 / 친구 고스트**
- **섹터 분할 비교**: 구간별 시간 차 표시

### 특징 및 차별점

고스트는 **레이싱 장르가 발명한 가장 우아한 비동기 경쟁 장치**입니다. 상대가 실시간으로 존재하지 않아도 경쟁이 성립하며, 충돌하지 않으므로 방해가 없고, 자신의 과거와 경쟁하므로 실력 격차 문제가 없습니다.

**고스트는 러버밴딩의 완전한 대안이기도 합니다.** 고스트는 결코 캐치업하지 않지만, 플레이어가 자신의 이전 기록을 고스트로 삼으면 격차는 자연히 좁혀집니다 — 어제의 나와 오늘의 나는 항상 비슷하기 때문입니다. 즉 **자기 자신을 상대로 삼으면 러버밴딩 없이 접전이 보장됩니다.** 3-7의 결정론이 여기서 실질적 의미를 갖습니다 — 결정론적 물리는 고스트를 입력 시퀀스로 저장할 수 있게 해주며, 이는 랭킹 서버의 저장 비용과 직결됩니다.

### 설계 고려사항

- **섹터 분할이 학습을 만듭니다**: 총 랩타임 차이만 보여주면 플레이어는 어디서 잃었는지 모릅니다.
- **고스트의 시각적 방해**: 반투명 처리와 충돌 무시는 필수이나, 시야를 가리는 문제는 남습니다.
- **물리 패치와 기록의 무효화**: 물리를 수정하면 기존 기록의 정당성이 사라집니다. 시즌 분리나 기록 아카이빙 정책이 필요합니다.
- **리플레이는 검증 도구입니다**: 치팅 판별과 러버밴딩 검증 모두에 쓰이므로, 개발 초기부터 내부 도구로 확보하십시오.

## 4-5. 접근성

### 메커니즘

- **조작 접근성**: 자동 가속/제동, 한 손 조작, 입력 리매핑, 감도 조절
- **시각 접근성**: 색맹 대응(주행선·미니맵·순위 색상), UI 크기 조절, 고대비 모드
- **청각 접근성**: 시각적 사운드 큐(뒤에서 접근하는 차량 표시)
- **전정 접근성(멀미)**: 모션 블러·카메라 흔들림·FOV 조절, 화면 고정 옵션
- **인지 접근성**: 주행선, 코너 예고, 미니맵 확대

### 특징 및 차별점

레이싱은 **멀미 유발도가 특히 높은 장르**입니다. 고속 이동, 카메라 흔들림, 시야 회전이 결합되기 때문이며, 4-1에서 다룬 속도감 연출 기법의 상당수가 그대로 멀미 유발 요인입니다. **즉 이 장르에서 접근성과 연출은 직접적으로 충돌합니다.** 해법은 연출을 포기하는 것이 아니라 **연출 강도를 개별 슬라이더로 분리**하는 것입니다 — 모션 블러, 화면 흔들림, FOV, 속도선을 각각 독립 조절 가능하게 하면 민감한 플레이어가 자신에게 맞는 지점을 찾을 수 있습니다.

**주행선은 접근성 기능이면서 동시에 난이도 기능입니다.** 이 이중성은 4-2의 어시스트 사다리 설계와 충돌할 수 있습니다 — 주행선을 "초심자용 보조 바퀴"로 프레이밍하면 인지 접근성이 필요한 플레이어가 사용을 주저하게 됩니다. 어시스트와 접근성 설정을 UI에서 분리하는 것이 안전합니다.

### 설계 고려사항

- **자동 가속을 부끄러운 옵션으로 만들지 마십시오**: 이는 운동 장애가 있는 플레이어에게 필수 기능입니다.
- **색상에만 의존하지 마십시오**: 주행선의 제동 구간 표시가 색상만으로 되어 있으면 색맹 플레이어에게 전달되지 않습니다. 패턴이나 형태를 병용하십시오.
- **오디오 큐를 시각화하십시오**: 뒤에서 접근하는 차량의 엔진음은 중요한 정보이며, 시각적 대체물이 필요합니다.
- **햅틱을 정보 채널로 쓰십시오**: 그립 상실 경고는 진동으로 전달할 수 있으며, 이는 청각·시각 채널의 부담을 덜어줍니다.

상세 지침은 [접근성 가이드라인](175-accessibility_guidelines.md)을 참조하십시오.

---

# 5. 주요 사례 분석

이 장의 모든 사례는 부록에 명시된 출처에서 직접 확인한 내용만을 다루며, 개발사의 자기 진술은 귀속 서술로 처리합니다.

## 5-1. 그란 투리스모와 소니 AI의 Nature 논문 (2022) — 귀속의 교과서

### 사실 관계

2022년 2월 9일, Nature 602권 223-228쪽에 "Outracing champion Gran Turismo drivers with deep reinforcement learning"이 게재되었습니다. 저자는 27명이며, **전원이 소니 AI(Sony AI) 소속**입니다 — 뉴욕, 도쿄, 취리히 세 거점에 분포합니다. 그란 투리스모를 개발한 폴리포니 디지털(Polyphony Digital)은 **저자 소속으로 등재되어 있지 않으며, 감사(Acknowledgements) 절에만 등장합니다.**

논문은 도입부에서 다음과 같이 기술합니다.

> "Automobile racing represents an extreme example of these conditions; drivers must execute complex tactical manoeuvres to pass or block opponents while operating their vehicles at their traction limits. Racing simulations, such as the PlayStation game Gran Turismo, faithfully reproduce the non-linear control challenges of real race cars..."

논문은 모델 프리 심층 강화학습 에이전트 GT Sophy가 세계 최고 수준의 그란 투리스모 드라이버 4명과의 맞대결에서 승리했다고 보고하며, 해당 이벤트는 **2021년 7월 2일과 2021년 10월 21일**에 개최되었다고 밝힙니다.

### 이 사례를 인용할 때의 규칙

**이 문서는 "Nature가 GT의 물리 충실도를 입증했다"고 쓰지 않습니다.** 이 표현은 세 가지 이유로 사실과 다릅니다.

**첫째, 이해충돌 구조입니다.** 저자 27명 전원이 소니 AI 소속이고, 폴리포니 디지털 역시 소니의 자회사입니다. 즉 이 문장은 **소니가 소니 제품을 평가한 것**입니다. 논문의 경합이익(competing interests) 진술은 미국 임시특허출원 63/267,136만을 다루며, 이 관계를 공개하지 않습니다. 동료 심사를 거친 Nature 논문이라는 사실이 이 구조를 상쇄하지 않습니다 — 심사는 강화학습 방법론의 타당성을 검토한 것이지, 도입부의 무인용 프레이밍 문장을 검증한 것이 아닙니다.

**둘째, 범위의 문제입니다.** 논문이 충실하다고 주장하는 것은 **"비선형 제어 과제(non-linear control challenges)"**이지 물리 충실도 일반이 아닙니다. 그리고 결정적으로, **논문은 GT의 타이어 그립·에어로다이내믹·서스펜션 모델을 일절 문서화하지 않습니다.** 논문은 GT 물리 엔진을 블랙박스로 다룹니다. 따라서 이 논문에서 "GT의 타이어 모델이 정확하다"는 결론을 끌어낼 수 없습니다 — 논문에 타이어 모델에 대한 기술 자체가 없기 때문입니다.

**셋째, 시의성입니다.** 논문 본문은 제품을 "Gran Turismo"로만 지칭하며 특정 버전을 명시하지 않습니다(참고문헌에 "Super-human performance in Gran Turismo Sport using deep reinforcement learning"이라는 선행 연구가 인용되어 있습니다). 확실한 것은 **맞대결 이벤트가 2021년에 열렸고 게재가 2022년 2월 9일이라는 점**이며, GT7이 2022년 3월에 발매되었으므로 **이 논문은 현행 GT7 물리에 대한 진술이 아닙니다.** GT7은 이후에도 물리 개정을 포함한 업데이트를 출하해 왔습니다.

### 올바른 표기 형식

- **잘못된 표기**: "Nature 논문이 그란 투리스모의 물리 충실도를 입증했다."
- **올바른 표기**: "소니 AI 연구진의 Nature 논문(2022)은 그란 투리스모가 실제 레이스카의 비선형 제어 과제를 충실히 재현한다고 **규정합니다**."

### 설계 교훈

1. **접지 한계 운용은 장르의 과제 정의입니다**: 논문의 주행 과제 정의 — 차량을 접지 한계에서 운용하며 추월·블로킹의 전술 기동을 수행하는 것 — 는 하위 장르를 막론하고 레이싱 설계의 좌표를 잡는 데 유용합니다 (1-2 참조).
2. **AI 상대의 대안 경로가 존재합니다**: 이 논문은 학습 기반 AI가 최상위 인간 드라이버와 맞대결에서 승리한 사례를 보고합니다. 이는 러버밴딩 외의 AI 설계 경로가 연구 수준에서 실증되고 있음을 보여줍니다. 다만 이것이 상용 게임의 표준 AI로 이식 가능한지는 이 논문이 답하는 문제가 아닙니다.
3. **가장 중요한 교훈은 인용 규율 자체입니다**: 최고 권위지에 게재된 동료 심사 논문조차, 저자 구성과 진술 범위를 확인하지 않으면 오인용됩니다. **권위는 출처의 속성이지 문장의 속성이 아닙니다.**

## 5-2. Just Cause 4 (2018) — 아케이드 타이어 모델의 제안

### 사실 관계

어밸런치 스튜디오의 해미시 영(당시 Just Cause 4의 리드 메카닉 디자이너)은 GDC 2019에서 "Vehicle Physics and Tire Dynamics in 'Just Cause 4'"를 발표했습니다. 공식 세션 설명에 따르면 이 세션은 다양한 탈것과 플레이어 스타일을 타이어 동역학 설계로 수용하는 방법을 다루며, 발표자는 매직 포뮬러 타이어 모델을 검토하고 그것이 아케이드·심케이드 핸들링에 최적이 아닐 수 있는 이유를 살핀 뒤, 준경험적이지 않고 파라미터가 현저히 적으며 더 플레이어 친화적인 결과를 내는 디자이너 친화적 접근을 제안한다고 기술되어 있습니다.

**강연 본문의 논증은 3-1에 정리했습니다.** 이 절은 그 논증을 사례로서 어떻게 읽고 인용할 것인가에 집중합니다.

### 초록과 본문의 간극 — 이 사례가 가르치는 인용 규율

이 사례는 **공식 초록만 읽었을 때와 강연 본문을 확인했을 때 도달하는 결론이 달라지는** 드문 예입니다. 두 단계를 구분해 두는 것이 이 문서의 규율입니다.

**초록만으로 알 수 있었던 것**: 초록은 "may not be optimal"이라는 헷지를 쓰고, **파라미터 감소·비준경험적 구성·더 나은 결과를 병렬로 나열할 뿐 인과를 진술하지 않습니다.** 따라서 초록만 근거로 "매직 포뮬러가 아케이드에 부적합하다"거나 "파라미터를 줄여서 더 나은 결과를 얻었다"고 쓰는 것은 원문을 넘어선 서술이었습니다.

**본문이 실제로 채운 것**: 강연 본문은 헷지 뒤에 있던 논증을 드러냅니다. 영이 실측 타이어 모델을 물리치는 근거는 **비용이 아니라 설계 목표의 충돌**이며(제동 중 언더스티어로 T자 교차로를 못 도는 것은 오픈월드에서 수용 불가), 파라미터 감소는 **목표가 아니라 마찰 클램프의 부산물**입니다 — 클램프가 안정성을 떠맡으면 곡선은 현실을 재현할 의무에서 풀려나고, 그때 비로소 계측 피팅용 계수가 불필요해집니다.

**그럼에도 유지되는 한계**: 본문이 인과의 방향을 밝혀 주었다고 해서 이것이 **검증된 결과**가 되지는 않습니다. 이는 **한 팀이 하나의 출시작에서 택한 경로에 대한 실무자의 회고**이며, 통제된 비교 실험이 아닙니다. 영 자신도 유효 질량을 "휴리스틱"이라 부르고 "완벽하지 않다"고 명시합니다. **"영은 …라고 설명한다"는 귀속 서술이 여전히 정확한 표기이며, "파라미터가 적은 모델이 아케이드에 더 낫다"는 일반 명제로 승격시켜서는 안 됩니다.**

### 설계 교훈

1. **목표가 모델을 배제합니다 — 비용이 아니라**: 실측 타이어를 쓰지 않는 이유가 "비싸서"가 아니라 **"우리 게임에서는 그 정확한 거동이 틀린 답이라서"**일 수 있습니다. 영이 명시한 목표는 **제한된 인지 부하** — 운전하면서 다른 일을 생각할 수 있어야 한다 — 였고, 그 목표 아래에서 정확한 언더스티어는 결함입니다. **모델 선택 논의를 성능 논의로 시작하면 이 층위를 놓칩니다.**
2. **안정성을 사실성에서 분리하면 자유가 생깁니다**: 마찰 클램프가 물리적 타당성의 하한을 구조적으로 보장하기 때문에 그 위의 곡선은 자유로워집니다. **"디자이너가 마음껏 하게 하라"는 태도가 아니라, 마음껏 해도 깨지지 않는 구조를 먼저 만드는 공학**입니다. 영은 강연 말미에 카트 레이서 물리에서 출발한 인디 개발자에게 **"먼저 바퀴의 마찰 클램프를 제대로 잡아라, 그러면 타이어 모델은 원하는 대로 해도 된다"**는 순서를 권합니다.
3. **탈것의 다양성이 모델 요구를 규정합니다**: 단일 핸들링 체계로 이질적 탈것을 수용해야 한다면, 각 탈것마다 실측 데이터를 요구하는 모델은 구조적으로 불리합니다. 영의 팀은 곡선을 정규화해 게임 전체를 소수의 타이어 세트로 덮고, 탈것별 저작을 **앞·뒤 그립 값**으로 축소했습니다.
4. **거짓말은 국소화하십시오**: 과잉 그립의 부작용을 서스펜션을 비현실적으로 만들어 상쇄하는 대신, **힘 적용 단계에서 롤·피치 성분만 축소**해 나머지 물리를 정직하게 남겼습니다. 아케이드 물리의 품질은 거짓말의 양이 아니라 **국소성**이 결정합니다.
5. **믿을 만함과 재미는 명시적 교환 관계입니다**: 영은 목표를 "실제로 사실적인" 것이 아니라 **따옴표 친 "믿을 만한(believable)"** 것으로 두었다고 밝힙니다. **충실도를 기본 목표로 두지 않는 태도** 자체가 이 사례의 핵심 전달점입니다.

## 5-3. 마리오 카트 월드 (2025) — 단일 월드 전환과 24인

### 사실 관계

닌텐도는 2025년 5월 21일 공식 인터뷰 시리즈 "Ask the Developer Vol. 18 — Mario Kart World Part 1"을 게재했습니다. 참여자는 프로듀서 야부키 코스케(Kosuke Yabuki), 프로그래밍 디렉터 사토 켄타(Kenta Sato), 아트 디렉터 이시카와 마사아키(Masaaki Ishikawa), 기획 팀 리드 지쿠마루 신타로(Shintaro Jikumaru), 음악 리드 아사히 아츠코(Atsuko Asahi)입니다.

**트랙 구조의 전환에 대해** 닌텐도는 다음과 같이 밝혔습니다.

> "Up to the previous game, we created each course separately as an independent element, but this time, we ended up with a map like this because all the courses are connected as part of the same world."

야부키는 이 구조의 실현 조건에 대해 이렇게 밝혔습니다.

> "I thought that with modern technology, being able to seamlessly transition between courses and realize a single, vast world wasn't beyond the realm of possibility."

**동시 주행 인원에 대해** 야부키는 이렇게 밝혔습니다.

> "Yes, the previous game was for up to 12 players, and we decided fairly early on that this game would be for up to 24 players."

그 사유에 대해서는 이렇게 밝혔습니다.

> "By creating long routes in a vast world, you could end up with players spread out in various places, which could diminish the sense that they're racing against each other."

아트 디렉터 이시카와는 시각적 측면을 덧붙였습니다.

> "once players spread out, the course starts to look sparse, and the visuals give off a sort of lonely feel."

프로젝트의 성격에 대해 야부키는 이렇게 밝혔습니다.

> "If the idea had just been to add more courses, then I think we would've called it Mario Kart 9."

### 이 사례를 인용할 때의 규칙

**첫째, 인원 증가를 캐치업의 대체재로 서술해서는 안 됩니다.** 이 문서를 작성하며 해당 인터뷰를 직접 확인한 결과, **러버밴딩·캐치업·AI 난이도는 이 결정과 연결되어 단 한 번도 언급되지 않습니다.** 따라서 "캐치업 메커니즘에 의존하는 대신 인원을 늘렸다"는 식의 서술은 **출처에 근거가 없는 창작**입니다. 인터뷰가 밝히는 인과는 오직 하나입니다 — **넓은 세계의 긴 경로가 플레이어를 분산시키고, 분산이 경쟁감을 희석시키므로, 인원을 늘려 접전 밀도를 확보한다.**

**둘째, 기술은 동기가 아니라 실현 조건입니다.** 야부키의 "modern technology" 발언은 **단일 월드가 가능해진 조건**을 말한 것이지, 기술 때문에 그런 결정을 내렸다는 동기 진술이 아닙니다. 그가 밝힌 동기는 별도로 존재합니다 — 마리오 카트 8 디럭스에서 기존 공식을 완성했기 때문에 이번에는 넓은 세계를 주행하는 게임플레이를 원했다는 것입니다.

**셋째, 출처의 성격을 감안해야 합니다.** Ask the Developer는 게임 출시(2025년 6월) 약 2주 전에 게재된 **닌텐도 자체 프로모션 시리즈**입니다. 이는 **"개발진이 표명한 설계 의도"의 최적 출처이지, 감사된 의사결정 기록이 아닙니다.** 따라서 이 문서는 모든 진술을 "…라고 밝혔습니다" 형태로 귀속합니다.

**넷째, 미세한 정정**: "이전작"은 엄밀히 말해 마리오 카트 8(2014)이 아니라 마리오 카트 8 디럭스(2017)이나, 두 버전 모두 12명이므로 실질적 오류는 아닙니다.

### 설계 교훈

1. **공간 규모는 경쟁 밀도의 적입니다**: 이것이 이 사례의 이식 가능한 핵심입니다. 세계를 넓히면 플레이어가 흩어지고, 흩어지면 경쟁감이 사라집니다. 이는 2-5의 오픈월드 레이싱이 구조적으로 안고 있는 문제이며, 마리오 카트 월드가 택한 대응은 **분모(공간)를 줄이는 대신 분자(인원)를 늘리는 것**이었습니다.
2. **밀도 해법의 선택지는 여럿입니다**: 인원 증가는 그중 하나일 뿐입니다. 경로를 좁히기, 체크포인트로 재집결시키기, 캐치업 강화 등이 모두 가능합니다. **닌텐도가 인원을 택했다는 사실이 다른 해법이 열등하다는 근거는 아닙니다.**
3. **시각적 공허함은 별도의 문제입니다**: 이시카와의 발언은 분산이 게임플레이뿐 아니라 **화면의 시각적 밀도**도 훼손한다는 점을 지적합니다. 넓은 맵의 공허감은 게임플레이 지표로 측정되지 않을 수 있습니다.
4. **구조적 재발명과 반복적 속편은 다른 프로젝트입니다**: 야부키의 "Mario Kart 9" 발언은 **코스 추가가 목표였다면 다른 이름을 붙였을 것**이라는 선언이며, 명명 자체를 프로젝트 성격의 신호로 사용한 사례입니다.

## 5-4. Forza Motorsport의 Drivatar — 표명된 의도와 출시된 동작의 간극

### 사실 관계와 이 문서의 취급

Turn 10 스튜디오는 2023년 7월 17일, 크리에이티브 디렉터 크리스 에사키(Chris Esaki) 명의로 자사 사이트에 신규 AI와 물리를 설명하는 글을 게재했습니다. 이는 2023년 10월 10일 출시 **약 3개월 전에 나온 1차 개발사 홍보 자료**입니다.

이 글에서 Turn 10은 새로운 AI 컨트롤러가 모든 차량-트랙 조합을 마스터했으며, **"The computer drives each track 26,000 times to achieve the fastest line through every layout!"**라고 밝혔습니다.

### 하나의 발표문, 두 종류의 주장

**이 사례의 핵심은 같은 발표문 안에 성격이 전혀 다른 두 종류의 주장이 나란히 있다는 것입니다.** 이 구분이 이 절 전체의 요점입니다.

| | 타이어 물리 주장 | AI 거동 주장 |
|---|---|---|
| 내용 | 접지점 8개·초당 360사이클, 이전 세대는 1개·60사이클 | "치트·핵·러버밴딩을 일절 쓰지 않고" 최속 인간 드라이버 수준의 속도를 낸다 |
| 주장의 종류 | **검증 가능한 기술 사양** — 구현의 사실이며 해결하려는 문제(연석 이산화 오차)가 함께 서술됨 | **설계 목표 선언** — 출시 빌드의 실제 동작이 아니라 의도의 표명 |
| 이 문서의 처리 | **채택** (3-1에 인과와 함께, 3-7에 수치 독법과 함께) | **배제** |
| 배제/채택 근거 | 원문에 축자로 존재하며 비교 대상까지 명시. 단 "48배 충실도"는 해상도 기준임을 병기해야 함 (3-7) | 부재는 증명 불가하나 존재는 플레이어가 고발함. 출시 후 공식 포럼에 AI 러버밴딩 관련 이슈 스레드가 등재된 정황이 보고된 바 있어, **출시 빌드의 동작 근거로 인용하기에 부적격** |

**이 대비가 이 사례를 선명하게 만듭니다.** 개발사 홍보물이라는 이유만으로 문서 전체를 기각하는 것도, 1차 출처라는 이유만으로 전체를 수용하는 것도 모두 잘못된 독법입니다. **같은 저자의 같은 글에서도 주장마다 검증 가능성이 다릅니다** — 타이어 접지점 수는 구현하면 그렇게 되는 사양이지만, "러버밴딩을 쓰지 않는다"는 출시된 소프트웨어의 모든 상황에서의 거동에 대한 전칭 명제입니다. **전자는 사양서의 문장이고 후자는 약속의 문장이며, 약속은 깨질 수 있습니다.**

**26,000회 수치 역시 개발사의 자기 진술이며**, 이 문서는 이를 "Turn 10이 …라고 밝혔습니다"로만 인용합니다. 이 값이 어떻게 측정되었는지, 어떤 트랙에 대한 값인지, 출시 빌드에 반영된 값인지는 해당 글에서 확인되지 않습니다.

### 설계 교훈

**이 사례의 진짜 교훈은 Drivatar의 기술이 아니라, 표명된 의도와 출시된 동작 사이의 간극을 다루는 방법입니다.**

1. **"러버밴딩 없음"은 검증 가능한 주장이 아니라 마케팅 포지션이 되기 쉽습니다**: 러버밴딩의 부재는 증명하기 어렵고, 존재는 플레이어가 체감으로 고발합니다. 이 비대칭 때문에 "우리는 치트를 쓰지 않는다"는 선언은 **약속이 아니라 부채**가 됩니다. 출시 후 단 한 번의 이상 동작이 그 선언을 반증 사례로 만듭니다.
2. **오프라인 학습은 러버밴딩의 대안이지만 비용이 있습니다**: 러버밴딩 없이 경쟁력 있는 AI를 만들려면 AI 자체가 실제로 빨라야 하며, Turn 10의 진술은 그 비용의 규모를 시사합니다. 다만 오프라인 학습으로 **빠른** AI를 만드는 것과, 플레이어 실력에 **맞는** AI를 만드는 것은 다른 문제입니다. 후자는 여전히 난이도 조정을 요구합니다.
3. **기획서에 "우리는 러버밴딩을 쓰지 않는다"고 쓰기 전에 3-3-5를 읽으십시오**: 멜더가 열거한 세 가지 비활성화 상황은, 러버밴딩을 **쓰는** 팀조차 그것을 끄는 규칙을 정교하게 설계해야 함을 보여줍니다. 러버밴딩을 아예 쓰지 않겠다는 선언은 그 모든 상황을 AI의 순수 실력만으로 처리하겠다는 뜻이며, 이는 훨씬 어려운 약속입니다.
4. **출처 단위가 아니라 주장 단위로 판정하십시오**: 개발사 블로그를 통째로 신뢰하거나 통째로 기각하는 것은 둘 다 게으른 독법입니다. 위 대비표가 보여주듯 **같은 글 안에서도 사양의 문장과 약속의 문장은 검증 가능성이 다릅니다.** 전자는 귀속 서술로 인용할 수 있고, 후자는 출시 빌드의 동작 근거가 될 수 없습니다.

## 5-5. Split/Second — 캐치업이 코어 루프의 전제인 구조

### 사실 관계

데이브 마크는 2010년 6월 2일 Game Developer에 "Rubber-banding as a Design Requirement"를 기고했습니다. 이 글은 Split/Second의 러버밴딩을 난이도 조율이 아니라 **메커닉의 전제 조건**으로 분석합니다.

핵심 논리는 이 게임의 파워 플레이 메커닉이 **앞선 차량을 향해서만** 발동 가능하다는 점입니다. 마크는 이렇게 씁니다. "being in the lead isn't a particularly fun experience when you can't trigger the game's main selling point."

### 설계 교훈

1. **캐치업의 정당화 논리는 하나가 아닙니다**: 마리오 카트형 모델에서 캐치업의 목적은 "무리를 가깝게 유지하는 것"이고, Split/Second형 모델에서는 "플레이어를 메커닉이 작동하는 위치에 두는 것"입니다. **후자에서 캐치업 제거는 공정성 개선이 아니라 게임의 정지입니다.**
2. **메커닉의 발동 조건이 순위 구조를 규정합니다**: "뒤에 있을 때만 쓸 수 있는 메커닉"을 설계하는 순간, 그 게임은 **선두를 벌로 만드는 구조**를 갖게 됩니다. 이는 2-4에서 지적한 카트 게임의 "선두의 경험" 문제와 같은 계열입니다.
3. **필요하다고 해서 지각 가능해도 되는 것은 아닙니다**: 4-3에서 인용한 Thunderbolt 리뷰어의 인상(6초 격차를 2초 만에 좁힌다)은 **주관적 지각이지 검증된 수치가 아니지만**, 캐치업이 코어 루프에 필요한 게임에서도 강도가 과하면 비판받는다는 점을 보여줍니다. **필요성은 강도를 정당화하지 않습니다.**
4. **패턴은 장르를 넘습니다**: 마크는 슈터의 AI 난이도 조정이 마리오 카트 러버밴딩과 같은 템플릿("Do well, and then lose")이라고 일반화합니다. 레이싱의 러버밴딩 설계는 DDA 일반의 사례 연구로 읽을 수 있습니다.

## 5-6. MotorStorm — 물리의 부산물을 콘텐츠로 승격시킨 사례

### 사실 관계

리비오 데 라 크루즈는 2016년 8월 12일 Game Developer 기고문에서, 모터스톰의 물리가 작은 노면 요철에서도 차량이 쉽게 접지를 잃게 만들었다고 서술합니다. 그리고 에볼루션 스튜디오가 이를 "그냥 일어나는 일"로 취급하는 대신 **"designing tracks that explicitly called attention to this mechanic, thus challenging players to master it"** 하는 쪽을 택한 것으로 보인다고 씁니다.

### 이 사례를 인용할 때의 규칙

**이는 기고자의 외부 추론이지 에볼루션 스튜디오의 진술이 아닙니다.** 원문 자체가 "seemed to"라는 표현으로 추정임을 밝히고 있습니다. 따라서 이 사례는 **"에볼루션이 그렇게 했다"가 아니라 "트랙 설계가 물리를 뒤따를 수 있다"는 설계 원리의 예시**로만 사용해야 합니다.

### 설계 교훈

1. **창발적 물리 거동은 설계 분기점입니다**: 의도하지 않은 거동이 발견되면 선택지는 제거와 승격 두 가지입니다. 제거가 기본값이어야 할 이유는 없습니다.
2. **승격의 조건은 학습 가능성입니다**: 부산물을 스킬로 승격시키려면 그것이 **일관되고 예측 가능해야** 합니다. 무작위로 발생하는 접지 상실은 학습 대상이 아니라 짜증의 원인입니다.
3. **트랙이 물리를 조명합니다**: 3-4에서 지적했듯, 물리 특성과 무관하게 설계된 트랙은 어떤 물리 엔진에서도 밋밋합니다. **물리팀과 레벨팀의 분리는 이 사례의 반대 방향으로 작동합니다.**

---

# 6. 기획자를 위한 디자인 가이드라인

## 6-1. 설계 원칙

### 원칙 1: 좌표를 먼저 정하십시오

물리 충실도와 접근성의 스펙트럼(2-0) 위에서 **자기 게임의 좌표를 명시적으로 선언**하십시오. 이 선언이 없으면 팀은 매 결정마다 "더 현실적으로"와 "더 재미있게" 사이에서 표류합니다. 좌표가 정해지면 대부분의 하위 결정이 자동으로 따라옵니다 — 타이어 모델의 선택(3-1), 어시스트의 기본값(3-2), 러버밴딩의 입장(4-3)이 모두 이 좌표의 함수입니다.

### 원칙 2: 아케이드는 시뮬레이션의 부분집합이 아닙니다

아케이드를 목표한다면 **아무것도 없는 상태에서 필요한 메커닉만 쌓아 올리십시오**(1-1). 시뮬레이션을 먼저 만들고 덜어내는 접근은 복잡성은 남고 명료함은 없는 결과를 냅니다.

### 원칙 3: 스킬을 먼저, 파워는 마지막에

러버밴딩을 구현한다면 **드라이버 스킬 조작을 우선하고 파워 조작은 스킬이 포화한 뒤의 폴백으로 두십시오**(3-3-3). 이유는 도덕이 아니라 물리입니다 — 파워 조작은 물리 엔진이 스핀과 소리로 폭로합니다(3-3-4).

### 원칙 4: 데드존이 지각을 지배합니다

러버밴딩의 존재를 감추는 것은 정교한 곡선이 아니라 **플레이어 근처에서 시스템을 완전히 꺼버리는 것**입니다(3-3-2). 데드존 폭은 "AI가 부당한 가시적 우위를 갖지 않으면서도 백미러 안까지는 따라붙을 수 있는" 범위에서 정합니다(3-3-7).

### 원칙 5: 거리 하나로는 부족합니다

러버밴딩은 거리만 봅니다. **스타트·저속·랩 뒤처짐에서는 반드시 무효화**하십시오(3-3-5). 특히 스타트에서는 부호가 뒤집혀, 앞선 차량을 밀어주는 것이 옳습니다.

### 원칙 6: 필수 메커닉은 없습니다

드리프트도, 부스트도, 변속도, 타이어 슬립도 필수가 아닙니다(1-3). 각 메커닉은 **왜 필요한지 답할 수 있을 때만** 넣으십시오.

### 원칙 7: 러버밴딩 이전에 대안을 검토하십시오

접전을 만드는 방법은 러버밴딩만이 아닙니다. **실력 기반 매칭**(2-1), **피트스톱**(3-5), **추월 지점 설계**(3-4), **고스트**(4-4), **공개형 아이템 분배**(2-4)가 모두 대안입니다. 러버밴딩은 이들이 불가능하거나 불충분할 때의 선택입니다.

### 원칙 8: 출처를 귀속하십시오

개발사가 자사 제품에 대해 말한 것은 **의도의 증거이지 동작의 증거가 아닙니다**(5-4). 기획서에 경쟁사 사례를 인용할 때 이 구분이 무너지면, 팀은 마케팅 문구를 기술 목표로 착각하게 됩니다.

## 6-2. 개발 단계별 체크리스트

### 프리프로덕션

- [ ] 물리 충실도-접근성 스펙트럼 위의 목표 좌표를 문서로 선언했는가
- [ ] 타이어 모델의 선택 근거가 "정확도"가 아니라 "목표 감각 대비 비용"으로 서술되어 있는가
- [ ] 파라미터를 누가 튜닝할 것인가(물리 프로그래머 / 디자이너)를 결정했는가
- [ ] 러버밴딩의 입장(은폐형/공개형/배제형)을 팀이 합의했는가
- [ ] 러버밴딩 없이 접전을 만드는 대안을 최소 2개 검토했는가
- [ ] 결정론 요구 여부를 결정했는가 (리플레이·고스트·크로스 플랫폼 랭킹의 전제)
- [ ] 물리 틱레이트를 타이어 모델의 수렴 요구에서 역산했는가
- [ ] 어시스트 목록과 각 어시스트의 대상 플레이어를 정의했는가
- [ ] 속도 의존 조향각 제한 같은 "보이지 않는 보정"을 내부 문서에 명시했는가

### 프로토타입

- [ ] 어시스트를 모두 끈 상태에서 트랙 1랩 완주가 가능한가
- [ ] 어시스트를 모두 켠 상태에서 게임이 지루하지 않은가
- [ ] 물리가 극한 상황(고속 스핀, 공중 낙하, 벽면 접촉)에서 발산하지 않는가
- [ ] 창발적 물리 거동 중 승격 후보를 식별했는가 (5-6 참조)
- [ ] 러버밴딩 데드존 폭의 초기값을 정하고 근접 경쟁 시 AI 우위가 보이지 않음을 확인했는가
- [ ] 스타트 직후 첫 코너에서 파일업이 발생하지 않는가
- [ ] 속도감 연출 없이도 조작이 성립하는가 (연출로 문제를 덮고 있지 않은가)

### 프로덕션

- [ ] 러버밴딩 무효화 규칙 3종(스타트/저속/랩 뒤처짐)이 모두 구현되어 있는가
- [ ] 스타트에서 역방향 밴딩(앞선 차량 가속)이 적용되는가
- [ ] 뭉침 대응(평균 위치 또는 최원거리 기준)이 구현되어 있는가
- [ ] 1위 전용 러버밴딩(1위-2위 간 거리 기준)이 있는가
- [ ] 분할 화면에서 기준 차량 규칙(최후위=역방향, 최선두=전방)이 구현되어 있는가
- [ ] 파워 조작 시 AI가 코너 탈출에서 스핀하지 않는가 (추가 파워 전제 셋업 또는 그립 보강 적용 여부)
- [ ] 파워 조작 시 오디오가 리미터를 계속 때리지 않는가
- [ ] 저출력 AI가 언덕에서 멈추거나 변속 불능이 되지 않는가
- [ ] 어시스트 단계 간 난이도 간격이 균일한가
- [ ] 접근성 설정과 어시스트 설정이 UI에서 분리되어 있는가
- [ ] 연출 강도(모션 블러/흔들림/FOV)가 개별 슬라이더로 분리되어 있는가

### 출시 전

- [ ] 리플레이로 러버밴딩의 가시성을 검증했는가 (플레이어가 하는 검증을 먼저 하십시오)
- [ ] 텔레메트리로 AI의 파워 배수 분포를 측정했는가
- [ ] 색상에만 의존하는 정보 전달이 없는가
- [ ] 물리 패치 시 기존 랭킹 기록의 처리 정책이 있는가
- [ ] 홍보 자료에 검증 불가능한 절대 선언("치트를 쓰지 않는다")을 넣지 않았는가

## 6-3. 흔한 함정과 해결책

| 함정 | 증상 | 근본 원인 | 해결책 |
|------|------|----------|--------|
| 파워 우선 러버밴딩 | 근접 경쟁 중 AI가 직선에서 갑자기 앞서 나감 | 스킬 조작 없이 파워부터 건드림 | 스킬 조작을 우선하고 파워는 스킬 포화 후 폴백으로 (3-3-3) |
| 데드존 부재 | 나란히 달릴 때 AI 가속이 눈에 보임 | 거리 0 부근에서도 러버밴딩 작동 | region c 데드존 도입, 폭을 백미러 기준으로 튜닝 (3-3-7) |
| 데드존 과다 | AI가 영영 따라오지 못해 캐치업이 무의미 | 데드존이 너무 넓음 | 폭을 좁히되 근접 시 가시성을 재검증 (3-3-7) |
| 첫 코너 파일업 | 스타트 직후 다중 충돌로 레이스가 파괴됨 | 전방 밴딩이 차량을 뭉치게 함 | 스타트에서 러버밴딩 무효화, 앞선 차량에 역방향 밴딩 적용 (3-3-5) |
| 정지 AI | 충돌 후 AI가 재출발하지 못함 | 파워가 깎인 상태로 정지 상태에서 출발 | 저속 조건에서 러버밴딩 무효화 (3-3-5) |
| 기어다니는 AI | 랩을 앞선 AI가 비정상적으로 느리게 주행 | 트랙 거리 기준 최대 음의 밴딩 적용 | 랩 뒤처짐 시 해당 차량의 러버밴딩 무효화 (3-3-5) |
| AI 스핀 | 캐치업 중인 AI가 코너 탈출에서 스핀 | 추가 파워가 타이어 그립을 초과 | 추가 파워 전제로 차량 셋업, 또는 그립 보강 (3-3-4) |
| 리미터 소음 | AI 차량에서 리미터 효과음이 끊임없이 재생 | 파워 증가로 최고 기어에서 레브 리미터 상시 도달 | 오디오-물리 직결 구조 재검토 (3-3-4) |
| 무리 뭉침 | 한 번에 여러 대를 추월해 버려 재미가 소멸 | 차량마다 다른 배수를 받아 간격이 붕괴 | 임계 거리 밖 차량에 평균 위치 기준 동일 배수 적용 (3-3-6) |
| 선두 도주 | 1위가 혼자 달아나 레이스가 성립하지 않음 | 선두는 완벽한 라인을 탈 수 있어 자연히 격차 발생 | 1위-2위 거리 기준의 별도 러버밴딩 (3-3-9) |
| 분할 화면 기준 혼란 | 2인 플레이에서 러버밴딩이 한쪽만 편애 | 기준 플레이어가 정의되지 않음 | 최후위=역방향 기준, 최선두=전방 기준, 사이 AI는 미적용 (3-3-8) |
| 어시스트 절벽 | 특정 어시스트를 끄면 갑자기 완주 불가 | 어시스트가 이진 스위치로만 존재 | 단계적 강도 조절 도입, 사다리 간격 균일화 (4-2) |
| 보상 강제 | 숙련되지 않은 플레이어가 고통스럽게 어시스트를 끔 | 어시스트 해제 보상 배율이 과함 | 배율 하향, 또는 별도 진행 축으로 분리 (2-2) |
| 연출로 덮기 | 조작이 나쁜데 이펙트로 가림 | 속도감 연출을 조작감 문제의 해결책으로 오용 | 연출 전부 끈 상태로 조작 검증 (4-1) |
| 멀미 유발 | 플레이어가 장시간 플레이를 못함 | 연출 요소가 단일 프리셋으로 묶임 | 모션 블러·흔들림·FOV 개별 분리 (4-5) |
| 트랙-물리 분리 | 어떤 물리 엔진에서도 트랙이 밋밋함 | 트랙이 물리 특성과 무관하게 설계됨 | 물리의 특징적 거동을 조명하는 구간 설계 (5-6) |
| 추월 불가 트랙 | 러버밴딩이 잘 작동해도 순위가 고정됨 | 제동 구간이 짧아 추월 지점이 없음 | 긴 직선 끝의 저속 코너를 의도적으로 배치 (3-4) |
| 마케팅 선언의 부채화 | "러버밴딩 없음" 선언이 출시 후 반증됨 | 검증 불가능한 절대 선언 | 선언 대신 설계 의도를 서술, 예외 규칙을 인정 (5-4) |
| 홍보 수치의 기획서 유입 | 경쟁사 마케팅 수치가 기술 목표가 됨 | 1차 출처를 검증된 사실로 오인 | 개발사 자기 진술은 귀속 서술로 격리 (5-1, 5-4) |

---

# 7. 결론 및 미래 전망

## 7-1. 장르의 핵심 요약

레이싱은 **"먼저 도착한다"는 단순한 승리 조건 위에 두 개의 어려운 문제를 쌓아 올린 장르**입니다.

첫째 문제는 **접지 한계의 표현**입니다. 타이어와 노면의 마찰을 어떤 해상도로 모사할지가 게임의 정체성을 결정하며, 이 결정은 정확도의 문제가 아니라 목표 감각과 튜닝 비용의 교환입니다(3-1). "더 정확한 물리가 더 좋은 게임"이라는 명제는 참이 아니며, 어밸런치의 GDC 2019 세션은 아케이드·심케이드에서 오히려 파라미터가 적은 모델이 나을 수 있다는 **제안**을 내놓았습니다(5-2).

둘째 문제는 **경쟁의 관리**입니다. 실력 차가 시간 차로 누적되면 경쟁자는 시야에서 사라지고 게임은 붕괴합니다. 이 문제에 대한 장르의 표준 대응이 러버밴딩이며, 닉 멜더의 Game AI Pro 42장은 이를 완결된 아키텍처로 문서화합니다 — 데드존을 중심으로 한 구간 구조(3-3-2), 스킬 우선·파워 폴백의 규칙(3-3-3), 물리가 거짓말을 폭로하는 실패 모드들(3-3-4), 거리 기준이 오도하는 세 가지 무효화 상황(3-3-5), 뭉침과 선두 도주에 대한 대응(3-3-6, 3-3-9), 그리고 레이스 페이스 시스템으로의 확장(3-3-10).

**이 두 문제는 독립적이지 않습니다.** 물리가 정교할수록 파워 조작이 스핀과 소리로 드러나므로, 경쟁을 인위적으로 관리할 여지가 줄어듭니다. 이것이 2-0의 스펙트럼에서 물리 충실도와 경쟁 관리가 역상관을 이루는 이유입니다. **레이싱 설계의 핵심 결정은 이 두 축 위에서 자기 좌표를 고르는 것**이며, 나머지는 대체로 그 결정의 귀결입니다.

## 7-2. 공정성이라는 미해결 문제

러버밴딩의 윤리는 이 장르가 완전히 해결하지 못한 문제입니다.

멜더의 입장은 명확합니다 — 문제는 조작의 존재가 아니라 **노출**이며, 스킬 조작이 파워 조작보다 나은 이유는 그것이 물리 법칙 안에 머물러 "부정이 일어나지 않기" 때문입니다. 이 관점에서 좋은 러버밴딩은 **들키지 않는 러버밴딩**입니다.

그러나 이 입장에는 구조적 취약성이 있습니다. **리플레이와 텔레메트리가 있는 게임에서 은폐는 결국 실패합니다.** 마리오 카트형 공개 캐치업이 배신감을 유발하지 않는 이유는 그것이 더 공정해서가 아니라 **약속을 어기지 않았기 때문**입니다. 반대편 극단에서 "러버밴딩을 쓰지 않는다"는 선언은 검증 불가능한 절대 명제이며 **약속이 아니라 부채**가 됩니다(5-4) — 부재는 증명할 수 없지만 존재는 플레이어가 체감으로 고발하기 때문입니다.

**따라서 현재로서 가장 방어 가능한 위치는 중간에 있습니다** — 캐치업의 존재를 부인하지 않되, 그 작동을 물리 법칙 안에 두고, 무효화 규칙을 정직하게 설계하는 것입니다.

## 7-3. 기술 발전의 영향

**학습 기반 AI**: 소니 AI의 연구는 학습 기반 에이전트가 최상위 인간 드라이버를 상대로 승리한 사례를 보고합니다(5-1). 이것이 러버밴딩을 대체할지는 아직 열린 문제입니다. **빠른 AI를 만드는 것과 플레이어에게 맞는 AI를 만드는 것은 다른 문제**이며, 후자는 여전히 난이도 조정을 요구합니다. 학습 기반 AI의 잠재적 기여는 러버밴딩의 제거가 아니라, **스킬 축의 해상도를 높여 파워 조작에 의존할 필요를 줄이는 것**일 가능성이 큽니다 — 이는 멜더의 규칙(3-3-3)을 더 넓은 범위에서 지킬 수 있게 해준다는 뜻입니다.

**해상도의 상승**: 3-1과 3-7에서 봤듯 타이어의 노면 샘플링 해상도는 세대를 거치며 올라가고 있으며, 이는 연석처럼 이산화 오차가 집중되던 지점의 거동을 개선합니다. 다만 **해상도 상승이 곧 충실도 상승은 아니며**(3-7), 해상도가 오를수록 파워 기반 러버밴딩의 노출도 정밀해진다는 역설이 함께 따라옵니다(3-3-4).

**하드웨어와 구조적 실험**: 마리오 카트 월드 사례(5-3)는 하드웨어 진보가 트랙 구조 자체의 재설계를 가능하게 한 경우입니다. 다만 야부키의 발언이 보여주듯 기술은 **실현 조건이지 동기가 아니었습니다.** 단일 월드 구조가 낳은 분산 문제와 그 대응은 **새로운 기술이 새로운 설계 문제를 만든다**는 일반 법칙의 사례입니다.

**절차적 생성과 결정론**: 트랙은 절차적 생성과 궁합이 나쁘며(1-2), **"달릴 수 있는 트랙"과 "재미있는 트랙" 사이의 간극이 이 기술의 실질적 시험대**입니다. 한편 고스트·랭킹의 크로스 플랫폼 공유는 부동소수점 결정론이라는 오래된 문제에 걸려 있으며(3-7), 이는 기술 발전보다 초기 아키텍처 결정에 좌우됩니다.

## 7-4. 기획자를 위한 최종 조언

### 필수 요소

1. **좌표 선언**: 물리 충실도-접근성 스펙트럼 위의 위치를 문서로 명시할 것
2. **타이어 모델의 목적 정합**: 정확도가 아니라 목표 감각 대비 튜닝 비용으로 선택할 것
3. **스킬 우선 러버밴딩**: 파워 조작은 스킬 포화 후의 폴백으로만 쓸 것
4. **데드존**: 플레이어 근처에서 시스템을 완전히 끌 것
5. **맥락 무효화**: 스타트·저속·랩 뒤처짐에서 반드시 끌 것
6. **어시스트 사다리**: 각 단계가 그 자체로 완결된 게임이 되게 할 것
7. **귀속 규율**: 개발사 자기 진술을 검증된 사실로 취급하지 말 것

### 밸런싱 기준

```
러버밴딩 데드존: AI가 가시적 우위 없이 백미러 안까지 따라올 수 있는 폭
조작 우선순위: 드라이버 스킬 → (포화 시) 차량 파워
무효화 조건: 레이스 시작(부호 반전) / 저속 / 랩 뒤처짐
뭉침 대응: 임계 거리 밖 차량은 평균 위치 기준 동일 배수
선두 도주: 1위-2위 거리 기준의 별도 밴딩
장거리 레이스: 거리 외 기준을 추가하여 레이스 페이스 시스템으로 확장
어시스트 사다리: 단계 간 난이도 간격을 균일하게
```

### 차별화 전략

8. **캐치업의 대안 탐색**: 매칭·피트스톱·추월 지점·고스트가 러버밴딩을 대체할 수 있는지 먼저 물을 것
9. **물리 부산물의 승격**: 창발적 거동을 제거 대상이 아니라 콘텐츠 후보로 볼 것
10. **트랙이 물리를 조명하게 할 것**: 물리팀과 레벨팀을 분리하지 말 것
11. **연출과 접근성의 분리**: 속도감 요소를 개별 슬라이더로 나눌 것
12. **공개형 캐치업의 재검토**: 은폐가 유일한 답은 아님

## 7-5. 레이싱 디자인의 핵심 공식

**"보이지 않는 손의 딜레마"**

```
레이싱 = 접지 한계의 표현 × 경쟁의 관리 × 속도의 지각 × 반복의 보상
```

**구체적 설계 공식:**
```
지각된 공정성 = (조작의 물리적 정당성 × 데드존의 적절성) / (파워 조작의 크기 × 노출 채널의 수)

목표: 캐치업이 작동하되 관측되지 않는 지점 — 그리고 그것이 불가능하다면, 캐치업을 공개하는 정직함
```

이 공식이 레이싱 디자인의 중심을 담고 있습니다. 게임은 플레이어에게 접지 한계라는 냉정한 물리적 제약을 부여하지만, 동시에 경쟁자가 항상 곁에 있는 극적인 무대를 약속합니다. 이 두 약속은 서로 모순됩니다 — **물리가 정직할수록 경쟁은 흩어지고, 경쟁을 붙들수록 물리는 거짓말을 해야 합니다.**

멜더의 문장은 이 장르의 설계자가 매일 마주하는 현실을 요약합니다 — 잘 하면 플레이어는 알아채지 못하고, 못 하면 속았다고 느낍니다. 러버밴딩은 기법이 아니라 약속입니다. 플레이어에게 접전을 약속하고, 그 접전이 어떻게 만들어졌는지는 말하지 않겠다는 약속. **이 약속을 지킬 자신이 없다면, 처음부터 공개하는 편이 낫습니다.**

---

## 부록: 출처 목록

이 문서를 작성하며 **직접 확인한 출처만** 아래에 나열합니다. 확인에 실패했거나 접근이 제한된 경우 그 사실을 명시했습니다.

### A. 확인 완료 — 1차 출처

| 출처 | URL | 확인 내용 | 확인 시점 |
|------|-----|----------|----------|
| Nic Melder, "A Rubber-Banding System for Gameplay and Race Management", *Game AI Pro* 42장 (Part VI. Racing, 507-512쪽) | https://www.gameaipro.com/GameAIPro/GameAIPro_Chapter42_A_Rubber-Banding_System_for_Gameplay_and_Race_Management.pdf | **전문 확인.** 3장 3-3의 모든 내용(구간 구조, 스킬/파워 규칙, 실패 모드, 무효화 3종, 뭉침 대응, 파라미터 선택, 분할 화면, 1위 밴딩, 50랩 레이스 페이스)의 근거 | 2026-07-16 |
| Sony AI 연구진, "Outracing champion Gran Turismo drivers with deep reinforcement learning", *Nature* 602, 223-228 (2022-02-09 게재) | https://www.nature.com/articles/s41586-021-04357-7 | 저자 27명 전원 Sony AI(뉴욕/도쿄/취리히) 소속 확인, 폴리포니 디지털은 감사 절에만 등장, 충실도/접지 한계 문장 축자 확인, 타이어·에어로·서스펜션 모델 미문서화 확인, 경합이익 진술이 특허 US 63/267,136만 다룸을 확인, 맞대결 이벤트 2021-07-02 및 2021-10-21 확인 | 2026-07-16 |
| 닌텐도, "Ask the Developer Vol. 18 — Mario Kart World Part 1" (2025-05-21 게재) | https://www.nintendo.com/us/whatsnew/ask-the-developer-vol-18-mario-kart-world-part-1/ | 참여자 5인 확인, 12→24인 결정 및 조기 결정 진술 축자 확인, 분산 사유 축자 확인, 이시카와의 시각적 공허함 발언 확인, 단일 월드 전환 및 "modern technology" 발언 확인, "Mario Kart 9" 명명 발언 확인, **러버밴딩·캐치업 언급 0회 확인** | 2026-07-16 |
| Turn 10 Studios (Chris Esaki), "Forza Motorsport's New AI and Physics Make Every Race Competitive" (2023-07-17 게재, 게임 출시 2023-10-10) | https://forza.net/news/forza-motorsport-drivatars-tire-physics | 1차 개발사 홍보 자료임을 확인. **타이어 물리 축자 확인** — 신규: "8 points of contact ... running at 360 cycles per second", 이전 세대: "a single point of tire contact ... that moved at 60 cycles per second", 단일 접지점의 문제(연석·럼블스트립에서 "inconsistencies and unrealistic changes in suspension forces", 요철 통과 시 "unrealistic loss of traction") 및 "48x improvement in tire fidelity" 문장 확인. **48의 도출 근거는 원문에 설명되어 있지 않음을 확인**(직전 문장은 "bigger than Forza Motorsport 5, 6 and 7 combined"). Drivatar 훈련 진술("26,000 times") 확인. **"치트·러버밴딩 미사용" 주장은 아래 유의사항 5의 근거로 배제** | 2026-07-16 |
| Hamish Young, "Vehicle Physics and Tire Dynamics in 'Just Cause 4'", GDC 2019 (Avalanche Studios) — **강연 자막(자동 생성)으로 내용 확인** | https://gdcvault.com/play/1026468/Vehicle-Physics-and-Tire-Dynamics | 세션 제목·발표자·소속·연도 및 공식 세션 설명 축자 확인. **강연 본문 전체를 자막으로 확인** — 핸들링 스펙트럼(카트 레이싱 ↔ 제약 기반 SDK ↔ 드라이빙 심)과 "스위트 스팟" 프레이밍, 준경험적 모델의 수정 저항성·낮은 시뮬레이션 주파수에서의 하중 이동 피드백 문제·언더스티어의 수용 불가능성(T자 교차로)·오버스티어 보정 비용, 설계 목표로서의 "believable"과 제한된 인지 부하, 입력 파라미터(슬립 레이시오·슬립 앵글·캠버·휠 로드) 유지와 힘 함수 교체, 정지/동적 서스펜션 힘 블렌딩, **마찰 클램프**(발산 원인, 종·각·횡 3방향, 반복 솔버 대비 차이, 유효 질량 휴리스틱), 클램프 확보 후 그립 상향 자유 진술, 스플라인 툴 3단계 곡선 저작과 종·횡 곡선 상호작용, 드리프트의 창발, **기울기=그립감** 통찰, 곡선 정규화 → 소수 타이어 세트 → 앞·뒤 그립 값 저작, 롤·피치 성분 축소, 비자동차 탈것의 선형 램프, 마무리 권고(클램프 우선), Q&A(라인 테스트 배치, Havok API로 힘 적용) 확인. **수치·수식·튜닝 값은 슬라이드에 있어 자막에 미포함 — 유의사항 7 참조** | 2026-07-16 |
| GDC 공식, "Get under the hood of Just Cause 4's vehicle physics at GDC 2019!" | https://gdconf.com/article/get-under-the-hood-of-just-cause-4-s-vehicle-physics-at-gdc-2019/ | 공식 세션 설명 축자 재확인(Vault와 동일 문안). 발표자 직함을 "lead mechanics designer"로 기재 | 2026-07-16 |
| GDC 공식 유튜브, "Vehicle Physics and Tire Dynamics in Just Cause 4" (채널: GDC Festival of Gaming, 약 28분) | https://www.youtube.com/watch?v=0jsENVOmkxc | **영상의 존재와 무료 공개 상태를 확인**(제목·채널·재생 시간 확인). 위 항목의 자막은 이 강연의 것이며, **발표자 자기소개("Hamish Young", Just Cause 4 리드 메카닉 디자이너)가 자막 서두에서 육성으로 확인되어 세션 메타데이터와 일치** | 2026-07-16 |

### B. 확인 완료 — 2차 출처 (기고문)

| 출처 | URL | 확인 내용 | 확인 시점 |
|------|-----|----------|----------|
| Dave Mark, "Rubber-banding as a Design Requirement", Game Developer (2010-06-02) | https://www.gamedeveloper.com/design/rubber-banding-as-a-design-requirement | 러버밴딩의 양방향 정의 축자 확인, Split/Second 파워 플레이가 후위에서만 발동 가능하여 선두 유지가 재미를 훼손한다는 논지 확인, Thunderbolt 리뷰 인용문 확인, 슈터 AI로의 일반화 문장 확인 | 2026-07-16 |
| Livio De La Cruz, "Implementing Racing Games: An intro to different approaches and their game design trade-offs", Game Developer (2016-08-12) | https://www.gamedeveloper.com/design/implementing-racing-games-an-intro-to-different-approaches-and-their-game-design-trade-offs | 아케이드의 가산적 축조 문장 축자 확인, 속도 의존 조향각 제한 서술 및 예시 값 확인, 모터스톰/에볼루션 스튜디오 사례 확인, 필수 메커닉 부재 열거 확인 | 2026-07-16 |
| Game Developer, "Video: The physics powering Just Cause 4's many vehicles" | https://www.gamedeveloper.com/programming/video-the-physics-powering-i-just-cause-4-s-i-many-vehicles | **확인했으나 요약 문단만 게재된 랜딩 페이지로, 타이어 모델에 대한 추가 정보 없음.** 이 문서에 인용된 내용 없음 | 2026-07-16 |

### C. 출처 이용 시 유의사항

이 절은 위 출처들을 재사용할 때 반드시 함께 읽어야 할 한계 사항입니다.

**1. 이해충돌 — Nature/그란 투리스모 논문**

저자 27명 전원이 소니 AI 소속이며, 그란 투리스모 개발사 폴리포니 디지털 역시 소니의 자회사입니다. 즉 **소니가 소니 제품의 충실도를 평가하는 구조**입니다. 논문의 경합이익 진술은 특허 US 63/267,136만 다루며 이 관계를 공개하지 않습니다. 또한 충실도 문장은 논문 도입부의 **무인용 프레이밍**이며, 논문의 검증 대상(강화학습 방법론)이 아닙니다. **"Nature가 GT의 물리 충실도를 입증했다"는 서술은 금지되며, "소니 AI의 논문이 …라고 규정한다"로 귀속해야 합니다.**

**2. 범위 표류 — Nature 논문**

논문이 주장하는 것은 **"비선형 제어 과제"의 충실도**이지 열·공기압·에어로 서브시스템을 포함한 물리 충실도 일반이 아닙니다. 논문은 GT 물리 엔진을 블랙박스로 다루며 타이어·에어로·서스펜션 모델을 일절 문서화하지 않습니다. 이 논문에서 타이어 모델의 정확성에 관한 결론을 끌어낼 수 없습니다.

**3. 시의성 — Nature 논문**

논문 본문은 제품을 "Gran Turismo"로만 지칭하며 특정 버전을 명시하지 않습니다(참고문헌에 GT Sport 대상 선행 연구가 인용됨). 확인 가능한 사실은 **맞대결 이벤트가 2021년, 게재가 2022-02-09**이라는 점입니다. GT7은 2022년 3월 발매되었고 이후 물리 개정을 포함한 업데이트를 출하했으므로, **이 논문은 현행 GT7 물리에 대한 진술이 아닙니다.**

**4. 프로모션 성격 — 닌텐도 Ask the Developer**

이 시리즈는 게임 출시(2025년 6월) **약 2주 전에 게재된 닌텐도 자체 프로모션**입니다. **"개발진이 표명한 설계 의도"의 최적 출처이지 감사된 의사결정 기록이 아닙니다.** 모든 진술은 "…라고 밝혔습니다"로 귀속해야 합니다. 또한 이 인터뷰에는 **러버밴딩·캐치업·AI 난이도에 대한 언급이 전혀 없으므로**, 24인 결정을 캐치업의 대체재로 서술하는 것은 출처에 근거가 없습니다.

**5. 홍보 자료 — Forza Motorsport 관련 글**

출시 **약 3개월 전에 게재된 1차 개발사 홍보 자료**입니다. **같은 글 안에서도 주장에 따라 취급을 달리해야 합니다**(5-4의 대비표 참조).

- **채택 — 타이어 물리의 해상도 수치**(접지점 1→8개, 초당 60→360사이클): 원문에 **비교 대상과 해결하려는 문제까지 축자로 명시**되어 있어 검증 가능한 기술 사양으로 취급합니다. 3-1(이산화 문제)과 3-7(수치 독법)에서 귀속 서술과 시점을 병기하여 인용했습니다.
- **조건부 — "48x improvement in tire fidelity"**: 원문은 **48의 도출 근거를 설명하지 않습니다.** 발표문이 밝힌 두 배수(접지점 8배 × 연산 주기 6배)의 곱이 정확히 48이라는 산술적 대응은 확인되나, 이것이 도출 근거인지 우연인지는 **출처가 확정해 주지 않습니다.** 더 중요하게, **연산 해상도의 곱은 주행 리얼리즘의 독립 측정치가 아닙니다** — 해상도 증가는 충실도의 필요조건일 뿐 충분조건이 아니며(잘못된 모델을 더 자주 계산해도 여전히 잘못된 모델입니다), "충실도"는 실측 대비 오차로 정의되는 양이지 연산량으로 정의되는 양이 아닙니다. 이 수치를 인용한다면 반드시 **"연산 해상도 기준"임을 병기**하고 마케팅 프레이밍으로 취급하십시오.
- **배제 — "치트·핵·러버밴딩을 일절 쓰지 않는다"는 주장**: 출시 전 **설계 목표 선언**이며 출시 빌드의 실제 동작에 대한 검증된 진술이 아닙니다. 이후 공식 포럼에 AI 러버밴딩 관련 이슈 스레드가 등재된 정황이 보고된 바 있어, 출시 빌드의 동작 근거로 인용하기에 부적격합니다.

이 글에 남은 Drivatar 훈련 관련 진술("26,000회") 역시 **개발사의 자기 진술**이며, 이 문서는 귀속 서술로만 인용합니다.

**6. 헷지의 보존과 그 해소 — Just Cause 4 GDC 세션**

공식 세션 설명은 **"may not be optimal"**이라는 헷지를 사용하며, 파라미터 감소·비준경험적 구성·더 나은 결과를 **병렬 나열할 뿐 인과를 진술하지 않습니다.** 따라서 **초록만 근거로** "매직 포뮬러를 비판했다"거나 "파라미터를 줄여서 더 나은 결과를 얻었다"로 강화하는 것은 원문을 넘어선 서술입니다.

**강연 본문(유의사항 7의 자막)은 이 공백을 상당 부분 메웁니다.** 본문에는 초록에 없는 논증이 있습니다 — 실측 모델 배제의 근거는 설계 목표와의 충돌이고, 파라미터 감소는 목표가 아니라 **마찰 클램프가 안정성을 떠맡은 결과**입니다. 다만 이는 **한 팀의 하나의 출시작에 대한 실무자 회고이지 통제된 비교가 아니며**, 발표자 자신이 유효 질량을 "휴리스틱"이자 "완벽하지 않다"고 명시합니다. 따라서 **"영은 …라고 설명한다"는 귀속 서술을 유지하고, "파라미터가 적은 모델이 아케이드에 더 낫다"는 일반 명제로 승격시키지 마십시오.**

**7. 자막 기반 확인의 한계 — Just Cause 4 GDC 세션**

**GDC Vault 페이지는 구독을 요구하지만 동일한 강연이 GDC 공식 유튜브 채널에 무료로 공개되어 있으며**, 이 문서는 그 강연의 **자동 생성 자막**으로 본문 내용을 확인했습니다. 따라서 "이 강연은 유료 장벽 뒤에 있다"는 서술은 사실이 아닙니다.

**이 강연 근거는 자동 생성 자막으로 확인했으므로 개념과 논증은 신뢰할 수 있으나 슬라이드의 수치·수식은 자막에 포함되지 않습니다. 정확한 모델 파라미터가 필요하면 강연 영상의 슬라이드를 직접 확인하십시오.** 발표자는 강연 중 수식과 코드를 슬라이드로 띄우고 "읽기 어렵게 만들어 두었으니 나중에 노트를 받으라"고 안내하므로, **정식화의 세부는 음성에 존재하지 않습니다.** 또한 자동 자막은 고유명사·수치·수식에 구조적으로 취약합니다(같은 코퍼스의 다른 강연에서 단어 및 발표자명 왜곡 사례를 확인했습니다). 이 때문에 **3-1과 5-2는 자막에서 읽힌 구체 수치를 일절 인용하지 않고 정성 서술로 대체했으며**(시뮬레이션 주파수, 탈것 종수, 타이어 세트 수, 곡선 개수, 슬립 레이시오의 수치 범위 등), 개념·설계 논리·인과 관계만을 근거로 삼았습니다. 인용은 짧은 의역으로 처리했습니다.

**8. 출처 간 진술 충돌 — GDC 세션의 트랙 분류**

두 공식 GDC 페이지가 이 세션의 분류에 대해 **서로 다르게 기재**하고 있습니다 — GDC Vault는 Track/Format을 **Design**으로, gdconf.com 기사는 **Programming**으로 표기합니다. 따라서 **"이 세션이 Design 트랙으로 분류되었으므로 기술 강연이 아니라 설계 방법론으로 제시되었다"는 식의 추론은 근거가 불충분합니다.** 이 문서는 트랙 분류에 기반한 주장을 사용하지 않습니다.

**9. 예시 값과 계측 값의 구분 — 조향각 제한**

3-2에 언급된 조향각 예시(저속 최대 약 10도 → 고속 최대 약 2도)는 **기고자가 개념 설명을 위해 든 예시 값이며 출시된 특정 타이틀에서 계측된 수치가 아닙니다.** 실제 게임의 튜닝 값으로 인용해서는 안 됩니다.

**10. 주관적 인상의 격리 — Split/Second 캐치업 강도**

4-3과 5-5에 언급된 "6초 격차를 2초 만에 좁힌다"는 표현은 **당대 리뷰어의 주관적 인상을 블로그 기고문이 간접 인용한 것**이며, 개발사의 텔레메트리나 공개된 튜닝 값이 아닙니다. **"캐치업 강도가 과하다고 지각된 사례"로만 인용해야 하며, 검증된 수치로 인용해서는 안 됩니다.**

**11. 외부 추론의 격리 — MotorStorm 사례**

5-6의 에볼루션 스튜디오 관련 서술은 **기고자의 외부 추론**이며 스튜디오의 공식 진술이 아닙니다(원문이 "seemed to"로 추정임을 명시). **"트랙 설계가 물리를 뒤따를 수 있다"는 설계 원리의 예시로만 사용하고, 에볼루션의 의사결정 기록으로 인용해서는 안 됩니다.**

**12. 도표 축 값의 비인용 — Game AI Pro 42장**

멜더의 챕터에는 러버밴딩 배수 곡선을 보여주는 도표(그림 42.1, 42.2)가 있으나, **이 문서는 도표의 축 눈금 값을 튜닝 권장치로 인용하지 않았습니다.** 해당 값들은 개념 설명용 도해이며 본문이 권장 파라미터로 제시한 값이 아니기 때문입니다. 이 문서가 인용한 유일한 구체 수치는 **본문 산문에 명시된 50랩·10/30/10 레이스 페이스 프로파일**이며, 이 역시 멜더가 "typically"로 일반적 경향을 서술한 것입니다.

**13. 커버리지의 공백**

이 문서 작성의 근거가 된 리서치는 **하위 장르 분류 체계와 대표작 계보, 밸런싱·튜닝 방법론, 흔한 설계 실패 사례**의 세 축에서 1차 출처를 확보하지 못했습니다. 따라서 2장의 분류 체계와 6장의 체크리스트·함정 표는 **확인된 출처(주로 Game AI Pro 42장)에서 도출 가능한 원리와 일반적 설계 지식을 결합해 구성한 것**이며, 개별 항목마다 1차 출처가 뒷받침되지는 않습니다. 특정 타이틀의 밸런싱 수치가 필요한 경우 별도 조사가 필수입니다.

---

# 관련 문서 (See Also)

- [컨트롤·입력 & 카메라(3C)](115-controls_camera_3c.md)
- [실시간 액션 모듈 (Real-time Action)](302_Real-time_Action_Module_Guideline.md)
- [AI / NPC 행동 & 적/몬스터 난이도 설계](121-ai_npc_behavior_difficulty.md)
- [장르 분류 가이드라인](032-Genre-Guide.md)
- [기획 요소 리뷰 목록](033-Review.md)
