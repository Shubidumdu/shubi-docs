# any의 진화를 이해하기

TS 상에서 일반적인 타입들은 모두 정제되기만 하는 반면, 
`noImplicitAny`가 설정된 상태에서 암시적으로 `any` 또는 `any[]` 타입으로 추정되는 변수는 **진화**(evolve)합니다. 
다시 말해, 이는 문맥에 따라 다른 타입으로 확장해나갑니다.

```ts
const result = []; // any[]
result.push('a'); // string[]
result.push(1); // (string | number)[]

let val = null; // any -> null인 경우도 암시적 any입니다.
if (Math.random() < 0.5) {
  val = /hello/; // RegExp
} else {
  val = 12; // number
}
val // number | RegExp
```

## 명시적인 any에서는 진화하지 않습니다

다만, 명시적으로 직접 `any` 타입으로 선언한 경우에는 진화가 일어나지 않습니다.

```ts
let val: any;
if (Math.random() < 0.5) {
  val = /hello/; // any
} else {
  val = 12; // any
}
val // any
```