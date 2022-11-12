# 웹 성능이란 무엇인가?

## 1. 웹

### 웹의 대표적인 요소

- URL
- 네트워크 프로토콜 (대개는 HTTP)
- HTML

## 2. 웹 성능이 중요한 이유

**웹 성능**이란 *콘텐츠가 신속하게 전달되어 사용자가 원하는 서비스를 빠르게 전달받을 수 있도록 하는 시스템들의 성능*을 의미한다. (≒ 웹 로딩 시간)

> 3초의 법칙 ~ 3초 안에 웹 사이트에 접속한 사용자의 관심을 끄는 것이 필요하다. (그렇지 않으면 사용자가 이탈한다.)

## 3. 웹 성능 측정 방법

### 대표적인 서비스

- 브라우저 개발자 도구
- [WebPageTest](http://www.webpagetest.org/)
- [구글 PageSpeed](https://pagespeed.web.dev/)

## 4. 웹 성능을 만드는 지표

- 스티브 사우더스의 14가지 웹 성능 최적화 기법

<table class="tg">
<thead>
  <tr>
    <th class="tg-0pky">최적화</th>
    <th class="tg-0pky" colspan="4">내용</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td class="tg-0pky">백엔드</td>
    <td class="tg-0pky" colspan="4">1.Expires 헤더를 추가한다.<br>2. gzip으로 압축한다.<br>3. redirect를 피한다.<br>4. ETag를 설정한다.<br>5. 캐시를 지원하는 AJAX를 만든다.</td>
  </tr>
  <tr>
    <td class="tg-0pky">프런트엔드</td>
    <td class="tg-0pky" colspan="4">1. HTTP 요청을 줄인다.<br>2. 스타일 시트는 상단에 넣는다.<br>3. 스크립트는 하단에 넣는다.<br>4. CSS 표현식은 피한다.<br>5. 자바스크립트와 CSS는 외부 파일에 넣는다.<br>6. 자바스크립트는 작게 한다.<br>7. 중복 스크립트는 제거한다.</td>
  </tr>
  <tr>
    <td class="tg-0pky">네트워크</td>
    <td class="tg-0pky" colspan="4">1. 콘텐츠 전송 네트워크(CDN)을 사용한다.<br>2. DNS 조회를 줄인다.</td>
  </tr>
</tbody>
</table>

- [yslow](http://yslow.org/)

## 5. 웹 성능과 프런트엔드

대다수 웹 사이트의 웹 성능 측정 시 가장 주요한 것은 프론트엔드 영역이며, 이는 웹 성능의 측정 기준이 *사용자 관점에서 원하는 콘텐츠를 전달받았는지*가 되기 때문이다.

### 브라우저 렌더링

- FCP (First Contentful Paint) : 첫 번째 텍스트 또는 이미지가 표시되는 데 걸린 시간
- SI (Speed Index) : 페이지 콘텐츠가 얾마나 빨리 표시되는지에 대한 정보
- LCP (Largest Contentful Paint) : 가장 큰 텍스트 또는 이미지가 표시된 시간
- TTI (Time to Interactive) : 사용자와 페이지가 상호 작용할 수 있게 된 시간
- TBT (Total Blocking Time) : FCP와 TTI 사이 모든 시간의 합
- CLS (Cumulative Layout Shift) : 표시 영역 안에 보이는 요소들이 얼마나 이동하는지에 대한 정보

## 6. 웹 성능 예산

웹 성능 예산(web performance budget)이란, *웹 성능에 영향을 미치는 다양한 요소를 제어하는 한계값*을 의미한다. 웹 성능 지표를 계량할 수 있도록 수치화하여 최적화의 목표치로 삼기 위해 사용한다.

### 정량 기반 지표 (quantity based metrics)

웹 페이지 구성 요소에 대한 한계값

ex.) 이미지 파일의 최대 크기, JS 파일 크기 합...

### 시간 기반 지표 (timing based metrics ~ milestone timing)

실제로 브라우저에서 측정 가능한 시간적 수치에 대한 시간에 대한 한계값

ex.) FCP, TTI

### 규칙 기반 지표 (rule based metrics)

성능 측정 도구들을 통해 측정된 점수에 대한 한계값

ex.) WebPageTest의 성능 점수, 구글 Lighthouse의 성능 점수
