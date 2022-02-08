# 테스팅 타입의 함정에 주의하기

프로젝트를 공개하려면 테스트 코드를 작성하는 것이 필수적이며, 타입 선언 역시 이러한 테스트를 거쳐야 합니다.
그러나 타입 선언을 테스트하는 것은 실제로 상당히 어렵습니다.

다음과 같은 함수가 있다고 가정합시다.

```ts
const square = (x: number) => x * x;
```

이를 테스트하는 가장 쉬운 방법은 이를 단순히 실행해보는 것입니다.

```ts
test('square a number', () => {
  square(1);
  square(2);
});
```

이러한 테스트의 문제점은, 오직 "실행"에 대해서 오류가 발생하는지 아닌지에 대해서만 체크를 한다는 것입니다.
반환값에 대해서는 체크하지 않기 때문에, 그 결과에 대해서는 관심이 없는 셈입니다.

이러한 문제를 해결하고자 반환 결과를 체크하기 위해 별도의 변수를 둘 수도 있습니다.

```ts
declare function map<U, V>(array: U[], fn: (u: U) => V): V[];

const lengths: number[] = map(['john', 'paul'], name => name.length);
```

다만 이 경우 두 가지 문제가 발생합니다.

- **불필요한 변수**(ex. `lengths`)를 만듭니다. 이는 린팅과 관련한 경고를 유발할 수 있습니다.
- 타입이 **동일한지**가 아니라, **할당가능한지**에 대해서만 체크가 이루어집니다.

## 헬퍼 함수와 유틸 타입을 활용하세요

이 문제들을 해결하기 위한 일반적인 선택은 헬퍼 함수를 정의한 후, 유틸 타입과 함께 사용하여 Input과 Output에 대한 체크를 수행하는 것입니다.

```ts
// Helper func.
function assertType<T>(x: T) {}

type CustomMap<U, V> = (array: U[], fn: (u: U) => V) => V[];

// 매개변수 타입에 대한 테스트
const params: [number[], (n: number) => number] = null!;
assertType<Parameters<CustomMap<number, number>>>(params);

// 반환 타입에 대한 테스트
const result: number[] = null!;
assertType<ReturnType<CustomMap<number, number>>>(result);
```

## 콜백에서의 this 역시 고려되어야 합니다

콜백 함수에서 `this`를 사용하는 경우 역시 타입을 가질 수 있으므로, 이 역시 테스트 시에 체크해주어야 합니다.

```ts
declare function map<U, V>(
  array: U[],
  fn: (u: U, i: number, array: U[]) => V
): V[];

const beatles = ['john', 'paul', 'george', 'ringo'];

// ma
assertType<number[]>(map(
  beatles,
  function(name, i, array) {
    assertType<string>(name);
    assertType<number>(i);
    assertType<string[]>(array);
    assertType<string[]>(this); // this에 대해서도 타입을 체크합니다.
    return name.length;
  }
));
```

## 테스트에서 any를 주의하세요

테스트에서도 `any`는 여전히 나쁜 영향을 끼칩니다.
`any` 타입을 사용하는 경우 테스트는 전부 통과하겠지만, 타입 안정성을 포기하게 됩니다.
`noImplicitAny`를 설정하더라도, 타입 선언을 통해 여전히 `any` 타입은 생겨나게 되며, 테스트를 하는 것이 매우 어려워집니다.

이러한 어려움 때문에 타입 체커와 독립적으로 동작하는 도구를 사용해 타입 선언을 테스트하는 방법이 권장되는데, [dtslint](https://github.com/microsoft/dtslint)가 좋은 예시가 됩니다.

```ts
map(beatles, function(
  name,  // \$ExpectType string
  i,     // \$ExpectType number
  array  // \$ExpectType string[]
) {
  this   // \$ExpectType string[]
  return name.length;
});  // \$ExpectType number[]
```

dtslint는 특별한 형태의 주석을 통해 동작하며, 이는 할당 가능성 체크가 아닌, 각 심벌의 타입을 추출하여 글자가 일치하는지를 비교합니다.
우리가 에디터 상에서 변수에 커서를 올려 타입을 확인하는 과정을 자동화하는 것과 유사합니다.
물론 이 경우 미묘한 단점은 있는데, `string | number`와 `number | string`과 같은 경우 사실 상 같은 타입이지만 다른 타입으로 인식해버립니다.
