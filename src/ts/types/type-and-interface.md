# 타입과 인터페이스의 차이점 알기

타입스크립트에서는 명명된 타입(named type)을 정의하기 위해 <b>타입 별칭(Type Alias, 이하 타입)</b>과 <b>인터페이스(Interface)</b>라는 두 가지 방법을 사용할 수 있습니다.

```ts
// type
type TState = {
  name: string;
  capital: string;
}

// interface
interface IState {
  name: string;
  capital: string;
}
```

## 타입과 인터페이스의 유사점

### 인덱스 시그니처를 사용할 수 있습니다.

```ts
type TDict = { [key: string]: string };

interface IDict {
  [key: string]: string;
}
```

### 함수 타입을 정의할 수 있습니다.

```ts
type TFn = (x: number) => string;

interface ifN {
  (X: number): string;
}
```

JS에서의 함수는 곧 객체라는 점을 떠올려보면, 아래처럼 추가로 프로퍼티를 정의할 수도 있습니다.

```ts
type TFnWithProperties = {
  (x: number): number;
  prop: string;
}

interface IFnWithProperties {
  (x: number): number;
  prop: string;
}
```

### 제네릭을 사용할 수 있습니다.

```ts
type TPair<T> = {
  first: T;
  second: T;
}

interface IPair<T> {
  first: T;
  second: T;
}
```

### 타입과 인터페이스는 서로 간에 확장이 가능합니다.

```ts
interface IStateWithPop extends TState {
  population: number;
}

type TStateWithPop = IState & { population: number; };
```

### 클래스의 구현(implements)에 사용할 수 있습니다.

```ts
class StateT implements TState {
  name: string = '';
  capital: string = '';
}

class StateI implements IState {
  name: string = '';
  capital: string = '';
}
```

## 타입과 인터페이스의 차이점

### 유니온(`|`)은 타입으로만 표현할 수 있습니다.

```ts
type AorB = 'a' | 'b';
```

유니온을 인터페이스로 표현할 방법은 없습니다. 이러한 이유로 타입은 일반적으로 인터페이스보다 더 쓰임새가 많습니다.

```ts
type Input = { /* ... */ };
type Output = { /* ... */ };

interface VariableMap {
  [name: string]: Input | Output;
}

type NamedVariable = (Input | Output) & { name: string };
```

### 튜플, 배열 타입은 type 키워드를 이용해야 합니다.

```ts
type Pair = [number, number];
type Stringlist = string[];
type NamedNums = [string, ...number[]];
```

인터페이스로도 유사하게 구현할 수는 있으나, 이 경우엔 `.map`, `.concat` 등의 배열 메서드를 사용할 수 없게 됩니다.

```ts
interface Tuple {
  0: number;
  1: number;
  length: 2;
}

const t: Tuple = [10, 20]; // 정상
t.concat([30, 40]); // ERROR : Property 'concat' does not exist on type 'Tuple'.
```

### 인터페이스는 보강(augment)이 가능합니다.

```ts
interface IState {
  name: string;
  capital: string;
}

interface IState {
  population: number;
}

const wyoming: IState = {
  name: 'Wyoming',
  capital: 'Cheyenne',
  population: 500_000,
}
```

위의 예제처럼 프로퍼티를 확장하는 것을 선언 병합(Declaration merging)이라고 합니다. 이는 주로 타입 선언 파일에서 사용됩니다. (일반 코드에서 쓰지 못하는 것은 아닙니다.) 다시 말해, 타입 선언 파일을 작성할 때는 선언 병합을 지원하기 위해 반드시 인터페이스를 사용해야 하며, 표준을 따라야 합니다.

## 결론 : 그래서 둘 중 무엇을 써야 할까요?

- 복잡한 타입의 정의가 필요한 경우 => **타입**
- 보강의 가능성이 있는 경우 (ex. API) => **인터페이스**

그 외에 프로젝트의 일관된 스타일을 유지하게끔 일관적으로 타입 또는 인터페이스를 사용하면 됩니다.