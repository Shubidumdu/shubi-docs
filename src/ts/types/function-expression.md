# 함수 표현식에 타입 적용하기

JS 및 TS에서는 다음의 방법들로 함수를 나타낼 수 있습니다. 이는 큰 관점에서 Statement냐 Expression이냐로 나뉩니다.

```ts
function rollDice1(sides: number): number { ... } // Statement
const rollDice2 = (sides: number): number => { ... } // Expression
const rollDice3 = function(sides: number): number { ... } // Expression
```

## TS에서는 함수 표현식(Function Expression)을 사용하세요.

결론부터 말하면, TS에서는 함수 표현식을 사용하는 것이 좋습니다.
함수 전체를 하나의 함수 타입으로 선언하여 여러 곳에 재사용할수 있다는 장점이 있기 때문입니다.

```ts
type DiceRollFn = (sides: number) => number;
const rollDice: DiceRollFn = sides => { ... };
```

위의 예시가 짧아서 장점이 와닿지 않는다면 다음의 코드도 참고하세요.

```ts
type BinaryFn = (a: number, b: number) => number;
const add: BinaryFn = (a, b) => a + b;
const sub: BinaryFn = (a, b) => a - b;
const mul: BinaryFn = (a, b) => a * b;
const div: BinaryFn = (a, b) => a / b;
```

## 함수 타입에도 typeof를 사용할 수 있습니다.

함수 타입에도 `typeof`를 사용할 수 있는데, 이는 기존에 이미 작성되어 있던 네이티브 및 라이브러리 함수들의 타입을 재차 활용하고자 할 때 유용합니다.
아래 예시는 네이티브 `fetch` 함수에 대한 에러 핸들링을 추가하는 예시입니다.

```ts
const checkedFetch: typeof fetch = async (input, init) => {
  const response = await fetch(input, init);
  if (!response.ok) {
    throw new Error('Request failed: ' + response.status);
  }
  return response;
}
```