# 공개 API에 등장하는 모든 타입을 export하기

공개된 메서드에 등장한 어떤 형태의 타입이든 익스포트를 하는 것이 좋습니다.
어차피 메서드 자체가 공개된 이상, 라이브러리 사용자가 추출이 가능하므로, 애초에 익스포트하여 이용하기 쉬운 형태로 만드는 편이 좋습니다.

```ts
interface SecretName {
  first: string;
  last: string;
}

interface SecretSanta {
  name: SecretName;
  gift: string;
}

export function getGift(name: SecretName, gift: string): SecretSanta {
  // COMPRESS
  return {
    name: {
      first: 'Dan',
      last: 'Van',
    },
    gift: 'MacBook Pro',
  };
  // END
}
```

이를테면, 위의 경우에 타입을 숨기기 위해서 일부러 각 인터페이스에 대해 `export`를 하지 않았다고 하더라도, 어차피 사용자는 해당 인터페이스의 타입을 추출할 수 있습니다.

```ts
type MySanta = ReturnType<typeof getGift>;  // type SecretSanta
type MyName = Parameters<typeof getGift>[0];  // type SecretName
```

다시 말해, 공개 API에 해당 타입들이 이용되는 순간, 어차피 해당 타입들은 노출된 상태이기 때문에 굳이 숨기려 하지 말고 라이브러리 이용자들을 위해 명시적으로 export하는 것이 좋습니다.
