# Copilot 지침

한국어 우주 탐사 퀴즈. 빌드·의존성·서버 없이 **`index.html` 단일 파일**(HTML + CSS + 바닐라 JS)로 구성된다. 번들러, 린터, 테스트 러너, git 저장소가 없다.

## 실행 및 검증

- 실행: 브라우저에서 `index.html`을 직접 연다(`file://`). 서버 불필요.
- 자동 테스트가 없다. 통합 브라우저의 `evaluate_javascript`로 `.opt` 버튼을 클릭하며 점수(`.score`), 진행 바(`.bar>div`의 `style.width`), 피드백 클래스(`.correct`/`.wrong`), 결과 화면(`.emoji`)을 검증한다.
- 통합 브라우저가 "동작 줄이기"(`prefers-reduced-motion`)를 켠 상태일 수 있다. 이 경우 전역 `animation:none!important` 규칙 때문에 흔들림/팝 애니메이션이 계산 스타일에 나타나지 않는다. 이는 결함이 아니다.

## 구조

- 상태는 모듈 전역 `i`(현재 문제), `score`, `answered` 세 개뿐이다.
- 렌더링은 `render()`(문제 화면) → `answer(idx)`(정오 처리) → `showResult()`(결과 화면) 흐름이며, 매번 `#card`의 `innerHTML`을 통째로 다시 그린다. 따라서 `.score`, `.bar>div`, `.fb`, `.btn` 같은 요소는 렌더링할 때마다 새로 생기므로 참조를 캐시하지 말고 매번 `card.querySelector`로 찾는다.
- 문제 데이터는 `QUESTIONS` 배열 `{q, o[4], a(정답 인덱스), e(해설)}`이다. 문제 수는 배열 길이에서 계산하므로 추가·삭제 시 다른 곳을 고칠 필요가 없다(제목의 "10문제" 같은 하드코딩은 없음). 선택지는 항상 4개를 전제로 한 레이아웃이다.
- 결과 이모지/문구는 `showResult()`의 정답률 구간(100%, 80%, 60%, 40%, 그 미만)으로 결정된다.

## 규칙

- 사용자가 어떤 언어(영어 등)로 입력하더라도 항상 한국어로 응답한다.
- 모든 사용자 노출 텍스트(문제, 해설, UI 문구)는 한국어다. 문서도 한국어로 작성한다.
- 색상은 `:root` CSS 변수로만 정의하고, 다크 테마는 `@media (prefers-color-scheme:dark)`에서 같은 변수를 덮어쓴다. 새 색을 하드코딩하지 말고 변수(`--good`, `--bad`, `--accent` 등)를 사용하며, 투명도 혼합은 `color-mix`를 쓴다.
- 정답 피드백은 `.correct`+`pop`, 오답은 `.wrong`+`shake` 애니메이션이다. 새 애니메이션을 추가해도 `prefers-reduced-motion` 전역 규칙으로 꺼지므로, 색과 텍스트만으로도 정오를 구분할 수 있어야 한다.
- 접근성: 선택지는 `<button>`, 진행 바는 `role="progressbar"`, 피드백 영역은 `aria-live="polite"`이다. 답을 고르면 포커스를 "다음 문제" 버튼으로 이동한다. 장식용 요소(예: HUD 상태 줄)는 `aria-hidden="true"`로 숨긴다.
- 레이아웃은 `main`의 `max-width:440px` 좁은 가운데 열이며, 본문은 15px 산세리프(`-apple-system`/`Segoe UI`/`Malgun Gothic`)다. 헤더(`.hud`)는 모노스페이스 "관제 센터" 스타일이다.
- 외부 의존성(CDN, 폰트, 이미지)을 추가하지 않는다. 오프라인 `file://`에서 동작해야 한다.
