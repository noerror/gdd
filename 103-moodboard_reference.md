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

### 4.4 선택 및 심화 (Optional & Advanced)
- **레퍼런스 저작권 관리**: 레퍼런스 출처 명시 의무화, 상업 프로젝트 시 저작권 확인, 트레이싱 금지 정책, 원리 분석 중심 활용
- **무드보드 버전 관리**: 프리프로덕션/프로덕션/폴리싱 단계별 무드보드 진화, 주요 변경사항 이력 관리, 팀 동의 절차
- **팀 간 공유 플랫폼**: Miro/Milanote(협업 무드보드), Notion(설명 + 이미지), PureRef(로컬 레퍼런스), Figma(인터랙티브 프로토타입)
- **AI 도구 활용**: MidJourney/DALL-E로 컨셉 이미지 생성, 프롬프트 엔지니어링 기법, AI 생성물 저작권 고려사항
- **무드보드 업데이트 주기**: 마일스톤마다 검토, 프로토타입 결과 반영, 팀 피드백 반영 워크플로우

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

### 6.1 무드보드 구성
- **Moodboard**: 시각적 방향성을 전달하는 이미지 모음
- **Reference**: 영감을 주는 참고 자료
- **Visual Identity**: 게임의 독특한 시각적 특징과 스타일

### 6.2 색상 & 조명
- **Color Palette**: 게임에서 사용하는 색상 조합
- **Lighting Mood**: 조명으로 표현하는 분위기(Low Key, High Key, Rim Light 등)
- **Tonal Range**: 색상의 명도 범위 및 대비 설정

### 6.3 실무 도구
- **Image Search**: Pinterest, ArtStation, ShotDeck 등 레퍼런스 검색 플랫폼
- **Mood Board Tools**: Milanote, Genery, PureRef, Figma 등 무드보드 작성 도구
- **AI Image Generation**: MidJourney, DALL-E, Stable Diffusion 등 AI 이미지 생성 도구

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

**Case 4: Journey의 미니멀 무드보드**
- **좋은 예 (Good)**: "콘셉트: 광활한 사막, 고독한 순례자의 정서적 여정. 색상 팔레트: 금색(모래), 파란색(하늘), 주황색(석양) 3가지로 제한. 레퍼런스: 영화 '라스트 템플라'의 모래 광경, 애니메이션 '스피릿의 말' 풍경, 사막 일몰 사진. 선택 이유: 최소한의 색상으로 '적막함'과 '경외감' 동시 표현, 초보 개발자도 구현 가능한 효율성"
- **안티패턴 (Bad)**: "여러 장르의 무작위 이미지를 모아 통일감 없고, 선택 이유가 '예쁘다'는 주관적 평가만 있음"

**Case 5: Inside의 공포 무드보드**
- **좋은 예 (Good)**: "콘셉트: 어두운 지하 시설, 생존의 공포, 정체불명의 위협. 색상 팔레트: 검정(80%), 진회색(15%), 빨강(5%, 위험 신호). 조명: Low-key 조명, 림 라이트로 실루엣 표현, 고명암비. 레퍼런스: 영화 '블레이드 러너' 어두운 장면, 촬영장 '더 코어' 지하 설정, 산업 폐허 사진. 선택 이유: 색상 제한으로 긴장감 극대화, 플레이어가 항상 '쫓기는 느낌' 유지, 스타일라이즈드 2D로 리소스 절약"
- **안티패턴 (Bad)**: "무서운 이미지만 수집하되 시각적 일관성 고려 없음, 팀이 무드보드를 만들었으나 실제 게임은 밝은 색감으로 완성"

**Case 6: Hades의 다채로운 무드보드**
- **좋은 예 (Good)**: "콘셉트: 그리스 신화 × 현대 핫라인 마이애미, 황금 조각상의 웅장함 + 네온 색감의 반항. 색상 팔레트: 금색(주요), 진보라(경기장), 선홍색(혼돈), 검정(배경). 레퍼런스: 고대 그리스 미술, 1980년대 보드게임 미학, 바로크 미술의 드라마틱 조명, 사이버펑크 인터페이스. 지역별 무드보드 분리(타르타로스/아스포델/엘리시움 등 각 5-7개 레퍼런스). 선택 이유: 각 지역이 시각적으로 구분되면서도 일관된 스타일 유지, 플레이어 감정의 점진적 고조 표현, 레트로-파이 스타일로 개발 난이도 관리"
- **안티패턴 (Bad)**: "좋은 무드보드를 만들었지만 지역별로 통일되지 않거나, 팀 일부만 무드보드를 참고하여 UI/아트 간 스타일 불일치"

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
- [Prompt Base](https://promptbase.com/): 다양한 스타일의 이미지 생성 프롬프트 판매
- [MidJourney](https://midjourney.com/): AI 이미지 생성

