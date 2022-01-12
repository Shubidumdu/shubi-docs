# any 타입은 가능한 한 좁은 범위에서만 사용하기

## any는 선언보다 단언을 활용하기

```ts
// 이것보다는
const x: any = expressionReturningFoo();
processBar(x);

// 이게 낫습니다.
const x = expressionReturningFoo();
processBar(x as any);
```

위에서 앞선 예시의 문제는 `any` 타입으로 선언된 `x`를 이용함에 따라 다른 코드에도 지속적으로 영향을 미치기 때문입니다.
반면, 뒤쪽 예시의 경우는 `processBar`의 호출에 대해서만 x를 `any` 타입으로 단언한 것이므로 다른 코드에는 영향을 미치지 않습니다.

## 가능한 좁게 any 타입을 사용하기

특정 객체의 한 프로퍼티가 타입 에러를 갖는 상황에 `any`를 사용해야만 한다면, 객체 전체를 `any`로 단언하기 보다는 해당 속성만 `any`로 단언하는 것이 좋습니다.

```ts
// 이것보다는 
// Type is any
const config: Config = {
  a: 1,
  b: 2,
  c: {
    key: value
  }
} as any;

// 이게 낫습니다.
// Type is { a: number, b: number, c: { key: any } }
const config: Config = {
  a: 1,
  b: 2,
  c: {
    key: value as any
  }
};
```
