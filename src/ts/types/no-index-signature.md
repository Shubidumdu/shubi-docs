# number 인덱스 시그니처보다는 Array, 튜플, ArrayLike를 사용하기

JS에서 배열은 하나의 객체입니다. `number` 타입의 인덱스를 사용하지만, JS 내부적으로 사실 이는 문자열로 변환되어 사용된다는 특이점이 있습니다.
실제로도 배열에서의 인덱싱을 문자열로 하더라도 아무런 문제 없이 동작합니다.

```js
const arr = ['a', 'b', 'c'];
console.log(arr[0]) // 'a'
console.log(arr['1']) // 'b'
```

TS의 경우, **이와 같은 코드를 작성했을 때 에러를 발생해야 한다**라고 책에는 적혀있는데, 현 시점의 `v4.4` 이상의 TS에서는 이러한 에러가 출력되지 않습니다.

```ts
const xs = [1, 2, 3];
const x0 = xs[0];  // OK
const x1 = xs['1'];
           // 책(< v4.4)에서는 아래와 같은 에러가 출력된다고 이야기하지만, v4.4 이상에서는 그렇지 않습니다.
           // ~~~ Element implicitly has an 'any' type
           //      because index expression is not of type 'number'

function get<T>(array: T[], k: string): T {
  return array[k];
            // 이 경우는 현 시점에서도 에러가 발생합니다.
            // ~ Element implicitly has an 'any' type
            //   because index expression is not of type 'number'
}
```

## 인덱스 시그니처에 number를 사용하지 마세요.

다시 본론으로 돌아와서, `number` 타입을 통해 인덱싱을 해야하는 상황이라면, 굳이 객체에 인덱스 시그니처를 사용하기보다, Array 또는 Tuple 타입을 사용하세요.

```ts
// 굳이 이렇게 만들지 말고
type MyArray = {
  [index: number]: string;
}

const myArr: MyArray = {
  1: 'a',
  2: 'b',
}

// 그냥 Array나 Tuple을 쓰세요.
const arr: Array<string> = ['a', 'b'];
const tup: [string, string] = ['a', 'b'];
```

만약 Array 프로토타입의 프로퍼티들을 갖는 것을 원치 않는 상황이라면, `ArrayLike<T>` 타입을 사용하면 됩니다.

```ts
const tupleLike: ArrayLike<string> = {
  0: 'A',
  1: 'B',
  length: 2,
};

tupleLike[0] // 'A'
```