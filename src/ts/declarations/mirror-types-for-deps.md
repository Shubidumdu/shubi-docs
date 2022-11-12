# 의존성 분리를 위해 미러 타입 사용하기

직접 TS로 라이브러리를 작성하여 공개할 때는 필요 이상으로 의존성을 갖는 것을 피해야 합니다.
이를테면, 아래처럼 CSV 파일을 파싱하는 함수를 만든다고 할 때, NodeJS 사용자를 위해 매개변수에 Buffer 타입을 허용하였다고 가정합시다.

```ts
function parseCSV(contents: string | Buffer): {[column: string]: string}[]  {
  if (typeof contents === 'object') {
    // It's a buffer
    return parseCSV(contents.toString('utf8'));
  }
  // ...
}
```

여기서 쓰인 `Buffer` 타입은 NodeJS에 대한 타입 선언을 설치하여 얻을 수 있는데, 이 경우 작성한 라이브러리에 대한 타입 선언도 포함됩니다.
이는 `@types/node`에 의존하기 때문에, 결국 `devDependencies`로 포함하게 됩니다.
결국 이에 따라 `@types`와 무관한 JS 개발자나, NodeJS를 프로젝트에 이용하지 않는 TS 개발자의 경우 사용하지 않는 모듈을 포함해야 하는 문제가 생겨납니다.

## 구조적 타이핑을 활용하세요

이 경우에 저희가 초기에 다루었던 구조적 타이핑을 적용할 수 있습니다.
사용할 타입을 완전히 가져다 쓰는 대신, 필요한 메서드와 속성에 대해서만 별도로 타입을 작성하는 방법을 이용할 수 있습니다.

예를 들어, 위의 예시에서 `Buffer`의 경우는 다음과 같이 간략하게 타입을 선언하여 대체할 수 있습니다.

```ts
interface CsvBuffer {
  // parseCSV에서 쓰이는 함수에 대해서만 타입 선언을 합니다.
  toString(encoding: string): string;
}
```

구조적 타이핑의 관점에 따라, 해당 타입은 `Buffer`와도 호환되기 때문에 실제로 NodeJS 프로젝트에서 `Buffer` 인스턴스로도 `parseCSV`를 호출할 수 있습니다.

한편 다른 라이브러리에서의 타입 선언 대부분을 추출해야 하는 상황이라면, 차라리 명시적으로 `@types` 의존성을 추가하는 게 낫습니다.
