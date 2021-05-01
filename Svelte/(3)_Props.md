# Props

## Declaring props

특정 컴포넌트에서 하위 컴포넌트로 데이터를 전달해야할 때, 프로퍼티(props)를 지정해줄 필요가 있다.

Svelte에서는 해당 작업이 `export` 키워드를 통해서 이루어질 수 있다.

```svelte
<script>
	export let answer;
</script>
```

이를 상위 컴포넌트에서 사용하려면, 아래와 같은 식이다.

```svelte
<Nested answer={42}/>
```

## Default Values

아래와 같이 props가 전달되지 않은 경우에 대한 기본값을 지정해줄 수 있다.

```svelte
<script>
	export let answer = 'a mystery';
</script>
```

## Spread Props

별도로 objects로 props들을 전달하려는 경우, spread 연산자로 이를 처리할 수 있다. React랑 똑같다.

```svelte
<script>
	import Info from './Info.svelte';

	const pkg = {
		name: 'svelte',
		version: 3,
		speed: 'blazing',
		website: 'https://svelte.dev'
	};
</script>

<Info {...pkg}/>
```

만약, 별도로 `export` 키워드를 통해 props를 지정하지 않았음에도, 전달받는 값을 사용해야 하는 경우, `$$props`를 통해 컴포넌트에서 접근할 수 있다.
