# Dealing with Generalization

---

- [Dealing with Generalization](#dealing-with-generalization)
  - [필드 끌어올리기 (Pull Up Field)](#필드-끌어올리기-pull-up-field)
  - [메서드 끌어올리기 (Pull Up Method)](#메서드-끌어올리기-pull-up-method)
  - [생성자 바디 끌어올리기 (Pull Up Constructor Body)](#생성자-바디-끌어올리기-pull-up-constructor-body)
  - [메서드 밀어내리기 (Push Down Method)](#메서드-밀어내리기-push-down-method)
  - [필드 밀어내리기 (Push Down Field)](#필드-밀어내리기-push-down-field)
  - [서브클래스 추출하기 (Extract Subclass)](#서브클래스-추출하기-extract-subclass)
  - [상위클래스 추출하기 (Extract Superclass)](#상위클래스-추출하기-extract-superclass)
  - [인터페이스 추출하기 (Extract Interface)](#인터페이스-추출하기-extract-interface)
  - [계층 구조 축소하기 (Collapse Hierarchy)](#계층-구조-축소하기-collapse-hierarchy)
  - [템플릿 메서드 생성 (Form Template Method)](#템플릿-메서드-생성-form-template-method)
  - [상속을 위임으로 바꾸기 (Replace Inheritance with Delegation)](#상속을-위임으로-바꾸기-replace-inheritance-with-delegation)
  - [위임을 상속으로 바꾸기 (Replace Delegation with Inheritance)](#위임을-상속으로-바꾸기-replace-delegation-with-inheritance)

추상화는 주로 클래스 상속 계층 구조를 따라 기능을 이동시키고, 새로운 클래스 및 인터페이스를 생성하고, 상속을 위임으로 대체하거나 혹은 그 반대로 대체하는 것과 관련된 자체적인 리팩터링 테크닉들을 담고 있다.

## 필드 끌어올리기 (Pull Up Field)

> 두 클래스가 똑같은 필드를 갖고 있는 경우

서브클래스로부터 필드를 제거하고 부모 클래스로 필드를 옮기자.

서브클래스가 개별적으로 성장 및 발전하면서 동일한 필드가 메서드가 나타나게 된 경우이다.

- Before

```ts
class Soldier {
  health: number;
}

class Tank {
  health: number;
}
```

- After

```ts
class Unit {
  health: number;
}

class Soldier extends Unit {
  // ...
}

class Tank extends Unit {
  // ...
}
```

## 메서드 끌어올리기 (Pull Up Method)

> 여러 서브클래스들에 비슷한 동작을 수행하는 메서드가 있는 경우

하나의 메서드를 만들어 이를 상위 클래스로 옮긴다.

이는 서브클래스가 상위 클래스의 작업을 재정의하더라도, 본질적으로는 동일한 작업을 수행하는 경우에도 적용할 수 있다.

대부분의 언어에서 서브클래스 생성자는 상위 클래스의 매개변수와 다른 그들 본인만의 매개변수를 가질수 있기 때문에, 상위클래스의 생성자에서는 실제로 필요한 매개변수들만 사용하도록 해야한다.

- Before

```ts
class Soldier {
  getHealth() {
    // ...
  }
}

class Tank {
  getHealth() {
    // ...
  }
}
```

- After

```ts
class Unit {
  getHealth() {
    // ...
  }
}

class Soldier extends Unit {
  // ...
}

class Tank extends Unit {
  // ...
}
```

## 생성자 바디 끌어올리기 (Pull Up Constructor Body)

> 서브클래스들에 거의 동일한 코드를 가진 생성자가 있는 경우

상위 클래스의 생성자를 생성한 후, 서브클래스에서 상위 클래스의 생성자를 그대로 호출하여 사용하도록 한다.

- Before

```ts
class Manager extends Employee {
  constructor(name: string, id: string, grade: number) {
    this.name = name;
    this.id = id;
    this.grade = grade;
  }
  // ...
}
```

- After

```ts
class Manager extends Employee {
  constructor(name: string, id: string, grade: number) {
    super(name, id);
    this.grade = grade;
  }
  // ...
}
```

## 메서드 밀어내리기 (Push Down Method)

> 상위 클래스에 구현한 동작을 오직 하나(또는 일부)의 서브클래스에서만 사용하고 있는 경우

서브클래스로 해당 동작을 옮긴다.

메서드가 하나 이상의 하위 클래스에서 필요하지만, 모든 하위 클래스에서 필요한 것은 또 아닌 경우에는 중간 하위 클래스를 만들어 메서드를 해당 클래스로 옮기는 것이 유용할 수 있다.

- Before

```ts
class Unit {
  getFuel() {
    // ...
  }
}

class Soldier extends Unit {
  // ...
}

class Tank extends Unit {
  // ...
}
```

- After

```ts
class Unit {
  // ...
}

class Soldier extends Unit {
  // ...
}

class Tank extends Unit {
  getFuel() {
    // ...
  }
}
```

## 필드 밀어내리기 (Push Down Field)

> 오직 일부 서브클래스에서만 어떤 필드를 사용하는 경우

서브클래스로 해당 필드를 옮긴다.

- Before

```ts
class Unit {
  fuel: number;
  // ...
}

class Soldier extends Unit {
  // ...
}

class Tank extends Unit {
  // ...
}
```

- After

```ts
class Unit {
  // ...
}

class Soldier extends Unit {
  // ...
}

class Tank extends Unit {
  fuel: number;
  // ...
}
```

## 서브클래스 추출하기 (Extract Subclass)

> 특정한 상황에서만 사용되는 기능이 있는 경우

서브클래스를 따로 생성하여 그러한 "특정 상황"에서 이 서브클래스를 사용하도록 한다.

클래스에 특정 드문 사례를 구현하기 위한 메서드와 필드가 있을 때, 드물기야 하지만 클래스가 해당 사례를 담당하고 있는 것은 맞으므로 아예 다른 클래스를 만들어 해당 사례를 담당하는 것은 잘못되었다. 이런 경우에는 서브클래스를 만들어 해당 사례를 담당하도록 한다.

- Before

```ts
class JobItem {
  getTotalPrice() {
    // ...
  }
  getUnitPrice() {
    // ...
  }
  getEmployee() {
    // ...
  }
}
```

- After

```ts
class JobItem {
  getTotalPrice() {
    // ...
  }
  getUnitPrice() {
    // ...
  }
  getEmployee() {
    // ...
  }
}

class LaborItem extends JobItem {
  getUnitPrice() {
    // ...
  }
  getEmployee() {
    // ...
  }
}
```

## 상위클래스 추출하기 (Extract Superclass)

> 다른 두 클래스에 유사한 형태의 필드와 메서드가 있는 경우

둘 사이에 동일한 필드와 메서드를 가진 상위 클래스를 생성한다.

다만, 이미 상위클래스가 있는 경우에는 해당 테크닉을 적용할 수 없다.

- Before

```ts
class Employee {
  getAnnualCost() {
    // ...
  }
  getName() {
    // ...
  }
  getId() {
    // ...
  }
}

class Department {
  getTotalAnnualCost() {
    // ...
  }
  getName() {
    // ...
  }
  getHeadCount() {
    // ...
  }
}
```

- After

```ts
class Party {
  getAnnualCost() {
    // ...
  }
  getName() {
    // ...
  }
}

class Employee extends Party {
  getAnnualCost() {
    // ...
  }
  getId() {
    // ...
  }
}

class Department extends Party {
  getAnnualCost() {
    // ...
  }
  getHeadCount() {

  }
}
```

## 인터페이스 추출하기 (Extract Interface)

> 클래스 인터페이스의 동일한 부분을 여러 클라이언트에서 사용하는 경우
>
> 또는, 두 클래스의 인터페이스 일부가 똑같은 경우

동일한 부분을 자체적인 인터페이스로 추출한다.

인터페이스는 클래스가 서로 다른 상황에서 특별할 역할을 수행해야 하는 경우에 매우 적절하다. 이 경우 인터페이스를 사용하면 어떤 역할을 하는지 명시적으로 나타낼 수 있다.

또한, 인터페이스는 클래스가 서버에서 수행하는 작업을 설명해야 할 때 편리하다. 최종적으로는 여러 타입의 서버를 사용하도록 계획하고 있는 경우, 모든 서버들이 해당 인터페이스를 구현해야 한다.

이는 [상위클래스 추출](#상위클래스-추출하기-extract-superclass)과 비슷하지만, 인터페이스를 추출하면 공통 코드가 아닌 공통 인터페이스만을 분리할 수 있다. 즉, 클래스에 중복 코드가 포함되어 있는 경우는 인터페이스를 추출해도 중복 제거에는 도움이 되지 않는다.

반면, [클래스 추출](../moving-features-between-objects/index.md#클래스-추출-extract-class)을 적용하여 중복을 포함한 동작을 별도의 컴포넌트로 이동시키고 모든 작업을 위임하면 이 문제를 해결할 수 있다. 공통 동작의 크기가 큰 경우, 언제든 [상위클래스 추출](#상위클래스-추출하기-extract-superclass)을 적용할 수 있다. 물론 이 방법이 훨씬 더 쉽지만, 부모클래스는 하나만 생성할 수 있다는 점을 기억하자.

- Before

```ts
class Employee {
  getRate() {
    // ...
  }
  getName() {
    // ...
  }
  getDepartment() {
    // ...
  }
  hasSpecialSkill() {
    // ...
  }
}
```

- After

```ts
interface Billable {
    getRate: () => number;
    hasSpecialSkill: () => string;
}

class Employee implements Billable {
  getRate() {
    // ...
  }
  getName() {
    // ...
  }
  getDepartment() {
    // ...
  }
  hasSpecialSkill() {
    // ...
  }
}
```

## 계층 구조 축소하기 (Collapse Hierarchy)

> 서브클래스가 상위클래스와 실질적으로 동일한 클래스 계층 구조가 되어버린 경우

서브클래스와 상위클래스를 하나로 합친다.

시간이 지나면서 프로그램이 성장하고, 서브클래스와 슈퍼클래스가 거의 동일해지는 경우에 이를 적용할 수 있다.

단, 이 경우 리스코프 대체 원칙을 위반할 수도 있다는 점에 주의하라. 예를 들어, 실수로 `Transport` 슈퍼클래스를 `Car` 서브클래스로 합쳐버리면, `Plane` 클래스가 `Car`의 자식 클래스가 되어버린다.

- Before

```ts
class Employee {
  // ...
}

class Salesman extends Employee {
  // ...
}
```

- After

```ts
class Employee {
  // ...
}
```

## 템플릿 메서드 생성 (Form Template Method)

> 내 서브클래스들이 모두 동일한 순서로 유사하게 진행되는 과정을 포함하는 알고리즘을 구현하고 있는 경우

알고리즘 구조와 그 동일한 과정을 슈퍼클래스로 옮기고, 서로 다른 방식의 구현에 대해서는 서브클래스에 따로 구축시킨다.

서브클래스는 여러 사람들이 동시에 개발하는 경우가 많기 때문에, 코드가 중복되고 오류가 발생하며, 변경할 때마다 모든 서브클래스에 적용함에 따라 유지 관리에 어려움이 발생할 수 있다.

중복 제거가 항상 코드의 복사/붙여넣기를 없애는 것을 의미하는 것은 아니다. 보다 상위 수준의 중복을 제거하는 것 역시 포함된다.

템플릿 메서드 생성은 **개방/폐쇄 원칙**이 실제로 작동하는 예다. 새로운 알고리즘 버전이 나타나면 기존 코드를 변경할 필요 없이 새로운 서브클래스를 만들기만 하면 된다.

- Before

```ts
class Site {
  // ...
}

class ResidentialSite extends Site {
  getBillableAmount() {
    const base = this.units * this.rate;
    const tax = base * this.taxRate;
    return base + tax;
  }
}

class LifelineSite extends Site {
  getBillableAmount() {
    const base = this.units * this.rate * 0.5;
    const tax = basse * this.taxRate * 0.2;
    return base + tax;
  }
}
```

- After

```ts
class Site {
  getBillableAmount() {
    return this.getBaseAmount() + this.getTaxAmount();
  }
  getBaseAmount() {
    return this.units * this.rate;
  }
  getTaxAmount() {
    return this.getBaseAmount() * this.taxRate;
  }
}

class ResidentialSite extends Site {
  getBaseAmount() {
    return this.units * this.rate;
  }
  getTaxAmount() {
    return this.getBaseAmount() * this.taxRate;
  }
}

class LifelineSite extends Site {
  getBaseAmount() {
    return this.units * this.rate * 0.5;
  }
  getTaxAmount() {
    return this.getBaseAmount() * this.taxRate * 0.2;
  }
}
```

## 상속을 위임으로 바꾸기 (Replace Inheritance with Delegation)

> 서브클래스가 슈퍼클래스 메서드 중 일부만을 사용하거나, 슈퍼클래스의 데이터를 상속할 수 없는 경우

필드를 생성하여 그 안에 슈퍼클래스 객체를 넣고, 메서드는 슈퍼클래스 객체에게 위임한 다음, 상속을 제거한다.

상속을 조합(composition)으로 대체하면 클래스 디자인을 크게 개선할 수 있다.

- 서브클래스가 리스코프 치환 원칙을 위반하는 경우, 즉 상속이 공통 코드를 결합하기 위한 목적으로만 구현되었으나 서브클래스가 슈퍼클래스의 확장이기 때문에 상속을 구현할 수 없는 경우
- 서브클래스가 슈퍼클래스의 메서드 중 오직 일부만을 사용하는 경우, 이 때는 해당 서브클래스 객체를 통해 누군가 호출해선 안되는 슈퍼클래스의 메서드를 실행하는 것은 시간 문제다.

본질적으로, 이 테크닉은 두 클래스를 분할하고 슈퍼클래스를 부모가 아닌, 서브클래스의 도우미의 역할로 바꾼다. 모든 슈퍼클래스의 메서드를 상속하는 대신, 서브클래스는 슈퍼클래스의 메서드 중 필요한 것만을 호출한다.

- Before

```ts
class Vector {
  isEmpty() {
    // ...
  }
}

class Stack extends Vector {
  // ...
}
```

- After

```ts
class Vector {
  isEmpty() {
    // ...
  }
}

class Stack {
  private _vector: Vector;

  isEmpty() {
    return this._vector.isEmpty();
  }
}
```

## 위임을 상속으로 바꾸기 (Replace Delegation with Inheritance)

> 한 클래스가 간단한 여러 메서드들을 모두 다른 클래스의 모든 메서드들에 위임하고 있는 경우

클래스를 현재 위임하고 있는 대상에 대한 상속자로 만들면, 이러한 메서드 위임이 불필요해진다.

위임은 위임이 구현되는 방식을 변경할 수 있고, 다른 클래스 또한 배치할 수 있기 때문에 상속보다 더 유연한 접근 방식이다. 하지만 하나의 클래스와 그 클래스의 모든 공용 메서드에만 작업을 위임한다면, 위임의 이점이 사라진다.

이 경우, 위임을 상속으로 대체하면 수많은 위임 메서드를 정리할 수 있고, 위임 클래스에 새로운 메서드가 추가될 때마다 이에 따라 메서드를 추가로 생성하지 않아도 된다. 이에 따라 결국 코드의 길이도 줄어든다.

다만, 클래스가 위임 클래스의 공용 메서드 중 일부에 대해서만 위임을 하고 있는 경우에는 이 테크닉을 사용할 수 없다. (= 리스코프 치환 원칙 위반) 또한, 부모 클래스가 없는 경우에 대해서만 적용 가능하다.

- Before

```ts
class Employee {
  private _person: Person;

  getName() {
    return this._person.getName();
  }
}

class Person {
  getName() {
    // ...
  }
}
```

- After

```ts
class Person {
  getName() {
    // ...
  }
}

class Employee extends Person {
  // ...
}
```
