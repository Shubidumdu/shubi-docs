# 정보를 감추는 목적으로 private 사용하지 않기

JS는 클래스에 비공개 속성을 만들 수 없습니다. 비공개 속성임을 나타내기 위해 언더스코어(`_`)로 접두사를 붙이는 것이 관례로 인정될 뿐, 실제로는 일반적인 속성일 뿐입니다.

헌데 TS에는 `public`, `protected`, `private` 접근 제어자가 있기 때문에, 이를 통해 공개 규칙을 강제할 수 있는 것으로 오해할 수 있습니다.

```ts
class Diary {
  private secret = 'cheated on my English test';
}

const diary = new Diary();
diary.secret
   // ~~~~~~ Property 'secret' is private and only
   //        accessible within class 'Diary'
```

헌데 이러한 접근 제어자들은 TS 키워드일 뿐, 컴파일 이후에는 제거되며, 그 결과 다음처럼 일반적인 JS 속성이 됩니다.

```js
class Diary {
  constructor() {
    this.secret = 'cheated on my English test';
  }
}

const diary = new Diary();
diary.secret;
```

TS 접근 제어자들은 단지 컴파일 시점에만 오류를 표시해줄 뿐, 언더스코어 관례와 마찬가지로 런타임에는 아무런 효력이 없습니다.

JS 및 TS에서 정보를 숨기기 위해 가장 효과적인 방법은 **클로저**를 사용하는 것입니다. 해당 챕터에서 클로저에 대해 깊게 다루기엔 범주를 벗어나므로, [여기](https://ko.javascript.info/closure)를 참조하도록 합시다.

또 다른 선택지로는 **비공개 필드 기능**을 사용할 수 있습니다. 비공개 필드는 접두사로 `#`를 붙여서 타입 체크와 런타임 모두에서 비공개로 만드는 역할을 합니다.

```ts
class PasswordChecker {
  #passwordHash: number;

  constructor(passwordHash: number) {
    this.#passwordHash = passwordHash;
  }

  checkPassword(password: string) {
    return hash(password) === this.#passwordHash;
  }
}
```

해당 기능은 기존에(해당 책이 작성될 시점에도) 표준화가 진행 중인 단계였으나, ES2022에 접어들면서 [공식적인 스펙이 되었습니다](https://github.com/tc39/proposals/blob/HEAD/finished-proposals.md).

