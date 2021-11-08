# 구조적 타이핑에 익숙해지기

JS는 본질적으로 덕 타이핑(Duck Typing) 기반입니다. 어떤 함수의 매개변수 값이 모두 제대로 주어진다면, 그 값이 어떻게 만들어졌는지 신경 쓰지 않고 사용합니다.

TS도 이러한 특징을 그대로 고려하고 있습니다.

```ts
interface Vector2D {
  x: number;
  y: number;
}

interface NamedVector {
  name: string;
  x: number;
  y: number;
}

function calculateLength(v: Vector2D) {
  return Math.sqrt(v.x * v.x + v.y * v.y);
}
```

이와 같이 정의를 한 상황에서, 아래와 같이 작성을 하더라도 아무런 문제가 없습니다.

```ts
const v: NamedVector = { x: 3, y: 4, name: 'Zee' };
calculateLength(v); // 5
```

여기서 유의할 점은, `NamedVector`와 `Vector2D`의 관계에 대해서는 전혀 선언한 바가 없다는 것입니다. 기본적으로 TS의 타입 시스템은 JS의 런타임 동작을 모델링합니다. `NamedVector`의 구조가 `Vector2D`와 호환되기 때문에, 위의 코드는 정상으로 간주됩니다.
이렇듯 JS의 덕 타이핑을 모델링하기 위해 TS가 활용하는 타이핑 체계를 **구조적 타이핑(Structural Typing)**이라고 합니다.

이러한 특징이 오히려 문제를 일으키는 경우도 있을 수 있습니다. 하지만 이것이 좋든, 싫든 간에 TS에서 모든 타입은 열려(Open)있고, 봉인(Sealed)되어 있지 않습니다.

다만, 이러한 특징은 테스트에서는 오히려 도움이 됩니다. 특정 함수가 받는 매개변수의 인터페이스 규격에 맞기만 한다면, 이를 작성하고 테스트하는데 문제가 없기 때문이죠.

