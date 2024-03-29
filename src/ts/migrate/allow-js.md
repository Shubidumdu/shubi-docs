# allowJs로 타입스크립트와 자바스크립트 같이 사용하기

프로젝트의 규모가 큰 경우, 한꺼번에 모든 JS 코드를 TS로 전환하는 것이 불가능하므로, 점진적인 전환 과정이 필요합니다.
그러러면 마이그레이션 기간 중에 TS와 JS가 동시에 동작할 수 있도록 하는 것이 필요합니다.

이것의 핵심은 `allowJs` 컴파일러 옵션인데, 이는 TS와 JS 파일을 서로 임포트할 수 있게 해줍니다.

번들러에 TS가 통합되어 있거나, 플러그인 방식으로 통합이 가능하다면 이를 쉽게 적용할 수 있습니다.

예를 들어, `tsify`는 TS를 컴파일하기 위한 `browserify` 플러그인이며, 이를 다음과 같은 형태로 사용할 수 있습니다.

```bash
browserify index.ts -p [ tsify --allowJs ] > bundle.js
```

대부분의 유닛 테스트 도구 역시 동일한 역할을 하는 옵션이 있습니다.
예를 들어 `jest`를 사용할 때 `ts-jest`를 설치하고 `jest.config.js`에 전달할 TS 소스를 지정할 수 있습니다.

```js
module.exports = {
  transform: {
    '^.+\\.tsx?$': 'ts-jest',
  },
};
```

만약, 프레임워크 없이 빌드 체인을 직접 구성했다면, 다소 복잡하긴 하더라도 TS 컴파일링 후 `outDir`에 지정한 디렉토리를 기반으로 기존의 빌드 체인을 실행하면 됩니다.
이 경우, TS가 생성한 코드가 기존의 JS 룰을 따르도록 출력 옵션을 조정해야 할 필요는 있습니다. (ex. target, module)
