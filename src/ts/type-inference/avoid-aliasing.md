# 일관성있는 별칭 사용하기

별칭(alias)을 남발해서 사용하면 제어 흐름을 분석하기 어렵습니다.
TS에서도 마찬가지로 별칭을 신중하게 사용해야합니다.
그래야 코드를 잘 이해할 수 있고, 오류도 쉽게 찾을 수 있기 때문입니다.

```ts
interface Coordinate {
  x: number;
  y: number;
}

interface BoundingBox {
  x: [number, number];
  y: [number, number];
}

interface Polygon {
  exterior: Coordinate[];
  holes: Coordinate[][];
  bbox?: BoundingBox;
}
```

위와 같은 자료 구조가 있고, 이에 대해 아래와 같은 함수가 있다고 가정합시다.
현 시점에서 이는 타입에러도 없고, 잘 동작하지만 코드가 반복되는 부분이 존재합니다.

```ts
function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
  if (polygon.bbox) {
    if (pt.x < polygon.bbox.x[0] || pt.x > polygon.bbox.x[1] ||
        pt.y < polygon.bbox.y[1] || pt.y > polygon.bbox.y[1]) {
      return false;
    }
  }

  // ... more complex check
}
```

여기서 중복되는 부분들을 없애기 위해 별도로 `box`라는 이름의 별칭으로 `polygon.bbox`를 참조하도록 하는 방법을 사용할 수 있습니다.

```ts
function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
  const box = polygon.bbox;
  if (box) {
    if (pt.x < box.x[0] || pt.x > box.x[1] ||
        pt.y < box.y[1] || pt.y > box.y[1]) {  // OK
      return false;
    }
  }
  // ...
}
```

사실 제일 이상적인 방법은 Destructuring(비구조화) 문법을 통해 `bbox`라는 일관된 이름을 사용하도록 하는 것입니다. 이를 적용하면 아래와 같아집니다.

```ts
function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
  // Destructuring을 통해 일관된 이름을 사용할 수 있도록 합니다.
  const { bbox } = polygon;
  if (bbox) {
    const { x, y } = bbox;
    if (pt.x < x[0] || pt.x > x[1] ||
        pt.y < x[0] || pt.y > y[1]) {
      return false;
    }
  }
  // ...
}
```

객체 프로퍼티에 직접 접근하지 않고 별도의 지역변수로 분리해낸다는 점은 타입 관점에서 더 안전합니다. 아래와 같이 프로퍼티를 직접 참조하는 경우 기존에 좁혀졌던 타입이 함수 호출 등으로 신뢰할 수 없는 상태가 될 수 있기 때문입니다.

```ts
const deletePolygonBox = (polygon: Polygon) => {
    polygon.bbox = undefined;
}

function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
    polygon.bbox  // Type is BoundingBox | undefined
    if (polygon.bbox) {
    polygon.bbox  // Type is BoundingBox
    deletePolygonBox(polygon); // polygon.bbox = undefined;
    polygon.bbox  // Type is BoundingBox
    }
}
```

단, 지역변수로 분리한 경우 기존 프로퍼티 `polygon.bbox`와 `bbox`가 항상 같음을 보장할 수 없다는 점에 주의해야합니다.

```ts
const resetPolygonBox = (polygon: Polygon) => {
    polygon.bbox = {
      x: [0, 0],
      y: [0, 0],
    };
}

function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
    const { bbox } = polygon;
    if (bbox) {
    resetPolygonBox(polygon); 
    // 이제 bbox와 polygon.bbox는 동일하지 않습니다.
    // ...
  }
}
```
