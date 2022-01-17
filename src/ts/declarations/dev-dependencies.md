# devDependencies에 typescript와 @types 추가하기

npm은 3가지 종류의 의존성을 구분해서 관리하며, 각각의 의존성은 `package.json` 파일 내의 별도 영역에 들어 있습니다.

- dependencies: 현재 프로젝트 실행에 필수적인 라이브러리
- devDependencies: 현재 프로젝트의 개발/테스트에 사용되지만, 런타임에 필요없는 라이브러리
- peerDependencies: 런타임에 필요하긴 하지만, 의존성을 직접 관리하지 않는 라이브러리

TS와 관련된 대부분의 라이브러리는 일반적으로 런타임에는 영향을 미치지 않기 때문에 devDependencies에 속합니다.

모든 타입스크립트 프로젝트에서 공통적으로 고려해야 할 의존성 두 가지를 살펴보겠습니다.

## 1. 타입스크립트 자체 의존성을 고려해야 합니다

TS를 시스템 레벨로 설치할 수도 있지만, 다음의 두 이유 때문에 추천하지 않습니다.

- 팀원들 모두가 동일한 버전을 설치한다는 보장이 없습니다.
- 프로젝트 셋업 시 별도의 단계가 추가됩니다.

결국, 따로 시스템 레벨로 설치를 하기 보다는, devDependencies에 포함시켜 단순히 `npm install` 명령만으로 모두 동일한 버전의 타입스크립트를 쉽게 설치하도록 하는 편이 좋습니다.

## 2. 타입 의존성(@types)를 고려해야 합니다

사용하는 라이브러리 자체적으로 `@types`가 포함되어 있지 않더라도, [DefinitelyTypes](https://github.com/DefinitelyTyped/DefinitelyTyped)에서 타입 정보를 얻을 수 있습니다.
이 경우, 원본 라이브러리 자체는 dependencies에 있더라도 `@types` 의존성은 devDependencies에 위치해야 합니다.

예를 들어, React의 경우에는 다음과 같습니다.

```zsh
npm install react
npm install --save-dev @types/react
```
