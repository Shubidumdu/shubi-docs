# 타입 연산과 제너릭 사용으로 반복 줄이기

같은 코드를 반복해서 작성하지 말라는 DRY(Don't Repeat Yourself) 원칙은 타입에 대해서도 유효합니다.

### 타입에 이름 붙이기 (Named Type)

```ts
function distanc(a: {x: number, y: number}, b: {x: number, y: number}) {
  return Math.sqrt(Math.pow(a.x - b.x, 2) + Math.pow(a.y - b.y, 2));
}
```

이를 별도의 이름을 가진 타입으로 쪼개어 작성하면 훨씬 보기 편해지고, 반복도 줄어듭니다.

```ts
interface Point2D {
  x: number;
  y: number;
}

function distance(a: Point2D, b: point2D) { 
  return Math.sqrt(Math.pow(a.x - b.x, 2) + Math.pow(a.y - b.y, 2));
}
```

이는 함수에 있어서도 마찬가지입니다.

```ts
type HTTPFunction = (url: string, opts: Options) => Promise<Response>;
const get: HTTPFunction = (url, opts) => { /* ... */ };
const post: HTTPFunction = (url, opts) => { /* ... */ };
```

### 확장

이미 작성된 타입을 활용한 확장도 반복을 줄이는 방법 중 하나입니다. 이 중 `&`을 이용하는 방법은 유니온 타입에서 확장을 하고자 하는 경우에 특히 유용한 패턴입니다. 

```ts
interface Person {
  firstName: string;
  lastName: string;
}

interface PersonWithBirthDate extends Person {
  birth: Date;
}

// 또는
interface PersonWithBirthDate = Person & { birth: Date };
```

### 타입 인덱싱

기존에 존재하던 타입의 일부를 인덱싱으로 사용할 수도 있습니다. `Pick`, `Partial` 등 아래서 추가로 설명할 제너릭 타입들을 사용하면 더 쉽습니다.

```ts

interface State {
  userId: string;
  pageTitle: string;
  recentFiles: string[];
  pageContents: string;
}

type TopNavState = {
  userId: State['userId'];
  pageTitle: State['pageTitle'];
  recentFiles: State['recentFiles'];
};
```

단, 여전히 `State[...]`와 같이 반복되는 코드가 남아있는 데, 이부분에서 [<b>매핑된 타입</b>](https://www.typescriptlang.org/docs/handbook/2/mapped-types.html)을 사용하면 더 나아집니다.

```ts
type TopNavState = {
  [k in 'userId' | 'pageTitle' | 'recentFiles']: State[k]
};
```


유니온 타입에서 인덱싱을 사용하는 경우에도 유연하게 동작합니다.

```ts
interface SaveAction {
  type: 'save';
  // ...
}
interface LoadAction {
  type: 'load';
  // ...
}
type Action = SaveAction | LoadAction;
type ActionType = Action['type'];  // Type is "save" | "load"
```

### `typeof`와 `keyof`

`keyof`는 특정 타입이 가진 프로퍼티 `key`들의 유니온을 반환합니다.

```ts
interface Options {
  width: number;
  height: number;
  color: string;
  label: string;
}
type OptionsKeys = keyof Options;
// Type is "width" | "height" | "color" | "label"
```

반면, `typeof`는 특정 <b>값(value)</b>의 형태에 대한 타입들을 가져올 수 있습니다. 여기서 `typeof` 뒤에 오는 것은 타입이 아닌 값임에 주의하세요.

```ts
const INIT_OPTIONS = {
  width: 640,
  height: 480,
  color: '#00FF00',
  label: 'VGA',
};

type Options = typeof INIT_OPTIONS;

// interface Options {
//   width: number;
//   height: number;
//   color: string;
//   label: string;
// }
```

## 표준 라이브러리의 제너릭을 활용하세요. 

제너릭은 **타입의 관점에서 사용하는 함수**에 가깝습니다. 정의 시점에는 해당 타입이 명확하지 않지만, 이를 사용할 때 결과 타입을 반환받아 사용할 수 있게 됩니다. TS 표준 라이브러리에서는 [Utility Types](https://www.typescriptlang.org/docs/handbook/utility-types.html)라는 이름으로 여러 제너릭을 제공하고 있습니다.

### Partial

`Partial`는 기존 타입들의 프로퍼티를 선택적인 속성으로 만들어줍니다.

```ts
interface Todo {
  title: string;
  description: string;
}

type UpdateTodoFields = Partial<Todo>; 

// interface UpdateTodoFields {
//  title?: string | undefined;
//  description?: string | undefined;
// }
```

### Pick

`Pick`은 기존에 존재하던 타입 프로퍼티의 일부만을 가져와 새로 정의할 수 있도록 해줍니다.

```ts
interface State {
  userId: string;
  pageTitle: string;
  recentFiles: string[];
  pageContents: string;
}

type TopNavState = Pick<State, 'userId' | 'pageTitle' | 'recentFiles'>;

// interface State {
//   userId: string;
//   pageTitle: string;
//   recentFiles: string[];
// }
```

### ReturnType

`ReturnType`은 기존의 함수 타입의 반환 타입을 가져올 수 있게 해주는 제너릭입니다.

```ts
// 아래 getUserInfo는 함수 값(value)입니다.
type UserInfo = ReturnType<typeof getUserInfo>;
```

그 외에도 여러 유틸리티 타입들이 존재하는데, 여기선 모두 다루진 않도록 하겠습니다. 다른 유틸리티 함수에 대해서는 [여기](https://www.typescriptlang.org/docs/handbook/utility-types.html)를 참조하세요.

## 제너릭의 매개변수를 `extends`를 통해 제한하세요.

TS 함수에서 매개변수의 값을 타입을 통해 제한하는 것처럼, TS의 제너릭에 있어서도 `extends`를 통해 타입을 제한할 수 있습니다. 

```ts
interface Name {
  first: string;
  last: string;
}
type DancingDuo<T extends Name> = [T, T];

const couple1: DancingDuo<Name> = [
  {first: 'Fred', last: 'Astaire'},
  {first: 'Ginger', last: 'Rogers'}
];  // OK
const couple2: DancingDuo<{first: string}> = [
                       // ~~~~~~~~~~~~~~~
                       // Property 'last' is missing in type
                       // '{ first: string; }' but required in type 'Name'
  {first: 'Sonny'},
  {first: 'Cher'}
];
```