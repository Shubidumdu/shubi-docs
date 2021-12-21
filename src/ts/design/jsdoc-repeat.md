# 문서에 타입 정보를 쓰지 않기

기본적으로 주석은 코드와 동기화되지 않습니다. 다시 말해, 열심히 주석을 작성하더라도, 그것이 최신화된 것이며, 실제로 일치할 것이라는 보장이 없습니다.
또, TS의 타입 체커가 일일이 주석을 다는 것보다 훨씬 정교하기 때문에 때문에 주석을 일일이 작성하는 것은 의미가 없습니다.
결국, 한눈에 이해가 어려운 함수에 대한 설명을 위한 용도로만 주석을 간단하게 활용하는 것이 좋습니다.

```ts
/** 애플리케이션 또는 특정 페이지의 배경색을 가져옵니다. */
function getForegroundColor(page?: string): Color {
  return page === 'login' ? {r: 127, g: 127, b: 127} : {r: 0, g: 0, b: 0};
}
```

불변성(Immutability)의 경우에도, 직접 주석으로 언급하기 보다는, 애초에 `readonly` 타입 선언으로 타입스크립트가 규칙을 강제하도록 하는 편이 더 좋습니다.

```ts
// 이건 좋은 방법이 아닙니다.
/** nums를 변경하지 않습니다! XD */
function sort(nums: number[]) { /* ... */ }

// 주석보다는 타입의 관점에서 강제하세요.
function sort(nums: readonly number[]) { /* ... */ }
```

입출력에 대한 설명을 덧붙이고 싶다면 JSDoc을 활용하세요.

```ts
/** 
 * @param {string} [page] optional.
*/
function getForegroundColor(page?: string) {
  return page === 'login' ? {r: 127, g: 127, b: 127} : {r: 0, g: 0, b: 0};
}
```

주석 뿐만 아니라 변수명에 대해서도 이를 그대로 적용할 수 있습니다. 변수명에 굳이 타입 정보를 넣을 필요는 없습니다.

```ts
let ageNum; // 이러지 말고
let age: number; // 이렇게 하세요.
```

단, 단위가 존재하는 숫자들은 예외입니다. 단위가 무엇인지 확실하지 않은 경우에는 이를 변수명에 포함하여 명확하게 해주는 것이 좋습니다.

```ts
let time = 1000 // 이것 보다는
let timeMs = 10000 // 이게 좋고

let temperature = 36.5 // 이것보다는
let temperatureC = 36.5 // 이게 좋습니다.
```
