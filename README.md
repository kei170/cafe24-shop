# 골라담기 UI — 카페24 상품 상세 옵션 커스터마이징

## 프로젝트 개요

카페24 쇼핑몰 상품 상세페이지(`detail.html`)의 기본 텍스트/라디오 옵션 UI를 **카드형 골라담기 UI**로 교체하는 프로젝트입니다.

피그마 시안을 참고하여 골라담기 UI의 카드 구조, 간격, 선택 상태, 모바일 배치를 구현하였으며, 카페24 기본 구매 흐름과 옵션이 안정적으로 동기화되도록 개발하였습니다.

---

## 파일 구조

```
├── detail.html          # 상품 상세페이지 (골라담기 UI 삽입)
├── index.html           # 쇼핑몰 메인페이지
├── layout.html          # 공통 레이아웃
└── README.md            # 프로젝트 설명서
```

---

## 구현 체크리스트

| # | 요구사항 | 완료 |
|---|---------|------|
| 1 | 상품 상세페이지에 골라담기 형태의 커스텀 옵션 UI가 정상 노출된다 | ✅ |
| 2 | 커스텀 옵션 선택 시 카페24 기본 텍스트 버튼 옵션의 선택 상태와 구매 흐름이 함께 갱신된다 | ✅ |
| 3 | 옵션 선택 후 선택상품 목록이 정상 생성되고, 반복 클릭·재선택 시 오류나 중복이 생기지 않는다 | ✅ |
| 4 | 옵션값 suffix(`_1`, `_2`) 기준으로 개입수별 추가 가능 횟수가 제한된다 | ✅ |
| 5 | 개입 수별 금액/추가금액이 의도대로 계산되고, 옵션값 수만큼 상품 품목이 모두 생성되어 있다 | ✅ |
| 6 | 수량 변경, 총금액 계산, 장바구니 담기, 바로 구매가 정상 동작한다 | ✅ |
| 7 | 선택된 옵션의 활성화 상태가 사용자에게 명확히 표시된다 | ✅ |
| 8 | 옵션별 뱃지·설명·추천 문구·추가 가능 횟수·가격 표시 등 변경 가능한 데이터가 외부 설정 JS로 분리되어 있다 | ✅ |
| 9 | PC/모바일 반응형에서 옵션 UI 레이아웃이 깨지지 않는다 | ✅ |

---

## 기술 스택

- HTML5 / CSS3 / JavaScript (Vanilla)
- 카페24 스마트디자인 템플릿
- 카페24 옵션 API (`product_option_id1` 라디오 바인딩)

---

## 골라담기 UI 구현 상세

### 코드 삽입 위치

`detail.html`의 163번 줄, 카페24 옵션 테이블(`<table module="product_option">`) 바로 위에 삽입하였습니다.

```
162행: 카페24 기본 코드
163행: <!-- [구현] 골라담기 커스텀 옵션 UI 시작 -->
 ...    골라담기 style + HTML + script
440행: <!-- [구현 종료] 골라담기 커스텀 옵션 UI -->
441행: <table module="product_option"> ← 카페24 기본 옵션 (숨김 처리)
```

### 구현 구조

구현 코드는 CSS, HTML, JavaScript 세 부분으로 구성되며, 카페24 기본 옵션 테이블 바로 위에 하나의 블록으로 삽입됩니다.

```
┌─ <style> ─────────────────────────────────┐
│  기존 옵션 테이블 숨김 (JS로 처리)          │
│  골라담기 카드 UI 스타일                    │
│  선택상품 목록 스타일                       │
│  반응형 미디어 쿼리 (480px, 360px)          │
└────────────────────────────────────────────┘
┌─ <HTML> ──────────────────────────────────┐
│  .golladamgi-wrap (옵션 선택 UI)           │
│    ├─ .golladamgi-header (접기/펼치기)      │
│    └─ .golladamgi-list (옵션 행 목록)       │
│  .selected-list (선택상품 목록)             │
└────────────────────────────────────────────┘
┌─ <script> ────────────────────────────────┐
│  OPTION_CONFIG (외부 설정 데이터)           │
│  renderOptionList() — 옵션 목록 렌더링      │
│  golladamgiSelect() — 옵션 선택 처리        │
│  syncCafe24Option() — 카페24 옵션 동기화    │
│  renderSelectedList() — 선택상품 목록       │
│  changeQty() / removeItem() — 수량 관리    │
│  updateTotal() — 총금액 계산               │
│  initGolladamgi() — 초기화 (3중 폴백)       │
└────────────────────────────────────────────┘
```

---

## 체크리스트별 구현 설명

### #1 골라담기 UI 정상 노출

- `.golladamgi-wrap` 래퍼 안에 "옵션 선택 (필수)" 헤더와 옵션 행 목록을 렌더링합니다.
- `OPTION_CONFIG` 배열 기반으로 `renderOptionList()` 함수가 동적으로 옵션 카드를 생성합니다.
- 각 옵션 행에는 라디오 서클, 수량명, 가격, 할인율, 뱃지, 추가 가능 횟수가 표시됩니다.

### #2 카페24 옵션 동기화

- `syncCafe24Option(idx)` 함수가 카페24 기본 옵션과 동기화합니다.
- 카페24 라디오 버튼(`name="product_option_id1"`)을 `.click()`으로 직접 클릭하여 카페24 내부 옵션 선택 로직을 트리거합니다.
- 폴백 순서: 라디오(product_option_id1) → 라디오(name*="option") → select → EC_SDE_setOptionSelect()

### #3 선택상품 목록 / 중복·오류 방지

- `selectedItems` 배열로 선택된 상품 목록을 관리합니다.
- `itemKey`(예: `30_1`, `30_2`)로 중복을 방지합니다.
- `renderSelectedList()` 함수가 선택 목록을 렌더링하며, 수량 조절(+/−) 및 삭제(✕) 버튼을 제공합니다.
- 항목 제거 시 `renderOptionList()`를 재호출하여 추가 가능 횟수를 업데이트합니다.

### #4 suffix 기반 추가 횟수 제한

- `OPTION_CONFIG`의 `maxCount` 속성으로 옵션별 최대 추가 횟수를 설정합니다.
- 예: 30개입(`maxCount: 2`) → 30개입_1, 30개입_2까지 추가 가능
- 최대 횟수 초과 시 카드가 반투명(`.is-maxed`)으로 표시되고 "추가불가" 텍스트가 나타납니다.

### #5 개입수별 금액 계산

- 각 옵션의 `price`, `origin`, `pct`, `unit` 값이 `OPTION_CONFIG`에 정의되어 있습니다.
- 선택상품 목록에서 `가격 × 수량`으로 개별 금액이 계산됩니다.
- `updateTotal()` 함수가 전체 총금액을 합산하여 카페24 `.totalPrice` 영역을 갱신합니다.

### #6 수량 변경 / 장바구니 / 바로 구매

- `changeQty(index, delta)` 함수로 수량 증감을 처리합니다 (최소 1개).
- 카페24 구매 버튼(`.btnSubmit.sizeL`)과 장바구니 버튼(`.btnNormal.sizeL`)에 클릭 이벤트를 바인딩합니다.
- 옵션 미선택 시 `e.preventDefault()`로 구매를 차단하고 "옵션을 선택해주세요" 알림을 표시합니다.

### #7 선택 상태 명확 표시

- **hover 효과**: 마우스 올리면 배경색 변경(`#fff8f5`) + 라디오 서클 오렌지 테두리
- **선택 상태**: 오렌지 라디오 점 + 연한 오렌지 배경(`#fff3ee`) + `is-selected` 클래스
- **최대 횟수 도달**: 반투명 처리(`.is-maxed`, `opacity: 0.45`) + "추가불가" 텍스트
- **구매 버튼 hover 툴팁**: 옵션 미선택 시 "⚠ 옵션을 선택해 주세요" 툴팁 표시

### #8 외부 설정 JS 분리

- `OPTION_CONFIG` 객체 배열로 모든 옵션 데이터를 분리하였습니다.
- 운영자가 이 배열만 수정하면 UI에 자동 반영됩니다.

```javascript
var OPTION_CONFIG = [
    { id:'10',  qty:'10개입',  price:24700,  origin:33000,  pct:25, unit:2470, badge:null,            badgeClass:'',    maxCount:2 },
    { id:'30',  qty:'30개입',  price:70500,  origin:99000,  pct:29, unit:2350, badge:'가장 많이 사요',  badgeClass:'best', maxCount:2 },
    { id:'50',  qty:'50개입',  price:111500, origin:165000, pct:32, unit:2230, badge:null,            badgeClass:'',    maxCount:2 },
    { id:'100', qty:'100개입', price:196000, origin:330000, pct:41, unit:1960, badge:'최대할인',        badgeClass:'max',  maxCount:1 }
];
```

| 속성 | 설명 |
|------|------|
| `id` | 옵션 고유 식별자 |
| `qty` | 화면에 표시되는 수량명 |
| `price` | 할인 판매가 (원) |
| `origin` | 원래 정가 (원) |
| `pct` | 할인율 (%) |
| `unit` | 개당 가격 (원) |
| `badge` | 뱃지 텍스트 (null이면 미표시) |
| `badgeClass` | 뱃지 CSS 클래스 (`best` / `max`) |
| `maxCount` | 최대 추가 가능 횟수 |

### #9 PC/모바일 반응형

- 기본: 세로 리스트형 (PC/모바일 동일 구조)
- `480px 이하`: 카드 패딩·폰트 축소
- `360px 이하`: 극소형 화면 텍스트 깨짐 방지
- 터치 영역이 구매 버튼과 겹치지 않도록 옵션 UI와 구매 영역을 분리 배치하였습니다.

---

## 카페24 옵션 동기화 방식

카페24는 렌더링 시 `module="product_option"` 속성을 제거하기 때문에, CSS 셀렉터로 옵션 테이블을 직접 선택할 수 없습니다. 따라서 다음과 같은 동기화 전략을 사용합니다.

```
골라담기 카드 클릭
  → golladamgiSelect(opt, idx)
    → syncCafe24Option(idx)
      → [A] input[name="product_option_id1"] 라디오 .click()  ← 최우선
      → [B] input[name*="option"] 라디오 .click()             ← 폴백 1
      → [C] select[name*="option"] 값 변경                    ← 폴백 2
      → [D] EC_SDE_setOptionSelect(0, idx)                    ← 폴백 3
```

기존 옵션 UI 숨김은 JS `initGolladamgi()`에서 라디오 버튼의 부모 요소를 찾아 `visibility: hidden`으로 처리합니다.

---

## 초기화 전략

카페24 환경에서는 `DOMContentLoaded`가 이미 발생한 후 스크립트가 실행될 수 있어, 3중 폴백으로 안전하게 초기화합니다.

```javascript
if (!initGolladamgi()) {
    document.addEventListener('DOMContentLoaded', function() { initGolladamgi(); });
    setTimeout(function() { initGolladamgi(); }, 500);
    setTimeout(function() { initGolladamgi(); }, 1500);
}
```

---

## 주요 함수 설명

| 함수명 | 역할 |
|--------|------|
| `renderOptionList()` | OPTION_CONFIG 기반 옵션 카드 동적 생성 |
| `golladamgiSelect(opt, idx)` | 카드 선택 → 상태 갱신 → 카페24 동기화 |
| `syncCafe24Option(idx)` | 카페24 기본 옵션 라디오/select 동기화 |
| `renderSelectedList()` | 선택상품 목록 렌더링 (수량 조절, 삭제) |
| `changeQty(index, delta)` | 선택상품 수량 증감 |
| `removeItem(index)` | 선택상품 삭제 |
| `updateTotal()` | 총금액 계산 및 카페24 금액 영역 갱신 |
| `golladamgiToggle()` | 옵션 목록 접기/펼치기 |
| `initGolladamgi()` | 전체 초기화 (렌더링 + 기존 옵션 숨김 + 이벤트 바인딩) |

---

## 테스트 환경

- 카페24 테스트몰: `https://kei10.cafe24.com`
- 스마트디자인 편집기에서 `product/detail.html` 수정
- 상품 옵션 설정: 수량 선택 (10개입 / 30개입 / 50개입 / 100개입) 라디오버튼 타입
