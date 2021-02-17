# Mutation Observer

`MutationObserver`는 DOM 요소를 감시하다가 변화를 감지하면 콜백을 호출하는 내장 객체이다.


## 문법

`MutationObserver`를 사용하는 것은 간단하다.

먼저, 콜백함수를 인자로 넘기는 옵저버를 만든다.

```js
let observer = new MutationObserver(callback);
```

그리고, 이를 DOM 노드에 덧붙인다.

```JS
observer.observe(node, config);
```

`config`은 boolean 옵션들을 갖고 있는 객체인데, 이는 **어떤 종류의 변화에 반응할 것인가**를 나타낸다.

- `childList` - `node` 본인의 바로 아래 자식 요소에서의 변화
- `subtree` - `node` 본인의 모든 자손
- `attributes` - `node`의 속성
- `attributeFilter` - 속성 이름들이 담긴 배열을 받는다. 오직 여기에 포함된 속성들만 감시한다.
- `characterData` - `node.data`(text content)를 감시할지에 대한 boolean

그 밖에 다른 옵션들도 있다.

- `attributeOldValue` - 만약 `true`라면, 콜백 함수 호출 시 '변경 전'과 '변경 후'의 값을 모두 넘겨준다. `false`라면 변경 후의 값만 넘긴다. (`attribute` 옵션이 필요하다.)
- `characterDataOldValue` - 만약 `true`라면, `node.data`의 '변경 전'과 '변경 후'의 값을 모두 넘겨준다. `false`라면 변경 후의 값만 넘겨준다. (`characterData` 옵션이 필요하다.)

이후 어떤 변화라도 감지된다면, `callback`이 실행된다. 변경된 내용은 `MutationRecord` 객체의 배열로 첫번째 인자로 넘겨진다. 그리고 옵저버 자체는 두번째 인자가 된다.

`MutaitonRecord` 객체는 다음과 같은 프로퍼티들을 갖는다.

- `type` - 뮤테이션 타입이다. 다음 중 하나다.
  - `attributes`: 수정된 속성
  - `characterData`: 수정된 `node.data`, 텍스트 노드로 쓰인다.
  - `childList`: 추가/삭제된 자식 요소들
- `target` - 변화가 감지된 곳의 요소
- `addedNodes/removedNodes` - 추가/삭제된 노드들
- `previousSibling/nextSibling` - 추가/삭제된 노드들의 이전/다음 형제 노드
- `attributeName/attributeNamespace` - 변경된 속성의 이름/네임스페이스(XML에서 사용)
- `oldValue` - 속성이나 텍스트가 변경되기 전의 값. `attributeOldValue`/`characterDataOldValue`이 `true`여야한다.

다음은 간단한 예시다.

```html
<div contentEditable id="elem">Click and <b>edit</b>, please</div>

<script>
let observer = new MutationObserver(mutationRecords => {
  console.log(mutationRecords); // console.log(the changes)
});

// observe everything except attributes
observer.observe(elem, {
  childList: true, // observe direct children
  subtree: true, // and lower descendants too
  characterDataOldValue: true // pass old data to callback
});
</script>
```

그리고 위에서 변화를 감지할 때마다 콜백함수에서 넘겨받는 `mutationRecords`는 아래와 같다.

```js
[{
  type: "characterData",
  oldValue: "edit",
  target: <text node>,
  // other properties empty
}];
```

만약, `<b>edit</b>`를 한번에 지우는 것과 같이 여러 작업이 동시에 일어나면, `mutationRecords`에도 여러 객체가 담긴다.

```
[{
  type: "childList",
  target: <div#elem>,
  removedNodes: [<b>],
  nextSibling: <text node>,
  previousSibling: <text node>
  // other properties empty
}, {
  type: "characterData"
  target: <text node>
  // ...mutation details depend on how the browser handles such removal
  // it may coalesce two adjacent text nodes "edit " and ", please" into one node
  // or it may leave them separate text nodes
}];
```

즉, `MutationObserver`는 DOM subtree에 발생하는 어떤 변화든지 대응할 수 있다.

## 활용 사례

그래서, 언제 이를  활용할 수 있을까?

만약, 서드파티 라이브러리를 사용하는데, 원치않는 광고가 포함되어 있다고 해보자. 이를테면 `<div class='ads'>...</ads>`와 같이.

`MutationObserver`를 사용하면, DOM에 생겨난 원치 않는 요소를 감지하여 제거할 수 있다.

그 밖에도, 여러가지를 감지하여 동적인 변화를 줄 수 있다. 이를 테면 어떤 요소의 사이즈를 변경한다던가.

## 아키텍쳐에 활용

`MutationObserver`가 구조적인 부분에서 유용하게 쓰이는 상황이 있다.

웹 프로그래밍과 관련한 웹사이트를 만들고자 한다고 하자. 각각의 문서들이 소스 코드 조각들을 담고 있을 것이다.

이때, 이 코드 조각들이 다음과 같은 모양을 띈다고 하자.

```js
...
<pre class="language-javascript"><code>
  // here's the code
  let hello = "world";
</code></pre>
...
```

이를 더 가독성이 좋게 하기 위해서, 꾸미고 싶다고 하자. 이 경우 우리는 **Prism.js**와 같은 문장 하이라이팅 라이브러리를 사용할 수 있다. 이는 `Prism.highlightElem(pre)`와 같은 식으로 특정 요소에 대해 하이라이트를 적용해준다.

그래서, 이제 이 메서드를 정확히 **언제** 사용해야 할까? `DOMContentLoaded` 이벤트 발생 시에 사용하는 것을 고려해볼 수 있겠다. 이후 각각의 코드 조각들에 대해 다음과 같이 하이라이팅을 적용시킬 수 있다.

```js
document.querySelectorAll('pre[class*="language"]').forEach(Prism.highlightElem);
```

지금까지는 수월해보인다. 근데, 만약에 서버를 통해 또 다른 코드 조각들을 가져와서 화면에 띄워주어야 한다면, 이는 어떻게 해결할 수 있을까?

```js
let article = /* fetch new content from server */
articleElem.innerHTML = article;

let snippets = articleElem.querySelectorAll('pre[class*="language-"]');
snippets.forEach(Prism.highlightElem);
```

위의 방법처럼, 해당 코드조각을 가져올 때 마다 다시 각각의 코드조각들에 대해 하이라이팅을 적용시켜줄 수 있다. 근데 이는 다소 비효율적인데, 코드 조각들이 있을만한 모든 요소들에 대해 위와 같은 코드를 추가해야 하기 때문이다.

결국, 이 역시 `MutationObserver`를 통해 페이지 내에 삽입되는 코드 조각들을 감지하여 처리할 수 있다.

```js
let observer = new MutationObserver(mutations => {

  for(let mutation of mutations) {
    // examine new nodes, is there anything to highlight?

    for(let node of mutation.addedNodes) {
      // we track only elements, skip other nodes (e.g. text nodes)
      if (!(node instanceof HTMLElement)) continue;

      // check the inserted element for being a code snippet
      if (node.matches('pre[class*="language-"]')) {
        Prism.highlightElement(node);
      }

      // or maybe there's a code snippet somewhere in its subtree?
      for(let elem of node.querySelectorAll('pre[class*="language-"]')) {
        Prism.highlightElement(elem);
      }
    }
  }

});

let demoElem = document.getElementById('highlight-demo');

observer.observe(demoElem, {childList: true, subtree: true});
```

## 추가적인 메서드

노드를 감시하는 것을 멈추는 메서드가 있다.

- `observer.disconnect()` - 감시를 멈춘다.

감시가 멈출 때, 해당 옵저버가 특정 작업을 처리하던 중이었을 수도 있다. 이런 경우에는 아래 메서드를 통해 확인할 수 있다.

- `observer.takeRecords()` - 처리되지 않은 `MutationRecord` 배열들을 가져온다. (변경은 감지했으나, 콜백이 호출되지 않은 경우를 말한다.)

이 메서드들은 함께 쓰일 수 있다.

```js
// get a list of unprocessed mutations
// should be called before disconnecting,
// if you care about possibly unhandled recent mutations
let mutationRecords = observer.takeRecords();

// stop tracking changes
observer.disconnect();
...
```