# Bindings

## Text inputs

일반적으로, Svelte에서의 데이터 흐름은 탑-다운 형식이다. 부모 컴포넌트에서 자식 컴포넌트에 props를 전달할 수 있고, 컴포넌트는 보유한 요소들에 대해 어트리뷰트를 설정할 수 있다.

가끔, 이러한 규칙을 깨야하는 경우가 있는데, 대표적인 경우가 컴포넌트 내에 `<input>` 태그를 보유한 경우다. 우리는 `on:input`에 대한 이벤트 핸들러를 추가하여 `event.target.name`에 따라 별도의 상태값을 변경하도록 구성할 수 있다. 다만, 이러한 과정 자체를 매번 반복하게 되면 너무 번거롭다.

대신에, 이러한 상황에 `bind:value` 명령을 사용할 수 있다.

```svelte
<script>
	let name = 'world';
</script>

<input bind:value={name}>

<h1>Hello {name}!</h1>
```

`bind`를 사용하는 경우, `input`의 `value`값 변경에 따라 `name`의 값을 변경할 뿐만 아니라, `name`의 값이 변경됨에 따라 `value`값 역시 변경된다.

## Numeric inputs

DOM 상에서, 모든 값들은 string 으로 다루어진다. 이런 경우, `type="number"` 혹은 `type="range"`인 상황에서, 값을 다루기에 다소 까다로워지며, 별도로 데이터 타입을 변환해주어야 한다.

Svelte에서는 `bind:value`를 사용하면, 알아서 이러한 과정을 처리해준다.

```svelte
<input type=number bind:value={a} min=0 max=10>
<input type=range bind:value={a} min=0 max=10>
```

## Checkbox inputs

체크박스들은 상태 값들을 토글링(toggling)하기 위해 사용된다. 이 경우, 이들의 상태값은 `value`가 아닌 `checked`가 되므로, 다음과 같이 사용해야 한다.

```svelte
<input type=checkbox bind:checked={yes}>
```

## Group inputs

동일한 값에 대한 여러 input들이 존재한다면, `bind:group`을 사용할 수 있다. 대표적으로 `radio` 혹은 `checkbox` 태그가 있다.

```svelte
<script>
	let scoops = 1;
	let flavours = ['Mint choc chip'];

	let menu = [
		'Cookies and cream',
		'Mint choc chip',
		'Raspberry ripple'
	];

	function join(flavours) {
		if (flavours.length === 1) return flavours[0];
		return `${flavours.slice(0, -1).join(', ')} and ${flavours[flavours.length - 1]}`;
	}
</script>

<h2>Size</h2>

<label>
	<input type=radio bind:group={scoops} value={1}>
	One scoop
</label>

<label>
	<input type=radio bind:group={scoops} value={2}>
	Two scoops
</label>

<label>
	<input type=radio bind:group={scoops} value={3}>
	Three scoops
</label>

<h2>Flavours</h2>

{#each menu as flavour}
	<label>
		<input type=checkbox bind:group={flavours} value={flavour}>
		{flavour}
	</label>
{/each}

{#if flavours.length === 0}
	<p>Please select at least one flavour</p>
{:else if flavours.length > scoops}
	<p>Can't order more flavours than scoops!</p>
{:else}
	<p>
		You ordered {scoops} {scoops === 1 ? 'scoop' : 'scoops'}
		of {join(flavours)}
	</p>
{/if}
```

## Textarea inputs

`<textarea>` 요소는 `<text>`와 거의 동일하게 동작하며, `bind:value`를 사용하면 된다.

```svelte
<textarea bind:value={value}></textarea>
```

만약, 변경할 상태의 변수명이 똑같이 `value`로 일치한다면, 아래와 같이 약식으로 작성할 수 있다.

```svelte
<textarea bind:value></textarea>
```

## Select bindings

`<select>` 요소에도 `bind:value`를 사용할 수 있다.

```svelte
<select bind:value={selected} on:change="{() => answer = ''}">
```

`<select>` 하위에 있는 `<option>` 값들이 string이 아닌 object임을 주의하자.

`selected`에 대한 기본값을 설정하지 않았기 때문에, binding 시에 기본적으로 `select`는 리스트의 첫번째 값을 `selected`로 설정해준다.

## Select multiple

`<select>`는 `multiple` 어트리뷰트를 사용할 수 있으며, 이 경우 하나 이상의 값들을 **Array 형태로 받아올 수 있게 된다.**

```svelte
<h2>Flavours</h2>

<select multiple bind:value={flavours}>
	{#each menu as flavour}
		<option value={flavour}>
			{flavour}
		</option>
	{/each}
</select>
```

## Contenteditable bindings

`contenteditable="true"` 어트리뷰트를 보유한 요소들은 `textContent`와 `innerHTML`에 대해서도 바인딩을 할 수 있다.

```svelte
<div
	contenteditable="true"
	bind:innerHTML={html}
></div>
```

## Each block bindings

`each` 문 내에서도 바인딩을 할 수 있다.

```svelte
<script>
	let todos = [
		{ done: false, text: 'finish Svelte tutorial' },
		{ done: false, text: 'build an app' },
		{ done: false, text: 'world domination' }
	];

	function add() {
		todos = todos.concat({ done: false, text: '' });
	}

	function clear() {
		todos = todos.filter(t => !t.done);
	}

	$: remaining = todos.filter(t => !t.done).length;
</script>

<h1>Todos</h1>

{#each todos as todo}
	<div class:done={todo.done}>
		<input
			type=checkbox
			bind:checked={todo.done}
		>

		<input
			placeholder="What needs to be done?"
			bind:value={todo.text}
		>
	</div>
{/each}

<p>{remaining} remaining</p>

<button on:click={add}>
	Add new
</button>

<button on:click={clear}>
	Clear completed
</button>

<style>
	.done {
		opacity: 0.4;
	}
</style>
```

이 경우 바인딩은 `todos` Array를 직접 변경하게 된다. 본인이 불변성을 유지하는 형태를 선호한다면, 이 경우엔 바인딩을 사용하지 말고 이벤트 핸들러를 직접 작성하는 편이 좋다.

## Media elements

`<audio>`와 `<video>` 요소는 바인딩 할 수 있는 수많은 프로퍼티들이 존재한다.

아래는 바인딩 가능한 읽기 전용 프로퍼티다.

- `duration` (readonly) — 비디오 및 오디오의 전체 시간, 초 단위
- `buffered` (readonly) — {start, end} 객체 Array
- `seekable` (readonly) — {start, end} 객체 Array
- `played` (readonly) — {start, end} 객체 Array
- `seeking` (readonly) — boolean
- `ended` (readonly) — boolean

비디오의 경우는 `videoWidth` 및 `videoHeight` 읽기 전용 속성이 추가적으로 존재한다.

아래는 쌍방향으로 바인딩 가능한 프로퍼티들이다.

- `currentTime` - 비디오 및 오디오의 현재 위치
- `playbackRate` - 비디오 및 오디오의 재생 속도, `1`이 기본.
- `paused` - 정지 여부
- `volume` - `0`부터 `1` 사이의 값
- `muted` - 음소거 여부, boolean

## Dimensions

모든 블록 요소들은 `clientWidth`, `clientHeight`, `offsetWidth`, 그리고 `offsetHeight`에 대한 바인딩을 할 수 있다.

```
<div bind:clientWidth={w} bind:clientHeight={h}>
	<span style="font-size: {size}px">{text}</span>
</div>
```

이들 바인딩은 읽기 전용이며, 이들 값을 직접 변경하는 것은 아무 의미가 없다.

> **주의** :
> 요소의 크기 등을 측정하기 위해서는 내부적으로 [이런 테크닉](http://www.backalleycoder.com/2013/03/18/cross-browser-event-based-element-resize-detection/)을 사용한다. 이 경우, 오버헤드에 대한 우려 때문에 너무 많은 수에 대해 해당 바인딩을 사용하지 않는 것을 추천한다.

## This

`this`에 대한 바인딩은 읽기 전용이며, 모든 요소 및 컴포넌트에 사용할 수 있다.

이는 React에서의 Ref와 유사하며, 요소 및 컴포넌트에 대한 참조값을 얻을 수 있다.

```svelte
<canvas
	bind:this={canvas}
	width={32}
	height={32}
></canvas>
```

위 예시에서 `canvas`는 컴포넌트가 마운트되기 전까지는 `undefined`임에 유의하자. 때문에, 마운트 시에 해당 컴포넌트 및 요소에 특정 로직을 적용하려면 `onMount` 등의 라이프사이클 메서드를 사용해야 한다. 이에 대해선 추후 살펴본다.

## Component bindings

DOM 요소들의 프로퍼티에 대해 바인딩을 할 수 있는 것처럼, 컴포넌트의 props에 대해서도 바인딩을 할 수 있다.

```svelte
<Keypad bind:value={pin} on:submit={handleSubmit}/>
```

이 경우, 하위 컴포넌트에서 상태 변화가 일어나면, 바인딩을 적용한 부모 요소에 대해서도 이에 따른 변경이 일어난다.

> **주의** : 가능하다면 컴포넌트 바인딩은 사용하지 않는 편이 좋다. 애플리케이션 규모가 커지면서, 데이터 흐름이 너무 많아지는 경우 이에 대한 추적이 어려울 수 있다. 이러한 문제는 single souce of truth가 없는 경우에 더욱 심각해진다.
