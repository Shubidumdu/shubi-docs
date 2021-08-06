# Component composition

## Slot

일반적으로 요소가 자식 요소들을 가질 수 있는 것처럼, 컴포넌트 역시 똑같이 적용될 수 있다.

이는 `<slot>` 요소를 통해 구현할 수 있는데, React에서의 `{children}`과 매우 유사하다.

```svelte
<div class="box">
	<slot></slot>
</div>
```

## Slot fallbacks

컴포넌트는 slot이 비어있을 경우에 제공할 fallback을 지정할 수 있다.

```svelte
<div class="box">
	<slot>
		<em>no content was provided</em>
	</slot>
</div>
```

## Named slots

앞선 예시들은 모두 default slot을 사용했으나, 좀 더 구체적으로 어떤 slot에 위치해야 하는지에 대해 지정해주어야 하는 경우가 있을 수 있다.

이러한 경우에 named slot을 사용할 수 있다.

```svelte
<article class="contact-card">
	<h2>
		<slot name="name">
			<span class="missing">Unknown name</span>
		</slot>
	</h2>

	<div class="address">
		<slot name="address">
			<span class="missing">Unknown address</span>
		</slot>
	</div>

	<div class="email">
		<slot name="email">
			<span class="missing">Unknown email</span>
		</slot>
	</div>
</article>
```

이후, 각각의 slot에 요소를 추가하기 위해서는 `slot` 어트리뷰트를 추가적으로 작성해야 한다.

```svelte
<ContactCard>
	<span slot="name">
		P. Sherman
	</span>

	<span slot="address">
		42 Wallaby Way<br>
		Sydney
	</span>
</ContactCard>
```

## Checking for slot content

`$$slots`를 통해, 컴포넌트 측에서 전달받은 slot들에 어떤 것들이 있는지 파악할 수 있다. 이는 아래와 같은 형태다.

```svelte
$$slots = {
  default: false,
  comments: true
}
```

따라서, 컴포넌트에서는 이를 통해 다음과 같이 조건부로 UI를 구성할 수 있다.

```svelte
<article class:has-discussion={$$slots.comments}>
```

```svelte
{#if $$slots.comments}
	<div class="discussion">
		<h3>Comments</h3>
		<slot name="comments"></slot>
	</div>
{/if}
```

## Slot props

일부 상황에서는, 하위 컴포넌트에서 부모 컴포넌트로 데이터를 전달하여, 이에 따라 slot으로 전달할 내용을 변경해주어야 하는 경우가 생긴다.

이러한 상황에서 slot props를 활용할 수 있다. 가령, 아래의 `Hoverable` 컴포넌트는, slot에 `hovering` 값을 전달한다.

```svelte
<div on:mouseenter={enter} on:mouseleave={leave}>
	<slot hovering={hovering}></slot>
</div>
```

이후, `hovering`을 `<Hoverable>` 컴포넌트의 내용에 전달하기 위해, `let` 명령어를 사용한다.

```svelte
<Hoverable let:hovering={hovering}>
	<div class:active={hovering}>
		{#if hovering}
			<p>I am being hovered upon.</p>
		{:else}
			<p>Hover over me!</p>
		{/if}
	</div>
</Hoverable>
```

별도로 변수명을 다시 지어도 된다. 아래의 경우에는 `hovering`을 `active`라는 이름으로 부모 컴포넌트 측에서 다시 이름지었다.

```svelte
<Hoverable let:hovering={active}>
	<div class:active>
		{#if active}
			<p>I am being hovered upon.</p>
		{:else}
			<p>Hover over me!</p>
		{/if}
	</div>
</Hoverable>
```

Named slot에서도 props를 가질 수 있는데, 이 경우 컴포넌트 자체보다는 `slot="..."` 어트리뷰트를 보유한 요소에서 `let` 명령어를 사용하자.
