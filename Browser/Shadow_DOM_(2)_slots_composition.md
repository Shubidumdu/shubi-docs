# Shadow DOM - slots, composition

탭, 갤러리 등 많은 종류의 컴포넌트들은 렌더링할 **내용**들이 필요하다.

내장 `<select>` 태그가 `<option>` 태그들을 요구하는 것처럼, 우리가 임의로 만든 태그 역시 임의의 태그를 요구할 수 있다.

```html
<custom-menu>
  <title>Candy menu</title>
  <item>Lollipop</item>
  <item>Fruit Toast</item>
  <item>Cup Cake</item>
</custom-menu>
```

우리는 이것을 동적으로 요소들의 내용을 분석하고, DOM 노드들을 조작해서 구현할 수 있다. 하지만, shadow DOM의 경우, 문서에서의 스타일링이 적용되지 않으며, 때문에 어느 정도의 추가 코드을 요구한다.

다행히도 Shadow DOM은 여기서 `<slot>` 요소를 제공한다. 이는 light DOM으로부터 가져온 내용들로 Shadow DOM의 내용을 채울 수 있게 해준다.

## Named slots

간단한 예시로부터 살펴보자. 여기 `<user-card>` shadow DOM은 두개의 슬롯(slot)을 사용하며, 이는 light DOM으로부터 아래와 같이 채워질 수 있다.

```HTML
<script>
customElements.define('user-card', class extends HTMLElement {
  connectedCallback() {
    this.attachShadow({mode: 'open'});
    this.shadowRoot.innerHTML = `
      <div>Name:
        <slot name="username"></slot>
      </div>
      <div>Birthday:
        <slot name="birthday"></slot>
      </div>
    `;
  }
});
</script>

<user-card>
  <span slot="username">John Smith</span>
  <span slot="birthday">01.01.2001</span>
</user-card>
```

섀도우 DOM에서, `<slot name='X'>`은 **삽입 지점**을 의미하며, 여기에는 추후 `slot='X'`를 light DOM에서 지정한 요소가 위치하게 된다.

이후 브라우저는 **합성_composition**을 수행하는데, 이는 light DOM에서 요소를 가져와 이에 대응하는 shadow DOM의 slot에 렌더링시키는 과정이다.

스크립트가 동작한 후, 아직 **합성_composition**이 동작하지 않은 상태의 DOM 구조는 아래와 같다.

```HTML
<user-card>
  #shadow-root
    <div>Name:
      <slot name="username"></slot>
    </div>
    <div>Birthday:
      <slot name="birthday"></slot>
    </div>
  <span slot="username">John Smith</span>
  <span slot="birthday">01.01.2001</span>
</user-card>
```

현 시점에서는, shadow DOM까지는 생성되었으나(이에 따라 `#shadow-root`가 보인다), 현재 요소는 light와 shadow DOM 모두를 갖고 있다.

렌더링을 하기 위해, shadow DOM에서의 각 `<slot name="...">`에서 브라우저는 light DOM에서 동일한 이름을 가진 `slot="..."`을 찾는다. 이후 이 요소들은 각 slot 안에 렌더링된다.

그 결과로 만들어진 아래의 DOM 구조를 **flatten DOM** 이라고 한다.

```html
<user-card>
  #shadow-root
    <div>Name:
      <slot name="username">
        <!-- slotted element is inserted into the slot -->
        <span slot="username">John Smith</span>
      </slot>
    </div>
    <div>Birthday:
      <slot name="birthday">
        <span slot="birthday">01.01.2001</span>
      </slot>
    </div>
</user-card>
```

다만, 유의해야 할 점이 있다. **flatten DOM은 오직 렌더링과 이벤트 핸들링의 목적으로 존재한다**. 이는 어떤 식으로 동작하는지를 보여주기 위한 것이며, 실제 문서의 노드들은 어디로도 이동하지 않는다.

이는 단순히 `querySelectorAll`를 통해 확인해볼 수 있다.

```js
// light DOM <span> nodes are still at the same place, under `<user-card>`
alert( document.querySelectorAll('user-card span').length ); // 2
```

결국, flatten DOM은 shadow DOM에서 slot에 대한 삽입을 통해 만들어진다. 브라우저는 이를 렌더링, 스타일 상속, 이벤트 전파의 목적으로 활용한다. 하지만, JS는 여전히 flatten이 이루어지기 전의 문서만을 볼 수 있다.

### **유의!**
```html
<user-card>
  <span slot="username">John Smith</span>
  <div>
    <!-- invalid slot, must be direct child of user-card -->
    <span slot="birthday">01.01.2001</span>
  </div>
</user-card>
```

> `slot-'...'` 속성을 가진 태그는 최상위의 자식 노드여야 한다. 보다 깊은 곳에 위치한 노드들은 무시된다.


한편, 똑같은 slot에 지정된 여러개의 요소들이 light DOM에 존재한다면, **이들은 갱신되는 것이 아니라, 순서대로 slot에 추가된다.**

예를 들어, 앞선 예시에 대해 아래와 같이 light DOM을 구성했다고 가정하자.

```html
<user-card>
  <span slot="username">John</span>
  <span slot="username">Smith</span>
</user-card>
```

그렇다면, 그 결과 생겨난 flatten DOM의 결과는 아래와 같다.

```HTML
<user-card>
  #shadow-root
    <div>Name:
      <slot name="username">
        <span slot="username">John</span>
        <span slot="username">Smith</span>
      </slot>
    </div>
    <div>Birthday:
      <slot name="birthday"></slot>
    </div>
</user-card>
```

## Slot fallback content

만약, `<slot>`태그에 어떤 값이 존재한다면, 이는 **fallback**(대비책)이 된다. 다시말해, **default**값이 된다. 브라우저는 light DOM에서 상응하는 `slot='...'` 요소를 찾지 못하는 경우 해당 기본값을 렌더링한다.

```html
<div>Name:
  <slot name="username">Anonymous</slot>
</div>
```

## Default slot: first unnamed

shadow DOM에서 `name`이 존재하지 않는 첫번째 `<slot>`은 **default slot**이 된다. 여기에는 light DOM에서부터 slot 처리가 되지 않은 모든 요소들이 추가된다.

예를 들어, 아래처럼 `<user-card>`에 default slot을 추가해보면, 별도로 slot을 지정해주지 않은 모든 요소들을 자동으로 default slot에 추가시킨다.

```html
<script>
customElements.define('user-card', class extends HTMLElement {
  connectedCallback() {
    this.attachShadow({mode: 'open'});
    this.shadowRoot.innerHTML = `
    <div>Name:
      <slot name="username"></slot>
    </div>
    <div>Birthday:
      <slot name="birthday"></slot>
    </div>
    <fieldset>
      <legend>Other information</legend>
      <slot></slot>
    </fieldset>
    `;
  }
});
</script>

<user-card>
  <div>I like to swim.</div>
  <span slot="username">John Smith</span>
  <span slot="birthday">01.01.2001</span>
  <div>...And play volleyball too!</div>
</user-card>
```

이 역시 기존의 slot과 마찬가지로 갱신을 하는 것이 아니라, 추가하는 방식으로 동작한다. 따라서 그 결과인 flatten DOM은 아래와 같아진다.

```HTML
<user-card>
  #shadow-root
    <div>Name:
      <slot name="username">
        <span slot="username">John Smith</span>
      </slot>
    </div>
    <div>Birthday:
      <slot name="birthday">
        <span slot="birthday">01.01.2001</span>
      </slot>
    </div>
    <fieldset>
      <legend>About me</legend>
      <slot>
        <div>Hello</div>
        <div>I am John!</div>
      </slot>
    </fieldset>
</user-card>
```

## Updating slots

만약 외부의 코드를 통해 slot에 들어간 item들을 동적으로 추가/삭제하고 싶다면 어떻게 하면 좋을까?

기본적으로, **브라우저가 slot들을 모니터링하며, 이에 따라 slot 처리된 요소들을 알아서 추가/삭제하여 렌더링해준다.**

또한, light DOM 노드들은 복제된 것이 아니라, 단순히 slot 안에 렌더링된 것이다. 때문에 변화가 즉시 가시적으로 반영된다.

따라서, 우리는 **렌더링 업데이트에 대해 신경 쓸 필요 없다.** 단, 만약 slot이 업데이트되는 **특정 시점에 대해 이벤트를 적용하고 싶다면 `slotchange` 이벤트를 활용하면 된다.**

`slotchange` 이벤트는, 최초에 1) 초기화 할 때 발생하고, 이후 2) slot에 변경이 생길 때마다 발생한다.

보다 상세한 처리가 요구되는 경우, **MutationObserver**를 사용할 수도 있다.

## Slot API
마지막으로, slot과 관련된 JS 메서드들을 살펴보자.

앞서 말했듯, JS는 오직 실제 DOM만을 바라본다. flatten DOM에 대해선 신경쓰지 않는다.

하지만, **만약 shadow tree가 `${mode: 'open'}` 옵션을 갖고 있다면, 요소로부터 slot을 유추할 수 있고, 반대의 경우도 가능하다.**

- `node.assignedSlot` : 노드가 지정된 `<slot>` 요소를 반환한다.
- `slot.assignedNodes({flatten: true/false})` : slot에 지정된 DOM 노드를 가져온다. 기본적으로 `flatten` 옵션은 `false`인데, 만약 `true`라면 flatten DOM을 통해 처리가 끝난 형태를 반환한다. 지정된 노드가 없다면 fallback을 반환한다.
- `slot.assignedElements({flatten: true/false})` : slot에 지정된 DOM 요소들을 반환한다. 위와 동일하지만, **요소만** 반환한다는 차이가 있다.

위와 같은 메서드들은 단순히 브라우저를 통해 출력하는 것 외에 slot 컨텐츠들을 추적해야할 필요가 있는 경우에 유용하다.

예를 들어, 아래에서 `<custom-menu>`는 `slotchange` 이벤트 핸들러를 통해 slot에 무엇이 보여지고 있는지를 추적하고 있다.

```html
<custom-menu id="menu">
  <span slot="title">Candy menu</span>
  <li slot="item">Lollipop</li>
  <li slot="item">Fruit Toast</li>
</custom-menu>

<script>
customElements.define('custom-menu', class extends HTMLElement {
  items = []

  connectedCallback() {
    this.attachShadow({mode: 'open'});
    this.shadowRoot.innerHTML = `<div class="menu">
      <slot name="title"></slot>
      <ul><slot name="item"></slot></ul>
    </div>`;

    // slottable is added/removed/replaced
    this.shadowRoot.firstElementChild.addEventListener('slotchange', e => {
      let slot = e.target;
      if (slot.name == 'item') {
        this.items = slot.assignedElements().map(elem => elem.textContent);
        alert("Items: " + this.items);
      }
    });
  }
});

// items update after 1 second
setTimeout(() => {
  menu.insertAdjacentHTML('beforeEnd', '<li slot="item">Cup Cake</li>')
}, 1000);
</script>
```