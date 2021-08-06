# Events

## DOM events

이미 앞서 살펴본 것 처럼, `on:` 명령어를 통해 특정 요소에 이벤트 핸들러를 부여할 수 있다.

```svelte
<script>
	let m = { x: 0, y: 0 };

	function handleMousemove(event) {
		m.x = event.clientX;
		m.y = event.clientY;
	}
</script>

<div on:mousemove={handleMousemove}>
	The mouse position is {m.x} x {m.y}
</div>

<style>
	div { width: 100%; height: 100%; }
</style>
```

## Inline handler

인라인으로 직접 부여할 수도 있다.

```svelte
<script>
	let m = { x: 0, y: 0 };

	function handleMousemove(event) {
		m.x = event.clientX;
		m.y = event.clientY;
	}
</script>

<div on:mousemove="{e => m = { x: e.clientX, y: e.clientY }}">
	The mouse position is {m.x} x {m.y}
</div>

<style>
	div { width: 100%; height: 100%; }
</style>
```

`"` 따옴표는 선택사항이다. 없어도 상관 없다.

일부 프레임워크에서는 이러한 인라인 이벤트 핸들러를 사용하는 것을 지양하지만, Svelte의 경우 컴파일링 단계에서 성능 상의 최적화가 진행되므로 문제가 없다.

## Event modifiers

DOM 이벤트 핸들러들은 그들 동작을 변경해주는 Event modifier를 가질 수 있다. 이를테면, 아래처럼 `once`를 이벤트 핸들러에 덧붙인 경우 해당 이벤트 핸들러는 딱 한번만 동작한다.

```svelte
<button on:click|once={handleClick}>
	Click me
</button>
```

아래는 Event modifier의 전체 목록이다.

- `preventDefault` - 핸들러를 동작시키기 전에 `event.preventDefault()`를 먼저 수행한다.
- `stopPropagation` - `event.stopPropagation()`을 호출한다. 다시 말해, 이벤트 버블링/캡처링을 막는다.
- `passive` - 터치 및 마우스 휠 이벤트에 대한 스크롤 성능을 향상시킨다. [[참조](https://ko.javascript.info/default-browser-action#ref-2368)]
- `nonpassive` - `passive: false`를 의미한다.
- `capture` - 이벤트 핸들러의 동작에 있어 버블링이 아닌 캡처링 단계를 사용한다.
- `once` - 이벤트 핸들러를 딱 한번만 동작시키고 제거한다.
- `self` - 오직 해당 요소 본인에서 이벤트가 발생했을 때만 이벤트 핸들러를 동작시킨다. (event.target === currentTarget)

이러한 Event modifier들은 `on:click|once|capture={...}`와 같이 여러 개를 한번에 체이닝할 수 있다.

## Component events

컴포넌트 역시 이벤트를 디스패치할 수 있다. 이를 위해서는 이벤트 디스패쳐를 만들어야한다.

```svelte
<script>
	import { createEventDispatcher } from 'svelte';

	const dispatch = createEventDispatcher();

	function sayHello() {
		dispatch('message', {
			text: 'Hello!'
		});
	}
</script>
```

위에서의 `text`는 `event.detail.text`를 통해 접근할 수 있다.

## Event forwarding

DOM 이벤트와 다르게, 컴포넌트 이벤트는 버블링되지 않는다. 따라서, 상위 컴포넌트에서 이벤트를 전달받기 위해선 각 층의 컴포넌트마다 이벤트를 디스패치 해주어야 한다.

```svelte
<script>
	import Inner from './Inner.svelte';
	import { createEventDispatcher } from 'svelte';

	const dispatch = createEventDispatcher();

	function forward(event) {
		dispatch('message', event.detail);
	}
</script>

<Inner on:message={forward}/>
```

단, 이러한 과정 자체가 너무 많은 코드를 작성하게 만드므로, Svelte에서는 이러한 내용을 다음과 같이 짧게 처리할 수 있다.

```svelte
<script>
	import Inner from './Inner.svelte';
</script>

<Inner on:message/>
```

## DOM event forwarding

DOM 이벤트 역시 이벤트 포워딩을 처리할 수 있다. 해당 요소 본인이 처리할 이벤트 핸들러가 존재하지 않더라도, 상위 컴포넌트로 이를 전달해주는 역할을 한다.

```svelte
<button on:click>
	Click me
</button>
```
