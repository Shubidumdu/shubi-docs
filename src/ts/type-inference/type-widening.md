# 타입 넓히기

TS에서의 각 변수들은 정적 분석 시점에 "가능한 값"들의 집합에 해당하는 **타입**을 갖게 됩니다.
변수를 초기화할 때 타입을 직접 명시하지 않는 경우, 타입 체커가 스스로 타입을 결정하게 되죠.
다시 말해, 지정된 타입 값들을 바탕으로 할당 가능한 값들의 집합을 유출해야 한다는 의미로, TS에서는 이를 <b>넓히기(widening)</b>이라는 명칭으로 부릅니다.

타입스크립트는 명확성과 유연성 사이의 균형을 유지하려고 합니다.
그래서 별도로 타입을 명시하지 않은 경우에는, 충분히 구체적으로 타입을 추론하려 하지만, 잘못된 추론(false positive)을 할 정도로 구체적으로 수행하지는 않습니다.
가령 아래와 같은 예시를 들 수 있습니다.

```ts
interface Vector3 { x: number; y: number; z: number; }
function getComponent(vector: Vector3, axis: 'x' | 'y' | 'z') {
  return vector[axis];
}
let x = 'x'; // 이 경우 `x` 변수는 string 타입이 됩니다.
let vec = {x: 10, y: 20, z: 30};
getComponent(vec, x);
               // ~ Argument of type 'string' is not assignable to
               //   parameter of type '"x" | "y" | "z"'
```
위 예시에서, `x`는 추후 재할당될 가능성이 있으므로, `string` 타입으로 추론되며, 이에 따라, `'x' | 'y' | 'z'` 타입만 할당 가능한 `axis` 매개변수에 할당할 수 없습니다.

이를 해결할 가장 간단한 방법은 `let`이 아닌 `const`를 사용하는 것입니다.
`x` 변수는 "재할당할 수 없음"이라는 정보를 전달해줌에 따라 TS가 확신을 갖고 `x` 리터럴 타입으로 추론할 수 있게 됩니다.

## 타입 추론의 강도를 직접 제어하기

타입 추론의 강도를 직접 제어하기 위해서는 TS의 기본 동작을 재정의해야 하는데, 여기에는 세 가지 방법이 있습니다.

### 타입 명시

첫번째는 직접 변수의 타입을 명시해주는 방법입니다.

```ts
const v: { x: 1|3|5 } = {
  x: 1, // type v.x = 1|3|5
};
```

### 추가적인 문맥 제공하기

두번째는 추가적인 문맥을 제공하는 방법입니다. 아래에서 매개변수로 넘겨지는 객체의 형태는 동일하지만, 문맥에 따라 에러의 발생 여부가 다릅니다.

```ts
type User = {
  type: 'customer' | 'guest';
  name: string;
  age: number;
}

const printUser = (user: User) => console.log(user);

const user = {
  type: 'customer', // string
  name: 'alan',
  age: 21,
};
printUser(user); 
// Argument of type '{ type: string; name: string; age: number; }' is not assignable to parameter of type 'User'.
//   Types of property 'type' are incompatible.
//     Type 'string' is not assignable to type '"customer" | "guest"'.(2345)

printUser({
  type: 'customer',
  name: 'alan',
  age: 21,
}) // 문제 없음
```

### const 단언문 사용

`const` 단언문은 변수 선언에 쓰이는 `let`과 `const`와는 별개의 것이므로 혼동해서는 안됩니다.
`as const` 단언을 사용하면 TS는 최대한 좁은 타입으로 이를 추론하고자 합니다.

```ts
interface Vector3 { x: number; y: number; z: number; }
function getComponent(vector: Vector3, axis: 'x' | 'y' | 'z') {
  return vector[axis];
}
const v1 = {
  x: 1,
  y: 2,
};  // type = { x: number; y: number; }

const v2 = {
  x: 1 as const,
  y: 2,
};  // type = { x: 1; y: number; }

const v3 = {
  x: 1,
  y: 2,
} as const;  // type = { readonly x: 1; readonly y: 2; }
```

이를 배열을 튜플 타입으로 만들고자 할 때도 사용할 수 있습니다.

```ts
interface Vector3 { x: number; y: number; z: number; }
function getComponent(vector: Vector3, axis: 'x' | 'y' | 'z') {
  return vector[axis];
}
const a1 = [1, 2, 3];  // Type is number[]
const a2 = [1, 2, 3] as const;  // Type is readonly [1, 2, 3]
```