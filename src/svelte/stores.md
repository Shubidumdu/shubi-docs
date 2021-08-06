# Stores

## Writable stores

Svelte에서는 *store*를 통해 상태 로직을 컴포넌트와 분리할 수 있다. store는 상태값이 변경되었을 때 관련 컴포넌트들에게 알려주는 `subscribe` 메서드를 가진 단순한 객체다.

store 중에는 **writable store**가 있으며, 이는 `set`과 `update`메서드를 갖는다.

```svelte
<script>
	import { count } from './stores.js';
	import Incrementer from './Incrementer.svelte';
	import Decrementer from './Decrementer.svelte';
	import Resetter from './Resetter.svelte';

	let count_value;

	const unsubscribe = count.subscribe(value => {
		count_value = value;
	});
</script>
```

`update`는 인수를 콜백함수로 받으며, `set`는 직접적인 값을 받는다.

```svelte
function increment() {
	count.update(n => n + 1);
}
```

```svelte
function reset() {
	count.set(0);
}
```

## Auto-subscriptions

만약, 컴포넌트가 초기화와 제거를 여러번 반복하게 되는 경우, `unsubscribe` 함수를 실행하지 않게되면 메모리 누수의 위험이 있다.

따라서, 이를 방지하기 위해선 `onDestroy` 생명주기 메서드에서 이를 호출해주어야 한다.

```svelte
<script>
	import { onDestroy } from 'svelte';
	import { count } from './stores.js';
	import Incrementer from './Incrementer.svelte';
	import Decrementer from './Decrementer.svelte';
	import Resetter from './Resetter.svelte';

	let count_value;

	const unsubscribe = count.subscribe(value => {
		count_value = value;
	});

	onDestroy(unsubscribe);
</script>

<h1>The count is {count_value}</h1>
```

헌데, 여러 컴포넌트에 대해 동일한 작업을 해주어야 하는 상황이라면 이런 방법이 다소 번거롭게 느껴질 수 있다.(boilerplatey) Svelte는 `$`만 덧붙이면 store 값에 대해 앞선 작업들을 알아서 처리해준다.

```svelte
<script>
	import { count } from './stores.js';
	import Incrementer from './Incrementer.svelte';
	import Decrementer from './Decrementer.svelte';
	import Resetter from './Resetter.svelte';
</script>

<h1>The count is {$count}</h1>
```

이러한 Auto-subscription은 store의 변수들이 컴포넌트 최상위 스코프에서 선언 및 import 되었을 때만 제대로 동작한다.

`$`로 변수명을 시작하는 것은 store 값을 참조하겠다는 것으로 간주되며, 그렇지 않은 경우에 대해 Svelte는 `$`로 임의의 변수를 선언하는 것을 방지한다.

## Readable stores

모든 경우에 store를 참조하는 컴포넌트들에게 쓰기 권한을 부여할 필요는 없다. 이를테면, 시간, 마우스 위치, 지리적 위치 등을 다루는 경우가 그렇다.

이러한 경우에 readable store를 사용할 수 있다.

```svelte
export const time = readable(new Date(), function start(set) {
	const interval = setInterval(() => {
		set(new Date());
	}, 1000);

	return function stop() {
		clearInterval(interval);
	};
});
```

`readable`의 첫번째 인수는 초기값이며, 설정할 필요가 없다면 `null` 혹은 `undefined`로 두면 된다.

두번째 인수는 `start` 콜백함수이며, 이는 `set` 콜백함수를 파라미터로 받고 `stop` 함수를 반환하는 함수다. `start` 함수는 최초의 subscriber에 의해 상태값이 참조되는 경우에 호출되며, `stop`은 마지막 subscriber가 unscribe를 했을 때에 실행되는 cleanup 함수다.

## Derived stores

특정 store의 값에 의존하는 다른 값이 있는 경우, `derived`를 통해 새로운 store를 만들어 줄 수 있다. 아래는 특정 페이지가 열리고 나서의 시간을 측정한 상태값을 보유한 derived store다.

```svelte
export const elapsed = derived(
	time,
	$time => Math.round(($time - start) / 1000)
);
```

여러 inputs들로부터 derive store를 갖추고, 값을 반환하는 대신 명시적으로 `set`를 통해 값을 변경해줄 수도 있다. 이에 대해서는 [여기](https://svelte.dev/docs#derived)를 참조하자.

## Custom stores

어떤 객체든 `subscribe` 메서드가 적절하게 실행되기만 하면, 이는 store로 취급된다. 이를 통해 유저가 임의로 커스텀 store를 만들어 로직을 처리할 수 있다.

해당 문서의 앞쪽에서 `writable`을 통해 store를 다루었던 내용을 커스텀 store를 통해 리팩토링해보자.

```svelte
function createCount() {
	const { subscribe, set, update } = writable(0);

	return {
		subscribe,
		increment: () => update(n => n + 1),
		decrement: () => update(n => n - 1),
		reset: () => set(0)
	};
}
```

## Store bindings

writable store를 사용하는 경우, 로컬 컴포넌트의 상태값과 store의 값을 binding 해줄 수 있다.

아래 예시에서는 `$name` store 값을 `input`의 `value`값에 바인딩을 해주는 예시다.

```svelte
<input bind:value={$name}>
```

이후, input의 `value`에 변경이 있다면 `name` store값 역시 자동으로 업데이트된다.

컴포넌트 내부에서 값을 직접 할당해줄 수도 있다.

```svelte
<button on:click="{() => $name += '!'}">
	Add exclamation mark!
</button>
```

위에서의 `$name += '!'`은 `name.set($name + '!')`의 호출과 동일하다.
