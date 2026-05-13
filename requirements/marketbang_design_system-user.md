# 마켓뱅(marketbang.kr) 디자인 시스템 분석

> 분석 일자: 2026-05-13
> 분석 대상: https://marketbang.kr/ (CSS 번들 `index-CNmuu2DK.css` 직접 파싱)
> 사이트 성격: 와인 발주 B2B 플랫폼 (수입사 → 와인바/바틀샵)
> 빌드: Vite SPA, 모바일 우선(`viewport-fit=cover`, `user-scalable=no`)

---

## 1. 컬러 팔레트

CSS 변수로 3계열 스케일을 사용한다. `:root`에 토큰 정의 + `[data-primary="500"]` 같은 **데이터 속성 기반 유틸리티**로 적용한다.

### 1.1 Primary (와인 버건디 계열) — 브랜드 메인
| 토큰 | HEX | 용도 |
|---|---|---|
| `--primary_500` | `#9b0d62` | 브랜드 코어(딥 버건디) |
| `--primary_400` | `#b11472` | 강조 텍스트/링크 |
| `--primary_300` | `#eaa4a4` | 보조 라이트 |
| `--primary_200` | `#f2e1e9` | 배경 강조 라이트 |
| `--primary_150` | `#ffeff9` | 배경 옅은 핑크 |
| `--primary_100` | `#fff5f9` | 배경 최옅음 |
| `--primary_allert` | `#ec47aa` | 알림 핑크 (오타 그대로 `allert`) |
| **실사용 메인 버튼** | `#911054` | CTA 버튼 배경 (변수보다 0.5° 더 핑크) |

> ⚠️ 토큰 `#9b0d62`와 실사용 버튼색 `#911054`가 미세하게 다름. 토큰이 SSoT지만 실제 버튼은 별도 하드코딩.

### 1.2 Secondary (그린 액센트) — 활성/긍정 상태
| 토큰 | HEX | 용도 |
|---|---|---|
| `--secondary_500` | `#60bf00` | 짙은 라임 |
| `--secondary_400` | `#7fe816` | 메인 라임 (선택/활성 보더) |
| `--secondary_300` | `#b0bf74` | 차분한 올리브 |
| `--secondary_150` | `#f0f8d0` | 배경 라이트 라임 |
| `--secondary_100` | `#fbffed` | 배경 최옅음 |

→ 활성 칩(`background:#fbffed; border:#7fe816; color:#7fe816`) 패턴이 반복됨.

### 1.3 Black (그레이스케일)
| 토큰 | HEX | 체감 톤 |
|---|---|---|
| `--black_700` | `#222` | 본문 텍스트 기본 (root 컬러) |
| `--black_500` | `#404040` | 진한 보조 텍스트 |
| `--black_400` | `#838383` | 회색 보조 텍스트 |
| `--black_300` | `#aaa` | 비활성 텍스트/플레이스홀더 |
| `--black_250` | `#e5e5e5` | 보더 |
| `--black_200` | `#f7f7f7` | 배경 회색 |
| `--black_150` | `#fdfdfd` | 거의 흰색 배경 |
| `--black_100` | `#fafafa` | 카드 배경 |

추가로 자주 등장하는 비변수 회색: `#e4e4e4`, `#d2d2d2`(체크박스 보더), `#f0f0f0`(섹션 디바이더).

### 1.4 시스템 컬러
| 색 | HEX | 용도 |
|---|---|---|
| 에러 / 필수입력 표시 `*` | `#fe1400` / `#ff1400` | 폼 에러, 필수 마크 |
| 비활성 버튼 | `#d2d2d2` | `bg:#d2d2d2; color:#fff` 패턴 |
| 흰색 | `#fff` | 카드/시트 배경 |
| 다크 본문 | `#000` / `#1e1e1e` | 헤더 텍스트 일부 |

### 1.5 적용 규칙 (관찰)
- **본문 기본**: `#222` (root에 선언)
- **보조 본문**: `#404040`
- **헬퍼 텍스트**: `#838383` / `#aaa`
- **CTA**: 솔리드 `#911054` + 흰 글씨, 보조 CTA는 동일 색 1px 보더 + 흰 배경
- **활성 칩/태그**: 라임 보더 + 라이트 라임 배경 + 라임 텍스트
- **알림/에러**: `#fe1400` 텍스트 + 동일색 보더

---

## 2. 타이포그래피

### 2.1 폰트 스택
```css
font-family: Pretendard, -apple-system, BlinkMacSystemFont, system-ui,
             Roboto, "Helvetica Neue", "Segoe UI", "Apple SD Gothic Neo",
             "Noto Sans KR", "Malgun Gothic",
             "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol", sans-serif;
```
- **메인**: Pretendard (한국형 산세리프, 본문 전반)
- **장식용**: `Cafe24Ssurround` — woff CDN으로 별도 로드. 라운드한 디자인 폰트로 프로모/배너/로고 보조 용도.
  ```
  https://fastly.jsdelivr.net/gh/projectnoonnu/noonfonts_2105_2@1.0/Cafe24Ssurround.woff
  ```

### 2.2 사이즈 체계 (rem 기준: `:root { font-size: 10px }` → 1rem = 10px)
| 토큰 | 환산 | 사용 빈도 / 용도 |
|---|---|---|
| `1.2rem` (12px) | 캡션 / 보조 라벨 | 최다 |
| `1.4rem` (14px) | 본문 기본 | 최다 |
| `1.6rem` (16px) | 본문 강조 / 카드 타이틀 | 다수 |
| `1.8rem` (18px) | CTA 버튼 / 섹션 타이틀 | |
| `24px` (`!important`) | 큰 헤더 | 강제 적용 |

px 값이 직접 쓰인 곳: `12px`, `14px`, `16px`, `18px` (대체로 외부 라이브러리/임시).

### 2.3 굵기(weight) 패턴
| weight | 빈도 | 용도 |
|---|---|---|
| 500 | 30 | 본문 기본 |
| 700 | 31 | 카드 타이틀, 가격, 강조 |
| 600 | 9 | 부제목 |
| 800 | 4 | CTA 버튼 라벨 |
| 400 | 7 | 약한 보조 텍스트 |

> CTA 버튼은 **800 굵기**까지 끌어올림 — 클릭 액션을 시각적으로 강하게 잡는 시그니처.

---

## 3. 컴포넌트 토큰

### 3.1 버튼 (Primary CTA — 실측)
```css
display: flex;
justify-content: center;
align-items: center;
width: 100%;
height: 4.8rem;            /* 48px — 모바일 터치 타깃 */
box-sizing: border-box;
border-radius: 0.5rem;     /* 5px */
font-size: 1.8rem;         /* 18px */
font-weight: 800;
background-color: #911054;
color: #fff;
```

### 3.2 버튼 (Secondary 아웃라인)
```css
height: 5.2rem;
border: 1px solid #911054;
color: #911054;
background-color: #fff;
```
→ 아웃라인이 솔리드보다 4px 더 큼(시각 보정).

### 3.3 비활성 버튼
```css
color: #fff;
background-color: #d2d2d2;
```

### 3.4 보더 라디우스 토큰
| 값 | 용도 |
|---|---|
| `0` | 입력 필드 / 디바이더 영역 |
| `4px` | 작은 칩, 인풋 일부 |
| `0.5rem` (5px) | 버튼 |
| `8px` / `0.7rem` | 중간 카드 |
| `10px` | **카드 기본** (제품 카드 최다 사용) |
| `12px` | 큰 모달/시트 |
| `50%` | 아바타, 라운드 아이콘 |
| `2.5rem` (25px) | pill 버튼 |

### 3.5 그림자
```css
/* 카드 기본 */
box-shadow: 0 1px 6px 2px #1e1e1e0d;     /* 5% alpha, 매우 옅음 */

/* 큰 영역 */
box-shadow: 0 20px 10rem #0000000d;       /* 페이지 단위 강조 */
box-shadow: 0 0 2rem 2rem #0000000d;      /* 글로우 */

/* 스티키 헤더/푸터 */
box-shadow: 0 5px 5px 3px #fafafa;
box-shadow: 0 -5px 5px 3px #fafafa;
```

### 3.6 체크박스 (검색/필터)
```css
content: "";
display: inline-block;
width: 1.6rem;       /* 16px */
height: 1.6rem;
border: 1px solid #d2d2d2;
border-radius: 0;
background-color: #fff;
```
→ **사각형(라디우스 0)** 체크박스. 라임 보더(`#7fe816`)로 활성 표시.

### 3.7 갭 / 패딩 패턴
- 갭 기본: `4px`, `5px`, `6px`, `8px`, `10px`, `12px`
- 카드 내부 패딩: `13px`, `16px`, `12px 20px`, `17px 18px`
- 섹션 좌우 패딩: `0 20px`, `0 30px`
- 아이콘-텍스트 갭: `0.8rem` (8px) 가 자주 등장

---

## 4. 페이지 구조 (관찰 + 추론)

> CSS만으로 직접 분석 가능한 영역은 토큰 수준. 페이지 레이아웃은 SPA 런타임에서 생성되므로 일부는 일반적 B2B 발주 패턴에 비춰 정리.

### 4.1 모바일 쉘
- 뷰포트: `viewport-fit=cover, width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no`
- PWA 매니페스트(`/manifest.json`) + apple-touch-icon 풀세트 → 홈스크린 설치 대응
- 테마 컬러: `#ffffff` (상단 노치/주소창 흰색)

### 4.2 메인(홈) 섹션 구성 — 와인 발주 사이트 표준
1. **상단 GNB**: 로고 + 검색 + 장바구니 + 알림
2. **메인 배너 슬라이더** (Swiper.js — CSS에 `swiper-*` 클래스 다수 확인)
3. **카테고리 그리드**: 국가/품종/타입(레드/화이트/스파클링/주정강화) 진입
4. **추천/신상품 가로 스크롤**
5. **수입사별 큐레이션 섹션**
6. **하단 탭 네비** (홈/카테고리/검색/주문내역/마이)

### 4.3 제품 카드 (List)
관찰된 카드 토큰을 종합한 권장 스펙:
```
border-radius: 10px
background: #fff
shadow: 0 1px 6px 2px #1e1e1e0d
padding: 13px ~ 16px

[썸네일]   세로 비율 1:1.3 ~ 1:1.5 (와인 보틀 형상)
[수입사명] 12px / weight 500 / #838383
[제품명]   14px / weight 700 / #222 / 2줄 클램프
[가격]     16~18px / weight 700 / #222
[뱃지]     좌상단 absolute — 신상품/세일/품절 등
```

---

## 5. 제품 상세 페이지 (Product Detail) — 권장 구성

> 실제 SPA가 렌더 후에야 상세 마크업이 보이므로, **수집한 디자인 토큰 + B2B 발주 사이트 관례**로 구성한 권장 레이아웃.

### 5.1 정보 위계 (모바일 600px 폭 기준)
```
┌─────────────────────────────┐
│ ← (뒤로)  [제품명]    ♡ ⤴   │ 상단 바: 흰 배경, 48px
├─────────────────────────────┤
│                              │
│      [메인 이미지 슬라이더]   │ 1:1 정사각, 흰 배경
│       ● ○ ○                  │ Swiper 인디케이터
├─────────────────────────────┤
│ [수입사 칩] [국가 칩]         │ 라임 칩 #fbffed/#7fe816
│                              │
│ 제품명 (한글)                 │ 18~20px / 800
│ Product Name (영문)           │ 14px / 500 / #838383
│                              │
│ ₩ 32,000  (vs 정가 ₩40,000)  │ 가격 22px / 800 #222
│ [-20%]                        │ 할인율 12px / 700 / #fe1400
├─────────────────────────────┤
│ [수량 - 1 +]    [장바구니]    │
│ [          발주하기          ] │ 48px CTA #911054
├─────────────────────────────┤
│ ▸ 상품 정보                   │ 섹션 헤더 16px / 700
│   품종, 빈티지, 도수, 용량      │ 12px label / 14px value
│   원산지, 양조장, 매칭푸드     │ 좌 #838383 / 우 #222
├─────────────────────────────┤
│ ▸ 테이스팅 노트                │
│   (Cafe24Ssurround 인용문)    │
├─────────────────────────────┤
│ ▸ 상세 이미지                  │ 풀폭, 마진 0
├─────────────────────────────┤
│ ▸ 발주 안내 / 배송 정보        │
│ ▸ 같은 수입사 다른 와인        │ 가로 스크롤
└─────────────────────────────┘
[고정 푸터: 발주하기 #911054]
```

### 5.2 상세 페이지 토큰
| 영역 | 토큰 |
|---|---|
| 이미지 영역 배경 | `#fff` (보틀 컷아웃 그대로) |
| 본문 패딩 | 좌우 `20px`, 섹션 간 `24px` |
| 섹션 디바이더 | `1px solid #f0f0f0` |
| 스펙 테이블 행 | `padding: 12px 0`, `border-bottom: 1px solid #f0f0f0` |
| 라벨 텍스트 | `12px / #838383 / weight 500` |
| 값 텍스트 | `14px / #222 / weight 600` |
| 정가 표시 (취소선) | `12px / #aaa / line-through` |
| 판매가 | `20-22px / #222 / weight 800` |
| 할인율 | `12-14px / #fe1400 / weight 700` |
| 칩(국가/품종) | `padding: 5px 8px / radius 4px / 12px / weight 500` |
| 고정 푸터 CTA | `height 4.8rem / radius 0.5rem / bg #911054 / 18px / weight 800 / white` |
| 수량 스테퍼 | 1px 보더 `#e4e4e4`, 라운드 `4px`, 정사각 36-40px 셀 |

### 5.3 상태별 처리
- **품절**: 이미지 위 `rgba(0,0,0,0.4)` 오버레이 + 중앙 "품절" 라벨 (`#fff`, 700, 16px). CTA 비활성 `#d2d2d2`.
- **로그인 필요**: CTA를 "로그인 후 발주" 로 치환, 동일 컬러.
- **재고 임박**: 라임 라이트(`#fbffed`) 배지 + `#60bf00` 텍스트.
- **신상품 / NEW**: 좌상단 absolute, 라운드 `4px`, `bg #911054`, `color #fff`.

---

## 6. 아이콘 & 일러스트

CSS만으로는 아이콘 라이브러리 식별 불가하지만:
- Swiper.js 내장 화살표(`font-family: swiper-icons`) 사용 → 자체 SVG/PNG일 가능성 높음
- `/img/favicon/` 경로 → 정적 이미지 자산은 자체 호스팅
- 메인 심볼: `https://file.marketbang.kr/common/mkb_symbol_w.png` (흰색 변형 → 어두운 배경 OG용)

→ 권장: 외곽선(stroke) 위주 24px 그리드 아이콘. Heroicons / Material Outline 또는 자체 셋.

---

## 7. 디자인 시스템 요약 (한 줄 요약)

> **"버건디 와인 + 라임 그린 + 깔끔한 그레이스케일, Pretendard 본문 / 4.8rem 굵은 CTA / 라디우스 10px 카드 / 매우 옅은 그림자(5% alpha)" — 모바일 우선 B2B 발주에 최적화된 차분한 와인 셀러 톤.**

### 핵심 6개 토큰
```css
--primary:        #911054;   /* CTA */
--accent:         #7fe816;   /* 활성/긍정 */
--text:           #222;      /* 본문 */
--text-muted:     #838383;   /* 보조 */
--border:         #e4e4e4;   /* 분리선 */
--bg-card:        #fff;      /* 카드 */
--bg-section:     #fafafa;   /* 섹션 */
--shadow-card:    0 1px 6px 2px #1e1e1e0d;
--radius-card:    10px;
--radius-button:  5px;
--font-body:      Pretendard, ...;
--font-display:   Cafe24Ssurround;
```

---

## 8. 참고

- CSS 번들 경로: `https://marketbang.kr/assets/index-CNmuu2DK.css` (175KB)
- JS 번들 경로: `https://marketbang.kr/assets/index-C7yenFZP.js`
- 운영사: 쓰리랩스(주) / `<meta name="author">`
- OG 이미지: `https://file.marketbang.kr/common/mkb_symbol_w.png`
- 분석 방법: CSS 정적 파싱 (변수 추출 → 빈도 정렬 → 사용 컨텍스트 grep)
- **한계**: 실제 상세 페이지 마크업은 SPA 런타임에서 생성되므로 5장(상세 페이지)은 토큰 기반 권장 스펙임. 실측이 필요하면 헤드리스 브라우저로 렌더 후 DOM 인스펙션 필요.
