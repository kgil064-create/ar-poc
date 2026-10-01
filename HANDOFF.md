# 인계 노트 — 2026-10-01 세션 종료

## 0. 현재 상황 한 줄

**PoC 5단계 완료.** 인쇄 마커 위에 Revit 배관 모델이 렌더링되며, **로컬 PC와 갤럭시 S24 울트라 양쪽에서 검증 완료.**

---

## 1. 오늘 세션 요약

지난 세션에서 1순위 가설로 남겨둔 "`embedded` + 컨테이너 높이 없음"을 F12 콘솔로 확인해
**캔버스가 `width 2197.6 × height 0`** 임을 확정했다. 마커 인식·추적·JS는 전부 정상이고
**그릴 면적이 0이라 아무것도 안 보였던 것**이다. 공식 예제의 `.example-container` 패턴을
`#arContainer`(absolute + 100%)로 옮겨 적용하자 모델이 즉시 나타났다. 이어서 사용자가
슬라이더로 찾은 자연스러운 값(7.24x / 5° / 95° / 0.94)을 코드에 고정하고 진단용 실험
요소(기준판·큐브·좌표축)를 제거했다. GitHub Pages로 배포해 폰에서도 같은 결과를 확인,
**로컬 → 폰 검증 순서를 모두 통과**했다.

---

## 2. 확정된 사실

| # | 확정 사실 | 근거 |
|---|---|---|
| 1 | 원인은 **캔버스 높이 0**이었다 | `canvas.getBoundingClientRect()` → `width 2197.6 / height 0` |
| 2 | `#arContainer` 수정이 **로컬·폰 양쪽에서 유효** | 로컬 2752×983, 폰 384×644 — 둘 다 양수 |
| 3 | A-Frame 1.6.0 + MindAR 1.2.5 + GitHub Pages HTTPS 조합 **안정** | 폰에서 라이브러리 로드·카메라·추적 전부 정상 |
| 4 | `mep_test.glb` 로드 정상 | 폰에서 1,933,373 bytes 수신 성공 |
| 5 | 하드코딩 기본값이 **로컬·폰 공통 유효** | 슬라이더 미조작 상태로 양쪽 동일하게 표시 |
| 6 | 추적 안정성 양호 | 폰에서 F1/L0 → F4/L3, 끊김 없음 |

### GLB 전송량 — 정확한 수치 (중요)

```
디스크   7,870,792 bytes (7.51 MB)
전송량   1,933,373 bytes (1.93 MB)   ← GitHub Pages 가 gzip 자동 적용
```

폰 로그의 1.93MB는 **압축 전송량**이고 파일 자체는 여전히 7.5MB다.
따라서 **네트워크 비용은 이미 1/4로 해결된 상태**이고, 남은 부담은
기기에서 풀어 올릴 때의 **파싱 시간과 메모리(7.5MB)** 뿐이다.
→ 지난 세션 "미해결 부채"에 적어둔 Draco 압축의 **우선순위를 낮춰도 된다.**

### 확정된 기본값 (코드에 고정됨)

| 항목 | 값 | 적용 위치 |
|---|---|---|
| 자동맞춤 배율 | `FIT_MARKER_SPAN = 7.24` → 최종 배율 `1.307e-1` | `ar.html` `FIT_MARKER_SPAN` |
| X축 회전 | 5° | `DEFAULTS.rotX` |
| Y축 회전 | 95° | `DEFAULTS.rotY` |
| 띄우기 | 0.94 (**+Z축**. 마커 평면에서 카메라 쪽) | `DEFAULTS.lift` |

> 띄우기는 **Y가 아니라 Z**다. MindAR 마커 좌표계에서 +Z가 마커에서 떠오르는 방향이고,
> 사용자가 0.94를 찾은 축이 바로 그 Z다. Y로 바꾸면 확인된 화면이 깨진다.
>
> 7.24를 슬라이더가 아니라 자동맞춤 쪽에 넣은 이유: 슬라이더 범위가 0.01x~10x라
> 7.24를 출발점으로 삼으면 **위로 1.38배밖에 못 키운다.** 자동맞춤에 흡수하면
> 슬라이더가 1.00x 중립에서 시작해 위아래로 여유가 생긴다.

### 검증된 CDN 경로 (재확인 불필요 — 지난 세션에서 이어짐)

```
https://aframe.io/releases/1.6.0/aframe.min.js                                  (200)
https://cdn.jsdelivr.net/npm/mind-ar@1.2.5/dist/mindar-image-aframe.prod.js     (200)
```
- `cdn.jsdelivr.net/gh/hiukim/mind-ar-js@.../dist/...` 는 **404**. `dist`는 저장소에 없음. 절대 쓰지 말 것.
- `mindar-image-three.prod.js` 는 **ES 모듈**이라 일반 `<script>`로 로드 불가.

---

## 3. 커밋 범위 투명성 공지

**커밋 `733150f` (`fix(ar): 캔버스 높이 0 문제 수정 + 모델 기본값 하드코딩 + 진단 요소 제거`)
은 커밋 메시지가 설명하는 범위보다 실제 변경 범위가 넓다.**

| | |
|---|---|
| 직전 커밋의 `ar.html` | **69줄** |
| 이 커밋의 `ar.html` | **396줄** |
| 차이 | 386 insertions / 60 deletions |

지난 세션(2026-07-24)에 만든 UI 전체 — **슬라이더 조절 패널, 화면 내 로그창,
상태 표시줄, 자동맞춤 로직, 파일 존재 확인** — 가 한 번도 커밋되지 않은 상태였고,
이번 커밋에 **함께** 들어갔다. 코드의 최종 상태는 커밋 메시지 내용과 정확히 일치하지만,
커밋이 담은 범위는 메시지보다 넓다.

**히스토리는 정정하지 않는다.** 이미 푸시된 커밋이라 수정에 force-push가 필요하고,
PoC 단계에서 치를 비용이 아니다. **이 문서에 기록만 남긴다.**

> 다음 세션 교훈: 실험용 변경도 세션 끝에 한 번은 커밋해 두면 이런 누적이 안 생긴다.

---

## 4. 다음 세션 주요 과제 — 트래킹 모드 재검토

### 문제

현재 구조는 **마커가 카메라 화면에서 벗어나면 모델도 사라진다**(MindAR 이미지 트래킹의
본질적 동작). 현장ON의 실제 사용 시나리오는 **작업자가 폰을 들고 현장을 이동하며
배관을 확인**하는 것이므로, 마커를 계속 화면에 담아야 하는 제약은 시나리오와 맞지 않는다.

### 옵션

| 옵션 | 내용 | 1차 인상 |
|---|---|---|
| **A** | MindAR 유지 + UX로 회피. 마커를 현장에 분산 배치해 **하나만 보여도 유지** | 비용 0, 학습곡선 0. 단 마커 인쇄·부착 작업이 현장 부담 |
| **B** | **WebXR Anchor API** 전환 (SLAM 기반 공간 추적) | 무료·웹 유지. 안드로이드 ARCore 의존, 기종 편차 위험 |
| **C** | **8th Wall** 유료 평가판 (상용급 SLAM) | 품질 최상·구현 빠름. 구독 비용, 벤더 종속 |

### ✅ 결정: **옵션 B 확정** (2026-10-01)

| 옵션 | 결과 | 근거 |
|---|---|---|
| A | 탈락 | 마커 다중 배치 UX는 "검증단계에서 못 쓰겠다" 판정 받을 가능성이 큼 |
| **B** | **채택** | 품질 적정 + 비용 0 + WebXR은 W3C 표준이라 미래 확장성 |
| C | 보류 | 월 ~20만원 영구 의존. 지금 단계에 과투자 |

> **분기 조건**: 사업이 진전되고 정확도 요구가 올라가면 C를 재검토한다.

> 참고: `ar.html` 의 `missTolerance: 5` / `warmupTolerance: 5` 는 추적 끊김을
> 약간 버티게 하는 값이다. 옵션 A로 되돌아갈 경우 이 수치부터 조정해 볼 수 있다.

### ⚠️ 조사 결과 — B는 "MindAR → WebXR 전환"이 아니라 **"MindAR + WebXR 결합"**이다

조사(2026-10-01, Chrome Platform Status API·Google ARCore 문서 직접 확인):

| 기능 | Chrome Android | 플래그 |
|---|---|---|
| WebXR AR Module (`immersive-ar`) | Enabled by default (M81) | 불필요 |
| **WebXR Anchors** | **Enabled by default (M79)** | 불필요 |
| **WebXR Raw Camera Access** | **Enabled by default (M107)** | 불필요 |
| **WebXR Image Tracking** | **"No active development"** | 플래그 있어도 **Chrome이 개발 중단** |

**WebXR에는 쓸 수 있는 마커 인식 기능이 없다.** Google의 ARCore↔WebXR 비교표도
"Augmented Images"를 **미지원**으로 명시한다. 따라서 마커 인식은 계속 MindAR이 담당하고,
공간 추적만 WebXR anchors로 넘기는 **결합 구조**가 된다.

**그 대신 `targets.mind` 는 재활용된다** (조사 전 전제는 "사용 불가"였으나 틀렸음).
MindAR `Controller` 소스 확인 결과:

```
addImageTargets(fileURL)                        // './targets.mind' 그대로 로드
async detect(input)                             // 임의 프레임 투입 가능 (canvas drawImage 기반)
getWorldMatrix(modelViewTransform, targetIndex) // 포즈 행렬 반환
```

→ MindAR을 A-Frame 없이 **마커 인식 엔진으로만** 떼어 쓸 수 있다
(`mindar-image.prod.js` = 코어 전용 빌드, CDN 200 확인).
**버리는 것은 A-Frame 의존성뿐이고, 자산 4종은 전부 재활용된다.**

**설계상 결정적 포인트**: 인식은 **1회만** 필요하다(마커 잡는 순간 anchor 생성 후 MindAR 종료).
매 프레임 tfjs를 돌릴 필요가 없어 성능 부담이 급감한다.

### 역방향 증분 계획 — 공식 예제에서 하나씩 더하기

이번 세션에 효과를 본 전략(공식 예제 → 우리 것 추가)을 그대로 적용한다.

| 단계 | 파일 | 증명 대상 | 상태 |
|---|---|---|---|
| **1** | `xr-test.html` | immersive-ar + 카메라 텍스처 접근 | ✅ **통과** (아래) |
| **2** | `xr-test-2.html` | hit-test + anchors → 걸어가도 제자리 | 작성 완료, 검증 대기 |
| 3 | 미작성 | MindAR Controller + 기존 `targets.mind` 로 1회 detect | 대기 |
| 4 | 미작성 | `mep_test.glb` 얹기 + 기존 기본값 적용 | 대기 |

> 1·2단계가 전체 리스크의 대부분이다. 2단계까지 통과하면 B 확정, 실패하면
> 코드를 거의 안 버리고 C로 선회할 수 있다.

### ✅ 1단계 완료 — `xr-test.html` (2026-10-01)

**테스트 환경**: 갤럭시 S24 울트라 / Android Chrome (stable 155 계열) / GitHub Pages HTTPS

```
[7] AR 세션 시작됨. enabledFeatures = ["local","camera-access","viewer","dom-overlay"]
[8] 카메라 텍스처 획득 성공 (886 × 1920) — 1단계 통과
[10-12] 두 번째 세션 재현 — 동일 결과
```

| 확인 사항 | 결과 |
|---|---|
| `immersive-ar` 세션 승인 | ✅ |
| `camera-access` 가 **enabledFeatures 에 실제 포함** | ✅ |
| 카메라 WebGLTexture 획득 | ✅ **886 × 1920** |
| 세션 재현성 | ✅ 2회 연속 동일 |
| `dom-overlay` 자동 활성화 | ✅ **보너스** (요청 안 했는데 부여됨) |

> **조사 때 "최대 기술 리스크"로 꼽았던 항목이 해소됐다.** WebXR 카메라가
> opaque WebGLTexture 로만 나오는 점을 우려했으나, 획득 자체는 확실히 된다.
> 남은 과제는 그 텍스처를 MindAR 이 요구하는 canvas 로 옮기는 blit 단계(3단계).
>
> `dom-overlay` 가 실제로 부여되는 것이 확인됐으므로, 2단계부터는 **세션 중
> HUD**(거리 숫자)를 띄울 수 있다. ±15cm 판정은 눈대중으로 불가하므로 꼭 필요하다.

**커밋 메시지 주의**: 1단계 커밋(`e95f43b`)의 메시지는 **"PC 로컬 로드 확인"**까지만
적혀 있다. 폰 통과는 그 커밋 **이후**에 확인됐고, 메시지를 번복하지 않기로 해
**이 문서가 폰 통과의 기록**이다.

### ✅ 2단계 완료 — `xr-test-2.html` (2026-10-01)

**테스트 환경**: 갤럭시 S24 울트라 / 복도 / **광택 타일 + 유리벽** (ARCore에 불리한 조건)

```
enabledFeatures = ["local","camera-access","viewer","hit-test","dom-overlay","anchors"]
```

| 확인 사항 | 결과 |
|---|---|
| **요청한 feature 전부 승인** (`anchors`, `hit-test` 포함) | ✅ |
| `hit.createAnchor()` 1순위 경로 성공 | ✅ (폴백 불필요) |
| **실사용 시나리오 — anchor 1개 + 걸어갔다 돌아오기** | ✅ **거의 제자리. 드리프트 수용 가능** |
| 스트레스 테스트 — anchor 11개 | ⚠️ `trackedAnchors` 상실 관찰 |

**사용자 판정: 통과.** 11개 상실은 **환경(광택 타일·유리벽) + anchor 과다 생성의 복합 원인**으로
판단했다. 실제 설계는 **anchor 1개**(마커 1개 → anchor 1개)이므로 해당 시나리오에서
작동함이 확인된 것으로 충분하다.

> **교훈으로 남길 것**: 광택 바닥·유리벽은 ARCore에 불리하다. 현장 테스트 장소 선정 시
> 바닥 무늬가 있고 반사면이 적은 곳을 고를 것. anchor 개수는 **최소로 유지**한다.

### 조사 단계의 미해결 리스크 — 전부 해소됨

| 조사 때 "확인 불가"였던 항목 | 결과 |
|---|---|
| S24 Ultra에서 `anchors` 실제 동작 | ✅ 확인 (2단계) |
| WebGLTexture 획득 가능 여부 | ✅ 확인 (1단계, 886×1920) |
| 공간 추적이 쓸 만한 수준인지 | ✅ 확인 (anchor 1개 시나리오) |

---

## 4-A. 3단계 착수 전 확인한 기술 사실 (2026-10-01)

MindAR 소스를 직접 읽어 확인한 것. **다음 세션에서 재조사하지 말 것.**

### `mindar-image.prod.js` 는 ES 모듈이다 (266 bytes 짜리 shim)

```js
// mindar-image.prod.js 전문 — 형제 청크를 import 하는 ES 모듈
import { C as o, a as r } from "./controller-mGt1s8dJ.js";   // 2.2 MB (tfjs 포함)
import { U as i } from "./ui-fBadYuor.js";
window.MINDAR.IMAGE = { Controller: o, Compiler: r, UI: i };
export { r as Compiler, o as Controller, i as UI };
```

- **일반 `<script>` 로는 로드 불가.** `<script type="module">` 또는 importmap 필요.
- 청크 경로가 상대경로라 **CDN 절대경로로 로드하면 자동 해결**된다.
- **Web Worker 는 base64 → Blob → `createObjectURL` 로 인라인**되어 있고
  실패 시 `data:` URL 폴백까지 있다. → **CDN 로드에서 깨지지 않는다.**
- tfjs 가 번들에 포함되어 있어 별도 로드 불필요.

### 🔴 MindAR 은 카메라 FOV 를 45° 로 **하드코딩**한다

`src/image-target/controller.js:36-47`:
```js
const fovy = 45.0 * Math.PI / 180;            // ← 하드코딩
const f = (this.inputHeight/2) / Math.tan(fovy/2);
this.projectionTransform = [[f,0,W/2],[0,f,H/2],[0,0,1]];
```

핀홀 모델에서 추정 거리는 `Z = f × S / s(픽셀)` 이므로,
**가정한 f 가 틀리면 거리가 그 비율만큼 틀어진다**:

```
Z_실제 = Z_MindAR × (f_실제 / f_가정)
f_가정 에 해당하는 proj[5] = 1/tan(22.5°) = 2.41421
f_실제 에 해당하는 proj[5] = view.projectionMatrix[5]  (WebXR 이 알려줌)
→ 보정계수 = view.projectionMatrix[5] / 2.41421
```

A-Frame 환경에서는 `MindARThree.resize()` 가 **카메라 FOV 를 MindAR 쪽에 맞춰
역산**해서 쓰기 때문에 이 오차가 드러나지 않았다. WebXR 은 카메라 투영을 우리가
고칠 수 없으므로 **반대로 포즈를 보정해야 한다.**

### MindAR 포즈의 단위와 좌표계

`src/image-target/three.js` 가 쓰는 변환 그대로:
```js
M_카메라_마커 = Matrix4(controller.getWorldMatrix(mvt, idx)) × postMatrix
// postMatrix: position(markerW/2, markerW/2+(markerH-markerW)/2, 0), scale(markerW,markerW,markerW)
```
- 결과는 **카메라(view) 공간** 기준, three.js/GL 규약(오른손, -Z 전방)
- **그룹 로컬 공간에서 마커 가로폭 = 1 단위** ← `ar.html` 의 `FIT_MARKER_SPAN=7.24` 전제와 동일
- 따라서 미터로 바꾸려면 **인쇄된 마커의 실제 가로폭(m)** 이 필요하다

### ✅ 인쇄 마커 실측값 (확정, 2026-10-01)

```
가로 18 cm × 세로 10 cm   →  xr-test-3.html 접속 시  ?mw=0.18
```

**교차검증 통과**: `test-official.html` 의 공식 카드 비율은 `width="1" height="0.552"` 이고
`18 cm × 0.552 = 9.94 cm ≈ 10 cm` 로 실측과 일치한다. 즉 인쇄물이 원본 비율대로
출력되었음이 확인됐다 (찌그러진 인쇄가 아님).

> 이 값이 틀리면 anchor 가 엉뚱한 거리에 생긴다. 마커를 **다시 인쇄하면 반드시 재실측**할 것.
> `xr-test-3.html` 은 URL 파라미터로 받으므로 코드 수정 없이 바꿀 수 있다 (기본값 0.10m).

---

## 5. 그 외 백로그 (우선순위 낮음)

| 항목 | 내용 |
|---|---|
| GLB 재질 복원 | **현재 흰색만 렌더링됨.** Revit → GLB 변환 과정에서 재질이 유실된 것으로 추정. PoC 합격선 밖 |
| 폰 FOV 보정 | 지금은 로컬과 거의 동일하지만 **다른 기종에서는 다를 수 있음** |
| 자동맞춤 로직 범용화 | `FIT_MARKER_SPAN = 7.24` 는 **`mep_test.glb` 전용 실측값**. 다른 모델에는 안 맞음 |
| Draco 압축 | **우선순위 하향** (§2 참조 — 네트워크는 이미 gzip으로 1.93MB) |

---

## 6. 사업 관점 노트

- **모두의 창업 2차 재도전**: 작동하는 데모 확보. 더 이상 "계획"이 아니라 "작동함"으로 설명 가능
- **연동주민센터 현장 실측 테스트** 가능 단계 진입. 다음은 실제 배관 위에서 ±15cm 확인
- **투자자·파트너 시연 가능**. URL만 보내면 상대 폰에서 바로 열림 (앱 설치 불필요 — 웹 AR의 핵심 강점)

> 단, 시연 시 **§4의 제약(마커를 화면에 계속 담아야 함)을 먼저 밝힐 것.** 시연 중에
> 모델이 사라지면 설명 없이는 결함으로 보인다.

---

## 7. 파일 상태

| 파일 | 크기 | 역할 | 상태 |
|---|---|---|---|
| `ar.html` | 18.1 KB | MindAR AR 페이지 — **본체** | **로컬·폰 정상 작동.** 커밋됨 (`733150f`) |
| `index.html` | 428 B | model-viewer 3D 뷰어 (AR 아님) | 정상 작동. 백업용 유지. 커밋됨 |
| `test-official.html` | 4.6 KB | MindAR 공식 예제 복제 — **대조군** | 정상 작동. 커밋됨. **보존할 것** (회귀 판별용) |
| `xr-test.html` | 11.2 KB | WebXR **1단계** (camera-access) — **대조군** | ✅ **로컬·폰 통과.** 커밋됨 (`e95f43b`). **보존할 것** |
| `xr-test-2.html` | 19.6 KB | WebXR **2단계** (hit-test + anchors) — **대조군** | ✅ **폰 통과** (anchor 1개 시나리오). 커밋됨 (`6296938`) |
| `xr-test-3.html` | 32.9 KB | WebXR **3단계** (MindAR 인식 + anchor 결합) | PC 로컬 통과. **폰 검증 대기** |
| `mep_test.glb` | 7.51 MB | Revit 변환 배관 모델 | 커밋됨. **AR에서 검증 완료** |
| `targets.mind` | 256 KB | `marker.png` 학습 파일 | 커밋됨. **AR에서 검증 완료** |
| `marker.png` | 61.7 KB | 마커 이미지 (= 공식 card.png) | 커밋됨. 인쇄본 보유 |

### `ar.html` 에 남겨둔 진단 코드 (의도적 — 제거하지 말 것)

| 코드 | 이유 |
|---|---|
| `renderstart` 시 캔버스 크기 로그 | 다른 기종에서 **같은 문제가 재발하면 즉시 보이게** |
| 화면 내 로그창 (`#logbox`) | 폰에는 콘솔이 없음. **필수** |
| 상태줄의 `F_ / L_` 카운터 | 추적 안정성 판단 지표 |
| 슬라이더 4종 + 초기화 버튼 | 현장에서 기종별 미세조정용 |

---

## 8. 변하지 않는 전제

- **PoC 합격선**: 마커 1개 위에 Revit 배관 모델이 **±15cm 이내**. 폰 1대(갤럭시 S24 울트라), 마커 1개, 파일 1개.
  → **렌더링 단계는 통과.** 실측 오차 확인은 현장 테스트에서.
- **검증 순서**: 반드시 **로컬 → 폰**. (이번 세션에서 이 순서가 유효함을 재확인)
- **PoC 범위 밖** (요청 와도 유보): 색상·재질·텍스처, UI 디자인, 다중 마커, 로그인, 기종 대응, 클라우드.
- **화면 내 로그창은 필수** (폰에 콘솔 없음).
- 커밋·푸시는 **사용자 승인 후에만**.

---

## 9. 배포 정보

```
저장소   https://github.com/kgil064-create/ar-poc
브랜치   main  (GitHub Pages 가 여기서 서빙)
URL      https://kgil064-create.github.io/ar-poc/ar.html
```

- 푸시 후 Pages 재빌드에 **30초~2분** 걸린다. 바로 확인하면 구버전이 뜬다.
- 배포 확인법: `Last-Modified` 헤더 날짜 또는 응답 본문에 `arContainer` 문자열 존재 여부.
- 폰에서 구버전이 뜨면 캐시다. 주소 끝에 `?v=2` 를 붙이거나 사이트 데이터 삭제.
- 이 환경에는 `gh` CLI 가 **없다.** Actions 탭 확인 대신 배포본을 직접 HTTP로 확인할 것.
