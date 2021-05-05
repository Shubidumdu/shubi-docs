# Special elements

## <svelte:self>

Svelte는 여러 빌트인 요소들을 제공한다. 먼저, `<svelte:self>`는 컴포넌트를 재귀적으로 활용할 수 있게 해준다. 기본적으로 모듈은 스스로를 import 하는 것이 불가능하기 때문에, `<svelte:self>`가 필요하다.

이는 폴더 안에 다른 폴더가 포함될 수 있는 폴더 트리 뷰와 같은 것들을 구성할 때 유용하다.

```svelte
{#if file.type === 'folder'}
	<svelte:self {...file}/>
{:else}
	<File {...file}/>
{/if}
```

## <svelte:component>

`<svelte:component>`는 동적으로 특정 컴포넌트가 해당 요소의 위치가 올 수 있을 경우에 활용할 수 있다.

이를테면, 아래처럼 조건부로 컴포넌트를 렌더링하고자 할 때, `<svelte:component>`를 통해 하나의 동적 컴포넌트로 처리해줄 수 있다.

```svelte
{#if selected.color === 'red'}
	<RedThing/>
{:else if selected.color === 'green'}
	<GreenThing/>
{:else if selected.color === 'blue'}
	<BlueThing/>
{/if}
```

위의 코드는 아래의 한줄로 대체될 수 있다.

```svelte
<svelte:component this={selected.component}/>
```

## <svelte:window>

바닐라 JS 상에서 어떤 DOM 요소에든 이벤트 리스너를 추가할 수 있듯, Svelte에서 `window` 오브젝트에 이벤트 리스너를 추가하고자 한다면 `<svelte:window>`를 이용하면 된다.

```svelte
<svelte:window on:keydown={handleKeydown}/>
```

## <svelte:window> bindings

`window`의 특정 프로퍼티에 바인딩을 해줄 수도 있다.

```svelte
<svelte:window bind:scrollY={y}/>
```

아래는 바인딩 가능한 프로퍼티의 리스트다.

- `innerWidth`
- `innerHeight`
- `outerWidth`
- `outerHeight`
- `scrollX`
- `scrollY`
- `online` - `window.navigator.onLine`과 동일

`scrollX`와 `scrollY`을 제외한 모두는 읽기 전용 프로퍼티다.

## <svelte:body>

`<svelte:window>`와 비슷하게, `<svelte:body>` 요소 역시 `document.body`에 직접적으로 이벤트 리스너를 추가해야 하는 경우에 사용할 수 있다.

`window`에서는 발생하지 않는 `mouseenter` 혹은 `mouseleave` 이벤트를 사용해야 하는 경우에 유용하다.

```svelte
<svelte:body
	on:mouseenter={handleMouseenter}
	on:mouseleave={handleMouseleave}
/>
```

## <svelte:head>

`<svelte:head>` 요소는 문서의 `<head>` 내에 요소를 추가해야할 때 사용할 수 있다.

```svelte
<svelte:head>
	<link rel="stylesheet" href="tutorial/dark-theme.css">
</svelte:head>
```

SSR 모드의 경우, `<svelte:head>`의 내용은 HTML의 나머지와 별도로 반환된다.

## <svelte:options>

`<svelte:options>`는 컴파일링 옵션을 설정할 수 있게 해준다.

예를 들어, 해당 컴포넌트에서 설정할 수 있는 `immutable` 옵션이 존재하는데, 이는 해당 컴포넌트가 immutable 데이터를 기반으로 동작함을 의미한다.

이를테면, 아래와 같이 `Todo` 컴포넌트가 존재한다고 하자.

```svelte
<script>
	import { afterUpdate } from 'svelte';
	import flash from './flash.js';

	export let todo;

	let div;

	afterUpdate(() => {
    // 아래의 flash는 애니메이션 효과.
		flash(div);
	});
</script>

<!-- the text will flash red whenever
     the `todo` object changes -->
<div bind:this={div} on:click>
	{todo.done ? '👍': ''} {todo.text}
</div>

<style>
	div {
		cursor: pointer;
		line-height: 1.5;
	}
</style>
```

헌데, 아래처럼 여러 개의 `Todo`를 갖는 리스트를 구현하려고 하는 상황을 생각해보자.

`toggle`이 실행될 때, `map` 메서드에 의해서 새로운 `todos`가 반환되며, 이에 따라 하위의 일부 `Todo`에 대해서는 결론적으로는 변경된 데이터가 없어 굳이 새로 리렌더링할 필요가 없음에도 모든 `Todo` 컴포넌트가 리렌더링 과정을 거치게 된다.

```js
let todos = [
  { id: 1, done: true, text: 'wash the car' },
  { id: 2, done: false, text: 'take the dog for a walk' },
  { id: 3, done: false, text: 'mow the lawn' },
];

function toggle(toggled) {
  todos = todos.map((todo) => {
    if (todo === toggled) {
      // return a new object
      return {
        id: todo.id,
        text: todo.text,
        done: !todo.done,
      };
    }

    // return the same object
    return todo;
  });
}
```

이러한 상황을 방지하기 위해서, immutable 옵션을 통해 props 값의 참조가 유지된다면 리렌더링을 수행하지 않도록 한다.

```svelte
// <svelte:options immutable/>을 써도 된다.

<svelte:options immutable={true}/>
```

아래는 `<svelte:options>`에서 설정할 수 있는 옵션들이다.

- `immutable={true}` - mutable 데이터를 사용하지 않겠다는 뜻으로, 컴포넌트는 값의 변경 여부를 확인하기 위해 단순 참조 비교(simple referential quality check)를 거친다.
- `immutable={false}` - 기본값. 값이 변경되었는지의 여부에 대해 더 엄격하게 체크한다.
- `accessors={true}` - 컴포넌트의 props에 대한 getter와 setter를 추가한다.
- `accessors={false}` - 기본값.
- `namespace="..."` - 컴포넌트가 사용될 네임스페이스. 일반적으로 `svg`에서 사용된다.
- `tag="..."` - 컴포넌트를 커스텀 요소로 컴파일링할 때 사용할 이름.

## <svelte:fragment>

`<svelte:fragment>` 요소는 named slot에 요소를 전달하고자 할 때, 굳이 별도의 컨테이너(ex. `div`)를 통해 요소를 묶어주지 않아도 곧바로 전달할 수 있게끔 해준다. React에서의 Fragment와 동일하다.(`<></>`)

```svelte
<svelte:fragment slot="footer">
	<p>All rights reserved.</p>
	<p>Copyright (c) 2019 Svelte Industries</p>
</svelte:fragment>
```
