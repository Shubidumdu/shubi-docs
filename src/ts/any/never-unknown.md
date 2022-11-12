# 모르는 타입의 값에는 any 대신 unknown을 사용하기

`any`, `unknown`, `never` 타입은 각각 다음의 특징을 갖고 있습니다.

- `any`는
  - 어떠한 타입이든 `any` 타입에 할당 가능하고
  - `any` 타입 본인도 어떤 타입에든 할당 가능합니다.
- `unknown`은
  - 어떠한 타입이든 `unknown` 타입에 할당 가능하지만
  - `unknown` 타입은 오직 `any`와 `unknown`에만 할당 가능합니다.
- `never`는
  - 어떠한 타입도 `never` 타입에 할당할 수 없지만
  - `never` 타입은 어디에든 할당할 수 있습니다.

이를 코드로 간단하게 살펴보면 다음과 같습니다.

```ts
let anyVal: any;
let unknownVal: unknown;
let neverVal: never;

anyVal = '123'; // 무엇이든 할당 가능하고
const stringVal: string = anyVal; // 어디에든 할당 가능합니다.

unknownVal = 123; // 무엇이든 할당 가능하지만
const numVal: number = unknownVal; // 어디든 할당할 수는 없습니다. (any와 unknown에만 가능)
// ERROR: Type 'unknown' is not assignable to type 'number'.(2322)

// ERROR: Type 'string' is not assignable to type 'never'.(2322)
neverVal = '123'; // 어떤 타입도 할당할 수 없지만
const stringVal2: string = neverVal; // 어디든 할당할 수는 있습니다.
```

## unknown 타입으로 타입 변환을 강제하세요

기본적으로 `unknown` 타입은 그대로 사용할 수가 없는 상태이기 때문에, 별도의 **타입 단언**이나 **타입 좁히기**가 이루어져야 합니다.

```ts
// 타입 단언
interface Book {
  name: string;
  author: string;
}

function safeParseYAML(yaml: string): unknown {
  return parseYAML(yaml);
}

const book = safeParseYAML(`
  name: Villette
  author: Charlotte Brontë
`) as Book; // 이제 이건 Book 타입입니다.

// 타입 좁히기
function isBook(val: unknown): val is Book {
  return (
      typeof(val) === 'object' && val !== null &&
      'name' in val && 'author' in val
  );
}

function processValue(val: unknown) {
  if (isBook(val)) {
    val;  // Type is Book
  }
}
```

## 제너릭 대신에 `unknown`을 이용할 수 있는 경우, `unknown`이 낫습니다

제너릭을 사용한 스타일은 타입 단언문과 달라 보이지만, 기능은 동일합니다.
제너릭보다는 `unknown`을 반환하고 사용자가 직접 단언문을 사용하거나 원하는 대로 타입을 좁히도록 강제하는 것이 좋습니다.

```ts
// 이것보다는
function safeParseYAML<T>(yaml: string): T {
  return parseYAML(yaml);
}

safeParseYAML<Book>(`
  name: Villette
  author: Charlotte Brontë`
);

// unknown이 낫습니다.
function safeParseYAML(yaml: string): unknown {
  return parseYAML(yaml);
}

const book = safeParseYAML(`
  name: Villette
  author: Charlotte Brontë
`) as Book;
```

## 단언의 경우에도 any보다는 unknown을 사용하세요

```ts
declare const foo: Foo;
let barAny = foo as any as Bar;
let barUnk = foo as unknown as Bar;
```

기능적으로 위의 `barAny`와 `barUnk`는 동일하지만, 추후 리팩토링을 염두한다면 `unknown`을 사용하는 쪽이 더 안전합니다.
`any`의 경우 분리되는 순간 그 영향력이 널리 퍼지게되지만, `unknown`의 경우 그 시점에 즉시 에러를 발생시키므로 더 안전합니다.

## {}와 object

`unknown` 타입과 유사한 두 가지 타입이 있습니다.

- `{}` 타입은 `null`과 `undefined`를 제외한 모든 값이 될 수 있습니다.
- `object` 타입은 모든 비기본형(non-primitive) 타입으로 이루어집니다.

`unknown` 타입이 생겨나기 전에는 `{}`가 일반적으로 사용되었으나, 최근에는 이를 이용하는 경우가 드뭅니다.
정말로 `null`과 `undefined`가 불가능하다고 판단되는 경우에만 `unknown` 대신 `{}`를 사용하면 됩니다.

```ts
let bracket: {};
bracket = [];
bracket = {
    a: 1,
    b: 2,
}
bracket = 1;
bracket = undefined;
bracket = null;

let obj: object;
obj = [];
obj = {
    a: 1,
    b: 2,
}
obj = 1;
obj = 'a';
obj = undefined;
obj = null;
```
