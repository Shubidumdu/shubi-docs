# Transitions

## The transition directive

Svelte는 `transition` 선언을 통해 트랜지션을 매우 쉽게 구현할 수 있다.

```svelte
<script>
	import { fade } from 'svelte/transition';
	let visible = true;
</script>

<label>
	<input type="checkbox" bind:checked={visible}>
	visible
</label>

<p transition:fade>Fades in and out</p>
```

## Adding parameters

트랜지션 함수들은 추가적으로 매개변수를 가질 수도 있다. 이번엔 `fade`가 아닌 `fly`의 예를 보자.

```svelte
<script>
	import { fly } from 'svelte/transition';
	let visible = true;
</script>

<p transition:fly="{{ y: 200, duration: 2000 }}">
	Flies in and out
</p>
```

트랜지션이 **reversible**하다는 점을 눈여겨보자. Svelte에서 제공하는 함수를 통해 구현된 트랜지션은 진행되는 도중에도 다시 되돌아올 수 있다.

## In and out

요소가 나타날 때와, 없어질 때 각각의 Transition을 다르게 구현하고자 하는 경우, `transition` 명령 대신, 요소에 `in`과 `out` 명령을 따로 지정할 수 있다.

```svelte
import { fade, fly } from 'svelte/transition';

<p in:fly="{{ y: 200, duration: 2000 }}" out:fade>
	Flies in, fades out
</p>
```

## Custom CSS transitions

`svelte/transition` 모듈에는 자체적으로 유용한 빌트인 트랜지션들이 많이 있으나, 직접 트랜지션을 구성하는 것도 쉽다.

아래는 `fade` 함수의 소스코드다.

```js
function fade(node, { delay = 0, duration = 400 }) {
  const o = +getComputedStyle(node).opacity;

  return {
    delay,
    duration,
    css: (t) => `opacity: ${t * o}`,
  };
}
```

이 함수는 두 개의 매개변수를 받는다. 하나는 트랜지션이 적용될 노트이고, 다른 하나는 아래의 옵션들이다.

- `delay` - 트랜지션이 시작되기 전의 딜레이, ms 단위
- `duration` - 트랜지션의 전체 길이, ms 단위
- `easing` - `p => t` easing 함수
- `css` - `(t, u) => css`함수, 여기서 `u === 1 - t` 이다.
- `tick` - 노드에 효과를 적용하는 `(t, u) => {...}` 함수

`t`는 인트로의 시작 또는 아웃트로의 끝에서 0이고, 인트로의 끝 또는 아웃트로의 시작에서 1이다.

가능하다면 대부분은 `tick` 프로퍼티가 아닌 `css` 프로퍼티를 반환해야 하는데, CSS 애니메이션은 가능하다면 브라우저의 버벅거림을 방지하기 위해 메인 스레드에서 실행되지 않기 때문이다. Svelte는 트랜지션을 시뮬레이션하고, CSS 애니메이션을 구성한 뒤 이를 실행한다.

이를테면, `fade` 트랜지션은 아래와 같은 CSS 애니메이션을 생성한다.

```css
0% {
  opacity: 0;
}
10% {
  opacity: 0.1;
}
20% {
  opacity: 0.2;
}
/* ... */
100% {
  opacity: 1;
}
```

좀 더 창의적이고 쓸데없는 애니메이션을 하나 만들어보자.

```svelte
<script>
	import { fade } from 'svelte/transition';
	import { elasticOut } from 'svelte/easing';

	let visible = true;

	function spin(node, { duration }) {
		return {
			duration,
			css: t => {
				const eased = elasticOut(t);

				return `
					transform: scale(${eased}) rotate(${eased * 1080}deg);
					color: hsl(
						${~~(t * 360)},
						${Math.min(100, 1000 - 1000 * t)}%,
						${Math.min(50, 500 - 500 * t)}%
					);`
			}
		};
	}
</script>
```

## Custom JS transitions

일반적으로는 가능하다면 CSS를 이용한 트랜지션을 사용하는 것이 좋지만, 일부 경우에는 JS 없이 구현하기 어려운 효과가 있을 수 있다. 대표적인 것이 타자기 효과다.

```svelte
<script>
	let visible = false;

	function typewriter(node, { speed = 50 }) {
		const valid = (
			node.childNodes.length === 1 &&
			node.childNodes[0].nodeType === Node.TEXT_NODE
		);

		if (!valid) {
			throw new Error(`This transition only works on elements with a single text node child`);
		}

		const text = node.textContent;
		const duration = text.length * speed;

		return {
			duration,
			tick: t => {
				const i = ~~(text.length * t);
				node.textContent = text.slice(0, i);
			}
		};
	}
</script>

<label>
	<input type="checkbox" bind:checked={visible}>
	visible
</label>

{#if visible}
	<p in:typewriter>
		The quick brown fox jumps over the lazy dog
	</p>
{/if}

```

## Transition events

트랜지션의 시작과 끝이 언제인지를 아는 것이 유용할 때가 있다. Svelte는 다른 DOM 이벤트들과 마찬가지로 해당 시점에 이벤트를 디스패치해준다.

```svelte
<p
	transition:fly="{{ y: 200, duration: 2000 }}"
	on:introstart="{() => status = 'intro started'}"
	on:outrostart="{() => status = 'outro started'}"
	on:introend="{() => status = 'intro ended'}"
	on:outroend="{() => status = 'outro ended'}"
>
	Flies in and out
</p>
```

## Local transitions

일반적으로 트랜지션은 컨테이너 블록이 추가되거나 없어지는 모든 경우에 실행된다.

만약, 모든 경우가 아니라, 요소 본인에 대한 직접적인 추가/삭제에 대해서만 트랜지션 효과를 주고자 한다면, local transition을 사용할 수 있다.

```svelte
<div transition:slide|local>
	{item}
</div>
```

## Deferred transitions

Svelte의 트랜지션 엔진이 갖는 강력한 특징은 트랜지션을 지연시킬 수 있다는 점이다. 따라서, 여러 개의 요소 간에도 이를 조정할 수 있다.

`crossfade` 함수는 `send`와 `receive`라는 두 쌍의 트랜지션을 만들어낸다. 어떤 요소가 `send`될 때, 해당 요소는 여기에 상응하는 `received` 요소를 찾고나서, 찾아낸 요소의 위치로 이동하며 트랜지션 효과를 실행한다.

```svelte
  <script>
	import { quintOut } from 'svelte/easing';
	import { crossfade } from 'svelte/transition';

	const [send, receive] = crossfade({
		duration: d => Math.sqrt(d * 200),

		fallback(node, params) {
			const style = getComputedStyle(node);
			const transform = style.transform === 'none' ? '' : style.transform;

			return {
				duration: 600,
				easing: quintOut,
				css: t => `
					transform: ${transform} scale(${t});
					opacity: ${t}
				`
			};
		}
	});

  // ...
  <script>
```

이후 아래와 주고 받게 될 각각의 요소에서 사용한다.

```svelte
<label
	in:receive="{{key: todo.id}}"
	out:send="{{key: todo.id}}"
>

<label
	class="done"
	in:receive="{{key: todo.id}}"
	out:send="{{key: todo.id}}"
>
```
