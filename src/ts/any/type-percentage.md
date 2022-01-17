# 타입 커버리지를 추적하여 타입 안정성 유지하기

`noImplicitAny`를 설정하더라도, 여전히 `any` 타입은 프로그램 내에 존재할 수 있습니다.

- 명시적 `any` 타입
- 서드파티 타입 선언 (`@types`)

`any` 타입은 프로그램 전반에 부정적 영향을 끼칠 수 있으므로 개수를 추적하는 것이 좋습니다.
다음의 `type-coverage` 패키지를 활용하면 `any`를 추적할 수 있습니다.

```zsh
npx type-coverage
```

`--detail` 플래그를 붙이면, `any` 타입이 있는 곳을 전부 출력해줍니다.

```zsh
npx type-coverage --detail
```
