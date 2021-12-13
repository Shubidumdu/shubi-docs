# 한꺼번에 객체 생성하기

타입스크립트의 타입은 일반적으로 변경되지 않기 때문에, 객체를 생성할 때는 속성을 하나씩 추가하기 보다는 여러 프로퍼티를 포함해 한꺼번에 생성해야 타입 추론에 유용합니다.

```ts
interface Point { x: number; y: number; }

// Don't
const pt = {};
pt.x = 3; // Property 'x' does not exist on type '{}'.
pt.y = 4; // Property 'y' does not exist on type '{}'.

// Don't
const pt: Point = {}; // Type '{}' is missing the following properties from type 'Point': x, y
pt.x = 3;
pt.y = 4;

// Do
const pt: Point = {
  x: 3,
  y: 4,
};
```

객체 전개 연산자(Spread operator) `...`를 사용하면 여러 객체들을 통해 하나의 새로운 객체를 만들어내기에 용이합니다.

```ts
interface Point { x: number; y: number; }
const pt = {x: 3, y: 4};
const id = {name: 'Pythagoras'};
const namedPoint = {...pt, ...id};
// type {
//     name: string;
//     x: number;
//     y: number;
// }
```

이를 통해 별도로 타입 명시를 하지 않고도 조건부 속성을 추론하게끔 할 수도 있습니다.
(아래 예시는 책에서 이야기한 것과는 다르게 의도한대로 추론됩니다.)

```ts
declare let hasDates: boolean;
const nameTitle = { name: 'Khufu', title: 'Pharaoh' };
const pharaoh = {
    ...nameTitle,
    ...(hasDates ? {start: -2589, end: -2566}: {}),
}
// type {
//     start?: number | undefined;
//     end?: number | undefined;
//     name: string;
//     title: string;
// }
```