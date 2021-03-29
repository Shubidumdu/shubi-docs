# Shadow DOM styling

shadow DOM은 `<style>` 태그와 `<link rel='stylesheet' href='...'>` 태그를 모두 포함할 수 있다. 그 중 `<link>` 태그의 경우, **HTTP 캐싱**이 되며, 여러번 다운로드 되지 않는다.

일반적인 스타일 규칙으로는, shadow DOM은 오직 shadow tree 내의 로컬 스타일 규칙에만 영향을 받는다. 하지만 몇가지 예외가 존재한다.

## :host

`:host` 선택자는 shadow 호스트(shadow tree를 보유한 요소)를 선택하는 것을 허용한다.

예를 들어, `<custom-dialog>`요소가 가운데에 위치하길 원한다면, 아래와 같은 방법으로 스타일을 추가할 수 있다.

```html
<template id="tmpl">
  <style>
    /* the style will be applied from inside to the custom-dialog element */
    :host {
      position: fixed;
      left: 50%;
      top: 50%;
      transform: translate(-50%, -50%);
      display: inline-block;
      border: 1px solid red;
      padding: 10px;
    }
  </style>
  <slot></slot>
</template>

<script>
  customElements.define(
    'custom-dialog',
    class extends HTMLElement {
      connectedCallback() {
        this.attachShadow({ mode: 'open' }).append(
          tmpl.content.cloneNode(true),
        );
      }
    },
  );
</script>

<custom-dialog> Hello! </custom-dialog>
```

## Cascading

shadow 호스트(`<custom-dialog>` 태그 그 자체)는 light DOM에 위치한다. 따라서, 이는 문서 자체의 CSS 규칙에 영향을 받는다.

만약, shadow tree에 로컬로 `:host` 스타일이 존재함과 동시에, 문서 자체에도 스타일이 존재한다면, 문서의 스타일링이 더 우선시된다.

따라서, 위의 코드에서 아래와 같이 문서에 스타일링을 추가하는 경우

```html
<style>
  custom-dialog {
    padding: 0;
  }
</style>
```

`<custom-dialog>`는 더 이상 padding을 갖지 않는다.

이는 제법 편리한데, 이를 통해 `:host`에는 **기본(default) 컴포넌트 스타일**을 지정하고, 문서를 통해서 스타일링을 쉽게 덮어씌울 수 있기 때문이다.

예외는 로컬 스타일링에 `!important`를 적용하는 경우다.

## :host(selector)

`:host`와 동일하되, 주어진 선택자(`selector`)에 해당하는 경우에만 적용된다.

예를 들어, 앞선 `<custom-dialog>`에서, `center` 속성(attribute)를 보유한 경우에만 가운데 정렬을 하고싶다면, 아래와 같이 활용할 수 있다.

```html
<template id="tmpl">
  <style>
    :host([centered]) {
      position: fixed;
      left: 50%;
      top: 50%;
      transform: translate(-50%, -50%);
      border-color: blue;
    }

    :host {
      display: inline-block;
      border: 1px solid red;
      padding: 10px;
    }
  </style>
  <slot></slot>
</template>

<script>
  customElements.define(
    'custom-dialog',
    class extends HTMLElement {
      connectedCallback() {
        this.attachShadow({ mode: 'open' }).append(
          tmpl.content.cloneNode(true),
        );
      }
    },
  );
</script>

<custom-dialog centered> Centered! </custom-dialog>

<custom-dialog> Not centered. </custom-dialog>
```

## :host-context(selector)

`:host`와 동일하되, shadow 호스트 자신, 혹은 그 상위에 있는 요소 중 해당 `selector`에 해당하는 경우에만 적용된다.

예를 들어, `:host-context(.dark-theme)`를 사용한 아래 예시에서, `<custom-dialog>`에 `dark-theme` 클래스가 존재하는 경우에만 스타일링이 적용된다.

```html
<body class="dark-theme">
  <!--
    :host-context(.dark-theme) applies to custom-dialogs inside .dark-theme
  -->
  <custom-dialog>...</custom-dialog>
</body>
```

요약하자면, `:host` 종류들은 컴포넌트의 메인 요소들을 스타일링 하기 위해 활용할 수 있는 선택자이다. 이를 활용해 적용한 스타일들은 문서 자체에서의 스타일링에 덮어씌여질 수 있다.

## slotted content 스타일링

이제, `slot`을 사용하는 경우를 보자.

slot 처리 된 요소 자체는 light DOM에서 온다. 따라서, 그들 요소는 문서의 스타일링을 따르며, shadow tree 측에서의 로컬 스타일링은 여기에 영향을 미치지 않는다.

예를 들어보자. 아래에 slot으로 삽입된 `<span>`은 문서 스타일링에 따라 `bold` 폰트 굵기를 갖는다. 하지만 shadow Root에서의 스타일링에 영향 받지 않기 때문에 붉은 바탕(`background: red`)이 아니다.

```html
<style>
  span {
    font-weight: bold;
  }
</style>

<user-card>
  <div slot="username"><span>John Smith</span></div>
</user-card>

<script>
  customElements.define(
    'user-card',
    class extends HTMLElement {
      connectedCallback() {
        this.attachShadow({ mode: 'open' });
        this.shadowRoot.innerHTML = `
      <style>
      span { background: red; }
      </style>
      Name: <slot name="username"></slot>
    `;
      }
    },
  );
</script>
```

만약, slot 처리된 요소들에 대해 컴포넌트 안에서 스타일링하고 싶다면, 두가지 선택지가 있다.

**첫번째는,** 컴포넌트 내에서 CSS 상속에 기반해 `<slot>` 그 자체를 스타일링하는 것이다.

```HTML
<user-card>
  <div slot="username"><span>John Smith</span></div>
</user-card>

<script>
customElements.define('user-card', class extends HTMLElement {
  connectedCallback() {
    this.attachShadow({mode: 'open'});
    this.shadowRoot.innerHTML = `
      <style>
      slot[name="username"] { font-weight: bold; }
      </style>
      Name: <slot name="username"></slot>
    `;
  }
});
</script>
```

이제 `<p>John Smith</p>`는 bold 굵기가 된다. 왜냐하면 CSS 상속에 의해 하위 요소들에게도 영향을 미치기 때문이다. 단, CSS 자체의 속성들이 상속되지는 않는다.

**두번째 선택지는** 바로 `::slotted(selector)` 의사 클래스를 사용하는 것이다. 이 때는 두가지 조건에 따라 해당하는 요소를 구분한다.

1. light DOM를 통해서 전달된 slot 처리된 요소(`slot='...'`를 포함)여야 한다. `name` 자체는 중요하지 않다. 단, 오직 그 요소 자체에만 해당하며, 하위 요소들은 해당하지 않는다.
2. 요소가 `selector` 선택자에 해당해야 한다.

예를 들어, `::slotted(div)`는 정확히 `<div slot='username'>`에만 적용되며, 하위 요소들에는 적용되지 않는다.

```html
<user-card>
  <div slot="username">
    <div>John Smith</div>
  </div>
</user-card>

<script>
  customElements.define(
    'user-card',
    class extends HTMLElement {
      connectedCallback() {
        this.attachShadow({ mode: 'open' });
        this.shadowRoot.innerHTML = `
      <style>
      ::slotted(div) { border: 1px solid red; }
      </style>
      Name: <slot name="username"></slot>
    `;
      }
    },
  );
</script>
```

기억하자. `::slotted` 선택자는 하위 요소들을 확인하지 않는다.

```css
::slotted(div span) {
  /* our slotted <div> does not match this */
}

::slotted(div) p {
  /* can't go inside light DOM */
}
```

## 커스텀 프로퍼티를 이용한 CSS Hook

메인 문서를 통해 shadow DOM 컴포넌트 내부의 요소들을 스타일링하려면, 어떻게 해야할까?

`:host` 선택자는 `<custom-dialog>` 자체에 대해서 스타일링을 적용할 수 있다. 그런데, 그것보다 깊숙히 위치한 요소들에 스타일링을 적용하고 싶다면 어떻게 할까?

사실, 문서에서 shadow DOM의 스타일에 직접 영향을 줄 수 있는 선택자는 없다. 그러나, 원한다면, CSS 변수(custom CSS properties)를 활용해 이를 구현할 수 있다.

왜냐하면, **커스텀 CSS 프로퍼티는 light와 shadow 모두에 존재하기 때문이다.(공유한다)**

예를 들어, 먼저 아래처럼 `--user-card-field-color`라는 CSS 변수를 사용해 `.field`를 기본 스타일링할 수 있다.

```html
<style>
  .field {
    color: var(--user-card-field-color, black);
    /* if --user-card-field-color is not defined, use black color */
  }
</style>
<div class="field">Name: <slot name="username"></slot></div>
<div class="field">Birthday: <slot name="birthday"></slot></div>
```

이후, 문서에서 `<user-card>`에 대해 앞서 만든 property를 활용하여 스타일링을 변경할 수 있다.

```css
user-card {
  --user-card-field-color: green;
}
```

커스텀 CSS 프로퍼티는 shadow DOM 전반에 유효하기 때문에, 어디서든 활용할 수 있다.

```HTML
<style>
  user-card {
    --user-card-field-color: green;
  }
</style>

<template id="tmpl">
  <style>
    .field {
      color: var(--user-card-field-color, black);
    }
  </style>
  <div class="field">Name: <slot name="username"></slot></div>
  <div class="field">Birthday: <slot name="birthday"></slot></div>
</template>

<script>
customElements.define('user-card', class extends HTMLElement {
  connectedCallback() {
    this.attachShadow({mode: 'open'});
    this.shadowRoot.append(document.getElementById('tmpl').content.cloneNode(true));
  }
});
</script>

<user-card>
  <span slot="username">John Smith</span>
  <span slot="birthday">01.01.2001</span>
</user-card>
```
