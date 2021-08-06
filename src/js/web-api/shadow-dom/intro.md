# Shadow DOM

섀도우 DOM은 캡슐화를 위해 제공된다. 이는 하나의 컴포넌트가 스스로의 **shadow DOM tree**를 가질 수 있게 하며, 메인 문서에서 접근 할 수 없으며, 로컬 스타일 규칙을 보유할 수 있다.

## 빌트인 섀도우 DOM

`<input type="range">`과 같은 것들은 브라우저마다 다른 자체적인 스타일을 보유한다. 이는 섀도우 DOM을 통해 브라우저 자체적으로 스타일링을 하고 있기 때문인데, 기본적으로 이는 이용자들로부터 숨겨져있다. 이를 보고자 한다면 개발자 도구의 옵션을 건드려야 한다.

## 섀도우 트리 (Shadow Tree)

하나의 DOM 요소에는 두가지 유형의 DOM 서브트리가 존재한다.

1. Light Tree - 일반적인 DOM 서브트리. 우리가 기존에 알고 쓰던 모든 서브트리는 이에 해당한다.
2. Shadow Tree - 숨겨진 DOM 서브트리. HTML 상에 보여지지 않는다.

만약, 두 가지 유형의 서브트리를 모두 갖는 요소가 있다면, 브라우저는 오직 섀도우 트리만을 렌더링한다. 물론 각각을 적절히 조합하도록 설정할 수도 있는데, 이에 대해선 추후에 설명한다.

섀도우 트리는 커스텀 요소 내에서 **컴포넌트 내부 요소들을 숨기고**, 컴포넌트 자체적인 **로컬 스타일링**을 위해 사용된다.

예를 들어, 아래와 같이 `<show-hello>`라는 커스텀 요소를 만들어낼 수 있다.

```html
<script>
  customElements.define(
    'show-hello',
    class extends HTMLElement {
      connectedCallback() {
        const shadow = this.attachShadow({ mode: 'open' });
        shadow.innerHTML = `<p>
      Hello, ${this.getAttribute('name')}
    </p>`;
      }
    },
  );
</script>

<show-hello name="John"></show-hello>
```

커스텀 요소를 만들기 위해서는 먼저 `elem.attachShadow({mode: ...})`를 호출해야 한다. 여기엔 두 가지 제한이 있다.

1. 각 요소 당 하나의 섀도우 루트(shadow-root)만 가질 수 있다.
2. `elem`은 반드시 커스텀 요소이거나, 다음 중 하나여야 한다. (“article”, “aside”, “blockquote”, “body”, “div”, “footer”, “h1…h6”, “header”, “main” “nav”, “p”, “section”, “span”)

`mode`옵션은 캡슐화 레벨을 설정한다. 다음의 둘 중 하나여야 한다.

- `open` : 어디서든 해당 요소의 섀도우 루트에 `elem.shadowRoot`로 접근할 수 있다.
- `closed` : `elem.shadowRoot`가 항상 `null`이 된다.

대부분의 브라우저 자체적인 섀도우 트리들은 `closed` 상태이며, 때문에 이들의 섀도우 트리에 접근할 방법이 없다.

`attachShadow`를 통해 반환되는 **섀도우 루트(shadow root)**는 요소와 같다. `innerHTML`이나 `append` 같은 DOM 프로퍼티 및 메서드를 사용할 수 있다.

- 섀도우 루트가 있는 요소는 **섀도우 트리 호스트(shadow tree host)**라고 불리며, 이는 섀도우 루트의 `host` 프로퍼티는 통해 접근할 수 있다.

```js
// assuming {mode: "open"}, otherwise elem.shadowRoot is null
alert(elem.shadowRoot.host === elem); // true
```

## 캡슐화 (Encapsulation)

섀도우 DOM은 메인 문서(document)으로부터 완전히 구분된다.

1. light DOM에서의 `querySelector`에 의해 탐색되지 않는다. 때문에 light DOM에 동일한 id가 존재하더라도 섀도우 트리 내에서만 고유하다면 상관없다.

2. 섀도우 DOM은 스스로의 스타일시트를 보유한다. 그 외의 스타일 규칙은 이에 적용되지 않는다.

이에 따라, 아래 예시를 보자.

```html
<style>
  /* document style won't apply to the shadow tree inside #elem (1) */
  p {
    color: red;
  }
</style>

<div id="elem"></div>

<script>
  elem.attachShadow({ mode: 'open' });
  // shadow tree has its own style (2)
  elem.shadowRoot.innerHTML = `
    <style> p { font-weight: bold; } </style>
    <p>Hello, John!</p>
  `;

  // <p> is only visible from queries inside the shadow tree (3)
  alert(document.querySelectorAll('p').length); // 0
  alert(elem.shadowRoot.querySelectorAll('p').length); // 1
</script>
```

1. 문서 자체에서 적용한 `style`은 섀도우 트리에 아무 영향도 미치지 않는다.
2. 허나, `elem.shadowRoot.innerHTML`에서 직접 지정한 스타일링은 적용된다.
3. 섀도우 트리에서 요소를 가져오고자 하는 경우, 반드시 섀도우 트리 내에서 `querySelector`와 같은 메서드를 사용해야 한다.
