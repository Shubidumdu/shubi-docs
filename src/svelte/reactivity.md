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

Svelte는 컴포넌트의 상태가 변하면 자동으로 DOM을 업데이트한다. 종종 일부 상황에서, 특정 상태에 의존적인 다른 상태가 존재할 수 있는데, 이 경우 **Reactive Declaration** 기능을 통해 이를 처리할 수 있다. 정확한 원리는 JS의 [Label]을 참조하자. 다시 말해, 아래와 같이 `$`를 통해 이를 적용하는 경우, **참조하는 다른 값이 변하는 경우 해당 코드를 다시 실행하라**는 의미를 전달할 수 있다.

```svelte
<script>
	let count = 0;
	$: doubled = count * 2;

	function handleClick() {
		count += 1;
	}
</script>

<button on:click={handleClick}>
	Clicked {count} {count === 1 ? 'time' : 'times'}
</button>

<p>{count} doubled is {doubled}</p>
```

물론, 단순히 `doubled`를 새로 선언하지 않고 `{count * 2}`와 같이 사용해도 된다.

## Statements

Declaration 뿐 아니라 Statement를 처리할 수도 있다.

```svelte
$: {
	console.log(`the count is ${count}`);
	alert(`I SAID THE COUNT IS ${count}`);
}
```

if 문을 적용할 수도 있다.

```svelte
$: if (count >= 10) {
	alert(`count is dangerously high!`);
	count = 9;
}
```

## Updating arrays and objects

React와 마찬가지로, 상태가 array 및 objects인 경우 `push`나 `splice` 같은 메서드들은 업데이트를 유발하지 않는다. 이를 해결하기 위한 방안 역시 동일하다.

```js
let numbers = [1, 2, 3, 4];

function addNumber() {
  numbers = [...numbers, numbers.length + 1];
}
```
