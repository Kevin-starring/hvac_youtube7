---
name: web-presentation
description: |
  HTML/CSS/JS로 웹 슬라이드 프레젠테이션을 만드는 스킬. 외부 라이브러리 없이 순수 vanilla HTML/CSS/JS만으로 reveal.js 스타일의 슬라이드를 생성한다.
  출력되는 디스플레이에 맞춰서 반응형으로 화면이 표시되도록 구성한다.

  다음 상황에서 반드시 이 스킬을 사용하라:
  - "슬라이드 만들어줘", "슬라이드 제작해줘"
  - "프레젠테이션 만들어줘", "발표 자료 만들어줘"
  - "웹 슬라이드", "HTML 슬라이드", "웹 프레젠테이션"
  - 특정 주제로 슬라이드를 요청할 때 (예: "AI에 대한 슬라이드 만들어줘")
  - 영어로도 트리거: "make slides", "create presentation", "web presentation", "HTML slides"
---

# 웹 프레젠테이션 스킬

사용자가 슬라이드나 프레젠테이션 제작을 요청하면, 순수 vanilla HTML/CSS/JS로 단일 HTML 파일을 생성한다.

## 핵심 기능 (반드시 포함)

모든 프레젠테이션에 다음 기능을 구현한다:

1. **키보드 내비게이션** — 좌우 방향키(←→)로 슬라이드 전환, 상하 방향키(↑↓)도 동일하게 동작
2. **전체화면** — `F` 키를 누르면 전체화면 토글 (Fullscreen API 사용)
3. **슬라이드 번호 표시** — 화면 우하단에 "현재 / 전체" 형식으로 항상 표시 (예: `3 / 8`)
4. **슬라이드 전환 애니메이션 (방향 인식)** — `→` 누르면 현재 슬라이드가 왼쪽으로 나가고 다음 슬라이드가 오른쪽에서 들어온다. `←` 누르면 현재 슬라이드가 오른쪽으로 나가고 이전 슬라이드가 왼쪽에서 들어온다.
5. **슬라이드 전환** - infra_eng.md 파일에서 비디오를 재생하거나 그림파일을 보여달라고 요청할 경우 해당 내용을 하나의 슬라이드와 동일하게 인식해서 요청하는 방향으로 슬라이드를 추가

## 출력 형식

- **단일 .html 파일** 하나로 모든 것을 완성한다 (외부 CDN, 라이브러리, 이미지 파일 금지)
- CSS와 JS는 모두 `<style>`, `<script>` 태그 안에 인라인으로 포함
- 파일명은 주제를 반영해서 짓는다 (예: `ai-trends-2025.html`)

## 슬라이드 구조 패턴

```html
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>프레젠테이션 제목</title>
  <style>
    /* 전체 레이아웃 */
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body {
      font-family: 'Segoe UI', sans-serif;
      background: #1a1a2e;
      overflow: hidden;
      width: 100vw; height: 100vh;
    }

    /* 슬라이드 컨테이너 */
    .slides-container {
      width: 100%; height: 100%;
      position: relative;
    }

    /* 개별 슬라이드 */
    .slide {
      position: absolute;
      width: 100%; height: 100%;
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      padding: 60px;
      opacity: 0;
      transform: translateX(100%);
      transition: all 0.5s cubic-bezier(0.4, 0, 0.2, 1);
    }
    .slide.active {
      opacity: 1;
      transform: translateX(0);
    }
    .slide.prev {
      opacity: 0;
      transform: translateX(-100%);
    }

    /* 슬라이드 번호 */
    #slide-counter {
      position: fixed;
      bottom: 20px; right: 30px;
      color: rgba(255,255,255,0.6);
      font-size: 16px;
      font-weight: 300;
      z-index: 100;
      letter-spacing: 2px;
    }

    /* 진행 바 (선택적이지만 권장) */
    #progress-bar {
      position: fixed;
      bottom: 0; left: 0;
      height: 3px;
      background: linear-gradient(90deg, #00d2ff, #7b2ff7);
      transition: width 0.4s ease;
      z-index: 100;
    }
  </style>
</head>
<body>
  <div class="slides-container">
    <div class="slide active" id="slide-1">
      <!-- 슬라이드 내용 -->
    </div>
    <!-- 추가 슬라이드들 -->
  </div>

  <div id="slide-counter">1 / N</div>
  <div id="progress-bar"></div>

  <script>
    const slides = document.querySelectorAll('.slide');
    let current = 0;
    const total = slides.length;

    function goTo(index) {
      slides[current].classList.remove('active');
      slides[current].classList.add('prev');

      // 이전 prev 클래스 정리
      setTimeout(() => {
        document.querySelectorAll('.slide.prev').forEach(s => s.classList.remove('prev'));
      }, 500);

      current = Math.max(0, Math.min(index, total - 1));
      slides[current].classList.add('active');

      // 카운터 & 진행 바 업데이트
      document.getElementById('slide-counter').textContent = `${current + 1} / ${total}`;
      document.getElementById('progress-bar').style.width = `${((current + 1) / total) * 100}%`;
    }

    // 키보드 이벤트
    document.addEventListener('keydown', e => {
      if (e.key === 'ArrowRight' || e.key === 'ArrowDown') goTo(current + 1);
      if (e.key === 'ArrowLeft' || e.key === 'ArrowUp') goTo(current - 1);
      if (e.key === 'f' || e.key === 'F') {
        if (!document.fullscreenElement) {
          document.documentElement.requestFullscreen();
        } else {
          document.exitFullscreen();
        }
      }
    });

    // 초기 진행 바 설정
    document.getElementById('progress-bar').style.width = `${(1 / total) * 100}%`;
  </script>
</body>
</html>
```

## 슬라이드 디자인 가이드

### 레이아웃 유형
사용자의 내용에 맞게 슬라이드 레이아웃을 선택한다:
- **타이틀 슬라이드** — 큰 제목 + 부제목 + 발표자 정보 (중앙 정렬, 임팩트 있게)
- **본문 슬라이드** — 제목 + 불릿 포인트 (좌측 정렬 또는 중앙, 읽기 쉽게)
- **두 컬럼** — 텍스트와 시각 요소(코드, 비교표) 나란히
- **강조 슬라이드** — 핵심 숫자나 인용구를 크게

### 슬라이드 배경 이미지
- 첫페이지와 마지막 페이지는 presentation template 폴더내의 Silde1.jpg 파일을 사용해주고 그외의 페이지들은 Silde2.jpg파일을 화면의 배경으로 사용해줘 

### 비주얼 퀄리티
- 각 슬라이드마다 배경 변화를 주거나 CSS 그라디언트로 시각적 다양성 부여
- 슬라이드 간 색상 팔레트는 일관되게 유지 (테마 색상 2-3개 정해서 사용)
- SVG 아이콘이나 CSS로 그린 도형으로 시각 요소 추가 가능 (외부 이미지 없이)
- 코드 예시가 필요하면 스타일링된 `<pre><code>` 블록 사용

### 슬라이드 수
- 명시적으로 지정하지 않으면: 주제 깊이에 맞게 **8~15장** 정도가 적절
- 각 슬라이드는 하나의 핵심 메시지만 담는다 (텍스트 과부하 금지)

## 사용자 입력 처리 방법

| 사용자가 준 것 | 처리 방법 |
|---|---|
| 주제만 (예: "AI 트렌드") | 적절한 내용을 직접 구성해서 완성도 높은 슬라이드 작성 |
| 목차나 개요 | 각 항목을 슬라이드로 확장 |
| 텍스트 본문 | 핵심만 추출해서 슬라이드화 |
| 슬라이드 수 지정 | 그 수에 맞게 내용 조절 |
| 스타일/색상 요청 | 해당 테마로 디자인 (예: "다크 모드", "파란색 계열") |

## 완성 후 안내

HTML 파일 생성 후 사용자에게 알려줄 것:
- 파일 위치
- 키보드 단축키 요약: `← → ↑ ↓` 슬라이드 이동, `F` 전체화면
- 총 슬라이드 수
