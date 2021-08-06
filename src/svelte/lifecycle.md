# Lifecycle

## onMount

모든 컴포넌트는 생성되는 시점에서부터 사라질 때까지 생명주기가 존재한다. Svelte 역시 이를 다루기 위한 몇가지 함수들을 제공한다.

`onMount`는 그중 가장 자주 사용하게 될 생명주기 함수로, 최초로 DOM에 렌더링된 이후에 실행된다.

```svelte
<script>
	import { onMount } from 'svelte';

	let photos = [];

	onMount(async () => {
		const res = await fetch(`https://jsonplaceholder.typicode.com/photos?_limit=20`);
		photos = await res.json();
	});
</script>
```

> **주의** : SSR에 의해, `fetch` 메서드는 `script` 태그의 최상위 보다 `onMount`에 위치하는 것이 좋다. `onDestroy`를 제외하면, SSR 중에는 생명주기 함수가 실행되지 않으며, 다시말해 DOM에 마운트 된 이후에야 실행되어야 하는 데이터를 페칭하는 경우를 방지할 수 있다.

`onMount`의 콜백함수에서 함수를 반환할 수 있는데, 이 경우 해당 함수는 컴포넌트가 사라질 때 호출된다. (clean up)

## onDestroy

함수에 사라질 때 특정 로직을 처리하고자 할 때 `onDestroy` 함수를 사용할 수 있다.

```svelte
<script>
	import { onDestroy } from 'svelte';

	let seconds = 0;
	const interval = setInterval(() => seconds += 1, 1000);

	onDestroy(() => clearInterval(interval));
</script>
```

이러한 생명주기 함수들을 React의 커스텀 Hook 처럼 사용할 수도 있다.

```svelte
import { onDestroy } from 'svelte';

export function onInterval(callback, milliseconds) {
	const interval = setInterval(callback, milliseconds);

	onDestroy(() => {
		clearInterval(interval);
	});
}
```

## beforeUpdate & afterUpdate

`beforeUpdate` 함수는 컴포넌트의 상태에 따라 DOM이 업데이트 되기 전마다 실행되며, 반대로 `afterUpdate`는 DOM이 업데이트 된 이후에 실행된다.

단순히 상태 중심적인 방식으로는 처리하기 어려운 로직을 처리하기 위한 경우에 종종 유용하다. (ex. 스크롤 위치 변경 등)

```svelte
let div;
let autoscroll;

beforeUpdate(() => {
	autoscroll = div && (div.offsetHeight + div.scrollTop) > (div.scrollHeight - 20);
});

afterUpdate(() => {
	if (autoscroll) div.scrollTo(0, div.scrollHeight);
});
```

## tick

`tick` 함수는 다른 생명주기 함수들과 다르게, 최초에 함수가 초기화되는 시점이 아닌, 어디서는 호출할 수 있다. 해당 함수는 보류 중인 상태 변경 사항이 DOM에 적용된 이후 즉시 resolved 되는 promise를 반환한다.

Svelte는 컴포넌트의 상태가 업데이트될 때, DOM을 바로 업데이트하지 않고, 적용할 다른 변경 사항이 없는지를 판단하기 위해 다음 마이크로태스트까지 대기한다. 이렇게 함으로써 불필요한 동작을 피하고, 브라우저가 더 효율적으로 작업을 일괄 처리할 수 있도록 도와준다.

```svelte
<script>
	import { tick } from 'svelte';

	let text = `Select some text and hit the tab key to toggle uppercase`;

	async function handleKeydown(event) {
		if (event.key !== 'Tab') return;

		event.preventDefault();

		const { selectionStart, selectionEnd, value } = this;
		const selection = value.slice(selectionStart, selectionEnd);

		const replacement = /[a-z]/.test(selection)
			? selection.toUpperCase()
			: selection.toLowerCase();

		text = (
			value.slice(0, selectionStart) +
			replacement +
			value.slice(selectionEnd)
		);

		await tick();
		this.selectionStart = selectionStart;
		this.selectionEnd = selectionEnd;
	}
</script>

<style>
	textarea {
		width: 100%;
		height: 200px;
	}
</style>

<textarea value={text} on:keydown={handleKeydown}></textarea>
```

위와 같이 사용하는 경우, `await tick();`의 이전까지의 내용들이 모두 적용되고, DOM이 업데이트 된 이후, 그 다음의 로직들이 처리된다. 즉, DOM이 업데이트되고 나서야 `await tick()` 이후의 코드가 실행된다.
