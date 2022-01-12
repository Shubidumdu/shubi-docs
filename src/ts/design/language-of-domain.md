# 해당 분야의 용어로 타입 이름 짓기

코드로 표현하고자 하는 모든 분야에는 주제를 설명하기 위한 전문 용어들이 이미 존재합니다.
자체적으로 용어를 만들어 내려고 하지 말고, 해당 분야에 이미 존재하는 용어를 사용해야 합니다.
이를 통해 보다 소통에 유리하게끔 하며, 타입의 명확성을 올릴 수 있습니다.

```ts
interface Animal {
  name: string;
  endangered: boolean;
  habitat: string;
}

// 변수명은 leopard지만, 프로퍼티 `name` 값은 `Snow Leopard`로, 별도의 의미를 지니는 것인지 모호함
const leopard: Animal = {
  name: 'Snow Leopard', // 동물의 학명인지? 일반적인 명칭인지?
  endangered: false, // 멸종 위기종 여부인지? 멸종 여부인지?
  habitat: 'tundra', // habitat라는 뜻 자체도 모호하고, 범위도 string으로 넓음
};
```

각 프로퍼티에 대한 의미가 모호하기 때문에, 이를 작성한 사람을 찾아 의도를 물어봐야만 하는 상황이 생깁니다.
이를 개선하자면 다음과 같이 변경할 수 있습니다.

```ts
interface Animal {
  commonName: string;
  genus: string;
  species: string;
  status: ConservationStatus;
  climates: KoppenClimate[];
}

type ConservationStatus = 'EX' | 'EW' | 'CR' | 'EN' | 'VU' | 'NT' | 'LC';
type KoppenClimate = |
  'Af' | 'Am' | 'As' | 'Aw' |
  'BSh' | 'BSk' | 'BWh' | 'BWk' |
  'Cfa' | 'Cfb' | 'Cfc' | 'Csa' | 'Csb' | 'Csc' | 'Cwa' | 'Cwb' | 'Cwc' |
  'Dfa' | 'Dfb' | 'Dfc' | 'Dfd' |
  'Dsa' | 'Dsb' | 'Dsc' | 'Dwa' | 'Dwb' | 'Dwc' | 'Dwd' |
  'EF' | 'ET';

// 각 프로퍼티를 보다 구체적인 용어로 대체
const snowLeopard: Animal = {
  commonName: 'Snow Leopard',
  genus: 'Panthera',
  species: 'Uncia', 
  status: 'VU',  // vulnerable => IUCN의 표준 분류 체계를 따르도록 함
  climates: ['ET', 'EF', 'Dfd'],  // alpine or subalpine => 쾨펜 기후 분류를 따르도록 함
};
```

IUCN 및 쾨펜 기후 분류에 대한 도메인 지식이 없다면 당황스러울 수는 있지만, 이 경우 정보를 찾기 위해 코드 작성자에 의존할 필요가 없습니다.
애초에 해당 분류 체계에 대한 정보를 습득하거나, 온라인에 있는 무수한 내용들을 바탕으로 이를 찾을 수 있을 겁니다.

## 이름을 정할 때 명심해야 할 세가지 규칙

1. 동일한 의미를 나타낼 때는 같은 용어를 사용합니다. 정말로 의미적으로 구분이 되어야 하는 경우에만 다른 용어를 사용합니다.
2. `data`, `info`, `thing`, `item`, `object`, `entity` 같은 모호하고 의미 없는 이름은 피해야 합니다. 만약 `entity`라는 용어가 해당 분야에서 특별한 의미를 지니다면 문제가 없겠지만, 귀찮다고 무심코 의미 없는 이름을 붙여서는 안됩니다.
3. 이름을 지을 때는 포함된 내용이나 계산 방식이 아니라 데이터 자체가 무엇인지를 고려해야 합니다. 예를 들어, `INodeList`보다는 `Directory`가 더 의미있는 이름입니다. 구현의 측면이 아니라 개념적인 측면에서 이름을 고려하세요.
