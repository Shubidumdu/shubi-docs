# Organizing Data

- [Organizing Data](#organizing-data)
  - [자체적으로 필드 캡슐화하기 (Self Encapsulate Field)](#자체적으로-필드-캡슐화하기-self-encapsulate-field)
  - [데이터 값을 객체로 전환하기 (Replace Data Value with Object)](#데이터-값을-객체로-전환하기-replace-data-value-with-object)
  - [값을 참조로 바꾸기 (Change Value to Reference)](#값을-참조로-바꾸기-change-value-to-reference)
  - [참조를 객체로 바꾸기 (Change Reference to Value)](#참조를-객체로-바꾸기-change-reference-to-value)
  - [배열을 객체로 대체하기 (Replace Array with Object)](#배열을-객체로-대체하기-replace-array-with-object)
  - [관찰된 데이터 중복 (Duplicate Observed Data)](#관찰된-데이터-중복-duplicate-observed-data)

해당 범주의 리팩터링 기술은 데이터 처리를 도와주며, primitive 타입을 풍부한 클래스 기능으로 대체한다.

또 다른 중요한 결과는 클래스 간의 연결을 끊어 클래스의 이식성(portable)과 재사용성을 향상시킬 수 있다는 것이다.

## 자체적으로 필드 캡슐화하기 (Self Encapsulate Field)

> 어떤 클래스 내의 프라이빗 필드에 직접 접근하고 있는 경우

해당 필드에 대한 getter/setter를 생성하여, 해당 필드에 접근할 때는 오직 이들만을 사용하도록 한다.

클래스 내 프라이빗 필드에 직접 액세스하는 것으로는 유연성이 충분하지 않을 수 있다. 쿼리가 수행될 때 필드값을 초기화하거나, 필드에 새로운 값을 할당할 때 부가적인 작업을 처리할 수 있는 등, getter/setter 사용 시 많은 경우에 더 유연한 대처가 가능해진다.

또한, 서브클래스에서 getter/setter를 재정의하는 것이 가능하다는 점도 큰 이점이다.

- Before

```ts
class Range {
  private low: number
  private high: number;
  includes(arg: number): boolean {
    return arg >= this.low && arg <= this.high;
  }
}
```

- After

```ts
class Range {
  private _low: number;
  private _high: number;
  includes(arg: number): boolean {
    return arg >= this.low && arg <= this.high;
  }
  get low(): number {
    return this._low;
  }
  get high(): number {
    return this._high;
  }
}
```

## 데이터 값을 객체로 전환하기 (Replace Data Value with Object)

> 클래스(또는 여러 클래스)에 어떤 데이터 필드가 있고, 해당 필드가 고유한 역할을 수행하고 관련 데이터를 보유하는 경우

새로운 클래스를 만들어, 기존 필드와 그것의 동작을 새 클래스로 옮긴 다음, 기존의 클래스에는 새로 만든 클래스의 객체를 보관한다.

이러한 리팩터링은 기본적으로 [클래스 추출](../moving-features-between-objects/index.md#클래스-추출-extract-class)로부터 확장되는 특수한 케이스다. 이것의 차이점은 리팩터링의 원인에 있다.

"클래스 추출"의 경우 서로 다른 작업을 담당하는 단일 클래스가 있어 그 책임을 분리하고자 하는 것이 목적이다.

반면, "데이터 값을 객체로 전환하기"의 경우, 데이터 값이 primitive 필드로 존재함에 따라, 여러 클래스에서 해당 필드를 이용하고, 그에 대해 유사한 작업을 요구할 가능성이 있기 때문에, 중복 코드가 발생할 가능성이 생겨 이를 방지하고자 하는 것이다.

- Before

```ts
class Order {
  customer: string;
  // ...
}
```

- After

```ts
class Order {
  customer: Customer;
  // ...
}

class Customer {
  name: string;
  // ...
}
```

## 값을 참조로 바꾸기 (Change Value to Reference)

> 하나의 객체로 대체해야 하는 하나의 클래스에 동일한 인스턴스가 여러 개 있는 경우

매번 생성할 필요가 없는 동일한 객체 여러개 생성하고 있는 경우, 이를 하나의 참조 객체로 대체한다.

이 경우 참조 객체에서 변경이 일어나면, 이를 참조하는 다른 곳에서도 이러한 변경 사항에 액세스할 수 있게 된다.

- Before

```ts
const customer = new Customer(customerData);
```

- After

```ts
const customer = customerRepository.get(customerData.id);
```

## 참조를 객체로 바꾸기 (Change Reference to Value)

> 참조 객체가 너무 작고 자주 변경되지 않아 라이프사이클 관리가 불필요하다고 느끼는 경우

해당 참조 객체(Reference Object)를 값 객체(Value Object)로 변경한다.

보통 참조에서 객체로 전환하고자 하는 생각은 참조를 사용하는 작업에서 불편함을 느끼는 경우에서 온다. 참조를 사용하는 경우 다음에 대한 관리가 필요하다.

- 항상 저장소로부터 필수 객체를 요청해야 한다.
- 메모리 내 참조는 작업에 불편할 수 있다.
- 분산 및 병렬 시스템에서는 참조를 다루는 것이 값에 비해 특히나 더 어렵다.

값 객체는 수명 동안에 자주 변경되는 객체보다는, 변경할 수 없는 객체를 다루는 경우에 특히나 더 유용하다. 객체 값을 반환하는 각 쿼리의 결과가 매번 동일하다면, 동일한 것을 나타내는 객체가 여러개 있어도 문제가 발생하진 않는다.

- Before

```ts
class Product {
  applyDiscount(val: number) {
    this._price.amount -= val;
  }
}
```

- After

```ts
class Product {
  applyDiscount(val: number) {
    this._price = new Money(this._price.amount - val, this._price.currency);
  }
}
```

## 배열을 객체로 대체하기 (Replace Array with Object)

> 여러 타입의 데이터를 담기 위한 용도로 배열을 사용하고 있는 경우

각 요소를 따로 필드로 분리하도록 하여 이를 객체로 대체한다.

배열은 단일한 유형의 데이터와 컬렉션을 저장하는 데에 탁월한 자료구조인 반면, 저마다 다른 타입의 데이터를 보관하는 경우에는 치명적인 오류로 이어질 수 있다.

클래스의 필드는 배열의 요소보다 문서화하기가 훨씬 쉽고, 결과 클래스에는 메인 클래스 및 다른 곳에 저장되어 있던 모든 관련된 동작을 배치할 수 있다.

- Before

```ts
let row = new Array(2);
row[0] = "Liverpool";
row[1] = "15";
```

- After

```ts
let row = new Performance();
row.setName("Liverpool");
row.setWins("15");
```

## 관찰된 데이터 중복 (Duplicate Observed Data)

> 클래스에 저장된 도메인 데이터가 GUI를 담당하고 있는 경우

데이터를 별도의 클래스로 구분하고, 도메인 클래스와 GUI 간 연결을 구축하여 동기화가 이루어질 수 있도록 보장하는 것이 좋다.

동일한 데이터에 대한 여러 형태의 인터페이스(ex. 데스크톱/모바일)를 갖추고자 하는 경우, GUI를 도메인으로부터 분리하지 않으면 코드 중복 및 여러 실수를 피하기가 매우 어렵다.

- 비즈니스 로직 클래스와 프레젠테이션 클래스 간에 책임을 분담(단일 책임 원칙)하여 프로그램을 더 읽기 쉽고, 이해하기 쉽게 만들 수 있다.

- 새로운 인터페이스 뷰를 추가해야 하는 경우, 새로운 프레젠테이션 클래스를 생성하기만 하면 되기 때문에, 비즈니스 로직 코드를 건들 필요가 없다. (개방/폐쇄 원칙)

- 여러 사람이 비즈니스 로직과 사용자 인터페이스를 작업할 수 있다.

- Before

```ts
class IntervalWindow {
  startField: TextField;
  endField: TextField;
  lengthField: TextField;

  startFieldFocusLost() {
    // ...
  }
 
  endFieldFocusLost() {
    // ...
  }
 
  lengthFieldFocusLost() {
    // ...
  }

  calculateLength() {
    // ...
  }

  calculateEnd() {
    // ...
  }
}
```

- After

```ts
class IntervalWindow {
  startField: TextField;
  endField: TextField;
  lengthField: TextField;
  interval: Interval;

  startFieldFocusLost() {
    // ...
  }
 
  endFieldFocusLost() {
    // ...
  }
 
  lengthFieldFocusLost() {
    // ...
  }
}

class Interval {
  start: Date;
  end: Date;
  length: number;

  calculateLength() {
    // ...
  }

  calculateEnd() {
    // ...
  }
}
```
