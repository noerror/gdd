---
title: 무드보드 & 레퍼런스 작성 가이드
description: 게임의 시각적·컨셉적 방향성을 무드보드와 레퍼런스를 통해 명확히 전달하고, 팀 전체가 동일한 비전을 공유할 수 있도록 하는 창의적 워크플로우 가이드
---

# 무드보드 & 레퍼런스 작성 가이드

# 1. 목적 (Purpose)
게임의 시각적·컨셉적 방향성을 무드보드와 레퍼런스를 통해 명확히 전달하여 팀 전체가 동일한 비전을 공유하기 위한 가이드입니다.

# 2. 범위 (Scope)
- **포함**: 무드보드 구성, 레퍼런스 이미지 수집, 비주얼 스타일 정의, 컨셉 아트 방향성
- **제외**: 구체적인 3D 모델링, 텍스처링 기법, 기술적 구현

# 3. 설계 목표 (Design Goals)

1. **명확한 비전 전달 (Clear Vision)**: 시각적 요소로 게임의 방향성을 명확히 전달합니다.
2. **팀 정렬 (Team Alignment)**: 모든 팀원이 동일한 시각적 비전을 공유합니다.
3. **영감 제공 (Inspiration)**: 레퍼런스를 통해 창의적 아이디어를 발견합니다.
4. **일관성 유지 (Consistency)**: 모든 비주얼 요소가 무드보드와 일치하도록 합니다.

# 4. GDD 문서 구조 (Structure)

## 4.1 무드보드 (Moodboard)
- **비주얼 스타일**: 색상 팔레트, 조명, 분위기
- **참고 이미지**: 영감을 주는 이미지 모음

### 4.2 레퍼런스 (References)
- **게임 레퍼런스**: 유사한 스타일의 게임
- **비게임 레퍼런스**: 영화, 아트, 사진 등

### 4.3 설명 & 의도 (Description & Intent)
- **선택 이유**: 각 레퍼런스를 선택한 이유
- **적용 방법**: 레퍼런스를 게임에 어떻게 적용할 것인가

# 5. 무드보드 검색 팁

## 5.1 장면별 요소 분리

- **월드 전체 무드**: 마스터 무드보드
- **지역/맵별 무드**: 숲/도시/던전 등
- **씬/컷별 무드**: 오프닝, 보스전, 감정 폭발 씬 등
- **캐릭터별 무드**: 프로필, 전투, 감정씬 등

→ 각 보드마다 키워드를 따로 세팅하면 중복 감소 & 팀 커뮤니케이션 향상

## 5.2 키워드 쪼개기

### 컨셉 문장 → 검색 키워드로 쪼개기

**예시**: "폐허가 된 근미래 서울, 네온 사이버펑크, 우울하고 고독한 여주인공"

- **장르/분위기**: cyberpunk, dystopian, post-apocalyptic, melancholic, lonely, neo-noir
- **공간/배경**: rainy street, back alley, neon lights, high-rise buildings, rooftop, abandoned subway
- **컬러/조명**: teal and orange, magenta highlights, neon pink sign, low key lighting, high contrast lighting, fog, haze
- **카메라/구도**: wide shot, low angle, dutch angle, over the shoulder, silhouette, shallow depth of field, long lens
- **시대/문화**: Seoul, Korean signs, hangul neon, Asian megacity

**조합 검색 예시**
- Pinterest/ArtStation: `cyberpunk rainy street`, `asian cyberpunk city neon`, `melancholic cyberpunk girl`
- ShotDeck/Frame Set: `rainy night neon street low key lighting`, `lonely woman wide shot back alley`, `teal orange city night`
- **팁**: 한국어 + 영어 혼용 시 결과 폭 확대 (예: `사이버펑크 서울 neon alley`, `어두운 판타지 숲 dark fantasy forest fog`)

### 조명·색감 키워드 활용

- **조명**: backlight, rim light, silhouette, high/low key, hard/soft light
- **색감**: monochrome, muted/saturated colors, teal & orange, warm light cool shadows
- **날씨/공기**: fog, mist, haze, rain, overcast, snow, golden/blue hour
- **예시**: `low key lighting silhouette corridor`, `golden hour fantasy field mist`, `muted colors war film trench`

### 실전 검색 테크닉

**기본 조합 공식** : `[장르/분위기] + [공간/배경] + [조명/색] + [감정]` (3~4개 요소)
- 예: `dark fantasy forest fog blue light fear`, `sci-fi corridor high contrast lighting tension`

**마이너스 검색**
- SF 우주 (코미디 제외): `sci-fi spaceship interior -comedy -parody -cartoon`
- 실사 무드만: `dark fantasy castle -anime -illustration`

**유사 이미지 검색**
- 구글/Bing/ShotDeck/Frame Set의 "similar images" 기능으로 마음에 드는 이미지 기준 확장

## 5.3 검색 워크플로우

1. **컨셉 정리**: 배경, 감정, 색감, 시대/장르를 1~2문단으로 작성
2. **키워드 추출**: 명사·형용사·동사를 영어/한국어 키워드로 변환
3. **플랫폼 분배**: Pinterest/Unsplash (무드/색감), ShotDeck/StillsLab (시네마틱), ArtStation (캐릭터/디자인)
4. **무드보드 작성**: Milanote/Genery/Figma/PureRef로 배열, 중요 포인트 텍스트 추가
5. **정리**: 톤 불일치 이미지 제거 → "한 영화/한 게임"처럼 통일감 확보

## 5.4 저작권 주의사항

- **트레이싱 금지**: 구조·조명·색감·분위기의 "원리"만 분석하여 활용
- **상업 프로젝트**: 스톡 이미지 라이선스 확인, 영화 스틸은 "참고용" 명시, 여러 레퍼런스 조합으로 새로운 결과물 창출

## 5.5 검색 키워드 예시

**장르별 키워드**
- **어두운 판타지 던전**: `dark fantasy dungeon torch light fog`, `medieval dungeon low key lighting horror`
- **밝은 카툰 판타지**: `colorful fantasy village daylight warm light`, `storybook town cobblestone street`
- **네온 사이버펑크**: `cyberpunk city street rain neon`, `asian cyberpunk alley magenta teal`
- **로맨스 노을**: `golden hour couple silhouette city rooftop`, `romantic sunset backlight wide shot`
- **스릴러 복도**: `thriller corridor low key lighting tension`, `dark hallway suspense high contrast`

# 6. 관련 개념 (Key Concepts)

- **Moodboard**: 시각적 방향성을 전달하는 이미지 모음
- **Reference**: 영감을 주는 참고 자료
- **Visual Identity**: 게임의 독특한 시각적 특징과 스타일
- **Color Palette**: 게임에서 사용하는 색상 조합

# 7. 품질 체크리스트 (Quality Checklist)

## 7.1 무드보드
- [ ] 무드보드가 게임 비전을 명확히 전달하는가?
- [ ] 색상 팔레트가 일관되는가?
- [ ] 분위기와 테마가 명확한가?

## 7.2 레퍼런스
- [ ] 레퍼런스가 다양하고 풍부한가?
- [ ] 각 레퍼런스의 선택 이유가 명확한가?
- [ ] 레퍼런스가 실현 가능한가?

## 7.3 팀 공유
- [ ] 팀 전체가 무드보드를 이해하고 동의하는가?
- [ ] 무드보드가 디자인 결정에 활용되는가?
- [ ] 무드보드가 정기적으로 업데이트되는가?

# 8. 예시 및 안티패턴 (Examples & Anti-Patterns)

**Case 1: 명확한 무드보드**
- **좋은 예 (Good)**: "다크 판타지 테마 - 어두운 색상(검정, 진회색, 진한 보라), 낮은 조명, 고딕 건축 스타일, Journey·Limbo 레퍼런스"
- **안티패턴 (Bad)**: "다양한 스타일의 이미지를 무작위로 모아 방향성 불명확"

**Case 2: 선택 이유 명시**
- **좋은 예 (Good)**: "Mirror's Edge 레퍼런스 - 빨간색으로 경로 안내하는 시각 언어를 우리 게임의 탐험 시스템에 적용"
- **안티패턴 (Bad)**: "예쁜 이미지를 모았지만 왜 선택했는지 설명 없음"

**Case 3: 실현 가능성**
- **좋은 예 (Good)**: "스타일라이즈드 아트 스타일 - 목표 플랫폼(모바일)에서 구현 가능, 팀의 기술 수준에 적합"
- **안티패턴 (Bad)**: "AAA급 리얼리즘을 목표로 하지만 인디 팀 규모로 실현 불가"

# 9. 핵심 인사이트 (Key Insights)

- **시각적 소통**: 말보다 이미지가 더 명확하게 비전을 전달하므로, 풍부한 무드보드를 구성하세요.
- **선택 이유 명시**: 각 레퍼런스를 왜 선택했는지, 어떻게 적용할 것인지 명확히 설명하세요.
- **일관성 유지**: 모든 레퍼런스가 동일한 방향성을 가리키도록 하여 일관된 비주얼을 구축하세요.
- **정기 업데이트**: 개발 진행에 따라 무드보드를 업데이트하여 최신 비전을 반영하세요.

# 10. 참고 자료

## 게임 컨셉 아트
- [Pinterest](https://www.pinterest.com/)
- [ArtStation](https://www.artstation.com/)
- [Behance](https://www.behance.net/)
- [Dribbble](https://dribbble.com/)
- [Game UI Database](https://www.gameuidatabase.com/)

## 영화 무드
- [ShotDeck](https://shotdeck.com/): 영화 스틸 최대급 라이브러리
- [Frame Set](https://frameset.app/): 필름 레퍼런스 검색
- [Shot.cafe](https://shot.cafe/): 영화 스틸 전용 DB들
- [StillsLab](https://stillslab.com/): 영화 스틸 전용 DB들

## 무드 도구
- [Milanote](https://milanote.com/): 게임/영화용 무드보드 템플릿 제공
- [Genery.io](https://genery.io/): 영화·영상 무드보드 특화
- [StudioBinder](https://studiosbinder.com/): 게임·영화 무드보드 튜토리얼

## 이미지 생성 활용
- [Prompt Base](https://promptbase.com/)
- [MidJourney](https://midjourney.com/)

