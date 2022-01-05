# 부정확한 타입보다는 미완성 타입을 사용하기

**타입이 없는 것보다, 타입이 잘못된 것이 더 나쁩니다.**
정확하게 타입을 모델링할 수 없는 상황이라면, 굳이 부정확하게 모델링하지 말아야 합니다.

일반적으로 `any`와 같이 매우 추상적인 타입은 정제하는 것이 좋지만, 매번 타입이 구체적일수록 정확도가 무조건적으로 올라가지는 않습니다.

## `any`와 `unknown`은 다릅니다

`any`와 `unknown`은 둘 다 어떤 값이든 될 수 있다는 특징을 갖지만, 주요한 차이점이 하나 있습니다.

먼저, `any`는 어떤 값이든 될 수 있고, 동시에 어디에든 할당되고, 어떤 방식으로든 사용 가능합니다.
물론 이러한 특징들이 오히려 독이 되기 때문에, 앞선 아이템들에서도 줄곧 말해왔듯 지양하는 것이 좋습니다.
any를 사용하지 말아야 하는 이유에 대해서는 앞서 [아이템 5](http://localhost:3000/ts/intro/no_any.html)에서 다룬 바가 있습니다.

```ts
let anyVal: any = 'abc';

anyVal.theresNoMethodLikeThis(); // 문제 없음

const num: number = anyVal; // 문제 없음
```

반면, `unknown`은 어떤 값이든 될 수는 있지만, **단언이나 타입 체킹 없이는 다른 타입에 할당하거나 메서드 및 프로퍼티를 참조할 수 없습니다**.

```ts
let unknownVal: unknown = 'abc';

unknownVal.toUpperCase(); // ERROR: Object is of type 'unknown'.(2571)

// unknown 타입인 변수를 이용하려면 
// 1. 타입을 좁히거나
if (typeof unknownVal === 'string') unknownVal.toUpperCase();

// 2. 단언을 사용해야 합니다.
const str: string = unknownVal as string;
```
