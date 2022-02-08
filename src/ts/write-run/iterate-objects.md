# 객체를 순회하는 노하우

TS는 특정 타입에 **할당 가능한** 모든 경우에 대해 고려되기 때문에, 이 때 단순히 `for` 키워드를 통해 객체 타입을 순회하려고 하면 타입이 예상치 못하게 추론되는 경우가 있을 수 있습니다.

```ts
interface ABC {
  a: string;
  b: string;
  c: number;
}

function foo(abc: ABC) {
  for (const k in abc) {  // const k: string 
                          // 'a' | 'b' | 'c' 가 아닙니다.
    const v = abc[k];
           // ~~~~~~ Element implicitly has an 'any' type
           //        because type 'ABC' has no index signature
  }
}
```

가령, 위의 매개변수 `abc`에는 인터페이스 `ABC`의 타입만 충족한다면 그 외에 추가적으로 어떤 속성을 갖더라도 타입 상으로 문제가 없기 때문에, 이는 정상적인 오류의 출력입니다.

만약, 이러한 타입을 좁히려면 `keyof` 키워드를 사용하면 됩니다.
단, 이 경우 실제로 해당 객체가 추가적인 키와 값을 가질 가능성이 없는지 판단해야할 필요가 있습니다.

```ts
function foo(abc: ABC) {
  let k: keyof ABC;
  for (k in abc) {  // let k: "a" | "b" | "c"
    const v = abc[k];  // Type is string | number
  }
}
```

타입에 신경쓰지 않고 단순히 객체의 키와 값을 순회하고자 한다면 `Object.entries`의 사용이 보다 일반적입니다.
물론 이 경우 키와 값의 타입이 추상적이기 때문에, 타입을 다루기에 까다롭습니다.

```ts
function foo(abc: ABC) {
  for (const [k, v] of Object.entries(abc)) {
    k  // Type is string
    v  // Type is any
  }
}
```
