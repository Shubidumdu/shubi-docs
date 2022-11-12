# API 주석에 TSDoc 사용하기

공개 API 이용자를 위한 주석을 덧붙이는 경우 JSDoc 형태로 작성해야 합니다.
TS 언어 서비스는 JSDoc 스타일을 지원하기 때문에 적극적으로 활용하는 것이 좋습니다.
이는 타입스크립트 관점에서 TSDoc이라 부르기도 합니다.

`@param`와 `@returns`를 추가하면 함수를 호출하는 부분에서 각 매개변수와 관련된 설명을 보여줍니다.

```ts
/**
 * Generate a greeting.
 * @param name Name of the person to greet
 * @param salutation The person's title
 * @returns A greeting formatted for human consumption.
 */
function greetFullTSDoc(name: string, title: string) {
  return `Hello ${title} ${name}`;
}
```

타입 정의에 TSDoc을 사용할 수도 있습니다.
이 경우 객체의 각 필드에 커서를 올려보면 필드 별로 설명을 볼 수 있게 됩니다.

```ts
interface Vector3D {}

/** A measurement performed at a time and place. */
interface Measurement {
  /** Where was the measurement made? */
  position: Vector3D;
  /** When was the measurement made? In seconds since epoch. */
  time: number;
  /** Observed momentum */
  momentum: Vector3D;
}

```

또, TSDoc 주석은 마크다운 형식으로 꾸며지므로 마크다운 문법을 사용할 수 있습니다.

```ts
/**
 * This _interface_ has **three** properties:
 * 1. x
 * 2. y
 * 3. z
 */
interface Vector3D {
  x: number;
  y: number;
  z: number;
}
```

주의해야 할 점으로, JSDoc의 경우 타입 정보를 명시하는 규칙(`@param {string} name`이 있지만, TS에서는 타입 정보를 코드를 통해 확인할 수 있으므로 이를 TSDoc 상에서 명시해선 안 됩니다.
