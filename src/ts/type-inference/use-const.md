# 다른 타입에는 다른 변수 사용하기

JS는 `let`을 통해 하나의 변수를 여러 타입으로 수번에 걸쳐 재할당하여 사용해도 무방합니다. 
타입스크립트 상에서도 이를 시스템 상으로 막고있지는 않지만, 에러가 발생할 여지가 많습니다.

```ts
function fetchProduct(id: string) {}
function fetchProductBySerialNumber(id: number) {}

let id = "12-34-56"; // 이건 string
fetchProduct(id);

id = 123456; // number로 재할당하려니 에러가 발생
// ~~ '123456' is not assignable to type 'string'.
fetchProductBySerialNumber(id);
                        // ~~ Argument of type 'string' is not assignable to
                        //    parameter of type 'number'
```

근본적인 이유는 값은 재할당되지만, 타입은 바뀌지 않기 때문인데, 이는 타입체커는 물론, 협업을 하는 동료에게도 혼란을 주기 쉽습니다.
기본적으로 TS에서는 **하나의 타입에 대해 하나의 변수**를 사용하는 것이 이상적인데, 그 이유는 다음과 같습니다.

- 서로 관련 없는 두 개의 값을 분리합니다.
- 변수명을 더 구체적으로 지을 수 있습니다.
- 타입 추론을 향상시키며, 불필요한 타입 구문을 줄일 수 있습니다..
- 타입이 더 간결해집니다. (`string | number`를 쪼개서 `string`, `number`로 따로 쓰는 쪽을 권장)
- `let` 대신에 `const` 변수를 선언하게 됩니다.
  - `const`로 변수를 선언하면 코드가 간결해지고, 타입 체커가 타입을 추론하기에도 좋습니다.

앞선 예시의 "재사용되는 변수"와 아래 예시의 "가려지는(shadowed) 변수"는 엄연히 다릅니다.
아래의 경우는 TS 상에서 문제없이 동작하겠지만, 여전히 다른 동료 개발자들에게 혼란을 주기 쉽습니다.
실제 이러한 이유로 별도의 린팅 규칙을 통해 스타일 규칙으로 이를 막는 개발팀도 많습니다.

```ts
function fetchProduct(id: string) {}
function fetchProductBySerialNumber(id: number) {}
const id = "12-34-56";
fetchProduct(id);

{
  const id = 123456;  // OK
  fetchProductBySerialNumber(id);  // OK
}
```