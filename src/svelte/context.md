# Context API

## setContext and getContext

context API는 데이터를 props로 전달하거나, store를 통해 디스패칭하지 않고도 컴포넌트 간에 소통할 수 있는 매커니즘이다.

이는 React에서의 context와도 흡사하게 이해해도 좋다.

한 컴포넌트에서 `setContext(key, context)`를 호출하면, 컴포넌트 본인을 포함한 하위 컴포넌트 모두에서 `const context = getContext(key)`로 해당 값을 가져올 수 있다.

```svelte
import { onMount, setContext } from 'svelte';
import { mapbox, key } from './mapbox.js';

setContext(key, {
	getMap: () => map
});
```

context는 어떤 타입이든지 가능하며, 생명주기 함수와 마찬가지로 `setContext`와 `getContext`는 반드시 컴포넌트 초기화 시점에 호출되어야 한다. 컴포넌트가 마운트되기 전까지 위에서 반환하는 `map`은 `undefined`가 되므로, `getMap` 함수를 통해 우회적으로 가져오는 방법을 사용한다.

이제, 하위 컴포넌트에서는 아래와 같이 `map`을 가져올 수 있다.

```svelte
import { getContext } from 'svelte';
import { mapbox, key } from './mapbox.js';

const { getMap } = getContext(key);
const map = getMap();
```

### Context keys

사실, 위에서의 `key`는 아래와 같다.

```js
const key = {};
```

`key`로는 어떤 것이든 사용 가능하며, 그냥 `setContext('mapbox', ...)`와 같이 string을 전달해도 된다.

단, `key`로 string을 사용할 경우, 여러 컴포넌트 라이브러리들 간에 서로 충돌되는 `key`를 사용할지도 모른다는 문제가 발생한다. 이를 방지하기 위해 위와 같이 객체 리터럴을 사용하면, 해당 key가 고유함을 보장받을 수 있다.

### Context vs. Stores

Context는 store와 비슷해 보일 수 있다. 둘 사이의 차이점은, store가 애플리케이션의 전역에서 접근할 수 있는 반면, context는 해당 컴포넌트와 그 하위 컴포넌트들에서만 접근할 수 있다는 것이다.

상황에 따라 둘을 적절히 사용할 수도 있다. context는 반응적이지(reactive) 않으므로, 실시간으로 값이 변화하는 값들에 대해서는 아래와 같은 형태로 store를 사용하는 것이 더 적합하다.

```svelte
const { these, are, stores } = getContext(...);
```
