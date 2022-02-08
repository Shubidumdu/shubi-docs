# 타입스크립트 기능보다는 ECMASCript 기능을 사용하기

JS는 본래 결함이 많고 개선해야 할 부분이 많은 언어였습니다.
이 때문에 각종 기능들을 프레임워크나 트랜스파일러로 보완하는 것이 일반적인 모습이었고, TS 또한 초기 버전에는 독립적으로 개발한 시스템을 포함한 형태였습니다.

하지만 시간이 지남에 따라 JS는 부족했던 부분들을 내장 기능으로 추가해나갔고, 기존에 독립적으로 개발된 기능들과 호환성 문제를 일으켰습니다.
현재 TS는 런타임 기능을 배제하고 오직 타입 기능만 발전시킨다는 명확한 원칙을 세우고 이에 따라 개발해나가고 있습니다.

헌데, 이러한 원칙 이전에, 기존에 존재하던 몇 가지 기능들이 있었으며, 이러한 기능들은 타입 공간과 값 공간 간의 경계를 혼란스럽게 만드므로 사용하지 않는 것이 좋습니다.

## 열거형 (enum)

```ts
enum Flavor {
  VANILLA = 0,
  CHOCOLATE = 1,
  STRAWBERRY = 2,
}

let flavor = Flavor.CHOCOLATE;  // Type is Flavor

Flavor  // Autocomplete shows: VANILLA, CHOCOLATE, STRAWBERRY
Flavor[0]  // Value is "VANILLA"
```

TS에서의 enum은 JS와 TS 간의 동작이 다르기 때문에 사용하지 않는 것이 좋습니다.
대신에 리터럴 타입의 유니온을 사용하면 됩니다.

```ts
type Flavor = 'vanilla' | 'chocolate' | 'strawberry';
```

## 매개변수 속성 (Parameter Properties)

TS의 [매개변수 속성](https://www.typescriptlang.org/docs/handbook/classes.html#parameter-properties)을 사용하면, 일반적으로 클래스를 초기화할 때 사용하는 경우의 문법을 보다 간결하게 작성할 수 있습니다.

```ts
// 일반적인 경우
class Person {
  name: string;
  constructor(name: string) {
    this.name = name;
  }
}

// 매개변수 속성을 사용하는 경우
class Person {
  constructor(public name: string) {}
}
```

매개변수 속성의 사용이 좋은지 나쁜지에 대해서는 찬반이 갈리는 문제입니다.
다만 기존 JS 문법과는 이질적이고 생소하다는 점과, 일반 속성과 같이 사용하는 경우 설계가 혼란스러울 수 있습니다.

## 네임스페이스와 트리플 슬래시 임포트(`///`)

기존의 JS에는 모듈 시스템이 존재하지 않았고, 이 때문에 TS 역시 독자적인 모듈 시스템의 마련이 필요했습니다.
그 결과가 트리플 슬래시 임포트와 `namespace` 키워드이며, 이는 호환성을 유지하기 위해 남아 있을 뿐, 이제는 ES6+ 스타일의 모듈을 사용하는 것이 좋습니다.

```ts
namespace foo {
  function bar() {}
}

/// <reference path="other.ts">
foo.bar();
```

## 데코레이터

데코레이터는 클래스, 메서드, 속성에 애너테이션(annotation)을 붙이거나 기능을 추가하는데 사용할 수 있습니다.
아래 예시는 클래스의 메서드 호출 시마다 `logged` 함수를 실행하는 경우입니다.

```ts
class Greeter {
  greeting: string;
  constructor(message: string) {
    this.greeting = message;
  }
  @logged
  greet() {
    return "Hello, " + this.greeting;
  }
}

function logged(target: any, name: string, descriptor: PropertyDescriptor) {
  const fn = target[name];
  descriptor.value = function() {
    console.log(`Calling ${name}`);
    return fn.apply(this, arguments);
  };
}

console.log(new Greeter('Dave').greet());
// Logs:
// Calling greet
// Hello, Dave
```

이는 처음에 앵귤러 프레임워크를 지원하기 위해 추가되었으며, 실제로도 `experimentalDecorators` 속성을 설정하고 사용해야 합니다.
현재까지도 표준화가 완료되지 않은 기능이기 때문에, 호환성이 깨질 가능성이 있습니다.
