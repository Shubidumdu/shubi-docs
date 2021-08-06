# Animations

## The animate directive

트랜지션이 적용되지 않는 컴포넌트들에 대해서도 애니메이션을 적용해야하는 경우가 있다. 이를테면, 리스트의 아이템 하나가 삭제됨에 따라 다른 아이템들을 서서히 이동하는 것을 구현해야 하는 경우다.

이를 위해서는 `flip` 함수를 사용한다. 이는 '[First, Last, Invert, Play](https://aerotwist.com/blog/flip-your-animations/)'의 준말이다.

```svelte
import { flip } from 'svelte/animate';
```

이후, 트랜지션 외에 애니메이션이 적용되어야 하는 컴포넌트 및 요소에 animation을 추가해준다.

```svelte
<label
	in:receive="{{key: todo.id}}"
	out:send="{{key: todo.id}}"
	animate:flip="{{duration: 200}}"
>
```

위에서의 `duration`은 `d => ms` 형태의 함수일수도 있다. 여기서의 `d`는 요소가 움직여야 할 픽셀의 갯수에 해당한다.

다시 한번, 모든 트랜지션과 애니메이션은 JS보다는 CSS 상에서 구현되어야 한다는 점을 기억하자. 이는 메인 스레드의 진행을 막는 것을 방지하기 위해서다.
