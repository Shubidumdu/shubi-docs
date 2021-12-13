# 비동기 코드에는 콜백 대신 async 함수 사용하기

비동기 동작을 다룰 때에, 콜백보다는 프로미스를 사용해야 합니다. 이유는 다음과 같습니다.

- 콜백보다는 프로미스가 코드를 작성하기 쉽습니다.
- 콜백보다는 프로미스가 타입을 추론하기 쉽습니다.

```ts
function fetchPagesCB() {
  let numDone = 0;
  const responses: string[] = [];
  const done = () => {
    const [response1, response2, response3] = responses;
    // ...
  };
  const urls = [url1, url2, url3];
  urls.forEach((url, i) => {
    fetchURL(url, r => {
      responses[i] = url;
      numDone++;
      if (numDone === urls.length) done();
    });
  });
}
```

위와 같은 콜백 기반의 비동기 함수는 프로미스를 통해 아래와 같은 형태가 될 수 있습니다.

```ts
async function fetchPages() {
  const [response1, response2, response3] = await Promise.all([
    fetch(url1), fetch(url2), fetch(url3)
  ]);
  // ...
}
```

그리고 프로미스보다는 async/await를 사용하는 편이 좋습니다. 이유는 아래와 같습니다.

- 일반적으로 더 간결하고 직관적인 코드가 됩니다.
- async 함수는 항상 프로미스를 반환하도록 강제합니다.

"함수가 항상 프로미스를 반환하도록 강제"하는 것은 각 함수가 항상 동기 또는 비동기로 실행되어야 한다는 원칙을 쉽게 지키도록 해줍니다.
콜백이나 프로미스를 사용하면 실수로 반(half)동기 코드를 작성할 수 있지만, async 함수에 기반하는 경우 항상 비동기 코드로 작성되기 때문입니다.

아래의 콜백 기반의 함수 `fetchWithCache`는 얼핏 제대로 만들어진 반동기 함수인 듯 하지만, 실제로 사용할 때 문제를 일으킬 가능성이 있습니다. 캐시가 되어있는 경우에는 `callback(..)`가 동기적으로 동작할 것이기 때문입니다.

```ts
const _cache: {[url: string]: string} = {};

function fetchWithCache(url: string, callback: (text: string) => void) {
  if (url in _cache) {
    callback(_cache[url]);
  } else {
    fetchURL(url, text => {
      _cache[url] = text;
      callback(text);
    });
  }
}

let requestStatus: 'loading' | 'success' | 'error';

// 캐시가 있는 경우 => requestStatus는 'loading'
// 캐시가 없는 경우 => requestStatus는 'success'
function getUser(userId: string) {
  fetchWithCache(`/user/${userId}`, profile => {
    requestStatus = 'success';
  });
  requestStatus = 'loading';
}
```

이를 async 함수로 대체하면 보다 간결하고, 일관적인 형태로 사용할 수 있게 됩니다.

```ts
const _cache: {[url: string]: string} = {};

async function fetchWithCache(url: string) {
  if (url in _cache) {
    return _cache[url];
  }
  const response = await fetch(url);
  const text = await response.text();
  _cache[url] = text;
  return text;
}

let requestStatus: 'loading' | 'success' | 'error';

async function getUser(userId: string) {
  requestStatus = 'loading';
  const profile = await fetchWithCache(`/user/${userId}`);
  requestStatus = 'success';
}
```

한가지 유의점으로, async 함수 내에서 프로미스를 반환한다고 해서 `Promise<Promise<T>>` 반환타입이 되지는 않습니다. 이 경우에도 동일하게 `Promise<T>`가 됩니다. 타입 체커를 통해서도 이를 확인할 수 있습니다.

```ts
// Function getJSON(url: string): Promise<any>
async function getJSON(url: string) {
  const response = await fetch(url);
  const jsonPromise = response.json();  // Type is Promise<any>
  return jsonPromise;
}
```
