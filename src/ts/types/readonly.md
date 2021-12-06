# 변경 관련된 오류 방지를 위해 readonly 사용하기

## readonly

Array 또는 Tuple 타입을 `readonly`로 선언하면 다음과 같은 일이 생깁니다.

- `length`를 포함한 해당 배열 요소들을 참조할 수는 있지만, 추가로 작성하거나 수정할 수는 없습니다.
- 배열에 변경을 가하는 `pop`, `push` 등의 메서드를 사용할 수 없습니다. (한편 `map`, `concat` 등은 가능합니다.)

이는 함수의 매개변수 또는 새로운 값을 선언할 때 배열의 불변성(Immutability)을 명시적으로 유지하고자 하는 경우에 활용될 수 있습니다.
이 경우, 어떤 동작에서 해당 배열을 변경하려고 하는 경우 즉각적으로 피드백을 받을 수 있어 즉각적으로 대처가 가능합니다.

```ts
function arraySum(arr: readonly number[]) {
  let sum = 0, num;
  while ((num = arr.pop()) !== undefined) {
                 // ~~~ 'pop' does not exist on type 'readonly number[]'
    sum += num;
  }
  return sum;
}
```

### readonly는 얕게(shallow) 동작한다는 점을 유의하세요.

`readonly`는 얕게 동작합니다. 다시 말해, 아래 예시와 같이 보다 깊게 위치한 배열에 대해서는 불변성을 보장할 수 없습니다.

```ts
const arrInArr: readonly string[][] = [[], ['a', 'b']];
arrInArr[1]?.pop(); // 문제 없음
```

### 인덱스 시그니처에서도 사용할 수 있습니다.

인덱스 시그니처에서도 `readonly`를 사용할 수 있는데, 이 경우 객체의 프로퍼티가 변경되는 것을 방지할 수 있습니다.

```ts
let obj: {readonly [k: string]: number} = {};
// Or Readonly<{[k: string]: number}
obj.hi = 45;
//  ~~ Index signature in type ... only permits reading
obj = {...obj, hi: 12};  // OK
obj = {...obj, bye: 34};  // OK
```