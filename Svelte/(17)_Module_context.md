# Module context

## Sharing code

지금껏 사용한 것 처럼, `<script>` 블록은 컴포넌트 초기화 시 각각의 컴포넌트 내에서 실행되는 로직들을 담고 있다. 그리고 대부분의 경우에는 이것으로 충분하다.

헌데, 아주 가끔 컴포넌트 외부에서 로직을 처리하여, 여러 컴포넌트에 해당 코드를 "공유"해야하는 경우가 생긴다. 이를테면, 음악 플레이어 컴포넌트를 만든 이후, 한 플레이어가 재생 중일 때 다른 플레이어를 중지시키는 로직을 구현하고자 하는 경우가 그렇다.

이 경우, `<script context="module">` 블록을 통해 처리할 수 있다. 해당 블록 내에서 실행되는 코드는 컴포넌트의 초기화 시점이 아닌, 최초로 evaluate되는 시점에 딱 한번만 실행된다.

```svelte
<script context="module">
	let current;
</script>
```

이제, 별도로 부모 컴포넌트에서 상태를 관리하지 않더라도, 동일한 컴포넌트의 인스턴스들끼리 소통하여 로직을 처리할 수 있다.

```svelte
function stopOthers() {
	if (current && current !== audio) current.pause();
	current = audio;
}
```

## Exports

`context="module"` 블록 내에서 `export`되는 변수들은 실제 모듈 자체에서 export한 것처럼 다루어진다.

다시 말해, 아래와 같이 `stopAll` 함수를 `export`한 경우,

```svelte
// AudioPlayer.svelte
<script context="module">
	const elements = new Set();

	export function stopAll() {
		elements.forEach(element => {
			element.pause();
		});
	}
</script>
```

이는 일반적인 JS 모듈처럼 `import`해서 사용할 수 있다.

```svelte
<script>
	import AudioPlayer, { stopAll } from './AudioPlayer.svelte';
</script>

<button on:click={stopAll}>
	stop all audio
</button>

// ...
```

Svelte에서는 컴포넌트가 default export로 다루어지기 때문에, `default export`를 사용할 수 없음에 주의하자.
