# 타입 공간과 값 공간의 심벌 구분하기

타입스크립트의 심벌(symbol)은 타입 공간이나 값 공간 중 한 곳에 존재합니다. 
이름이 같은 심벌이더라도 속하는 공간에 따라 서로 다른 것을 의미할 수 있기 때문에 혼란스러울 수 있습니다.

```ts
interface Cylinder {
  radius: number;
  height: number;
}

const Cylinder = (radius: number, height: number) => ({radius, height});
```

작성한 코드에 따라, 이후 상황에 따라서 `Cylinder`란 심벌은 값으로 쓰일 수도, 타입으로 쓰일 수도 있습니다.
이런 경우, 추후에 혼란을 일으킬 여지가 많습니다.

초기에 타입 공간과 값 공간, 각각에 대한 개념을 잡고자 한다면 [TS Playground](https://www.typescriptlang.org/play)를 활용해보세요.
TS가 JS로 컴파일링된 이후에도 심벌이 남아있다면 값일테고, 그렇지 않다면 타입일 겁니다.

## 타입과 값 구분하기

TS 코드 상에서 타입과 값 심벌은 번갈아 나올 수 있습니다. 
일반적으로 타입선언(`:`) 또는 단언문(`as`) 다음 나오는 심벌은 타입인 반면, `=` 다음 나오는 모든 심벌은 값이 됩니다.

## 타입과 값 모두로 사용될 수 있는 경우

### class

한편, `class`는 타입과 값 모두로 사용될 수 있습니다.

```ts
class Cylinder {
  radius = 1;
  height = 1;
}

// 여기서 Cylinder는 인터페이스로 사용되었습니다.
interface NamedCylinder extends Cylinder {
    name: string;
}

const namedCylinder: NamedCylinder = {
    radius: 1,
    height: 2,
    name: 'alan',
}

// 여기서 Cylinder는 생성자로 사용되었습니다.
const cylinder = new Cylinder();
```

클래스는 타입으로 쓰일 때 인터페이스로 사용되는 반면, 값으로 쓰일 때 생성자로 사용됩니다.

### typeof

연산자 `typeof`도 타입과 값 모두에서 사용될 수 있는데, 이는 비슷하면서도 상당히 다릅니다.

- 타입의 관점에서 `typeof`는 TS 상에서의 타입을 반환합니다.
- 값의 관점에서 `typeof`는 JS 런타임의 연산자가 됩니다.

위쪽의 예시의 연장선을 통해 이를 살펴보면 아래와 같습니다.

```ts
const v = typeof Cylinder; // v는 'function' string value입니다.
type T = typeof Cylinder; // T는 Cylinder 생성자 함수의 Type입니다.
```

여기서 흥미로운 것은, 아래에 있는 타입 관점의 `typeof Cylinder`는 인스턴스의 타입이 아닌, 생성자 함수의 타입이 된다는 점입니다.
만약 이것을 인스턴스 타입으로 활용하고자 한다면 아래와 같이 전환해야 합니다.

```ts
type T = typeof InstanceType<typeof Cylindar>;
```

### 타입 프로퍼티 접근자 `[]`

프로퍼티 접근자인 `[]`는 타입으로 쓰일 때에도 동일하게 동작합니다. 
하지만, `obj['field']`와 `obj.field`는 값이 동일하더라도 다른 타입을 가질 수 있기 때문에, 타입의 프로퍼티를 얻고자 한다면 반드시 첫 번째 방법을 사용해야 합니다.

```ts
const myName: NamedCylinder['name'] = 'alan';
```

이에 대한 내용은 아이템 14에서 더 자세히 다룹니다.

### 그 외에 두 공간 사이에서 다른 의미를 가지는 코드 패턴
- `this` 
  - 값으로 쓰일 때는 JS의 `this` 키워드
  - 타입으로 쓰일 때는 **다형성 this**라고 불리는 TS 타입. 서브클래스의 메서드 체인을 구현할 때 유용합니다.
- `&`, `|`
  - 값으로 쓰일 때는 AND와 OR 비트연산
  - 타입으로 쓰일 때는 intersection과 union입니다.
- `const`
  - `const`는 새 변수를 선언하는 키워드이지만,
  - `as const`는 리터럴 또는 리터럴 표현식의 추론된 타입을 바꿉니다.
- `extends`
  - JS에서 그렇듯 서브클래스를 정의하는 데 사용되거나
  - TS 상에서 서브타입(`interface A extends B`) 또는 제너릭 타입의 한정자(`Generic<T extends number>`)를 정의할 수 있습니다.

이렇듯, 타입 공간과 값 공간에서 동일한 키워드로 사용되는 심벌들이 여럿 존재합니다. 그렇기 때문에 TS 코드가 본인이 의도한 대로 동작하지 않는다면, 타입 공간과 값 공간을 혼동하여 잘못 작성했을 가능성이 큽니다.

