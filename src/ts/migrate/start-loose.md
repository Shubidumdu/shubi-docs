# 마이그레이션의 완성을 위해 noImplicitAny 설정하기

프로젝트 전체를 `.ts`로 전환했다면 매우 큰 진척을 이룬 것이지만, 마지막 단계로 `noImplicitAny`를 설정하는 것이 필요합니다.
`noImplicitAny`가 설정되지 않은 상태에서는 타입 선언에서 비롯된 실제 오류가 숨어있기 때문에 마이그레이션이 완료되었다고 할 수 없습니다.

예를 들어, 아래와 같이 `indices`라는 속성을 가진 클래스 `Chart`가 있다고 했을 때, `noImplicitAny`가 설정되어 있지 않다면 다음과 같이 작성되더라도 문제가 없습니다.

```ts
class Chart {
  indices: number[];
  // ...
  getRanges() {
    for (const r of this.indices) {
      const low = r[0];  // Type is any
      const high = r[1];  // Type is any
      // 에러가 출력되지 않습니다.
      // ...
    }
  }
}
```

만약, `noImplicitAny`가 설정되어 있다면 다음과 같이 `any`에 대한 에러가 제대로 발생합니다.

```ts
class Chart {
  indices: number[];
  // ...
  getRanges() {
    for (const r of this.indices) {
      const low = r[0];
              // ~~~~ Element implicitly has an 'any' type because
              //      type 'Number' has no index signature
      const high = r[1];
                // ~~~~ Element implicitly has an 'any' type because
                //      type 'Number' has no index signature
      // ...
    }
  }
}
```

처음에는 `noImplicitAny`를 로컬에만 설정하고 작업하는 것이 좋습니다.
원격 상에서는 설정에 변화가 없어 빌드에 실패하지 않을 것이기 때문입니다.

그 외에도 타입 체크의 강도를 높이는 설정에는 여러 가지가 있습니다.
지금껏 이야기한 `noImplicitAny`는 상당히 엄격한 설정이며, `strictNullChecks` 같은 설정을 적용하지 않더라도 대부분의 타입 체크를 적용한 것으로 볼 수 있습니다.

최종적으로 강력한 설정은 `"strict": true`이며, 타입 체크의 강도는 팀 내의 모든 사람이 TS에 익숙해진 다음에 조금씩 높여가는 것이 좋습니다.
