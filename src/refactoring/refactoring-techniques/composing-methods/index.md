# Composing Methods

___

- [Composing Methods](#composing-methods)
  - [메서드 추출하기 (Extract Method)](#메서드-추출하기-extract-method)
  - [인라인 메서드 (Inline Method)](#인라인-메서드-inline-method)
  - [변수 추출 (Extract Variable)](#변수-추출-extract-variable)
  - [인라인 임시변수 (Inline Temp)](#인라인-임시변수-inline-temp)
  - [임시변수를 쿼리로 전환 (Replace Temp with Query)](#임시변수를-쿼리로-전환-replace-temp-with-query)
  - [임시변수 쪼개기 (Split Temporary Variable)](#임시변수-쪼개기-split-temporary-variable)
  - [매개변수에 대한 할당 제거하기 (Remove Assignments to Parameters)](#매개변수에-대한-할당-제거하기-remove-assignments-to-parameters)
  - [메서드를 메서드 객체로 바꾸기 (Replace Method with Method Object)](#메서드를-메서드-객체로-바꾸기-replace-method-with-method-object)
  - [알고리즘 대체하기 (Substitute Algorithm)](#알고리즘-대체하기-substitute-algorithm)

리팩터링의 대부분은 메서드를 올바르게 구성하는 데에 할애하게 된다. 대부분의 경우 지나치게 긴 메서드는 만악의 근원이다. 이러한 메서드 내부의 모호한 코드는 실행 로직을 숨기고, 메서드를 이해하기 매우 어렵게 만들며 변경하기는 더욱 어렵게 만든다.

이 그룹의 리팩터링 기법은 메서드를 간소화하고 코드 중복을 제거하며, 향후 개선을 위한 기반을 마련한다.

---

## 메서드 추출하기 (Extract Method)

하나로 그룹화될 수 있는 코드조각을 별도의 메서드로 추출해낸다. 이는 다른 여러 리팩터링의 기초가 되기도 한다.

- Before

```ts
printOwing(): void {
  printBanner();

  // Print details.
  console.log("name: " + name);
  console.log("amount: " + getOutstanding());
}
```

- After

```ts
printOwing(): void {
  printBanner();
  printDetails(getOutstanding());
}

printDetails(outstanding: number): void {
  console.log("name: " + name);
  console.log("amount: " + outstanding);
}
```

## 인라인 메서드 (Inline Method)

메서드의 코드 내용 자체가 메서드의 이름보다 오히려 더 명확한 경우는 차라리 해당 메서드를 없애는 쪽이 더 분명하다.

- Before

```ts
class PizzaDelivery {
  getRating(): number {
    return moreThanFiveLateDeliveries() ? 2 : 1;
  }

  moreThanFiveLateDeliveries(): boolean {
    return numberOfLateDeliveries > 5;
  }
}
```

- After

```ts
class PizzaDelivery {
  // ...
  getRating(): number {
    return numberOfLateDeliveries > 5 ? 2 : 1;
  }
}
```

## 변수 추출 (Extract Variable)

이해하기에 어려운 표현식(expression)이 있을때는, 해당 표현식의 결과 또는 그 일부를 별도의 변수로 추출하여 이를 잘 설명하는 적절한 이름을 붙이자.

- Before

```ts
renderBanner(): void {
  if ((platform.toUpperCase().indexOf("MAC") > -1) &&
       (browser.toUpperCase().indexOf("IE") > -1) &&
        wasInitialized() && resize > 0 )
  {
    // do something
  }
}
```

- After

```ts
renderBanner(): void {
  const isMacOs = platform.toUpperCase().indexOf("MAC") > -1;
  const isIE = browser.toUpperCase().indexOf("IE") > -1;
  const wasResized = resize > 0;

  if (isMacOs && isIE && wasInitialized() && wasResized) {
    // do something
  }
}
```

## 인라인 임시변수 (Inline Temp)

오직 간단한 표현식의 결과를 할당하기만 할 뿐, 그 외에 아무 일도 하지 않는 임시변수가 있을 때는, 해당 임시변수를 없애고 인라인으로 대체한다.

해당 리팩터링 테크닉은 그 자체로는 거의 이점이 없다. 하지만 불필요한 변수를 제거하여 가독성을 약간 향상시킬 수 있다.

단, 이러한 임시변수가 캐싱의 역할을 하고 있는 상황이라면 해당 리팩터링이 부적절할 수 있으니, 성능에 영향을 주지 않는지 확인이 필요하다.

- Before

```ts
hasDiscount(order: Order): boolean {
  let basePrice: number = order.basePrice();
  return basePrice > 1000;
}
```

- After

```ts
hasDiscount(order: Order): boolean {
  return order.basePrice() > 1000;
}
```

## 임시변수를 쿼리로 전환 (Replace Temp with Query)

표현식의 결과를 코드에서 사용하기 위해 로컬 변수로 두고있는 경우, 그 표현식 전체를 쿼리 메서드로 전환하여, 임시변수를 제거하고 해당 메서드 호출로 대체한다.

- Before

```ts
calculateTotal(): number {
  let basePrice = quantity * itemPrice;
  if (basePrice > 1000) {
    return basePrice * 0.95;
  }
  else {
    return basePrice * 0.98;
  }
}
```

- After

```ts
calculateTotal(): number {
  if (basePrice() > 1000) {
    return basePrice() * 0.95;
  }
  else {
    return basePrice() * 0.98;
  }
}

basePrice(): number {
  return quantity * itemPrice;
}
```

## 임시변수 쪼개기 (Split Temporary Variable)

메서드 내부에 여러개의 중간값을 저장하기 위해 사용하는 로컬 변수가 있는 경우, 이를 아예 다른 값으로 분리한다. 각각의 변수는 오직 하나의 역할만 수행해야 한다.

- Before

```ts
let temp = 2 * (height + width);
temp = height * width;
```

- After

```ts
const perimeter = 2 * (height + width);
const area = height * width;
```

## 매개변수에 대한 할당 제거하기 (Remove Assignments to Parameters)

메서드 내부에서 파라미터를 직접 변경하지 말고, 별도의 임시변수를 두어야한다.

- Before

```ts
discount(inputVal: number, quantity: number): number {
  if (quantity > 50) {
    inputVal -= 2;
  }
  // ...
}
```

- After

```ts
discount(inputVal: number, quantity: number): number {
  let result = inputVal;
  if (quantity > 50) {
    result -= 2;
  }
  // ...
}
```

## 메서드를 메서드 객체로 바꾸기 (Replace Method with Method Object)

로컬 변수가 너무 복잡하게 얽혀 있어 [메서드 추출](#메서드-추출하기-extract-method)을 하기 어려울 때는, 메서드를 별도의 클래스로 전환하여 로컬 변수가 클래스의 필드가 될 수 있도록 처리한다. 이후 전환한 클래스 내 여러 메서드로 작업을 쪼갤 수 있다.

이를 통해 긴 메서드 로직을 자체 클래스로부터 분리하여 크기가 커지는 것을 막을 수 있고, 유틸리티 메서드로 기존 클래스를 오염시키지 않고 별도로 분리할 수 있다.

다만 이 경우, 새로운 클래스를 새로 추가하는 것이기 때문에, 프로그램의 전반적인 복잡성은 증가한다.

- Before

```ts
class Order {
  // ...
  price(): number {
    let primaryBasePrice;
    let secondaryBasePrice;
    let tertiaryBasePrice;
    // Perform long computation.
  }
}
```

- After

```ts
class Order {
  // ...
  price(): number {
    return new PriceCalculator(this).compute();
  }
}

class PriceCalculator {
  private _primaryBasePrice: number;
  private _secondaryBasePrice: number;
  private _tertiaryBasePrice: number;
  
  constructor(order: Order) {
    // Copy relevant information from the
    // order object.
  }
  
  compute(): number {
    // Perform long computation.
  }
}
```

## 알고리즘 대체하기 (Substitute Algorithm)

기존의 알고리즘이 부적절하다고 생각된다면, 새로운 알고리즘으로 대체한다.

**점진적 리팩터링**(Gradual Refactoring)만이 프로그램을 개선하는 유일한 방법은 아니다.

1. 때로는 메서드에 너무 많은 문제점이 있는 경우, 아예 새로 시작하는 것이 더 좋을 수도 있다. 또한 훨씬 더 간단하고 효율적인 알고리즘을 발견한 경우에도 이를 대체할 수 있다.

2. 시간이 지나면서 잘 구축된 라이브러리나 프레임워크를 사용하여 유지관리를 간소화하도록 할 수 있다.

3. 프로그램의 요구사항이 너무 많이 변경되어 기존 알고리즘으로 작업을 처리할 수 없을 수도 있다.

- Before

```ts
foundPerson(people: string[]): string{
  for (let person of people) {
    if (person.equals("Don")){
      return "Don";
    }
    if (person.equals("John")){
      return "John";
    }
    if (person.equals("Kent")){
      return "Kent";
    }
  }
  return "";
}
```

- After

```ts
foundPerson(people: string[]): string{
  let candidates = ["Don", "John", "Kent"];
  for (let person of people) {
    if (candidates.includes(person)) {
      return person;
    }
  }
  return "";
}
```
