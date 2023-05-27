# Moving Features Between Objects

___

- [Moving Features Between Objects](#moving-features-between-objects)
  - [메서드 이동 (Move Method)](#메서드-이동-move-method)
  - [필드 이동 (Move Field)](#필드-이동-move-field)
  - [클래스 추출 (Extract Class)](#클래스-추출-extract-class)
  - [인라인 클래스 (Inline Class)](#인라인-클래스-inline-class)
  - [위임 숨기기 (Hide Delegate)](#위임-숨기기-hide-delegate)
  - [미들맨 제거하기 (Remove Middle Man)](#미들맨-제거하기-remove-middle-man)
  - [외래 메서드 도입 (Introduce Foreign Method)](#외래-메서드-도입-introduce-foreign-method)
  - [로컬 확장 도입 (Introduce Local Extension)](#로컬-확장-도입-introduce-local-extension)

완벽하지 않은 방식으로 여러 클래스에 기능을 분산시킨 상황이어도, 여전히 희망은 있다.

이 범주의 리팩터링 기법은 클래스 간에 기능을 안전하게 이동시키고, 새 클래스를 만들고, 공개 액세스에서 구현 디테일을 숨기는 방법을 보여준다.

## 메서드 이동 (Move Method)

> 어떤 메서드가 본인 클래스보다도 다른 클래스에서 더 많이 사용되는 경우 

해당 메서드를 가장 많이 사용하는 클래스에 새 메서드를 만들고 기존 코드를 이동시킨다. 기존 메서드의 코드는 다른 클래스의 새 메서드에 대한 참조로 바꾸거나, 완전히 제거한다.

## 필드 이동 (Move Field)

> 어떤 필드가 본인 클래스보다도 다른 클래스에서 더 많이 사용되는 경우

이를 해당 클래스의 필드로 옮기고, 기존의 필드를 사용하던 코드를 새 필드를 사용하도록 수정한다.

이는 [클래스 추출](#클래스-추출-extract-class)의 일부로 활용되는 경우가 많으며, 종종 필드를 어느 클래스에 두어야하는지 결정하는 것이 꽤 어려울 수 있다. 하나의 룰을 정하자면, **필드를 사용하는 메서드와 같은 위치에 필드를 두는 것**이 좋다.

## 클래스 추출 (Extract Class)

> 하나의 클래스가 두 클래스의 작업을 모두 수행함에 따라 어색함이 생기는 경우

대신에 새 클래스를 만들고 그 안에 관련 기능을 담당하는 필드와 메서드를 배치하자.

클래스는 항상 처음에는 명확하고 이해하기 쉽게 시작하지만, 프로그램의 확장에 따라 점점 메서드와 필드가 추가되면서 일부 클래스가 필요 이상으로 많은 책임을 갖게된다.

해당 리팩터링은 **단일 책임 원칙**(Single Responsible Principle)을 준수하는데 도움이 된다.

## 인라인 클래스 (Inline Class)

> 클래스가 거의 아무 일도 하고 있지 않고, 어떤 것에도 책임이 없으며, 어떤 추가적인 책임도 계획된 바가 없는 경우

해당 클래스는 따로 있을 필요가 없는 것이다. 이 경우 클래스의 모든 기능을 다른 클래스에 합쳐 넣고, 원래의 빈 클래스는 삭제한다.

이는 종종 한 클래스의 기능이 다른 클래스로 이전되면서, 기존 클래스가 하는 일이 거의 없어진 경우에 발생한다.

## 위임 숨기기 (Hide Delegate)

> 클라이언트가 객체 A의 필드 또는 메서드를 통해 객체 B를 가져와, 이후 객체 B의 메서드를 호출하는 경우

이 경우, 클래스 A에 객체 B에 대한 호출을 위임하는 새 메서드를 생성한다. 이를 통해 클라이언트가 클래스 B에 의존하지 않는 형태로 만들 수 있다.

클라이언트로부터 위임을 숨기면, 클라이언트 코드가 객체 간의 관계에 대해 알아야할 이유가 없어진다. 이러한 세부 정보에 대해 알 필요가 없어지면, 프로그램을 더 쉽게 변경할 수 있다.

## 미들맨 제거하기 (Remove Middle Man)

> 클래스 내에 단순히 다른 객체에게 위임하는 메서드가 너무 많은 경우

해당 클래스 내 이러한 메서드들을 삭제하고 클라이언트가 직접 해당 객체를 사용하도록 수정한다.

서버 클래스가 자체적으로는 아무 일도 수행하지 않고 불필요하게 복잡성만 유발하는 경우에는, 이 클래스가 꼭 필요한지 먼저 생각해보아야 한다. 또, 위임에 새로운 기능이 추가될 때마다 서버 클래스에서 해당 기능에 대한 메서드를 추가해야 하기 때문에, 변경사항이 발생할 때마다 번거로울 수 있다.

## 외래 메서드 도입 (Introduce Foreign Method)

> 유틸리티 클래스에 필요한 메서드가 포함되어 있지 않은데, 해당 메서드를 클래스에 추가하는 것도 불가능한 상황인 경우

클라이언트 클래스에 메서드를 추가하고, 유틸리티 클래스의 객체를 해당 메서드의 argument로 넘긴다.

이를테면, 아래의 `nextDay`의 경우, 유틸리티인 Date 클래스에 포함되는 것이 이상적이지만, 네이티브 클래스인 Date에 이를 추가하는 것이 어려우므로 클라이언트 클래스인 `Report`에 이 `nextDay` 메서드를 추가하고, 매개변수 및 반환값으로 `Date` 객체를 사용한다.

이는 메서드가 적절한 위치에 있는 것이 아니더라도, 그를 통한 중복 제거가 더 유용하다고 보는 관점으로 적용하는 리팩터링이다.

다만, 기본적으로 이는 메서드를 적절한 클래스 위치에 두는 것이 아니므로, 아래의 [로컬 확장 도입](#로컬-확장-도입-introduce-local-extension)을 사용하는 편이 일반적으로는 더 좋다.

- Before

```ts
class Report {
  // ...
  sendReport(): void {
    let nextDay: Date = new Date(previousEnd.getYear(),
      previousEnd.getMonth(), previousEnd.getDate() + 1);
    // ...
  }
}
```

- After

```ts
class Report {
  // ...
  sendReport() {
    let newStart: Date = nextDay(previousEnd);
    // ...
  }

  private static nextDay(arg: Date): Date {
    return new Date(arg.getFullYear(), arg.getMonth(), arg.getDate() + 1);
  }
}
```

## 로컬 확장 도입 (Introduce Local Extension)

> 유틸리티 클래스에 필요한 메서드가 포함되어 있지 않은데, 해당 메서드를 클래스에 추가하는 것도 불가능한 상황인 경우

메서드를 포함하는 새 클래스를 만들고, 유틸리티 클래스의 서브클래스 또는 래퍼로 해당 클래스를 사용한다.

1. **서브 클래스**로 사용하는 경우, 부모로부터의 모든 것을 상속받기 때문에 쉬운 방법이지만, 유틸리티 클래스 자체적으로 이것이 차단되어 있는 경우에는 적용이 어렵다.

2. **래퍼 클래스**로 사용하는 경우, 모든 새 메서드를 포함하는 래퍼 클래스를 만들고, 그 외의 기본 사양은 원본 객체에 위임한다. 이는 객체 간 관계를 유지하기 위한 코드를 작성해야 할 뿐만 아니라, 유틸리티 클래스로 위임하는 여러 메서드들을 추가해야하기 때문에 보다 번거로운 작업이 된다.

- Before

```ts
class ClientClass {
  // ...
  private static nextDay(arg: Date): Date {
    return new Date(arg.getFullYear(), arg.getMonth(), arg.getDate() + 1);
  }
}
```

- After

```ts
class MfDate extends Date {
  // ...
  nextDay(): Date {
    return new Date(this.getFullYear(), this.getMonth(), this.getDate() + 1);
  }
}
```

