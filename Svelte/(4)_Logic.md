# Logic

Svelte는 조건 혹은 루프와 같은 로직을 처리할 수 있다.

## if

조건에 따라 특정 컴포넌트를 렌더링하고자 하는 경우, 아래와 같이 `if`를 통해 감싸주자.

```svelte
{#if user.loggedIn}
	<button on:click={toggle}>
		Log out
	</button>
{/if}

{#if !user.loggedIn}
	<button on:click={toggle}>
		Log in
	</button>
{/if}
```

## else

else에 대해서는 아래와 같이 처리할 수 있다.

```svelte
{#if user.loggedIn}
	<button on:click={toggle}>
		Log out
	</button>
{:else}
	<button on:click={toggle}>
		Log in
	</button>
{/if}
```

- `#` : 블럭을 여는 태그
- `/` : 블럭을 닫는 태그
- `:` : 블럭 내에서 사용되는 태그

## else-if

```svelte
{#if x > 10}
	<p>{x} is greater than 10</p>
{:else if 5 > x}
	<p>{x} is less than 5</p>
{:else}
	<p>{x} is between 5 and 10</p>
{/if}
```

## each

여러 개의 데이터가 Array 형태로 있는 경우, `each` 를 사용해 각각에 대한 순회 로직을 처리할 수 있다.

여기서의 Array는 배열 혹은 유사배열 객체라면 뭐든 가능하다.

또한, 두번째 인자로 `index`가 전달되기 때문에, 추가적으로 이를 이용할 수 있다.

```
{#each cats as cat, i}
	<li><a target="_blank" href="https://www.youtube.com/watch?v={cat.id}">
		{i + 1}: {cat.name}
	</a></li>
{/each}
```

## Keyed each

기본으로, `each`의 값에 대한 수정이 이루어지는 경우, 이들은 아예 처음부터 리렌더링 된다.

이 경우, 성능 상으로도 우려가 있을 뿐더러, 의도대로 동작하지 않을 가능성이 다분하다.

때문에, `each` 블럭에 별도의 고유 `id`를 지정해줌으로써 변경되지 않는 item에 대해서는 리렌더링을 방지해줄 수 있다.

```svelte
{#each things as thing (thing.id)}
	<Thing current={thing.color}/>
{/each}
```

사실 Svelte는 내부적으로 key 관리에 `Map`을 사용하기 때문에, 무엇이든지 `key`로써 사용할 수 있다. 다시말해, 위의 경우에는 굳이 `thing.id`가 아닌 `thing`을 사용해도 된다. **하지만, 일반적으로는 string 혹은 number를 사용하는 것이 안전하다.** key 변경 감지에 단순히 참조에 대한 동일성을 확인하기 때문.

## Await

대부분의 웹 애플리케이션은 비동기적으로 데이터를 다루어야만 하는 경우가 발생한다. Svelte는 마크업 내에서 직접적으로 promise를 다룰 수 있게끔 한다.

```svelte
{#await promise}
	<p>...waiting</p>
{:then number}
	<p>The number is {number}</p>
{:catch error}
	<p style="color: red">{error.message}</p>
{/await}
```

오직 가장 최신의 promise에 대해서만 고려되며, 다시 말해 race condition에 대해 신경 쓸 필요가 없다.

사용되는 promise가 rejected 상태가 될 염려가 없는 경우에는, `catch` 블럭을 제거할 수도 있으며, 심지어 resolved 상태 이전에 별도로 보여주고자 하는 내용이 없는 경우에는 이조차 없애도 상관없다.

```svelte
{#await promise then value}
	<p>the value is {value}</p>
{/await}
```
