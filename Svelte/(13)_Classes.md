# Classes

## The class directive

다른 어트리뷰트와 마찬가지로, JS 어트리뷰트를 통해 class를 지정할 수 있다.

```svelte
<button
	class="{current === 'foo' ? 'selected' : ''}"
	on:click="{() => current = 'foo'}"
>foo</button>
```

위와 같은 것은 UI 개발 상에서 일반적인 패턴으로, Svelte에서는 이를 단순화하기 위한 특별한 명령어를 갖고 있다. 아래 코드는 위와 동일하다.

```svelte
<button
	class:selected="{current === 'foo'}"
	on:click="{() => current = 'foo'}"
>foo</button>
```

## Shorthand class directive

종종, 클래스 이름은 그것이 의존하는 변수명과 동일한 경우가 많다.

```svelte
<div class:big={big}>
	<!-- ... -->
</div>
```

이 경우, 아래와 같이 짧게 작성할 수 있다.

```svelte
<div class:big>
	<!-- ... -->
</div>
```
