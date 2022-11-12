# DOM 계층 구조 이해하기

TS에서는 이벤트를 다룰 때, `EventTarget`에 달린 `Node`의 구체적인 타입을 안다면 개발 및 디버깅이 용이하며, 언제 타입 단언을 사용해야 하는지 판단하기 쉽습니다.

![img1](https://javascript.info/article/basic-dom-node-properties/dom-class-hierarchy.svg)

한편, `Event` 역시 `MouseEvent`, `UIEvent` 등 보다 구체적인 타입들이 존재합니다.

실제 개발 중에 이벤트 핸들러를 다룰 때에 `Event` 및 `EventTarget` 보다는 `HTMLDivElement`, `PointerEvent` 등 구체적인 타입을 선언 및 단언하여 개발을 해나가는 것이 좋습니다.

```ts
function addDragHandler(el: HTMLElement) {
  el.addEventListener('mousedown', eDown => {
    const dragStart = [eDown.clientX, eDown.clientY];
    const handleUp = (eUp: MouseEvent) => {
      el.classList.remove('dragging');
      el.removeEventListener('mouseup', handleUp);
      const dragEnd = [eUp.clientX, eUp.clientY];
      console.log('dx, dy = ', [0, 1].map(i => dragEnd[i] - dragStart[i]));
    }
    el.addEventListener('mouseup', handleUp);
  });
}

const div = document.getElementById('surface');

if (div) {
  addDragHandler(div);
}
```

특히 DOM을 다룰 때에는 TS 타입 체커보다 개발자인 우리가 더 타입에 대해 정확히 알고 있는 경우가 많으므로 타입 단언을 사용해도 좋습니다.

```ts
document.getElementById('my-div') as HTMLDivElement;
```
