# 사용할 때는 너그럽게, 생성할 때는 엄격하게

보통 매개변수 타입은 반환 타입에 비해 범위가 넓은 경향이 있으며, 실제로도 이 쪽이 사용하기 용이합니다.
이를 테면 다음과 같은 함수의 예를 들 수 있습니다.

```ts
interface User {
  id: number;
  name?: string;
  age?: number;
}

const updateUser = (option: User) => {
  // ...
};
```

한편, 반환 타입의 경우에는 선택적인 프로퍼티 없이 더 명확하고 엄격해야 합니다.
실제로 넓은 타입 범위를 갖는 반환 타입은 사용하기가 굉장히 불편합니다.
값을 반환받은 이후에도 타입 체킹을 해주어야하는 일이 다분하기 때문입니다.

```ts
const createUser = (option: User): User => {
  return {
    ...option,
  }
}

const { username } = createUser({ id: 1, username: '김앨런', age: 27 });
// type username = string | undefined

const firstName = username.charAt(0); 
// Object is possibly 'undefined'.
```

결국 이러한 문제를 해결하려면 기본 형태(반환 타입)와 느슨한 형태(매개변수 타입)으로 각각의 상황에 대한 타입을 별도로 두는 것이 좋습니다.

```ts
// 기본 형태
interface User {
  id: number;
  name: string;
  age: number;
}

// 느슨한 형태
// type UserOptions = {
//     name?: string | undefined;
//     age?: number | undefined;
//   }
type UserOptions = Partial<Omit<User, 'id'>> 

const updateUser = (id: number, options: UserOptions) => {
  // ...
}

let id = 1;

const createUser = (options: UserOptions): User => ({
  id: id++,
  name: '이름없음',
  age: 1,
  ...options,
})

const user = createUser({ name: '김앨런', age: 27 });
```
