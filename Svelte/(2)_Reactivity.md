# Reactivity

## Assignments

기본적인 이벤트 핸들링은 다음과 같이 할 수 있다.

```svelte
<script>
	let count = 0;

	function handleClick() {
		count += 1;
	}
</script>

<button on:click={handleClick}>
	Clicked {count} {count === 1 ? 'time' : 'times'}
</button>
```

## Declarations

Svelte는 컴포넌트의 상태가 변하면 자동으로 DOM을 업데이트한다.
