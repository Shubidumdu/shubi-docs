# Introduction

Canvas API에 대한 정리는 [MDN](https://developer.mozilla.org/ko/docs/Web/API/Canvas_API/Tutorial/Basic_usage#%3Ccanvas%3E_%EC%9A%94%EC%86%8C)의 문서를 쭉 따라갈 예정입니다.

먼저 `<canvas>` 태그의 형태에 대해 살펴봅시다.

```html
<canvas id="tutorial" width="150" height="150"></canvas>
```

`width`와 `height` 어트리뷰트를 지정하지 않는 경우, 캔버스의 최초 너비는 **300px**이고, 높이는 **150px**이 됩니다. 해당 요소는 CSS에 의해 임의로 크기가 변경될 수 있으나, 비율이 고려되지 않는 경우 왜곡되어 보입니다.

> 노트: 만약 렌더링이 왜곡된 것처럼 보인다면, CSS를 사용하지 않고, 직접 `<canvas>` 태그의 `width`와 `height` 어트리뷰트를 지정하는 것이 좋습니다.

## 대체 콘텐츠

`<canvas>` 태그 안에 콘텐츠가 삽입된 경우, `<canvas>` 태그를 지원하지 않는 브라우저에 대해서는 해당 콘텐츠를 보여줍니다. 브라우저가 `<canvas>`태그를 지원하는 경우, 이는 무시됩니다. 참고로, `<canvas>`는, 이러한 방식으로 인해 반드시 닫는 태그가 필요합니다.

```html
<canvas id="stockGraph" width="150" height="150">
  current stock price: $3.15 +0.15
</canvas>

<canvas id="clock" width="150" height="150">
  <img src="images/clock.png" width="150" height="150" alt="" />
</canvas>
```

## Rendering Context

캔버스는 최초에 비어있으며, 어떤 것을 표시하기 위해 스크립트를 통해 렌더링 컨텍스트에 접근하여, 이를 그려내야 합니다.

```js
const canvas = document.getElementById('tutorial');
const ctx = canvas.getContext('2d');
```

## 기본 예제

간단한 직사각형 두개를 그려낸 예제를 살펴보겠습니다. 현재는 아래를 통해 대략적인 형태에 대해서만 이해하면 됩니다.

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <script type="application/javascript">
      function draw() {
        const canvas = document.getElementById('canvas');
        if (canvas.getContext) {
          const ctx = canvas.getContext('2d');
          ctx.fillStyle = 'rgb(200,0,0)';
          ctx.fillRect(10, 10, 50, 50);
          ctx.fillStyle = 'rgba(0, 0, 200, 0.5)';
          ctx.fillRect(30, 30, 50, 50);
        }
      }
    </script>
  </head>
  <body onload="draw();">
    <canvas id="canvas" width="150" height="150"></canvas>
  </body>
</html>
```

<canvas id="canvas"></canvas>

<script>
  const ctx = canvas.getContext('2d');
  ctx.fillStyle = 'rgb(200,0,0)';
  ctx.fillRect(10, 10, 50, 50);
  ctx.fillStyle = 'rgba(0, 0, 200, 0.5)';
  ctx.fillRect(30, 30, 50, 50);
</script>
