# Simplifying Conditional Expressions

---

- [Simplifying Conditional Expressions](#simplifying-conditional-expressions)
  - [조건문 분해하기 (Decompose Conditional Expressions)](#조건문-분해하기-decompose-conditional-expressions)
  - [조건식 통합하기 (Consolidate Conditional Expression)](#조건식-통합하기-consolidate-conditional-expression)
  - [중복 조건부 조각 통합하기 (Consolidate Duplicate Conditional Fragments)](#중복-조건부-조각-통합하기-consolidate-duplicate-conditional-fragments)
  - [제어 플래그 제거 (Remove Control Flag)](#제어-플래그-제거-remove-control-flag)
  - [중첩 조건문을 보호 구문으로 바꾸기 (Replace Nested Conditional with Guard Clauses)](#중첩-조건문을-보호-구문으로-바꾸기-replace-nested-conditional-with-guard-clauses)
  - [조건을 다형성으로 바꾸기 (Replace Conditional with Polymorphism)](#조건을-다형성으로-바꾸기-replace-conditional-with-polymorphism)
  - [Null 객체 도입 (Introduce Null Object)](#null-객체-도입-introduce-null-object)
  - [단언 도입 (Introduce Assertion)](#단언-도입-introduce-assertion)

조건문은 시간이 지남에 따라 점점 더 논리가 복잡해지는 경향이 있으며, 이를 해결하기 위한 기법도 많이 개발되어 왔다.

## 조건문 분해하기 (Decompose Conditional Expressions)

> 복잡한 조건문이 있는 경우 (`if-then`, `else`, `switch`)

- Before

```ts
if (date.before(SUMMER_START) || date.after(SUMMER_END)) {
  charge = quantity * winterRate + winterServiceCharge;
} else {
  charge = quantity * summerRate;
}
```

- After

```ts
if (isSummer(date)) {
  charge = summerCharge(quantity);
} else {
  charge = winterCharge(quantity);
}
```

## 조건식 통합하기 (Consolidate Conditional Expression)

> 동일한 동작을 수행하거나, 동일한 결과를 반환하는 조건문이 여러 개로 나뉘어져 있는 경우

해당 조건문을 하나의 표현식으로 통일한다.

- Before

```ts
disabilityAmount(): number {
  if (seniority < 2) {
    return 0;
  }
  if (monthsDisabled > 12) {
    return 0;
  }
  if (isPartTime) {
    return 0;
  }
  // Compute the disability amount.
  // ...
}
```

- After

```ts
isNotEligibleForDisability(): boolean {
  return seniority < 2 || monthsDisabled > 12 || isPartTime;
}

disabilityAmount(): number {
  if (isNotEligibleForDisability()) {
    return 0;
  }
  // Compute the disability amount.
  // ...
}
```

## 중복 조건부 조각 통합하기 (Consolidate Duplicate Conditional Fragments)

> 조건부의 각 줄기에서 똑같은 코드가 사용되고 있는 경우

조건문의 바깥으로 해당 코드를 빼낸다. 이를 통해 코드 중복을 제거한다.

- Before

```ts
if (isSpecialDeal()) {
  total = price * 0.95;
  send();
}
else {
  total = price * 0.98;
  send();
}
```

- After

```ts
if (isSpecialDeal()) {
  total = price * 0.95;
}
else {
  total = price * 0.98;
}
send();
```

## 제어 플래그 제거 (Remove Control Flag)

> 여러 bool 표현식에 대한 제어 플래그 역할을 수행하는 bool 변수가 존재하는 경우

따로 만든 변수 대신에 `break`, `continue`, `return`을 사용한다.

- Before

```ts
let found = false;

for (const p of people) {
  if (!found) {
    if (p === "Don") {
      sendAlert();
      found = true;
    }
  }
}
```

- After

```ts
for (const p of people) {
  if (p === "Don") {
    sendAlert();
    break;
  }
}
```

## 중첩 조건문을 보호 구문으로 바꾸기 (Replace Nested Conditional with Guard Clauses)

> 중첩된 조건문들이 있으나, 코드 실행의 일반적인 흐름을 파악하기 어려운 경우

모든 특수한 경우와 엣지 케이스들을 주된 흐름 앞쪽으로 위치시킨다. 이상적인 조건문은 차례로 나열된 "평평한" 목록이다.

- Before

```ts
getPayAmount(): number {
  let result: number;
  if (isDead){
    result = deadAmount();
  }
  else {
    if (isSeparated){
      result = separatedAmount();
    }
    else {
      if (isRetired){
        result = retiredAmount();
      }
      else{
        result = normalPayAmount();
      }
    }
  }
  return result;
}
```

- After

```ts
getPayAmount(): number {
  if (isDead){
    return deadAmount();
  }
  if (isSeparated){
    return separatedAmount();
  }
  if (isRetired){
    return retiredAmount();
  }
  return normalPayAmount();
}
```

## 조건을 다형성으로 바꾸기 (Replace Conditional with Polymorphism)

> 객체의 타입 또는 속성에 따라 다른 동작을 수행하는 조건문이 있는 경우

조건 분기와 일치하는 서브클래스를 만들고, 그 안에 공유 메서드를 생성하여 조건문 내의 코드를 해당 메서드로 옮긴다. 이후 해당 조건을 관련 메서드 호출로 변경한다. 이를 통해 객체 클래스에 따른 다형성으로 적절한 구현을 얻을 수 있다.

새로운 객체 속성이나 유형이 나타나면, 유사한 조건문을 모두 검색하여 코드를 추가해야 하는데, 이 경우 모든 메서드에 여러 조건문이 흩어져 있는 경우 상당히 까다로워진다. 이런 상황에서 해당 테크닉이 빛을 발휘할 수 있다.

이 테크닉은 객체의 상태를 물어보고 그를 기반으로 동작을 수행하는 대신, 객체가 수행해야 할 작업을 알려주고 그 방법은 스스로 처리하도록 하는 **Tell-Don't-Ask**(TDA) 원칙을 준수한다.

- Before

```ts
class Bird {
  // ...
  getSpeed(): number {
    switch (type) {
      case EUROPEAN:
        return getBaseSpeed();
      case AFRICAN:
        return getBaseSpeed() - getLoadFactor() * numberOfCoconuts;
      case NORWEGIAN_BLUE:
        return (isNailed) ? 0 : getBaseSpeed(voltage);
    }
    throw new Error("Should be unreachable");
  }
}
```

- After

```ts
abstract class Bird {
  // ...
  abstract getSpeed(): number;
}

class European extends Bird {
  getSpeed(): number {
    return getBaseSpeed();
  }
}
class African extends Bird {
  getSpeed(): number {
    return getBaseSpeed() - getLoadFactor() * numberOfCoconuts;
  }
}
class NorwegianBlue extends Bird {
  getSpeed(): number {
    return (isNailed) ? 0 : getBaseSpeed(voltage);
  }
}

// Somewhere in client code
let speed = bird.getSpeed();
```

## Null 객체 도입 (Introduce Null Object)

> 실제 객체 대신 null을 반환하게 되어 코드의 많은 부분에서 null 체킹을 수행해야 하는 경우

실제 null 대신, 기본적인 동작을 수행하는 null 객체를 반환하도록 한다.

수십번이고 null을 체크하다보면 코드가 길고 지저분해진다.

- Before

```ts
if (customer === null) {
  plan = BillingPlan.basic();
}
else {
  plan = customer.getPlan();
}
```

- After

```ts
class NullCustomer extends Customer {
  isNull(): boolean {
    return true;
  }
  getPlan(): Plan {
    return new NullPlan();
  }
  // Some other NULL functionality.
}

// Replace null values with Null-object.
let customer = (order.customer !== null) ?
  order.customer : new NullCustomer();

// Use Null-object as if it's normal subclass.
plan = customer.getPlan();
```

## 단언 도입 (Introduce Assertion)

> 코드의 일부가 올바르게 동작하기 위해 특정 조건이나 값이 반드시 참이어야 하는 경우

이러한 가정을 구체적인 단언 체크(assertion check)로 대체한다. 단, JS/TS에는 빌트인 assertion이 없으므로, `console.error()` 등을 이용하여 assertion을 구현해야 한다.

- Before

```ts
getExpenseLimit(): number {
  // Should have either expense limit or
  // a primary project.
  return (expenseLimit != NULL_EXPENSE) ?
    expenseLimit:
    primaryProject.getMemberExpenseLimit();
}
```

- After

```ts
getExpenseLimit(): number {
  // TypeScript and JS doesn't have built-in assertions, so we'll use
  // good-old console.error(). You can always extract this into a
  // designated assertion function.
  if (!(expenseLimit !== NULL_EXPENSE ||
       (typeof primaryProject !== 'undefined' && primaryProject))) {
      console.error("Assertion failed: getExpenseLimit()");
  }

  return (expenseLimit !== NULL_EXPENSE) ?
    expenseLimit:
    primaryProject.getMemberExpenseLimit();
}
```
