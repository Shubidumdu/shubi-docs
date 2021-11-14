# 타입이 값들의 집합이라고 생각하기

타입스크립트에서의 타입은 **할당 가능한 값들의 집합**이라고 생각하면 이해가 쉽습니다.

## never

가장 작은 집합은 아무 값도 포함하지 않는 공집합이며, TS 상에서는 `never`가 됩니다. 여기에는 아무런 값도 할당할 수 없습니다.

```ts
const x: never = 12; // ERROR: '12' 형식은 never 타입에 할당할 수 없습니다.
```

## literal

그 다음 작은 집합은 한 가지 값만 포함하는 타입입니다. 이들은 TS 상에서 유닛(unit) 타입이라고도 불리는 리터럴(literal) 타입입니다.

```ts
type A = 'A';
type B = 'B';
type Twelve = 12;
```

## union

가능한 타입을 여러 개로 묶은 것을 유니온(union) `|` 타입이라고 합니다.

```ts
type AB = 'A' | 'B';
type AB12 = 'A' | 'B' | 12;
```

## intersection

인터섹션(intersection) `&` 타입은 각 인터페이스에 해당하는 모든 프로퍼티를 갖고 있어야 함을 의미합니다.

```ts
interface Person {
  name: string;
}

interface Lifespan {
  birth: Date;
  death?: Date;
}

type PersonSpan = Person & Lifespan;
```

## extends

다만 좀 더 일반적으로는 `extends` 키워드를 사용합니다. 타입은 일종의 집합이라는 관점에서, `extends`는 곧 *~의 부분집합* 이라는 의미로 이해할 수 있습니다.

```ts
interface Person {
  name: string;
}

interface PersonSpan extends Person {
  birth: Date;
  death?: Date;
}
```

`extends` 키워드를 제네릭 타입에서 한정자로 쓰이기도 하는데, 이 때도 *~의 부분집합*이라는 의미가 됩니다.

```ts
// 제네릭 타입 K는 string의 부분집합에 해당해야 합니다.
function getKey<K extends string>(val: any, key: K) {
  // ...
}

getKey({}, 'x'); // 정상
getKey({}, 12); // 12는 number이기 때문에 에러
```

## 타입스크립트에서의 타입은 상속보다는 집합으로 이해하는 것이 편합니다.


결국, "TS 상에서 어떤 값을 할당할 수 있느냐?"라는 것은 해당 변수가 요구하는 타입 집합에 할당하고자 하는 값의 타입 집합이 부분 집합으로 속하느냐를 판단하는 것입니다. 이건 앞선 아이템4인 "구조적 타이핑"에서 설명했던 바와 유사합니다. 

집합의 관점에서 위의 타입 시스템들을 이해한다면 아래와 같은 코드들도 쉽게 이해하고 작성할 수 있을겁니다.

```ts
interface Point {
  x: number;
  y: number;
}

type PointKeys = keyof Point; // 'x' | 'y'

// 제네릭 타입을 사용한 함수의 경우, 이를 사용하는 시점에야 구체적인 타입이 확정됩니다.
function sortBy<K extends keyof T, T>(vals: T[], key: K): T[] {
  // ...
}

const points = [{x: 1, y: 1}, {x: 2, y: -2}];

// type T = { x: number; y: number };
// type K = 'x' | 'y';
sortBy(points, 'y');
```