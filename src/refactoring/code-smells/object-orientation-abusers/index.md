# Object-Orientation Abusers

이러한 류의 코드 스멜은 객체 지향 프로그래밍(OOP)의 불완전하거나 잘못된 적용에 대한 것이다.

## 스위치 문(Switch Statements)

> 복잡한 switch 문 또는 if 문 시퀀스가 존재하는 경우

## 임시 필드(Temporary Field)

> 임시 필드는 오직 특정한 상황에서만 값을 가져오고, 그 외의 경우에는 항상 비어있다.

## 거부된 유증(Refused Bequest)

> 만약 하나의 서브클래스가 부모로부터 상속받은 일부 메서드와 프로퍼티만을 사용한다면, 계층 구조가 엉망이 된다. 불필요한 메서드는 단순히 사용되지 않거나, 재정의됨에 따라 예외를 발생시킬 수 있다.

## 인터페이스가 다른 대체 클래스 (Alternative Classes with Different Interfaces)

> 두 클래스가 동일한 기능을 수행하지만, 메서드 이름만 다른 경우
