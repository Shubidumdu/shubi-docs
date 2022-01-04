# 타입 주변에 null값 배치하기

다음의 최대, 최솟값을 계산하는 `extent` 함수가 있다고 생각해봅시다.

```ts
// WARNING : 실제로는 타입 에러가 발생합니다!
function extent(nums: number[]) {
  let min, max; // undefined, undefined
  for (const num of nums) {
    if (min === undefined) { // 최초에 min이 undefined라면 min, max 모두에 첫번째 값을 할당합니다.
      min = num;
      max = num;
    } else {
      // min에 대해서만 undefined 체킹이 이루어졌습니다.
      min = Math.min(min, num); // type min = number
      max = Math.max(max, num); // type max = number | undefined
    }
  }
  
  // 로직 만을 생각해본다면 반환 타입은 [number, number] | [undefined, undefined] 여야 하지만, 
  return [min, max];  // 실제로는 (number | undefined)[] 입니다.
}
```

위 함수는 로직 상 `min`, `max`는 둘 다 `undefined` 이거나, 둘 다 `undefined`가 아니어야 하지만, 이를 타입 체커가 인지하지 못 한다는 문제점을 갖고 있습니다.
그래서 실제로 `strictNullChecks` 환경에서 에러가 발생합니다.

## 함수의 반환 타입을 `null`이거나, `null`이 아니게 만드세요

이걸 해결하기 위해서는 해당 값들을 하나의 객체 또는 배열에 넣어 처리하면 됩니다.
반환값이 nullish하다면 반환 타입을 하나의 객체로 만들고 반환 타입 전체가 `null`이거나, 또는 전체가 `null`이 아니도록 만드는 편이 사람과 타입체커 모두에게 명료한 코드가 됩니다.

```ts
function extent(nums: number[]) {
  let result: [number, number] | null = null;
  for (const num of nums) {
    if (!result) {
      result = [num, num];
    } else {
      result = [Math.min(num, result[0]), Math.max(num, result[1])]
    }
  }
  return result; // [number, number] | null
}

const [min, max] = extent([0, 1, 2])!;
```

## 클래스 프로퍼티에는 `null`이 존재하지 않게 하세요

다음과 같이 프로퍼티에 nullish한 값이 존재한다면, 인스턴스를 생성하고 난 이후와 해당 클래스의 모든 메서드를 사용하기 어렵게 만듭니다.

```ts
class UserPosts {
  user: UserInfo | null;
  posts: Post[] | null;

  constructor() {
    // 최초 인스턴스 생성 시에 각 프로퍼티가 null 입니다.
    this.user = null;
    this.posts = null;
  }

  async init(userId: string) {
    // 인스턴스 생성 이후에야 데이터를 가져옵니다.
    return Promise.all([
      async () => this.user = await fetchUser(userId),
      async () => this.posts = await fetchPostsForUser(userId)
    ]);
  }

  getUserName() {
    // 각 메서드에서 매번 프로퍼티에 대한 null 체킹을 해주어야 합니다.
    if (this.user) return this.user.name;
    else return null;
  }
}

const userPost = new UserPosts();

// 인스턴스 생성 이후에도 계속 null 체킹이 필요합니다.
userPost.user // type UserInfo | null;
```

이것을 개선하려면, 인스턴스 생성 이후 프로퍼티를 채워넣는 것이 아니라, 정적 메서드를 통해 애초에 완성된 인스턴스를 생성하도록 해야합니다.

```ts
class UserPosts {
  user: UserInfo;
  posts: Post[];

  constructor(user: UserInfo, posts: Post[]) {
    // 최초 인스턴스 생성 시부터 모든 프로퍼티를 갖고 있습니다.
    this.user = user;
    this.posts = posts;
  }

  static async init(userId: string): Promise<UserPosts> {
    // 필요한 데이터들을 애초에 다 가져온 이후에
    const [user, posts] = await Promise.all([
      fetchUser(userId),
      fetchPostsForUser(userId)
    ]);
    // 인스턴스를 생성합니다.
    return new UserPosts(user, posts);
  }

  getUserName() {
    // 이제 null 체킹이 필요 없습니다!
    return this.user.name;
  }
}
```
