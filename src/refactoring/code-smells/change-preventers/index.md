# Change Preventers

![change-preventers](https://refactoring.guru/images/refactoring/content/catalog/change-preventers-2x.png)

이러한 종류의 냄새는 한 곳에서 어떤 변경이 일어날 경우 다른 곳에서도 많은 변경을 해야한다는 것을 의미한다.
결과적으로 프로그램 개발이 훨씬 더 복잡해지고 비용이 많이 들게 된다.

## 일치하지 않는 변경 (Divergent Change)

> 클래스에 변경이 생길 때 관련없는 여러 메서드들에도 변경이 요구되는 경우가 있을 수 있다. 예를 들어, 새로운 프로덕트 타입을 추가하게 되면 제품을 찾고/표시하고/주문하는 메서드에도 변경이 요구된다.

## 샷건 수술 (Shotgun Surgery)

> 어떤 하나의 변경이 여러 다른 클래스에 여러 작은 변경들을 요구하는 경우.

## 병렬 상속 구조 (Parellel Inheritance Hierarchies)

> 클래스의 서브클래스를 만들 때마다, 다른 클래스에 대해서도 서브클래스를 만들어야 하는 경우
