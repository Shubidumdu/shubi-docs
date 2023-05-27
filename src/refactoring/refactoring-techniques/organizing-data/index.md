# Organizing Data

- [Organizing Data](#organizing-data)
  - [자체적으로 필드 캡슐화하기 (Self Encapsulate Field)](#자체적으로-필드-캡슐화하기-self-encapsulate-field)
  - [데이터 값을 객체로 전환하기 (Replace Data Value with Object)](#데이터-값을-객체로-전환하기-replace-data-value-with-object)
  - [값을 참조로 바꾸기 (Change Value to Reference)](#값을-참조로-바꾸기-change-value-to-reference)
  - [참조를 객체로 바꾸기 (Change Reference to Value)](#참조를-객체로-바꾸기-change-reference-to-value)
  - [배열을 객체로 대체하기 (Replace Array with Object)](#배열을-객체로-대체하기-replace-array-with-object)
  - [관찰된 데이터 중복 (Duplicate Observed Data)](#관찰된-데이터-중복-duplicate-observed-data)
  - [단방향 연결을 양방향으로 변경 (Change Unidirectional Association to Bidirectional)](#단방향-연결을-양방향으로-변경-change-unidirectional-association-to-bidirectional)
  - [양방향 연결을 단방향으로 변경 (Change Bidirectional Association to Unidirectional)](#양방향-연결을-단방향으로-변경-change-bidirectional-association-to-unidirectional)
  - [매직 넘버를 기호 상수로 대체 (Replace Magic Number with Symbolic Constant)](#매직-넘버를-기호-상수로-대체-replace-magic-number-with-symbolic-constant)
  - [필드 캡슐화 (Encapsulate Field)](#필드-캡슐화-encapsulate-field)
  - [컬렉션 캡슐화 (Encapsulate Collection)](#컬렉션-캡슐화-encapsulate-collection)
  - [타입 코드를 클래스로 대체 (Replace Type Code with Class)](#타입-코드를-클래스로-대체-replace-type-code-with-class)
  - [타입 코드를 서브클래스로 대체 (Replace Type Code with Subclasses)](#타입-코드를-서브클래스로-대체-replace-type-code-with-subclasses)
  - [타입 코드를 상태/전략 패턴으로 대체 (Replace Type Code with State/Strategy)](#타입-코드를-상태전략-패턴으로-대체-replace-type-code-with-statestrategy)
  - [서브클래스를 필드로 대체 (Replace Subclass with Fields)](#서브클래스를-필드로-대체-replace-subclass-with-fields)

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

## 단방향 연결을 양방향으로 변경 (Change Unidirectional Association to Bidirectional)

> 서로의 기능을 필요로 하는 두 클래스가 있지만, 두 클래스 간의 연결은 단방향으로 이루어져 있는 경우

클래스가 필요로 하는 연결을 추가해준다.

원래 클래스는 단방향의 연결을 가지지만, 시간이 지남에 따라 클라이언트 코드가 연결의 양쪽 모두에 액세스 가능해야 할 수 있다.

단, 양방향 연결을 단방향보다 구현 및 유지관리가 훨씬 더 어렵고, 클래스를 상호 의존적으로 만든다는 문제가 있다. 단방향 연결은 둘 중 하나를 다른 클래스로부터 독립적으로 사용할 수 있게 해준다.

- Before

```ts
class Customer {
  name: string;
  order: Order;
}

class Order {}
```

- After

```ts
class Customer {
  name: string;
  order: Order;
}

class Order {
  customer: Customer;
}
```

## 양방향 연결을 단방향으로 변경 (Change Bidirectional Association to Unidirectional)

> 두 클래스 간의 양방향 연결이 구성되어 있으나, 둘 중 하나의 클래스가 다른 클래스의 기능을 필요로 하지 않는 경우

사용하지 않는 쪽의 연결을 없앤다.

양방향 연결을 일반적으로 단방향 연결보다 유지 관리가 어렵고, 관련된 객체를 올바르게 생성 및 삭제하기 위한 추가적인 코드가 필요하다. 이에 따라 프로그램이 더 복잡해진다. 또한 다음과 같은 문제점이 생길 수 있다.

또한, 양방향 연결을 잘못 구현하는 경우 가비지 컬렉션에 문제가 발생할 수 있으며, 결국 사용하지 않는 객체로 인해 메모리가 팽창할 수 있다.

또, 클래스의 상호 의존성 문제로, 클래스가 서로에 대해 알고있어야 하므로 분리되어 사용할 수 없고, 이러한 연결이 많아질 경우 프로그램의 여러 부분이 서로 지나치게 의존하게 됨에 따라 한 컴포넌트의 변경 사항이 다른 컴포넌트에 영향을 미칠 수 있다.

- Before

```ts
class Customer {
  name: string;
  order: Order;
}

class Order {
  customer: Customer;
}
```

- After

```ts
class Customer {
  name: string;
  order: Order;
}

class Order {}
```

## 매직 넘버를 기호 상수로 대체 (Replace Magic Number with Symbolic Constant)

> 코드에 특정한 의미를 담고있는 수가 사용되고 있는 경우

해당 수가 어떤 의미인지에 대해 설명해주는 인간 친화적인 이름을 부여한 상수에 이를 할당한다.

매직넘버는 소스에서 발견되지만 명확한 의미를 알 수 없는 숫자값으로, 이러한 안티 패턴은 프로그램의 이해와 리팩터링을 어렵게 만든다.

무엇보다도, 이 매직넘버를 변경해야 하는 상황에서 문제는 더 심각해지는데, 같은 숫자가 다른 위치에서 다른 용도로 사용될 수 있으므로, 이 숫자를 사용하는 모든 코드 라인을 확인해야만 한다.

- Before

```ts
potentialEnergy(mass: number, height: number): number {
  return mass * height * 9.81;
}
```

- After

```ts
static const GRAVITATIONAL_CONSTANT = 9.81;

potentialEnergy(mass: number, height: number): number {
  return mass * height * GRAVITATIONAL_CONSTANT;
}
```

## 필드 캡슐화 (Encapsulate Field)

> 퍼블릭 필드를 보유하고 있는 경우

필드를 프라이빗으로 만들고, 필드를 읽고 쓰는 접근자 메서드를 만든다.

OOP의 이점 중 하나는 캡슐화로, 객체의 데이터를 외부에게서 숨길 수 있다는 것이다. 모든 객체 데이터가 공개되는 경우, 객체가 서로 직접 데이터를 참조 및 수정할수 있게 되어 프로그램의 모듈성이 손상되고 유지 관리가 복잡해진다.

단, 경우에 따라서는 성능에 대한 고려사항으로 인해 캡슐화를 적용하는 것이 적절하지 않을 수도 있다. 이를테면, x/y 좌표축을 갖는 객체들이 무수하게 많이 포함된 그래픽 편집기가 있다고 가정하자. 이들 좌표에 액세스하는 별도의 메서드들을 각각 두기보다는, 좌표 필드에 직접 액세스할 수 있도록 구성한다면, 액세스 메서드를 호출할 때 차지할 상당한 CPU 사이클을 절약할 수 있다. (ex. Java의 Point 클래스)

- Before

```ts
class Person {
  name: string;
}
```

- After

```ts
class Person {
  private _name: string;

  get name() {
    return this._name;
  }

  setName(name: string): void {
    this._name = name;
  }
}
```

## 컬렉션 캡슐화 (Encapsulate Collection)

> 클래스에 컬렉션 필드와 컬렉션 작업을 위한 간단한 getter/setter가 존재하는 경우

getter가 반환하는 값을 읽기 전용으로 만들고 컬렉션 요소를 추가/삭제하는 메서드를 만든다.

클래스 내에 컬렉션을 포함하는 필드가 존재하는 경우, 이 때는 일반적인 필드와는 다르게 다루어야 한다. 만약 getter를 통해 컬렉션 자체가 직접 전달되는 경우, 클라이언트가 클래스도 모르게 임의로 컬렉션의 내용을 수정할 수도 있고, 필요 이상으로 많은 데이터가 클라이언트에게 노출되기 때문이다.

따라서, 컬렉션 요소를 가져오는 getter 메서드의 경우, 컬렉션을 변경할 수 없는 형태로 반환하거나, 컬렉션 구조에 대한 과도한 데이터를 공개하지 말아야 한다. 또한, 컬렉션 값을 할당하는 메서드 대신, 컬렉션 내에 요소를 추가/삭제할 수 있는 메서드를 제공해야 한다. 이를 통해 클라이언트가 아닌, 클래스 본인이 요소의 추가 및 삭제에 대한 제어권을 갖도록 할 수 있다.

- Before

```ts
class Person {
  private _courses: Set<Course>;

  getCourses() {
    return this._courses;
  }

  setCourses(courses: Set<Course>) {
    this._courses = courses;
  }
}

class Course {}
```

- After

```ts
class Person {
  private _courses: Set<Course>;

  getCourses() {
    return [...this._courses];
  }

  addCourse(course: Course) {
    this._courses.add(course);
  }

  removeCourse(course: Course) {
    this._courses.delete(course);
  }
}

class Course {}
```

## 타입 코드를 클래스로 대체 (Replace Type Code with Class)

> 어떤 클래스가 타입 코드를 포함하는 필드를 갖추었으나, 해당 타입 코드들은 연산자 조건에 사용되지 않고, 프로그램 동작에도 영향을 주지 않는 경우

새로운 클래스를 생성하여, 타입 코드 대신 해당 클래스의 객체를 사용한다.

타입 코드가 필요한 일반적인 이유 중 하나는 숫자나 문자열로 코딩된 복잡한 개념이 있는 필드가 있는 데이터베이스 작업을 해야하는 경우다.

필드 설정자는 보통 어떤 값이 전달되는지에 대해서는 체크하지 않으므로, 누군가 의도치 않은 값이나 잘못된 값을 이들 필드에 전달하게 되면 커다란 문제가 발생할 수 있다.

타입 코드가 숫자/문자열 등 primitive 타입인 경우, 적절한 타입 체크가 이루어지지 않는다는 문제점도 있다.

- Before

```ts
class Person {
  static O = 0;
  static A = 1;
  static B = 2;
  static AB = 3;
  private _bloodGroup: number;

  get bloodGroup() {
    return this._bloodGroup;
  }

  set bloodGroup(bloodGroup: string) {
    this._bloodGroup = bloodGroup;
  }
}
```

- After

```ts
class Person {
  private _bloodGroup: BloodGroup;

  get bloodGroup() {
    return this._bloodGroup;
  }

  set bloodGroup(bloodGroup: BloodGroup) {
    this._bloodGroup = bloodGroup;
  }
}

class BloodGroup {
  static O: BloodGroup;
  static A: BloodGroup;
  static B: BloodGroup;
  static AB: BloodGroup;
}
```

## 타입 코드를 서브클래스로 대체 (Replace Type Code with Subclasses)

> 프로그램 동작에 직접적인 영향을 미치는 타입 코드가 존재하는 경우 (해당 필드의 값이 조건부로 다양한 코드를 트리거하는 경우)

코딩된 타입의 각 값에 대한 하위 클래스를 만든다. 이후 기존 클래스에서 새로 만든 하위 클래스로 관련 동작을 추출해낸다. 그리고 제어 흐름 코드를 다형성으로 대체한다.

- Before

```ts
class Employee {
  static ENGINEER: number = 0;
  static SALESMAN: number = 1;
  private _type: number;

  get employeeType() {
    return this._type;
  }

  set employeeType(type: number) {
    this._type = type;
  }
}
```

- After

```ts
class Employee {
  // ...
}

class Engineer extends Employee {
  // ...
}

class Salesman extends Employee {
  // ...
}
```

## 타입 코드를 상태/전략 패턴으로 대체 (Replace Type Code with State/Strategy)

> 동작에 영향을 주는 타입 코드가 있지만, 서브클래스를 사용하여 타입 코드를 대체할 수 없는 경우

타입 코드를 상태 객체로 대체한다. 만약 필드 값을 타입 코드로 바꾸는 것이 필수적인 경우 다른 상태 객체가 연결되도록 한다. 이는 아래와 같은 경우에 적절하다.

1. 타입 코드가 있고, 클래스의 동작에 영향을 미치고 있어 타입 코드를 클래스로 바꿀 수 없는 경우

2. 타입 코드가 클래스의 동작에 영향을 주지만 기존 클래스 계층 구조 또는 다른 이유로 인해 타입 코드에 대한 서브클래스를 만들 수 없는 경우

해당 리팩터링 테크닉은 타입 코드의 필드값이 객체의 라이프타임 동안에 변경되는 것을 막아준다. 이 때 값의 교체는 원래 클래스가 참조하는 상태 객체를 교체하여 이루어져야 한다.

코드 타입에 새로운 값을 추가해야 하는 경우, 기존 코드를 변경하지 않고 새로운 상태 하위 클래스를 추가하기만 하면 된다. (OCP ~ Open/Closed Principle)

다만 해당 리팩터링 테크닉은 불필요한 클래스가 많이 추가된다.

- Before

```ts
class Employee {
  static ENGINEER: number = 0;
  static SALESMAN: number = 1;
  private _type: number;

  get employeeType() {
    return this._type;
  }

  set employeeType(type: number) {
    this._type = type;
  }
}
```

- After

```ts
class Employee {
  private _type: EmployeeType;

  get employeeType() {
    return this._type;
  }

  set employeeType(type: EmployeeType) {
    this._type = type;
  }
}

class EmployeeType {
}

class Engineer extends EmployeeType {

}

class Salesman extends EmployeeType {

}
```

## 서브클래스를 필드로 대체 (Replace Subclass with Fields)

> 서브클래스들이 단순히 상수 데이터(반환이 항상 같음)를 반환하는 메서드만을 가지고 있을 때

해당 메서드를 상위 클래스의 필드로 바꾸고, 하위 클래스를 삭제한다.

때때로 리팩터링은 타입 코드를 피하기 위한 단순한 티켓이 된다.

이러한 경우, 서브클래스의 계층 구조는 오직 특정한 메서드의 반환값에 대한 차이만 갖게된다. 이러한 메서드들은 어떤 연산의 결과도 아니며, 오직 메서드 자체적으로 엄격하게 설정된 값일 뿐이다. 클래스 구조를 단순화하기 위해서는 구조를 하나의 클래스로 압축하고 상황에 따라 필요한 값을 가진 하나 또는 여러 개의 필드를 추가한다.

이러한 테크닉은 클래스 계층 구조에서 다른 곳으로 많은 기능들을 이동하고 난 이후에 필요할 수 있다. 이 경우 계층 구조가 그다지 가치가 없어지고, 하위 클래스가 쓸모없어지는 경우가 생길 수 있기 때문이다.

- Before

```ts
type Code = 'M' | 'F';

abstract class Person {
  abstract getCode: () => Code;
}

class Male extends Person {
    getCode = (): Code => 'M';
}

class Female extends Person {
    getCode = (): Code => 'F';
}

```

- After

```ts
type Code = 'M' | 'F';

class Person {
  private _code: Code;

  getCode(): Code {
    return this._code;
  };
}
```
