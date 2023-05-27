# Simplifying Method Calls

- [Simplifying Method Calls](#simplifying-method-calls)
  - [메서드 이름 바꾸기 (Rename Method)](#메서드-이름-바꾸기-rename-method)
  - [매개변수 추가하기 (Add Parameter)](#매개변수-추가하기-add-parameter)
  - [매개변수 제거하기 (Remove Parameter)](#매개변수-제거하기-remove-parameter)
  - [수정자와 쿼리를 분리하기 (Separate Query from Modifier)](#수정자와-쿼리를-분리하기-separate-query-from-modifier)
  - [메서드를 매개변수화하기 (Parameterize Method)](#메서드를-매개변수화하기-parameterize-method)
  - [매개변수를 명시적인 메서드로 바꾸기 (Replace Parameter with Explicit Methods)](#매개변수를-명시적인-메서드로-바꾸기-replace-parameter-with-explicit-methods)
  - [객체 통째로 넘기기 (Preserve Whole Object)](#객체-통째로-넘기기-preserve-whole-object)
  - [매개변수를 메서드으로 바꾸기 (Replace Parameter with Methods)](#매개변수를-메서드으로-바꾸기-replace-parameter-with-methods)
  - [매개변수 객체 도입 (Introduce Parameter Object)](#매개변수-객체-도입-introduce-parameter-object)
  - [세팅 메서드 제거하기 (Remove Setting Method)](#세팅-메서드-제거하기-remove-setting-method)
  - [메서드 숨기기 (Hide Method)](#메서드-숨기기-hide-method)
  - [생성자를 팩토리 메서드로 대체 (Replace Constructor with Factory Method)](#생성자를-팩토리-메서드로-대체-replace-constructor-with-factory-method)
  - [에러 코드를 예외로 대체 (Replace Error Code with Exception)](#에러-코드를-예외로-대체-replace-error-code-with-exception)
  - [예외를 테스트로 대체 (Replace Exception with Test)](#예외를-테스트로-대체-replace-exception-with-test)

이러한 종류의 테크닉들은 메서드 호출을 단순화하고 코드를 더 이해하기 쉽게 만들어준다. 이는 결국 클래스 간의 상호작용을 위한 인터페이스를 단순화한다.

## 메서드 이름 바꾸기 (Rename Method)

> 메서드 이름이 메서드가 어떤 작업을 하는지 설명하지 못하는 경우

메서드 이름을 새로 짓자.

메서드는 애초에 잘못된 이름이었을 수도 있고, 시간이 지나면서 이름이 충분한 설명이 되지 못하게 되었을 수도 있다.

- Before

```ts
class Customer {
  getsnm() {
    // ...
  }
}
```

- After

```ts
class Customer {
  getSecondName() {
    // ...
  }
}
```

## 매개변수 추가하기 (Add Parameter)

> 메서드가 특정 작업을 수행하는 데에 충분한 데이터를 갖고 있지 않는 경우

새로운 매개변수를 추가하여 필요한 데이터를 전달하자.

이러한 상황에서는 앞서 설명한 것처럼 매개변수를 추가하거나, 새로운 프라이빗 필드를 추가하는 방법을 고려할 수 있다.

일반적으로 매개변수를 추가하는 것은 데이터가 가끔씩만 필요하거나, 데이터가 자주 바뀌어 객체에 항상 보유할 필요가 없는 경우에 효과적이다. 그렇지 않은 상황이라면 비공개 필드를 추가하는 방법이 더 좋을 수 있다.

- Before

```ts
class Customer {
  getContact() {
    // ...
  }
}
```

- After

```ts
class Customer {
  getContact(date: Date) {
    // ...
  }
}
```

## 매개변수 제거하기 (Remove Parameter)

> 매개변수가 더 이상 메서드 바디에서 사용되지 않을 때

사용되지 않는 매개변수는 제거한다.

- Before

```ts
class Customer {
  getContact(date: Date) {
    // ...
  }
}
```

- After
  
```ts
class Customer {
  getContact() {
    // ...
  }
}
```

## 수정자와 쿼리를 분리하기 (Separate Query from Modifier)

> 값을 반환하면서 동시에 객체 내부의 무엇인가를 변경하는 메서드가 있는 경우

메서드를 별개의 동작을 수행하는 각각의 메서드로 분할한다. 각 메서드는 하나의 역할만 수행해야 한다.

다만, 어떤 경우에는 특정한 수정 명령을 수행한 후, 데이터를 가져오는 것이 편리할 때도 있다. (ex. DB에서 무엇인가를 삭제할 때, 얼마나 많은 행이 삭제되었는지 알고 싶은 경우)

- Before

```ts
class Customer {
  getTotalOutstandingAndSetReadyForSummaries() {
    // ...
  }
}
```

- After

```ts
class Customer {
  getTotalOutstanding() {
    // ...
  }

  setReadyForSummaries() {
    // ...
  }
}
```

## 메서드를 매개변수화하기 (Parameterize Method)

> 여러 메서드가 내부적으로 사용되는 값, 숫자 또는 연산만 다른 유사한 작업을 수행하는 경우

필요한 특정 값을 매개변수로 넘겨받는 형태의 메서드를 만들어 기존의 메서드를 대체한다.

유사한 메서드가 있다면 코드 중복의 가능성이 높으며, 이는 코드를 수정할 때 불필요한 작업을 유발한다.

- Before

```ts
class Employee {
  tenPercentRaise() {
    this.salary *= 1.1;
  }

  fivePercentRaise() {
    this.salary *= 1.05;
  }
}
```

- After

```ts
class Employee {
  raise(percentage: number) {
    this.salary *= 1 + percentage;
  }
}
```

## 매개변수를 명시적인 메서드로 바꾸기 (Replace Parameter with Explicit Methods)

> 메서드가 여러 부분으로 나뉘어 있고, 각 부분의 실행이 매개변수의 값에 의존하고 있는 경우

메서드의 각 부분들을 별개의 메서드로 나눈다.

단, 메서드가 거의 변경되지 않고, 그 안에서 새로운 변형이 추가되지 않는 경우에는 이 리팩터링을 적용하지 않는다.

- Before

```ts
 setValue(name: string, value: number): void {
  if (name.equals("height")) {
    height = value;
    return;
  }
  if (name.equals("width")) {
    width = value;
    return;
  }
}
```

- After

```ts
setHeight(value: number): void {
  height = value;
}
setWidth(value: number): number {
  width = value;
}
```

## 객체 통째로 넘기기 (Preserve Whole Object)

> 객체에서 여러 개의 값을 따로 가져와 메서드의 매개변수로 전달하고 있는 경우

그냥 객체 전체를 통째로 넘기자.

반면, 이 경우 특정 인터페이스를 갖춘 객체만 사용할 수 있도록 제한되기 때문에 메서드의 유연성이 다소 떨어지게 되는 경우가 있다.

- Before

```ts
let low = daysTempRange.getLow();
let high = daysTempRange.getHigh();
let withinPlan = plan.withinRange(low, high);
```

- After

```ts
let withinPlan = plan.withinRange(daysTempRange);
```

## 매개변수를 메서드으로 바꾸기 (Replace Parameter with Methods)

> (메서드 내에서 호출되어도 상관없는) 쿼리 메서드를 사용해 그 결과를 그대로 메서드의 매개변수로 전달하고 있는 경우

매개변수로 해당 값을 전달하는 대신, 메서드 내부에서 직접 해당 쿼리를 호출하도록 하자.

- Before

```ts
let basePrice = quantity * itemPrice;
const seasonDiscount = this.getSeasonalDiscount();
const fees = this.getFees();
const finalPrice = discountedPrice(basePrice, seasonDiscount, fees);
```

- After

```ts
let basePrice = quantity * itemPrice;
let finalPrice = discountedPrice(basePrice);
```

## 매개변수 객체 도입 (Introduce Parameter Object)

> 매개변수의 특정 그룹이 반복적으로 메서드에 전달되는 경우

해당 매개변수들을 하나의 객체로 묶어 전달한다.

다만, 단순히 객체를 위해 새로운 클래스에 데이터만을 옮겨놓고 별다른 동작이나 관련 작업을 추가할 계획이 없다면, [데이터 클래스](../../code-smells/dispensables/index.md#데이터-클래스-data-class) 냄새가 나기 시작할 것이다.

```ts
amountInvoiced(startDate: Date, endDate: Date) {
  // ...
}

amountReceived(startDate: Date, endDate: Date) {
  // ...
}

amountOverdue(startDate: Date, endDate: Date) {
  // ...
}
```

- After

```ts
amountInvoiced(date: DateRange) {
  // ...
}

amountReceived(date: DateRange) {
  // ...
}

amountOverdue(date: DateRange) {
  // ...
}
```

## 세팅 메서드 제거하기 (Remove Setting Method)

> 필드의 값은 필드를 생성할 때만 설정되어야 하고, 이후에는 변경하지 않아야 한다.

따라서, 필드값을 설정하는 메서드를 삭제한다.

이를 통해 필드 값의 변경을 방지해야 한다.

- Before

```ts
class Customer {
  setImmutableValue() {
    // ...
  }
}
```

- After

```ts
class Customer {
  // ...
}
```

## 메서드 숨기기 (Hide Method)

> 다른 클래스에서 사용하지 않으며, 오직 클래스 구조 내부에서만 사용되는 메서드가 있는 경우

메서드를 private 또는 protected로 설정한다.

메서드를 은닉하면 코드의 확장이 더 쉬워진다. private 메서드의 코드를 변경할 때는, 어차피 클래스 외부에서는 사용할수 없다는 것을 알기 떄문에, 현재 클래스에 대한 영향만 걱정하면 되기 때문이다.

- Before

```ts
class Employee {
  aMethod() {
    // ...
  }
}
```

- After

```ts
class Employee {
  private aMethod() {
    // ...
  }
}
```

## 생성자를 팩토리 메서드로 대체 (Replace Constructor with Factory Method)

> 객체 필드에 단순히 값을 설정하는 것 이상으로 복잡한 로직을 가진 생성자가 있는 경우

생성자를 팩토리 메서드로 대체하여, 생성자 호출을 대신한다.

해당 테크닉은 [타입 코드를 서브클래스로 대체](../organizing-data/index.md#타입-코드를-서브클래스로-대체-replace-type-code-with-subclasses)하는 상황과도 관련이 있다. 리팩터링을 적용한 이후 생겨난 여러 서브클래스 객체들을 타입 코드에 따라 적절하게 반환하는 생성자가 필요한데, 이것을 기존의 생성자를 통해 구현하는 것은 불가능하므로, 이를 수행하는 정적 팩토리 메서드를 생성하여 기존 생성자의 역할을 대신하게 할 수 있다.

해당 리팩터링은 생성자를 사용하기 적합하지 않은 많은 상황에서 유용하다. 이를테면, 값을 참조로 변경하고자 할 때, 또한 매개변수의 수와 유형을 넘어서는 다양한 생성 모드를 갖추고자 할 때도 사용할 수 있다.

- Before

```ts
class Employee {
  constructor(type: number) {
    this.type = type;
  }
  // ...
}
```

- After

```ts
class Employee {
  static create(type: number): Employee {
    switch (type) {
      case 1:
        return new Engineer();
      case 2:
        return new Salesman();
      default:
        throw new Error("Incorrect type code value");
    }
  }
  // ...
}
```

## 에러 코드를 예외로 대체 (Replace Error Code with Exception)

> 메서드가 에러를 의미하는 임의의 특별한 값을 반환하고 있는 경우

그냥 예외를 던지자.

오류 코드의 반환은 절차적 프로그래밍에서 더 이상 사용되지 않는 방식이다. 모던 프로그래밍에서 오류 처리는 예외라는 이름의 특수 클래스에 의해 수행된다.

- Before

```ts
withdraw(amount: number): number {
  if (amount > _balance) {
    return -1;
  }
  else {
    balance -= amount;
    return 0;
  }
}
```

- After

```ts
withdraw(amount: number): void {
  if (amount > _balance) {
    throw new Error();
  }
  balance -= amount;
}
```

## 예외를 테스트로 대체 (Replace Exception with Test)

> 간단한 테스트로도 충분한 상황에 예외를 던지고 있는 경우

예외를 조건부 테스트로 수정한다.

예외는 예상치 못한 오류와 관련된 불규칙한 동작을 처리하는 데에 사용되어야 하지, 테스트를 대체해서는 안 된다. 실행하기 전에 조건을 간단히 확인하여 예외를 피할 수 있는 상황이라면, 그렇게 처리하라. 예외는 "진짜 오류"를 위해 남겨두어야 한다.

- Before

```ts
getValueForPeriod(periodNumber: number): number {
  try {
    return values[periodNumber];
  } catch (ArrayIndexOutOfBoundsException e) {
    return 0;
  }
}
```

- After

```ts
getValueForPeriod(periodNumber: number): number {
  if (periodNumber >= values.length) {
    return 0;
  }
  return values[periodNumber];
}
```
