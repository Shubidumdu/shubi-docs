# any를 구체적으로 변형해서 사용하기

`any`는 말그대로 만능 타입입니다. 무엇이든 될 수 있습니다. `any` 타입을 사용할 때는 **정말로 모든 값이 허용되어야 하는지**에 대해 검토해보아야 합니다.
거꾸로 말해, 대부분의 상황에서는 `any`보다 더 구체적으로 표현할 수 있는 타입이 존재할 가능성이 높기 때문에 가능한 더 구체적인 타입을 찾아 타입 안정성을 높이도록 해야합니다.

## 배열의 경우

```ts
// 이것보다는
function getLengthBad(array: any) {  
  return array.length;
}

// 이게 낫습니다
function getLength(array: any[]) {
  return array.length;
}
```

## 객체의 경우

```ts
// 이것보다는
function hasTwelveLetterKey(o: any) {
  // ...
}

// 이게 낫고
function hasTwelveLetterKey(o: {[key: string]: any}) {
  // ...
}

// 이것도 괜찮습니다
function hasTwelveLetterKey(o: object) {
  // ... 
}
```

위 예시에서 `{[key: string]: any}`와 `object`의 차이는 프로퍼티의 접근 가능 여부에 있습니다.

```ts
const obj: {[key: string]: any} = { a: 1 };
const obj2: object = { a: 1 };

obj.a
obj2.a // ERROR: Property 'a' does not exist on type 'object'.
```

## 함수의 경우

함수에 있어서도 단순히 `any`를 사용해서는 안 됩니다. 최소한으로나마 구체화할 수 있는 아래의 세가지 방법이 있습니다.

```ts
type Fn0 = () => any;
type Fn1 = (arg: any) => any;
type FnN = (...args: any[]) => any;
```
