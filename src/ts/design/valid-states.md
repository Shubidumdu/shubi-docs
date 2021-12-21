# 유효한 상태만 표현하는 타입을 지향하기

효과적으로 타입을 설계하려면, **유효한 상태만** 표현할 수 있는 타입을 만들어 내는 것이 가장 중요합니다.
좋지 않은 예시를 하나 들어봅시다.

```ts
interface State {
  pageText: string;
  isLoading: boolean;
  error?: string;
}
```

위와 같은 상태 구성은 다음과 같은 문제를 갖습니다.

- 로딩 중이면서 동시에 에러가 발생할 수 있습니다.
- 상태 변경에 실수를 할 여지가 있습니다. (`error`는 선택 프로퍼티이며, `isLoading`은 단순한 `boolean`이기 때문에, 타입 관점에서 실수를 줄일 방법이 없습니다.)

이 경우 상태를 좀 더 적절하게 표현하는 방법은 다음과 같습니다.

```ts
interface RequestPending {
  state: 'pending';
}

interface RequestError {
  state: 'error';
  error: string;
}

interface RequestSuccess {
  state: 'ok';
  pageText: string;
}

type RequestState = RequestPending | RequestError | RequestSuccess;

interface State {
  currentPage: string;
  requests: {[page: string]: RequestState};
}
```

작성해야 하는 코드의 양 자체는 늘어났지만, 이는 유효한 상태만을 다루고 있습니다.
이 덕분에, 당장에 의미없는 상태 프로퍼티를 갖게되는 경우는 없게 되어, 이를 다루기가 훨씬 편해졌습니다.
이는 코드가 길어지고, 또 표현하기 어려운 작업이지만, 결국은 시간을 절약하고 고통을 줄일 수 있는 방법입니다.
