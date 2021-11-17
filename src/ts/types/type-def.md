# 타입 단언보다는 타입 선언을 사용하기

타입스크립트에서 변수에 값을 할당하고 타입을 부여하는 방법은 두 가지 입니다.

```ts
interface Person { name: string };
const alice: Person = { name: 'Alice' }; // Type Declaration -> 타입 선언
const bob = { name: 'Bob' } as Person; // Type Assertion -> 타입 단언
```

위의 둘은 비슷하면서도 다릅니다.

결론부터 말하자면, 우리는 Assertion 보다는 Declaration을 사용하는 편이 좋습니다. 
Assertion은 타입을 강제로 지정하여 타입 체커가 이로부터 비롯된 오류를 무시하게끔 만들기 때문입니다.
대부분의 상황에서는 안전성 체크까지 되는 Declaration을 사용하는 것이 맞습니다.

### 언제 Assertion을 쓸까요?

Assertion은 타입 체커가 추론한 타입보다 우리가 생각하는 타입이 더 정확하다고 판단되는 경우에 사용되어야 합니다.

가령, 다음과 같은 상황입니다.

```ts
document.querySelector('#myButton').addEventListener('click', e => {
  e.currentTarget; // EventTarget
  const button = e.currentTarget as HTMLButtonElement;
  button; // HTMLButtonElement
})
```

TS는 DOM에 접근할 수 없기 떄문에, `#myButton` id를 가진 요소가 버튼에 해당할 것이라는 것을 예상하지 못합니다.
이러한 상황에서는 직접 Assertion을 통해 타입을 지정해주는 것이 필요합니다.

이와 유사한 것으로, `!`를 통한 Assertion도 활용할 수 있습니다.

```ts
const elNull = document.getElementById('foo'); // HTMLElement | null
const el = document.getElementById('foo')!; // HTMLElement
```

`!`는 `null`이 아님을 확신하는 단언문입니다.
이 역시 특정 상황에서 해당 값은 `null`이 아니라고 확신을 할 수 있을 때 사용하여야 합니다.

덧붙여, Assertion은 단언하고자 하는 타입이 타입체커가 추론한 타입의 서브타입에 해당하지 않는다면 사용할 수 없습니다.
이를 무시하고 타입을 변환하고자 한다면 `unknown` 타입을 활용하면 됩니다.

```ts
interface Person { name: string; }
const body = document.body;
const el = body as unknown as Person; 
```

`unknown`은 모든 타입의 서브타입이기 때문에, 어떤 타입으로도 변환할 수 있습니다.
하지만 주의해야 합니다. `unknown` 타입을 사용한 이상 무엇인가 위험한 동작을 하고 있다는 뜻이니까요.