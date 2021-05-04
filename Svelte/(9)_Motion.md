# Motion

## Tweened

Svelte는 상호작용에 따른 매끄러운 UI 애니메이션을 구성하기 위한 툴을 제공한다.

`tweened`를 통해 `progress` store를 변경해보자.

```svelte
<script>
	import { tweened } from 'svelte/motion';
	import { cubicOut } from 'svelte/easing';

	const progress = tweened(0, {
		duration: 400,
		easing: cubicOut
	});
</script>
```

`tweened`의 첫번째 인수에는 초기값이, 두번째 인수에는 `options`가 전달된다. 가능한 `option`에는 다음과 같은 것들이 있다.

- `delay` - tween이 시작되기 전의 딜레이. `ms`단위.
- `duration` - tween의 동작 시간, `ms`단위 혹은 `(from, to) => ms` 함수
- `easing` - `p => t` 함수
- `interplate` - 커스텀 `(from, to) => t => value` 함수. 기본적으로 Svelte는 number, date, 동일한 모양의 object, array에 대해서만 지원하기 때문에, 컬러스트링 등에 대해 적용하기 위해서는 별도로 커스텀 interpolator를 전달해야한다.

해당 옵션들을 `progress.set` 혹은 `progress.update`의 두번째 인수로 전달할 수 있으며, 이 경우 기본 옵션에 오버라이딩된다. `set`와 `update` 메서드는 tween이 완료될 때 resolved 되는 프라미스를 반환한다.

## Spring

`spring` 함수는 `tweened`보다 좀 더 자주 변경되는 값에 더 최적화된 함수이다.

```svelte
<script>
	import { spring } from 'svelte/motion';

	let coords = spring({ x: 50, y: 50 });
	let size = spring(10);
</script>
```

추가적으로 각각 `0`과 `1` 사이의 `{stiffness, damping}` 옵션을 넘겨줄 수 있다.

```svelte
let coords = spring({ x: 50, y: 50 }, {
	stiffness: 0.1,
	damping: 0.25
});
```
