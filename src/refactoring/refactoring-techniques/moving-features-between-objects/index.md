# Moving Features Between Objects

___

- [Moving Features Between Objects](#moving-features-between-objects)
  - [메서드 이동 (Move Method)](#메서드-이동-move-method)
  - [필드 이동 (Move Field)](#필드-이동-move-field)
  - [클래스 추출 (Extract Class)](#클래스-추출-extract-class)

완벽하지 않은 방식으로 여러 클래스에 기능을 분산시킨 상황이어도, 여전히 희망은 있다.

이 범주의 리팩터링 기법은 클래스 간에 기능을 안전하게 이동시키고, 새 클래스를 만들고, 공개 액세스에서 구현 디테일을 숨기는 방법을 보여준다.

## 메서드 이동 (Move Method)

어떤 메서드가 본인 클래스보다도 다른 클래스에서 더 많이 사용되는 경우, 해당 메서드를 가장 많이 사용하는 클래스에 새 메서드를 만들고 기존 코드를 이동시킨다. 기존 메서드의 코드는 다른 클래스의 새 메서드에 대한 참조로 바꾸거나, 완전히 제거한다.

## 필드 이동 (Move Field)

어떤 필드가 본인 클래스보다도 다른 클래스에서 더 많이 사용되는 경우, 이를 해당 클래스의 필드로 옮기고, 기존의 필드를 사용하던 코드를 새 필드를 사용하도록 수정한다.

이는 [클래스 추출](#클래스-추출-extract-class)의 일부로 활용되는 경우가 많으며, 종종 필드를 어느 클래스에 두어야하는지 결정하는 것이 꽤 어려울 수 있다. 하나의 룰을 정하자면, **필드를 사용하는 메서드와 같은 위치에 필드를 두는 것**이 좋다.

## 클래스 추출 (Extract Class)

하나의 클래스가 두 클래스의 작업을 모두 수행하고 있으면 어색함이 생긴다. 대신에 새 클래스를 만들고 그 안에 관련 기능을 담당하는 필드와 메서드를 배치하자.

클래스는 항상 처음에는 명확하고 이해하기 쉽게 시작하지만, 프로그램의 확장에 따라 점점 메서드와 필드가 추가되면서 일부 클래스가 필요 이상으로 많은 책임을 갖게된다.

해당 리팩터링은 **단일 책임 원칙**(Single Responsible Principle)을 준수하는데 도움이 된다.

- Before

```ts
class Person {
  name: string;
  officeAreaCode: string;
  officeNumber: string;

  getTelephoneNumber() {
    return `(${this.officeAreaCode}) ${this.officeNumber}`;
  }
}
```

- After

```ts
class Person {
  name: string;
  private _officeTelephoneNumber: TelephoneNumber;

  // ...

  getOfficeTelephoneNumber() {
    return this._telephoneNumber.getTelephoneNumber();
  }
}

class TelephoneNumber {
  private _areaCode: string;
  private _number: string;

  // ...

  getTelephoneNumber() {
    return `(${this._areaCode}) ${this._number}`;
  }
}
```
