# Couplers

![Couplers](https://refactoring.guru/images/refactoring/content/catalog/couplers-2x.png?id=86e33d80273d564bd4d48a554167a7c9)

해당 범주의 냄새들은 모두 클래스 간의 과한 커플링에 기여하거나, 커플링이 과도한 위임으로 대체될 때 어떤 일들이 발생하는지 보여준다.

## 기능 질투 (Feature Envy)

> 어떤 메서드가 본인의 데이터보다도 다른 객체의 데이터에 더 액세스하는 경우

## 부적절한 친밀감 (Inappropriate Intimacy)

> 어떤 클래스가 다른 클래스의 내부 필드와 메서드를 사용하는 경우

## 메시지 체인 (Message Chains)

> 코드에서 `$a -> b() -> c() -> d()`와 같은 형태의 일련의 호출이 보이는 경우

## 미들 맨 (Middle Man)

> 하나의 클래스가 단 하나의 작업만 수행하고, 그 외의 모든 작업은 다른 클래스에 위임하고 있는 경우
