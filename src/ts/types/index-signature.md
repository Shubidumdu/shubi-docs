# 동적 데이터에 인덱스 시그니처 사용하기

JS의 장점을 객체 생성 문법이 간단하다는 것입니다. TS에서도 이렇게 유연한 형태로 객체 타입을 정의하고 싶다면 다음과 같이 사용하면 됩니다.

```ts
type Rocket = {[property: string]: string};

const rocket: Rocket = {
  name: 'Falcon 9',
  variant: 'v1.0',
  thrust: '4,940 kN',
};
```

여기서 사용된 `[property: string]: string`이 <b>인덱스 시그니처</b>가 되며, 다음의 세 가지 의미를 담고 있습니다.

- 키 이름(`property`): 키의 위치만 표시하며, 타입 체커에서는 실질적으로 사용하지 않습니다.
- 키 타입(`string`): `string`, `number` 또는 `symbol` 이어야하는데, 보통은 `string`을 사용합니다.
- 값 타입(`string`): 무엇이든 될 수 있습니다.

다만, 위와 같은 방식으로 타입 체크를 수행하는 경우 다음의 문제들이 발생합니다.

- 모든 키를 포함합니다. 즉, 의도와 다르게 키를 잘못 작성하더라도 에러가 발생하지 않습니다.
- 특정 키가 필요하지 않습니다. `{}`도 위의 `Rocket`타입에 유효합니다.
- 키마다 다른 타입을 가질 수 없습니다. 예를 들어 `thrust`는 `number`로도 표현될 여지가 있습니다.
- TS 언어 서비스가 아무런 도움도 주지 못합니다. (자동 완성, 도움말 등..) 무엇이든 가능하기 때문입니다.

결국, 일반적인 상황에서는 인덱스 시그니처보다 그냥 인터페이스로 타입을 정의하는 것이 더 나은 방법입니다.

```ts
interface Rocket {
  name: string;
  variant: string;
  thrust_kN: number;
}
const falconHeavy: Rocket = {
  name: 'Falcon Heavy',
  variant: 'v1',
  thrust_kN: 15_200
};
```

## 인덱스 시그니처는 현 시점에서 알 수 없는 동적인 데이터를 표현할 때 사용합니다.

아래 코드는 CSV 파일의 string을 받아 여러 개의 Row들을 가진 배열을 반환하는 함수입니다. 이 시점에서 우리는 어떤 CSV 파일이 사용될지 알 수 없기 때문에, 이러한 경우에는 인덱스 시그니처를 사용해야 합니다.

```ts
// 현 시점에서 우리는 CSV 파일의 컬럼명이 무엇이 될지 알 수 없습니다.
function parseCSV(input: string): {[columnName: string]: string}[] {
  const lines = input.split('\n');
  const [header, ...rows] = lines;
  return rows.map(rowStr => {
    const row: {[columnName: string]: string} = {};
    rowStr.split(',').forEach((cell, i) => {
      row[header[i]] = cell;
    });
    return row;
  });
}
```

물론, 모든 열의 값들에 대해서 주어지지 않을 가능성이 있습니다. 이러한 부분을 염려하여 보다 엄격한 형태로 작성하고자 한다면 다음과 같이 활용해야 합니다. 물론 그만큼 추후의 타입체킹에 신경을 써주어야 합니다.

```ts
function safeParseCSV(
  input: string
): {[columnName: string]: string | undefined}[] {
  return parseCSV(input);
}
```

만약, 반환받은 값의 형태에 대해 명확히 알고 있는 경우라면 다음과 같이 Assertion을 활용하여 강제로 타입을 변환시킬 수 있습니다.

```ts
interface ProductRow {
  productId: string;
  name: string;
  price: string;
}

let csvData: string;
const products = parseCSV(csvData) as unknown as ProductRow[];
```

## 가능한 필드가 제한적이라면 인덱스 시그니처를 쓰지 마세요.

동적인 데이터를 다룬다고 해도, 어떤 타입에 사용될 수 있는 필드가 제한되어 있는 경우라면 인덱스 시그니처를 쓰지 말아야 합니다. 너무 광범위하기 때문입니다. 이 때는 선택적 프로퍼티(Optional Property)나 유니온 타입(`|`)을 사용하는 편이 좋습니다.

```ts
interface Row1 { [column: string]: number }  // 너무 광범위
interface Row2 { a: number; b?: number; c?: number; d?: number }  // 최선
type Row3 =
    | { a: number; }
    | { a: number; b: number; }
    | { a: number; b: number; c: number;  }
    | { a: number; b: number; c: number; d: number }; // 가장 정확하지만 번거로움
```

## 키 타입에 제한두기

어떤 타입에서 사용되는 키 타입을 인덱스 시그니처로 모두 `string`로 정의하기에 너무 광범위하다면, 다음의 두 가지 대안을 생각해볼 수 있습니다.

### Record 제너릭

Record는 키 타입에 유연성을 제공하는 제너릭 타입으로, `string`의 부분 집합을 사용할 수 있습니다.

```ts
type Vec3D = Record<'x' | 'y' | 'z', number>;
// Type Vec3D = {
//   x: number;
//   y: number;
//   z: number;
// }
```

### Mapped Types (매핑된 타입)

매핑된 타입은 Record 제네릭과 동일하게 사용할 수 있고, 조건부 타입(`?`)를 통해 키마다 별도의 타입을 사용하게 할 수도 있습니다. 조건부 타입에 대해서는 아이템 50에서 다룰 예정입니다.

```ts
type Vec3D = {[k in 'x' | 'y' | 'z']: number};
// Type Vec3D = {
//   x: number;
//   y: number;
//   z: number;
// }

type ABC = {[k in 'a' | 'b' | 'c']: k extends 'b' ? string : number};
// Type ABC = {
//   a: number;
//   b: string;
//   c: number;
// }
```
