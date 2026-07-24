# 인계 노트 — 2026-07-24 세션 종료

## 0. 현재 상황 한 줄

`ar.html`에서 **마커 인식은 되는데 마커 위 3D 콘텐츠가 하나도 렌더링되지 않음.**
공식 예제(`test-official.html`)는 같은 환경에서 **정상 작동**하므로, 문제는 우리 코드에 있음.

---

## 1. 오늘 확정된 사실 4가지

| # | 확정 사실 | 근거 |
|---|---|---|
| 1 | MindAR + A-Frame 조합은 **정상 작동한다** | `test-official.html`에서 카드·캐릭터 모델 표시 성공 |
| 2 | A-Frame **1.4.1, 1.6.0 둘 다 정상** | 같은 파일에서 버전만 바꿔 양쪽 성공. 버전 문제 아님 |
| 3 | 우리 `marker.png` = 공식 `card.png` **완전 동일** | MD5 `10b1bd5e7d8a55adc5df43eb9336eaac` 일치 |
| 4 | 문제는 **우리 `ar.html` 안에 있다** (원인 미확정) | 1~3에 의해 라이브러리·버전·마커 전부 배제 |

### 검증된 CDN 경로 (재확인 불필요)

```
https://aframe.io/releases/1.6.0/aframe.min.js                                  (200)
https://cdn.jsdelivr.net/npm/mind-ar@1.2.5/dist/mindar-image-aframe.prod.js     (200)
```
- `cdn.jsdelivr.net/gh/hiukim/mind-ar-js@.../dist/...` 는 **404**. `dist`는 저장소에 없음. 절대 쓰지 말 것.
- `mindar-image-three.prod.js` 는 **ES 모듈**이라 일반 `<script>`로 로드 불가 (`window.MINDAR` 안 생김).
- 단, `.../gh/hiukim/mind-ar-js@master/examples/...` (assets)는 정상 200.

---

## 2. 배제된 원인 후보

| 후보 | 배제 근거 |
|---|---|
| 스케일 부족 | 기준판을 100×100으로 키워도 안 보임 |
| 좌표계 방향 뒤집힘 | 공식 예제가 `rotation="0 0 0"`으로 정상 표시. `side:double`도 적용했으나 무효 |
| 마커 추적 불안정 | 5초간 Found 2 / Lost 1 — 정상 범위 |
| A-Frame 버전 | 1.4.1 / 1.6.0 둘 다 공식 예제 정상 작동 |
| 마커 이미지 품질 | 공식 파일과 MD5 동일 |
| 조명 부족 | ambient 2.0 + directional 2개 있음. 게다가 공식 예제는 조명이 **아예 없는데도** 표시됨 |
| GLB 재질 손실 | 증상이 "흰색으로 뜸"이 아니라 "아예 안 뜸". 해당 없음 |

---

## 3. 남은 원인 후보 (다음 세션 조사 대상)

### ★ 1순위 — `embedded` + 컨테이너 높이 없음 (신규 발견, 유력)

**공식 예제(작동)와 우리 `ar.html`(미작동)의 구조 차이:**

```html
<!-- test-official.html (작동) -->
<style>
  .example-container { overflow:hidden; position:absolute; width:100%; height:100%; }
</style>
<div class="example-container">
  <a-scene ... embedded>
</div>

<!-- ar.html (미작동) — 감싸는 컨테이너가 없음 -->
<style>
  html, body { margin:0; padding:0; overflow:hidden; }   /* height 지정 없음 */
</style>
<body>
  <a-scene ... embedded>      <!-- 부모가 그냥 body -->
```

- `ar.html:20` — `html, body`에 **`height: 100%`가 없음**
- `ar.html:123-132` — `<a-scene>`에 `embedded` 속성이 있으나 **감싸는 컨테이너 div 없음**
- `embedded` 모드에서 A-Frame은 WebGL 캔버스를 **부모 요소 크기에 맞춤**. 부모(body)에 높이가 없으면 캔버스 높이가 0이 될 수 있음.

**이 가설이 모든 관찰을 설명함:**

| 관찰 | 설명 |
|---|---|
| 카메라 화면은 정상 표시 | MindAR의 `<video>`는 캔버스와 **별개 요소**라 영향 없음 |
| `targetFound` 정상 발생 | 추적은 video 프레임 기반. 렌더링과 무관 |
| 자동맞춤 로그 정상 (배율 1.805e-2) | JS는 정상 실행. **렌더링만** 안 됨 |
| 기준판 100×100도 안 보임 | 캔버스 자체가 0 높이면 크기 무관 |
| 큐브·좌표축도 안 보임 | 위와 동일 |

**검증 방법 (다음 세션 첫 작업):** 브라우저 F12 콘솔에서
```js
document.querySelector('a-scene').canvas.getBoundingClientRect()
```
→ `height`가 0 또는 비정상이면 **확정**.
수정은 컨테이너 div 추가 또는 `html,body{height:100%}` 또는 `embedded` 제거 중 하나.

### 2순위 — `<a-scene>` 속성 차이

| 항목 | 공식 (작동) | 우리 (미작동) |
|---|---|---|
| `color-space` | `"sRGB"` | **없음** |
| `renderer` | `"colorManagement: true, physicallyCorrectLights"` | `"colorManagement: true; antialias: true"` |

### 3순위 — 자동맞춤 로직이 모델을 화면 밖으로 보냄
`ar.html:227-252`. 단, **기준판·큐브까지 안 보이는 건 설명 못 함**(자동맞춤은 모델에만 적용).

### 4순위 — 우리 `targets.mind` / `mep_test.glb` 파일 자체 문제
마찬가지로 **기준판·큐브 미표시를 설명 못 함.** 우선순위 낮음.

---

## 4. 다음 세션 첫 작업 제안

원래 제안된 순서(공식 mind → 우리 targets.mind → 우리 GLB → 자동맞춤 추가)는 타당하나,
**그보다 먼저 1순위(캔버스 크기)를 30초 만에 확인할 것.** 이유: 3·4순위 후보는
"초록 판과 빨간 큐브까지 안 보인다"는 핵심 관찰을 **설명하지 못함**. 파일 교체부터
시작하면 설명력 없는 후보에 시간을 쓰게 됨.

권장 순서:

1. **[30초] 캔버스 크기 확인** — `ar.html`에서 F12 → `document.querySelector('a-scene').canvas.getBoundingClientRect()`
   - height 0 → **원인 확정**. 컨테이너 div 추가로 수정 후 종료.
   - height 정상 → 2번으로.
2. **[역방향 교체]** `test-official.html` 복사본을 기준으로 우리 요소를 **하나씩** 얹으며 어디서 깨지는지 확인:
   1) 공식 mind → 우리 `targets.mind`
   2) 공식 GLB → 우리 `mep_test.glb`
   3) 우리 자동맞춤 로직 추가
   - 정방향(우리 파일에서 빼기)보다 역방향(공식에서 더하기)이 원인 격리가 빠름.

---

## 5. 파일 상태

| 파일 | 크기 | 역할 | 상태 |
|---|---|---|---|
| `index.html` | 428 B | model-viewer 3D 뷰어 (AR 아님) | **정상 작동**. 백업용 유지. 커밋됨 |
| `ar.html` | 15.6 KB | MindAR AR 페이지 — **작업 대상** | 미작동. 실험용 코드 포함. **미커밋** |
| `test-official.html` | 4.6 KB | 공식 예제 복제 — **대조군** | **정상 작동**. 다음 세션 계속 사용. 미커밋 |
| `mep_test.glb` | 7.5 MB | Revit 변환 배관 모델 | 커밋됨. AR에서 미검증 |
| `targets.mind` | 256 KB | `marker.png` 학습 파일 | 커밋됨. AR에서 미검증 |
| `marker.png` | 61.7 KB | 마커 이미지 (= 공식 card.png) | 커밋됨. 인쇄본 보유 |

### `ar.html`에 남아있는 실험용 변경 (다음 세션에 정리 판단)

| 실험 코드 | 위치 | 원래 값 |
|---|---|---|
| 기준판 100×100, opacity 0.5, position `0 0 1` | `ar.html:143-148` | 1×1, opacity 0.3, position `0 0 0` |
| 빨간 큐브 (대조군) | `ar.html:150-151` | 없었음 |
| 좌표축 3개 (X빨강/Y초록/Z파랑) | `ar.html:153-156` | 없었음 |
| 상단 바 큰 글씨 (26px, 가운데) | `ar.html:23-32` | 14px, 좌측 정렬 |
| Found/Lost 카운터 오버레이 | `ar.html:34-42, 82-86, 292-306` | 없었음 |

> 원인 확정 전까지는 **되돌리지 말 것.** 진단용으로 계속 필요함.

---

## 6. 변하지 않는 전제

- **PoC 합격선**: 마커 1개 위에 Revit 배관 모델이 **±15cm 이내**. 폰 1대(갤럭시 S24 울트라), 마커 1개, 파일 1개.
- **검증 순서**: 반드시 **로컬 → 폰**. 현재 로컬도 미해결이므로 폰 테스트는 아직 무의미.
- **PoC 범위 밖** (요청 와도 유보): 색상·재질·텍스처, UI 디자인, 다중 마커, 로그인, 기종 대응, 클라우드.
- **화면 내 로그창은 필수** (폰에 콘솔 없음).
- 커밋·푸시는 **사용자 승인 후에만**.

## 7. 미해결 부채 (PoC 통과 후 처리)

- `mep_test.glb` 7.5 MB — 모바일에 무거움. Draco 압축 시 1/5~1/10 예상. **MVP 단계 과제.**
