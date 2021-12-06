# 매핑된 타입을 사용하여 값을 동기화하기

[매핑된 타입(Mapped Type)](https://www.typescriptlang.org/docs/handbook/2/mapped-types.html)은 기존에 작성한 다른 타입에 기반하여 새로운 타입을 쉽게 작성할 수 있는 방법입니다. 주로 `keyof` 키워드와 함께 사용됩니다.

```ts
type FeatureFlags = {
  darkMode: () => void;
  newUserProfile: () => void;
};

type OptionsFlags<Type> = {
  [Property in keyof Type]: boolean;
};

type FeatureOptions = OptionsFlags<FeatureFlags>;
// type FeatureOptions = {
//     darkMode: boolean;
//     newUserProfile: boolean;
// }
```

이는 꼭 제네릭으로 사용되어야 하는 것은 아닙니다. 인덱스 시그니처에서도 매핑된 타입을 사용할 수 있습니다.

```ts
const options: {[k in keyof FeatureFlags]: boolean} = {
  darkMode: false,
  newUserProfile: true,
}
```
