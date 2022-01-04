# string 타입보다 더 구체적인 타입 사용하기

`string` 타입은 `any`와 유사한 문제를 갖고 있습니다. 잘못 사용하게 되는 경우 무효한 값을 허용하며, 타입 간의 관계도 감추어 버립니다.
리터럴 타입과 유니온을 통해 `string`의 부분 집합을 정의하여 타입 안정성과 가독성을 크게 높일 수 있습니다.

## 가능하다면 더 구체적인 타입을 사용하세요

```ts
// 이는 너무 광범위합니다.  
interface Album {
  artist: string;
  title: string;
  releaseDate: string;  // YYYY-MM-DD
  recordingType: string;  // E.g., "live" or "studio"
}

// 이렇게 쓰세요.
type RecordingType = 'live' | 'studio';

interface Album {
  artist: string;
  title: string;
  releaseDate: Date; // 굳이 string일 이유가 없습니다.
  recordingType: RecordingType;
}
```

## 객체 프로퍼티명을 매개변수로 가져와야 할 때는 `keyof`를 사용하세요

다음은 underscore 라이브러리에 존재하는 `pluck` 유틸함수입니다.
특정 타입의 배열에서 원하는 키의 값들만 가져온 하나의 배열을 반환합니다.

```ts
function pluck<T, K extends keyof T>(record: T[], key: K): T[K][] {
  return record.map(r => r[key]);
}

interface User {
  name: string;
}

const users: User[] = [{ name: '짱구' }, { name: '철수' }];

pluck(users, 'name'); // ['짱구', '철수']
```

이것이 만약 단순히 `key` 매개변수를 `string` 타입으로 가져오는 형태였다면 아래와 같았을겁니다.

```ts
function pluck(record: any[], key: string): any[] {
  return record.map(r => r[key]);
}
```

이 경우 해당 함수의 반환값은 `any[]` 타입이기 때문에 타입 체킹에 크게 방해가 됩니다.
따라서 객체의 프로퍼티명을 매개변수로 가져와야 하는 경우에는 타입 공간에서의 `keyof`를 적절히 사용해 더 명확한 타입을 지정해주어야 합니다.
