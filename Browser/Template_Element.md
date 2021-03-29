# Template Element

내장 `<template>` 요소는 HTML 마크업 템플릿에 대한 저장요소로 취급된다. 브라우저는 이를 무시하고, 오로지 문법 상 온전한지만을 체크하는데, 우리가 우너한다면, 이를 이용해서 다른 요소들을 만들어낼 수도 있다.

사실, 우리는 이미 이론 상 보이지 않는 요소들을 어디서든 만들어 낼 수 있다. 대체 이 `<template>`가 특별한 점은 무엇일까?

**먼저**, 이들의 내용(content)는 문법상 올바른 HTML이면 무엇이든 가능하다. 다시 말해, 적절히 태그만 열고닫았으면 문제가 없다.

무슨 소리냐고? 단순히 우리가 `<tr>`과 `<td>`만을 이용해서 테이블을 만들면, 브라우저는 부적절한 DOM 구조를 감지하고, 알아서 `<table>`을 추가해 DOM 구조를 수정해준다. 반면 `<template>`의 경우는 우리가 작성한 내용 그대로를 유지시켜준다.

```js
<template>
  <tr>
    <td>Contents</td>
  </tr>
</template>
```

마찬가지로, `<template>`에는 스타일과 스크립트 태그가 포함될 수 있다.

```js
<template>
  <style>
    p { font-weight: bold; }
  </style>
  <script>
    alert("Hello");
  </script>
</template>
```

브라우저는 `<template>`의 내용들을 **문서와 상관없는 것**으로 간주한다. 때문에 스타일은 적용되지 않고, 스크립트 역시 마찬가지다.

## 템플릿 삽입하기

템플릿의 내용은 `content` 프로퍼티를 통해 이용할 수 있는데, 이는 **DocumentFragment**라는 특수한 DOM 노드 타입이다.

이는 다른 DOM 노드들과 거의 동일하게 다룰 수 있으나, 유일한 차이점은, 어딘가에 삽입되는 경우, 노드 본인이 아닌 자식들이 대신 삽입된다는 점이다.

```HTML
<template id="tmpl">
  <script>
    alert("Hello");
  </script>
  <div class="message">Hello, world!</div>
</template>

<script>
  let elem = document.createElement('div');

  // 재사용을 위해 템플릿 컨텐츠를 복사하여 사용한다.
  elem.append(tmpl.content.cloneNode(true));

  document.body.append(elem);
  // append에 의해 body에 요소가 추가되고 나서야 위의 script가 동작한다.
</script>
```

Shadow DOM과 함께 사용해보자.

```HTML
<template id="tmpl">
  <style> p { font-weight: bold; } </style>
  <p id="message"></p>
</template>

<div id="elem">Click me</div>

<script>
  elem.onclick = function() {
    elem.attachShadow({mode: 'open'});

    elem.shadowRoot.append(tmpl.content.cloneNode(true)); // (*)

    elem.shadowRoot.getElementById('message').innerHTML = "Hello from the shadows!";
  };
</script>
```
