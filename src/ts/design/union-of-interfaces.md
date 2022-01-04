# 유니온의 인터페이스보다는 인터페이스의 유니온 사용하기

다음 형태의 유니온 프로퍼티 타입을 갖는 인터페이스가 있다고 가정해봅시다.

```ts
interface Family {
  parent: KimsParents | LeesParents | ParksParents;
  child: Kim | Lee | Park;
}
```

얼핏 문제가 없는 듯 보이지만, 해당 인터페이스는 추후에 사용하기가 어렵고, 에러가 발생할 여지가 많습니다.
타입 시스템 상으로 `KimsParent`가 부모일 때 `child`가 `Park`이거나 `Lee`인 경우를 허용하기 때문입니다.

만약 이것을 더 나은 방법으로 모델링하려면 각각의 타입 계층을 분리된 인터페이스로 두어야 합니다.

```ts
interface KimsFamily {
  parent: KimsParent;
  child: Kim;
}

interface LeesFamily {
  parent: LeesParent;
  child: Lee;
}

interface ParksFamily {
  parent: ParksParent;
  child: Park;
}

// 이제 부모 자식 간의 관계가 꼬일 일이 없습니다!
type Family = KimsFamily | LeesFamily | ParksFamily;
```

## Tagged Union

이러한 패턴을 활용하는 가장 일반적인 예시는 Tagged Union 입니다.

```ts
interface KimsFamily {
  lastName: 'kim';
  parent: KimsParent;
  child: Kim;
}

interface LeesFamily {
  lastName: 'lee';
  parent: LeesParent;
  child: Lee;
}

interface ParksFamily {
  lastName: 'park';
  parent: ParksParent;
  child: Park;
}

type Family = KimsFamily | LeesFamily | ParksFamily;
```

위의 각 인터페이스에서 쓰인 `lastName`(일반적으로는 `type`과 같은 이름)이 곧 **태그**가 됩니다.
이 태그는 런타임에 어떤 타입의 인터페이스가 쓰이는지 판단되어 **타입 좁히기**에 활용됩니다.

```ts
function getChild = (family: Family) => {
  if (family.lastName === 'kim') {
    // type family = KimsFamily
  } else if (family.lastName === 'lee') {
    // type family = LeesFamily
  } else {
    // type family = ParksFamily
  }
}
```

## 관련된 Optional 프로퍼티는 하나로 묶으세요

다음과 같이 관련이 깊은 두 속성의 경우는 하나의 객체로 묶는 것이 더 나은 설계입니다. 앞선 아이템에서 말한 내용과 유사합니다.

```ts
// 이것보다는
interface Person {
  name: string;
  // 아래는 둘 다 존재하거나, 둘 다 없어야 합니다.
  placeOfBirth?: string;
  dateOfBirth?: Date;
}

// 이게 낫습니다.
interface Person {
  name: string;
  // 이제 두 프로퍼티 중 하나만 존재하는 일은 없습니다.
  birth?: {
    place: string;
    date: Date;
  }
}
```

하지만, 타입 구조를 직접 손댈 수 없는 경우(ex. API의 결과)라면, 앞서 말한 인터페이스의 유니온을 사용해 관계를 모델링할 수 있습니다.

```ts

   
interface Name {
  name: string;
}

interface PersonWithBirth extends Name {
  placeOfBirth: string;
  dateOfBirth: Date;
}

type Person = Name | PersonWithBirth;

function eulogize(p: Person) {
  // placeOfBirth 프로퍼티가 존재한다면 PersonWithBirth 입니다.
  if ('placeOfBirth' in p) {
    p // type p = PersonWithBirth
    const { dateOfBirth } = p  // type dateOfBirth = Date
  }
}
```
