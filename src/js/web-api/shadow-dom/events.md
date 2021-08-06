# Shadow DOM and events

shadow tree에 담긴 기본 아이디어는 컴포넌트 내부적인 실행 세부사항에 대해 캡슐화를 적용하는 것이다.

`<user-card>`의 shadow DOM 내부에 클릭 이벤트가 발생했다고 가정하자. 이 때, 메인 문서는 shadow DOM 내부에 대해 알 수 있는 방법이 없다. 이는 특히 서드파티 라이브러리로부터의 컴포넌트를 활용할 때 더 두드러진다.

따라서, 캡슐화를 유지하기 위해, 브라우저는 이벤트를 리타겟팅(**retarget**)한다.

**shadow DOM 내부에서 일어나는 이벤트들은 컴포넌트 외부에서 볼 때, 그들의 **host** 요소를 `target`으로 삼는다.**

말이 좀 헷갈릴 수 있는데, 다시 말해, shadow DOM 측에서 봤을 때는 `div`, `span` 등에서 이벤트가 발생했더라도, 메인 문서 측에서는 해당 이벤트의 타겟을 항상 `user-card`와 같은 host로 본다.

아래는 간단한 예시다.

```html
<user-card></user-card>

<script>
  customElements.define(
    'user-card',
    class extends HTMLElement {
      connectedCallback() {
        this.attachShadow({ mode: 'open' });
        this.shadowRoot.innerHTML = `<p>
      <button>Click me</button>
    </p>`;
        this.shadowRoot.firstElementChild.onclick = (e) =>
          alert('Inner target: ' + e.target.tagName);
      }
    },
  );

  document.onclick = (e) => alert('Outer target: ' + e.target.tagName);
</script>
```

위 예시에서는, shadow DOM과 light DOM에서 바라보는 target이 달라지게 된다.

1. 내부 타겟: `BUTTON` - shadow DOM을 통한 컴포넌트 내부의 이벤트 핸들러는 올바른 `target`을 가져온다.

2. 외부 타겟: `USER-CARD` - 문서에서 사용하는 이벤트 핸들러는 shadow host(`user-card`)를 타겟으로 가져온다.

이벤트 리타겟팅은 마땅히 존재해야 하는데, 컴포넌트 내부에서 발생하는 일들에 대해 외부의 문서가 신경을 쓸 필요가 없기 떄문이다. 때문에, 문서의 관점에서는 단순히 `<user-card>`에서 발생한 이벤트라고만 인식하는 것이다.

**리타겟팅은 slot 처리된 요소에서는 발생하지 않는데, 왜냐하면 애초에 해당 요소는 light DOM에서부터 온 것이기 때문이다.**

예를 들어, 아래 예시에서 `<span slot='username'>`의 이벤트 타겟은 정확히 `span`이 된다. 이는 shadow와 light 이벤트 핸들러 양측이 동일하다.

```HTML
<user-card id="userCard">
  <span slot="username">John Smith</span>
</user-card>

<script>
customElements.define('user-card', class extends HTMLElement {
  connectedCallback() {
    this.attachShadow({mode: 'open'});
    this.shadowRoot.innerHTML = `<div>
      <b>Name:</b> <slot name="username"></slot>
    </div>`;

    this.shadowRoot.firstElementChild.onclick =
      e => alert("Inner target: " + e.target.tagName);
  }
});

userCard.onclick = e => alert(`Outer target: ${e.target.tagName}`);
</script>
```

단, 위 예시에서 `<b>Name:</b>`과 같이 slot에 해당하지 않는 요소로부터 발생한 이벤트는 마찬가지로 host를 타겟으로 삼는다.

## 버블링, event.composedPath()

flattened DOM은 이벤트 버블링을 위해서 사용된다.

따라서, 만약 slot 처리된 요소가 존재한다면, 그 안에서 발생한 이벤트는 `<slot>`을 거쳐 상위로 버블링된다.

shadow 요소들을 포함한 원래 이벤트 타겟에 대한 전체 경로(full-path)는 `event.composedPath()` 메서드를 통해 확인할 수 있다. 메서드의 이름에서부터 알 수 있듯이, 여기서 반환받는 경로는 composition 단계 이후의 path이다.

예를 들어, 아래의 flatten DOM이 있다고 가정하자.

```HTML
<user-card id="userCard">
  #shadow-root
    <div>
      <b>Name:</b>
      <slot name="username">
        <span slot="username">John Smith</span>
      </slot>
    </div>
</user-card>
```

이제, `<span slot='username'>`을 클릭했을 때, `event.composedPath()`를 확인한다면, 다음 배열을 반환한다.

`[span, slot, div, shadow-root, user-card, body, html, document, window]`

이는 composition 이후 생성된 flatten DOM에서의 타겟 요소로부터 부모로 뻗어나가는 체이닝이다.

### 주의

> shadow tree의 세부사항들은 오직 `{mode: 'open'}` 옵션이 있을 때만 제공된다.
> 만약, 그렇지 않다면 `event.composedPath()`역시 `user-card`에서부터 시작한다.
> 이는 shadow DOM이 동작하는 다른 메서드의 원칙과 유사한데, 닫힌(closed) 트리는 내부적으로 완전히 숨겨진다.

## event.composed

대부분의 이벤트들은 shadow DOM 경계를 거쳐 완전히 버블링된다. 하지만 몇개의 예외가 존재한다.

이 경우 `composed` 이벤트 객체 프로퍼티에 의해 제어될 수 있는데, 만약 `true`에 해당하는 경우, 해당 이벤트는 shadow DOM의 경계를 넘어간다. `false`인 경우, 이벤트는 오직 shadow DOM 내부에서만 탐색된다.

아래 대부분의 이벤트는 `composed: true`이다.

- `blur`, `focus`, `focusin`, `focusout`
- `click`, `dblclick`
- `mousedown`, `mouseup`, `mousemove`, `mouseout`, `mouseover`
- `wheel`
- `beforeinput`, `input`, `keydown`, `keyup`

모든 터치 이벤트와 포인터 이벤트 역시 `composed: true`로 설정된다.

아래는 `composed: false`에 해당하는 일부 이벤트들이다.

- `mouseenter`, `mouseleave` (얘넨 애초에 버블링이 없다.)
- `load`, `unload`, `abort`, `error`
- `select`
- `slotchange`

해당 이벤트들은 오직 해당 요소가 동일하게 위치한 DOM 내에서만 확인될 수 있다.

## 커스텀 이벤트

임의로 작성한 커스텀 이벤트를 발생시킬 때, `bubble`과 `composed` 프로퍼티를 설정할 수 있다.

예를 들어, 아래에서 `div#outer`의 shadow DOM에 `div#inner`을 만들고 거기에 두 이벤트를 트리거하자. 그러면, 오직 `composed: true`로 설정한 이벤트만이 DOM 경계를 넘어 문서 바깥으로 나올 수 있다.

```HTML
<div id="outer"></div>

<script>
outer.attachShadow({mode: 'open'});

let inner = document.createElement('div');
outer.shadowRoot.append(inner);

/*
div(id=outer)
  #shadow-dom
    div(id=inner)
*/

document.addEventListener('test', event => alert(event.detail));

inner.dispatchEvent(new CustomEvent('test', {
  bubbles: true,
  composed: true,
  detail: "composed"
}));

inner.dispatchEvent(new CustomEvent('test', {
  bubbles: true,
  composed: false,
  detail: "not composed"
}));
</script>
```
