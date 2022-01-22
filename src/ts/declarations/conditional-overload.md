# 오버로딩 타입보다는 조건부 타입을 사용하기

오버로딩 타입을 사용하면 하나의 함수가 여러 타입에 대해 동작하는 경우에 대한 타입을 명시할 수 있습니다. (아이템 3)
허나 경우에 따라 해결하기가 까다로운 타입 선언 문제가 있을 수 있습니다.

```ts
function double(x: any) { return x + x; }

// 1. 유니온 타입으로 한번에 선언하는 방법
// 선언이 모호합니다. string => number 또는 number => string의 경우도 인정합니다.
function double(x: number | string): number | string;
const num = double(12);  // string | number
const str = double('x');  // string | number

// 2. 제너릭을 사용하는 방법
// 타입이 구체적이긴 하나, 틀린 타입 선언입니다.
function double<T extends number | string>(x: T): T;
const num = double(12);  // Type is 12 => WRONG: 144여야 합니다.
const str = double('x');  // Type is "x" => WRONG: "xx"여야 합니다.

// 3. 여러 번에 걸쳐 타입 선언을 하는 방법
// 타입이 비교적 명확하나, 유니온 타입과 관련해서 문제가 발생합니다.
function double(x: number): number;
function double(x: string): string;
const num = double(12);  // Type is number
const str = double('x');  // Type is string
const f = (x: number | string) => double(x); // ERROR: Argument of type 'string | number' is not assignable to parameter of type 'string'.
```

이를 해결하기 위한 가장 좋은 해결책은 <b>조건부 타입(Conditional Type)</b>을 사용하는 것입니다.
조건부 타입은 타입 공간의 `if` 구문과 같습니다.
이 경우, 이전에 문제가 발생했던 모든 경우들에 대해 정상적으로 동작합니다.

```ts

function double<T extends number | string>(
  x: T
): T extends string ? string : number;

const num = double(12);  // Type is number
const str = double('x');  // Type is string
const f = (x: number | string) => double(x); // Type is (x: string | number) => string | number;
```

조건부 타입은 개별 타입의 유니온으로 일반화하는 과정을 거치기 때문에 타입이 보다 정확해집니다.
각각의 오버로딩 타입이 독립적으로 처리되는 반면, 조건부 타입은 타입 체커가 단일 표현식으로 받아들이기 때문에 유니온 문제를 해결할 수 있습니다.

이러한 장점으로 인해, 오버로딩 타입을 작성 중이라면, 조건부 타입을 사용해서 개선할 수 있을지 검토해 보는 것이 좋습니다.
