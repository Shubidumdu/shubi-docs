# 잉여 속성 체크의 한계 인지하기

## 잉여 속성 체크 (Excess property checking)

타입이 명시된 변수에 **객체 리터럴**을 할당할 때, TS는 해당 타입의 속성이 제대로 존재하는지, 그리고 **그 외의 속성**은 없는지 확인합니다.
이 *그 외의 속성이 없는지* 판단하는 과정을 **잉여 속성 체크(Excess property checking)**라고 합니다.

```ts
interface Room {
  numDoors: number;
  ceilingHeightFn: number;
}

const r: Room = {
  numDoors: 1,
  ceilingHeightFt: 10,
  elephant: 'present', // 에러 발생
}
```

### 잉여 속성 체크는 객체 리터럴에서만 발생합니다.

여기서 포인트는 **객체 리터럴**입니다. 다른 임시 변수를 통해 우회적으로 할당을 하는 경우에 이는 문제가 발생하지 않습니다.

```ts
const obj = {
    numDoors: 1,
    ceilingHeightFt: 10,
    elephant: 'present',
}

const room: Room = obj; // 문제 없음
```

여기서 우리가 앞서 살펴봤던 덕 타이핑과 이에 따른 TS의 구조적 타이핑 체계를 떠올려봅시다. 
그러한 관점에서 생각해보면, 앞선 코드에서 문제가 발생하지 않는 것이 이상한 일은 아닙니다.

결국 중요한 포인트는, 일반적인 구조적 할당 가능성 체크와 **잉여 속성 체크**는 엄연히 별도의 과정으로써 동작한다는 점입니다.
그리고, 잉여 속성 체크는 **객체 리터럴을 통해** 할당 또는 함수에 값을 넘겨줄 경우에 발생한다는 것을 기억해야 합니다.

### 잉여 속성 체크는 단언문(Assertion)에서는 발생하지 않습니다.

우리가 단언문(Assertion)보다는 선언문(Declaration)을 우선시 해야하는 단적인 이유 중 하나입니다.
객체 리터럴을 사용하더라도, 단언을 이용한 경우에는 잉여 속성 체크가 일어나지 않습니다.

```ts
const room = { 
    numDoors: 1,
    ceilingHeightFt: 10,
    elephant: 'present',
  } as Room; // 문제 없음
```

### 잉여 속성 체크를 원치 않는다면 인덱스 시그니처를 사용하세요.

잉여 속성 체크를 원치 않는 상황, 다시 말해 추가적인 속성을 가질 수 있다고 판단되는 경우에는 인덱스 시그니처를 사용하면 됩니다.
이 인덱스 시그니처에 대해서는 아이템15에서 상세하게 다루겁니다.

```ts
interface Options {
  darkMode?: boolean;
  [others: string]: unknown;
}

const options: Options = { darkmode: true }; // 문제 없음
```

### 약한 타입에 대한 공통 속성 체크

선택적 속성만 가지는 **약한(weak) 타입**에도 유사한 체크가 동작하는데, 이는 잉여 속성이 아닌 공통 속성이 있는지를 확인한다는 점에서 조금 다릅니다.

```ts
interface LineChartOptions {
  logscale?: boolean;
  inverteedYAxis?: boolean;
  areaChart?: boolean;
}

const opts = { logScale: true };

const options: LineChartOptions = opts; // 에러 발생
```

약한 타입에 대한 공통 속성 체크는 값과 타입 간에 공통된 속성이 있는지에 대해 확인하는 과정입니다.
그러나 잉여 속성 체크와는 다르게, 약한 타입과 관련된 할당마다 수행된다는 차이가 있습니다.