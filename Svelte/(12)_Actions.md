# Actions

## The use directive

Action은 기본적으로 요소 수준의 생명주기 함수다. 다음과 같은 내용들을 구현할 때 유용하다.

- 서드파티 라이브러리를 사용할 때
- 이미지에 대한 레이지 로딩
- 툴팁
- 커스텀 이벤트 핸들러 추가

여러 컴포넌트에 다양하게 사용될 로직들을 미리 모듈화 시켜놓고, `use`를 통해 사용하는 방식이라고 이해하면 편하다.

```svelte
export function pannable(node) {
	// setup work goes here...

	return {
		destroy() {
			// ...cleanup goes here
		}
	};
}
```

```svelte
import { pannable } from './pannable.js';

<div class="box"
	use:pannable
	on:panstart={handlePanStart}
	on:panmove={handlePanMove}
	on:panend={handlePanEnd}
	style="transform:
		translate({$coords.x}px,{$coords.y}px)
		rotate({$coords.x * 0.2}deg)"
></div>
```

## Adding parameters

트랜지션 및 애니메이션처럼, 액션 역시 인자를 전달받아서 로직 구성에 활용할 수 있다. 해당 매개변수는 `node` 다음의 두번째 매개변수로 전달받는다.

만약, 전달받는 매개변수가 변경될 수 있는 경우라면, `destroy` 외에도 `update`함수를 추가적으로 반환해주어야 한다.

```svelte
export function longpress(node, duration) {
	// ...

	const handleMousedown = () => {
		timer = setTimeout(() => {
			node.dispatchEvent(
				new CustomEvent('longpress')
			);
		}, duration);
	};

	// ...

  return {
    update(newDuration) {
      duration = newDuration;
    },
    destroy() {
      // ...
    }
  };
}
```

이 후, 아래와 같이 인수를 전달한다.

```svelte
<button use:longpress={duration}>
```

만약 여러개의 인자를 전달해야 하는 상황이라면, object를 전달하면 된다.

```svelte
<button use:longpress={{duration, spiciness}}>
```
