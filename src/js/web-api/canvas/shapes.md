# 도형 그리기

기본적으로 캔버스 상의 좌표 공간은 아래의 형태를 따릅니다. 좌상단을 기준으로 x, y 좌표를 판단합니다.

<img src="https://mdn.mozillademos.org/files/224/Canvas_default_grid.png" />

## 직사각형 그리기

캔버스 상에서 직사각형을 그리기는 데에는 세가지 함수가 있습니다.

1. `fillRect(x, y, width, height)`

- 색칠된 직사각형을 그립니다.

2. `strokeRect(x, y, width, height)`

- 직사각형 윤곽선을 그립니다.

3. `clearRect(x, y, width, height)`

- 직사각형 모양으로 해당 부분들을 완전히 지웁니다.

4. `rect(x, y, width, height)`

- 직사각형 모양으로 경로를 추가합니다. 이후 좌표가 해당 경로로 이동합니다.

이들을 이용해 하나의 예제를 살펴봅시다.

```js
ctx.fillRect(25, 25, 100, 100);
ctx.clearRect(45, 45, 60, 60);
ctx.strokeRect(50, 50, 50, 50);
```

<canvas id="example1" ></canvas>

<script>
const ctx1 = example1.getContext('2d');
ctx1.fillRect(25, 25, 100, 100)
ctx1.clearRect(45, 45, 60, 60);
ctx1.strokeRect(50, 50, 50, 50);
</script>

## 경로 그리기

경로는 직사각형 외의 유일한 원시형(primitive) 도형입니다. 경로를 통해 도형을 그리기 위해서는 다음의 단계를 거치게 됩니다.

1. 경로를 생성
2. 그리기 명령들을 통해 경로 상에 그립니다.
3. 그린 경로에 대한 윤곽선을 그리거나 도형 내부를 채웁니다.

이러한 단계들을 수행하게 아래의 함수들이 사용됩니다.

1. `beginPath()`

- 새로운 경로를 만듭니다.

2. [여러 Path 메서드들](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D#paths)

- 이후에 계속 살펴보겠지만, 여러 경로들을 설정하기 위해 사용됩니다.

3. `closePath()`

- 현재 경로의 시작점과 연결되는 직선을 추가합니다. 이는 옵션 사항입니다.

4. `stroke()`

- 윤곽선을 통해 도형을 그립니다.

5. `fill()`

- 경로 내부를 채워 색칠된 도형을 그립니다. 이 경우 별도로 `closePath()`를 해줄 필요가 없습니다.

이를 통해 간단한 삼각형을 그려봅시다.

```js
ctx.beginPath();
ctx.moveTo(75, 50);
ctx.lineTo(100, 75);
ctx.lineTo(100, 25);
ctx.fill();
```

<canvas id='example2' ></canvas>

<script>
const ctx2 = example2.getContext('2d');
ctx2.beginPath();
ctx2.moveTo(75, 50);
ctx2.lineTo(100, 75);
ctx2.lineTo(100, 25);
ctx2.fill();
</script>

## 펜 이동하기

- `moveTo(x, y)`
  - 이를 이용하면, 펜을 해당 좌표로 옮기기만 하고, 그리진 않습니다.

```js
ctx.beginPath();
ctx.arc(75, 75, 50, 0, Math.PI * 2, true); // Outer circle
ctx.moveTo(110, 75);
ctx.arc(75, 75, 35, 0, Math.PI, false); // Mouth (clockwise)
ctx.moveTo(65, 65);
ctx.arc(60, 65, 5, 0, Math.PI * 2, true); // Left eye
ctx.moveTo(95, 65);
ctx.arc(90, 65, 5, 0, Math.PI * 2, true); // Right eye
ctx.stroke();
```

<canvas id="example3"></canvas>

<script>
const ctx3 = example3.getContext('2d');
ctx3.beginPath();
ctx3.arc(75, 75, 50, 0, Math.PI * 2, true); // Outer circle
ctx3.moveTo(110, 75);
ctx3.arc(75, 75, 35, 0, Math.PI, false); // Mouth (clockwise)
ctx3.moveTo(65, 65);
ctx3.arc(60, 65, 5, 0, Math.PI * 2, true); // Left eye
ctx3.moveTo(95, 65);
ctx3.arc(90, 65, 5, 0, Math.PI * 2, true); // Right eye
ctx3.stroke();
</script>

## 선 그리기

- `lineTo(x, y)`
  - 이는 현재 위치에서 해당 좌표 위치까지 선을 그려냅니다.

```js
// Filled triangle
ctx.beginPath();
ctx.moveTo(25, 25);
ctx.lineTo(105, 25);
ctx.lineTo(25, 105);
ctx.fill();

// Stroked triangle
ctx.beginPath();
ctx.moveTo(125, 125);
ctx.lineTo(125, 45);
ctx.lineTo(45, 125);
ctx.closePath();
ctx.stroke();
```

<canvas id="example4"></canvas>

<script>
  const ctx4 = example4.getContext('2d');
  // Filled triangle
  ctx4.beginPath();
  ctx4.moveTo(25, 25);
  ctx4.lineTo(105, 25);
  ctx4.lineTo(25, 105);
  ctx4.fill();

  // Stroked triangle
  ctx4.beginPath();
  ctx4.moveTo(125, 125);
  ctx4.lineTo(125, 45);
  ctx4.lineTo(45, 125);
  ctx4.closePath();
  ctx4.stroke();
</script>

## 호(Arc) 또는 원 그리기

1. `arc(x, y, radius, startAngle, endAngle, anticlockwise)`

- 해당 좌표에, 반지름 `radius`를 갖도록 `startAngle` 각도에서 `endAngle`각도까지 `anticlockwise` 방향으로 호를 그려냅니다.

2. `arcTo(x1, y1, x2, y2, radius)`

- 주어진 각 제어점과 반지름으로 호를 그리고, 이전 점과 직선으로 연결합니다.
- 이에 대해서는 [여기](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/arcTo)를 살펴봅시다.

<img src=">

> **주의!**: `arc`함수에서의 각도는 degree가 아닌 radian 단위를 사용합니다. 따라서 degree 단위를 사용하려면 별도의 변환이 요구됩니다.
>
> `radians = (Math.PI/180)*degrees`

```js
const startAngle = 0;
const endAngle = (Math.PI / 180) * 90;
ctx.beginPath();
ctx.arc(120, 120, 100, startAngle, endAngle, true);
ctx.stroke();
```

<canvas id="example5" height="250"></canvas>

<script>
const ctx5 = example5.getContext('2d');
const startAngle = 0;
const endAngle = (Math.PI / 180) * 90;
ctx5.beginPath();
ctx5.arc(120, 120, 100, startAngle, endAngle, true);
ctx5.stroke();
</script>

## 베지어(Bezier) 곡선과 이차(Quadratic) 곡선

<img src="https://mdn.mozillademos.org/files/223/Canvas_curves.png" />

베지어 곡선은 주로 복잡한 형태를 그려내는데 사용됩니다.

1. `quadraticCurveTo(cp1x, cp1y, x, y)`

- `cp1x` 및 `cp1y`로 지정된 제어점을 통해 현재 펜 위치에서 `x`, `y`로 지정된 끝점까지 이차 베지어 곡선을 그립니다.

2. `bezierCurveTo(cp1x, cp1y, cp2x, cp2y, x, y)`

- 각 제어점을 통해 `x`, `y`로 지정된 끝점까지 삼차 베지어 곡선을 그립니다.

```js
ctx.beginPath();
ctx.moveTo(75, 25);
ctx.quadraticCurveTo(25, 25, 25, 62.5);
ctx.quadraticCurveTo(25, 100, 50, 100);
ctx.quadraticCurveTo(50, 120, 30, 125);
ctx.quadraticCurveTo(60, 120, 65, 100);
ctx.quadraticCurveTo(125, 100, 125, 62.5);
ctx.quadraticCurveTo(125, 25, 75, 25);
ctx.stroke();
```

<canvas id="example6"></canvas>

<script>
const ctx6 = example6.getContext('2d');
ctx6.beginPath();
ctx6.moveTo(75, 25);
ctx6.quadraticCurveTo(25, 25, 25, 62.5);
ctx6.quadraticCurveTo(25, 100, 50, 100);
ctx6.quadraticCurveTo(50, 120, 30, 125);
ctx6.quadraticCurveTo(60, 120, 65, 100);
ctx6.quadraticCurveTo(125, 100, 125, 62.5);
ctx6.quadraticCurveTo(125, 25, 75, 25);
ctx6.stroke();
</script>

```js
ctx.beginPath();
ctx.moveTo(75, 40);
ctx.bezierCurveTo(75, 37, 70, 25, 50, 25);
ctx.bezierCurveTo(20, 25, 20, 62.5, 20, 62.5);
ctx.bezierCurveTo(20, 80, 40, 102, 75, 120);
ctx.bezierCurveTo(110, 102, 130, 80, 130, 62.5);
ctx.bezierCurveTo(130, 62.5, 130, 25, 100, 25);
ctx.bezierCurveTo(85, 25, 75, 37, 75, 40);
ctx.fill();
```

<canvas id="example7"></canvas>

<script>
const ctx7 = example7.getContext('2d');
ctx7.beginPath();
ctx7.moveTo(75, 40);
ctx7.bezierCurveTo(75, 37, 70, 25, 50, 25);
ctx7.bezierCurveTo(20, 25, 20, 62.5, 20, 62.5);
ctx7.bezierCurveTo(20, 80, 40, 102, 75, 120);
ctx7.bezierCurveTo(110, 102, 130, 80, 130, 62.5);
ctx7.bezierCurveTo(130, 62.5, 130, 25, 100, 25);
ctx7.bezierCurveTo(85, 25, 75, 37, 75, 40);
ctx7.fill();
</script>

## Path2D 오브젝트

- `Path2D()`
  - `new` 키워드와 함께 사용되어 새로운 `Path2D` 객체를 반환합니다. 기존 경로 혹은 SVG 경로를 인자로 받을 수도 있습니다.

SVG 경로 데이터를 활용하는 경우, 아래와 같은 형태가 됩니다.

```js
const p = new Path2D('M10 10 h 80 v 80 h -80 Z');
```

이를 활용하면, 하나의 컨텍스트로 이리저리 옮겨가며 그리던 방식에서 벗어나, 객체의 형태로 각 경로를 변수에 저장할 수 있습니다.

```js
const rectangle = new Path2D();
rectangle.rect(10, 10, 50, 50);

const circle = new Path2D();
circle.moveTo(125, 35);
circle.arc(100, 35, 25, 0, 2 * Math.PI);

ctx.stroke(rectangle);
ctx.fill(circle);
```

<canvas id='example8'></canvas>

<script>
  const ctx8 = example8.getContext('2d');
  const rectangle = new Path2D();
  rectangle.rect(10, 10, 50, 50);

  const circle = new Path2D();
  circle.moveTo(125, 35);
  circle.arc(100, 35, 25, 0, 2 * Math.PI);

  ctx8.stroke(rectangle);
  ctx8.fill(circle);
</script>
