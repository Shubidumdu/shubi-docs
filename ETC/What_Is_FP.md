# Functional Programming

함수형 프로그래밍(Functional Programming)은 예전부터 들어왔지만, 그 자체로 되게 애매한 지식으로 내게 자리잡고 있었다.

[여기](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-functional-programming-7f218c68b3a0)의 문서를 살펴보면서 함수형 프로그래밍에 대한 개념을 잡고 가려고 한다.

---

FP는 다음과 같은 방식을 통해 소프트웨어를 만들어나가는 과정이다.

- 순수 함수의 합성(compose) 기반
- 공유 상태 / mutable 데이터 / 사이드 이펙트를 모두 회피
- 명령형 보다는 선언형
- 웹의 측면에서 보자면 순수함수들을 통한 상태 관리를 지향

이는 일반적으로 앱의 상태가 공유되고, 객체 메서드와 동일시되는 OOP와는 대조적인 성향을 띤다.

함수형 코드는 OOP 및 명령형 코드보다 명확하고, 예측가능하고, 테스트하기 쉽다.

사실, JS를 통해 프로그래밍을 해왔다면, 함수형 프로그래밍의 컨셉과 기능들을 본인도 모르는 사이에 적용해왔을 가능성이 높다.

## 왜 어렵게 들릴까?

**함수형 프로그래밍**은 뭔가 거창하게 들린다. 생소한 단어들이 여기저기 있기 때문이다.

- 순수 함수 (Pure func.)
- 합성 함수 (Function Composition)
- 공유 상태 방지
- Mutationg 상태 방지
- 사이드 이펙트 방지

## 순수 함수

동일한 입력이 있으면, 결과도 항상 동일하다

사이드 이펙트가 없다

## 합성 함수

둘 이상의 함수들을 합쳐 새로운 작업을 수행하는 함수다.

## 공유 상태

공유 상태는 공유 영역에 존재하는 변수, 객체, 메모리 공간들을 의미한다.

공유 상태가 갖는 문제점은, 단순히 함수의 실행 순서가 달라졌음에도 그 결과가 달라질 수 있다는 점에서 온다.

```js
// With shared state, the order in which function calls are made
// changes the result of the function calls.
const x = {
  val: 2,
};

const x1 = () => (x.val += 1);

const x2 = () => (x.val *= 2);

x1();
x2();

console.log(x.val); // 6

// This example is exactly equivalent to the above, except...
const y = {
  val: 2,
};

const y1 = () => (y.val += 1);

const y2 = () => (y.val *= 2);

// ...the order of the function calls is reversed...
y2();
y1();

// ... which changes the resulting value:
console.log(y.val); // 5
```

이를 회피하기 위해, 앞서 나온 개념들인 순수함수, 합성함수들을 활용함과 더불어 공유 상태를 회피한다.

```js
const x = {
  val: 2,
};

const x1 = (x) => Object.assign({}, x, { val: x.val + 1 });

const x2 = (x) => Object.assign({}, x, { val: x.val * 2 });

console.log(x1(x2(x)).val); // 5

const y = {
  val: 2,
};

// Since there are no dependencies on outside variables,
// we don't need different functions to operate on different
// variables.

// this space intentionally left blank

// Because the functions don't mutate, you can call these
// functions as many times as you want, in any order,
// without changing the result of other function calls.
x2(y);
x1(y);

console.log(x1(x2(y)).val); // 5
```

## Immutability

**불변성**은 일단 생성되고 난 후에는 수정될 수 없는 성질을 의미한다.

단순히 `const`로 변수를 선언하는 것과 헷갈리지 말아야 한다.

## Side Effects

사이드 이펙트는 함수의 실행 이후 반환되는 값이 아닌 다른 부분에서 상태 변화가 생기는 경우다.

사이드 이펙트를 유발하는 동작들은 소프트웨어에서 별도로 관리되어야 하며, 프로그래밍 로직에 직접적으로 관련되어선 안된다.

앞선 과정이 이루어지면, 소프트웨어의 확장, 리팩토링, 디버깅, 테스트 및 유지보수가 훨씬 수월해진다.

때문에 많은 프론트엔드 프레임워크들이 느슨하게 결합된 모듈을 통해 상태를 관리하고, 컴포넌트를 렌더링하게끔 유도한다.

## 고차 함수를 통한 재사용성

함수형 프로그래밍에서는 어떤 종류의 데이터든 동일하게 취급된다.

`map()` 메서드는 매개변수로 함수를 사용하기 때문에, 어떤 종류의 값이든 매핑을 할 수 있다.

JS는 1등급 함수를 갖는다. 이는 함수를 데이터로 다룰 수 있게 하며, 매개변수로 지정할 수도 있고, 인수로 전달될 수도 있고, 함수 자체가 반환될 수도 있다.

결국, 고차 함수란 함수를 매개변수로 받거나, 함수를 반환하는 함수로 여겨질 수 있다. 이들을 종종 다음과 같은 역할을 할 수 있다.

- 콜백, 프로미스 등을 통해 이벤트 핸들링이나 비동기 로직을 처리한다.
- 다양한 데이터 타입에 폭넓게 적용할 수 있는 유틸 함수를 만든다
- Partial Application 및 Currying
- 함수의 리스트를 받아 이들 함수들의 합성함수를 반환

## 선언형 vs 명령형

함수형 프로그래밍은 선언형이며, 이는 프로그램 로직이 내부적인 흐름을 설명하지 않고 표현되는 하나의 패러다임이다.

명령형 프로그래밍은 원하는 결과를 얻기 위한 순차적인 방법을 코드에 작성하며, **어떻게 처리하는가**를 주로 코드에 작성한다.

선언형 프로그래밍은 어떻게 이를 얻는지보다는 **무엇을 하는가**를 명시하는 코드를 주로 작성한다. 짧은 예시를 보자.

아래는 명령형 프로그래밍으로, 함수 실행을 통해 각 배열에 2를 곱한 결과를 얻고자 한다.

```js
const doubleMap = (numbers) => {
  const doubled = [];
  for (let i = 0; i < numbers.length; i++) {
    doubled.push(numbers[i] * 2);
  }
  return doubled;
};

console.log(doubleMap([2, 3, 4])); // [4, 6, 8]
```

아래는 선언형 프로그래밍으로, 앞의 코드와 동일한 역할을 하지만, `map` 메서드를 통해 각각의 값에 2를 곱하는 과정을 추상화하여 훨씬 데이터의 흐름을 명확하게 나타낸다.

결국, 명령형 프로그래밍은 Statement를 위주로 작성되는 코드에 해당한다. Statement는 일정한 액션을 수행하는 코드 조각들이며, `for`, `if`, `switch`, `throw`, 등이 이에 해당한다고 볼 수 있다.

반면, 선언형 프로그래밍은 Expression에 더 많이 의존한다. Expression은 어떤 값을 나타내는 코드 조각들이다. 일반적으로 결과 값을 생성하기 위해 함수 호출, 값 또는 연산자들의 조합들이다.

## 결론

함수형 프로그래밍은 다음을 선호하는 패러다임이다.

- 공유 상태 & 사이드 이펙트 대신에 순수함수
- 불변(Immutable) 데이터를 활용
- 명령형 흐름보다는 합성 함수
- 고차 함수를 통해 여러 종류의 데이터 타입에 폭넓게 재사용될 수 있는 유틸함수
- 명령형보다 선언형 (어떻게 하는지보다 무엇을 하는지에 초점)
- Statement보다 Expression
- 다형성(Polymorphism)보다는 컨테이너와 고차함수
