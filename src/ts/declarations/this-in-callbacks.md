# 콜백에서 this에 대한 타입 제공하기

JS에서의 `this` 키워드는 매우 혼란스러운 기능입니다.
`let`이나 `const`로 선언된 변수가 렉시컬 스코프(Lexical Scope)인 반면, `this`는 다이나믹 스코프(Dynamic Scope)입니다.
이는 정의된 방식이 아니라, 호출된 방식에 따라 가리키는 값이 달라집니다.

`this`는 전형적으로 객체의 현재 인스턴스를 참조하는 클래스에서 가장 많이 쓰입니다.

```js
class C {
  vals = [1, 2, 3];
  logSquares() {
    for (const val of this.vals) {
      console.log(val * val);
    }
  }
}

const c = new C();
c.logSquares();

// 1
// 4
// 9
```

위 상황에서 `logSquares`에서 사용된 `this`는 변수 `c`를 가리키게 됩니다.
한편, 이것을 외부 변수에 넣고 호출하면 어떻게 되는지 살펴봅시다.

```js
const c = new C();
const method = c.logSquares; // losing this
method(); // ERROR
```

이러한 에러가 발생하는 이유는, 사실 `c.logSquares`의 호출이 실제로는 두 가지 작업을 수행하기 떄문입니다.

- `this`의 값을 바인딩합니다.
- `C.prototype.logSquares`를 호출합니다.

이를 JS 상에서 온전히 제어하기 위해서는 명시적으로 `this`를 바인딩해주어야 하는데, 이를 위해 `call`, `apply`, `bind`와 같은 메서드들이 존재합니다.

이러한 `this` 바인딩은 종종 콜백함수에서 쓰입니다. React의 예시를 봅시다.
다음 예시에서 바인딩을 하지 않는다면 `render` 메서드 실행 시 `this`가 `undefined`가 되는 문제가 생깁니다.

```ts
class ResetButton {
  constructor() {
    this.onClick = this.onClick.bind(this);
  }
  render() {
    return makeButton({text: 'Reset', onClick: this.onClick});
  }
  onClick() {
    alert(`Reset ${this}`);
  
}
```

화살표 함수를 사용하면 더 쉽게 이를 해결할 수 있습니다.
화살표 함수로 메서드를 변경하면 해당 클래스의 인스턴스의 생성할 때마다 제대로 바인딩된 `this`를 가진 새 함수를 생성합니다.

```ts
class ResetButton {
  render() {
    return makeButton({text: 'Reset', onClick: this.onClick});
  }
  onClick = () => {
    alert(`Reset ${this}`);  // "this"가 항상 인스턴스를 참조합니다.
  }
}
```

이는 실제로는 다음과 같이 동작합니다.

```ts
class ResetButton {
  constructor() {
    var _this = this;
    this.onClick = function () { // 인스턴스의 생성 시점에 `onClick` 함수를 선언합니다.
      alert("Reset " + _this);
    };
  }
  render() {
    return makeButton({text: 'Reset', onClick: this.onClick});
  }
}
```

## TS에서 `this`가 사용되는 콜백 함수를 다루기

콜백 함수 상에서 `this`가 사용된다면 그 자체가 API의 일부가 되는 것이기 때문에 반드시 타입 선언에 포함되어야 합니다.

```ts
// 콜백함수인 `fn` 내부에서 `this`를 사용한다고 가정합시다.
function addKeyListener(
  el: HTMLElement,
  fn: (this: HTMLElement, e: KeyboardEvent) => void
) {
  el.addEventListener('keydown', e => {
    fn.call(el, e);
  });
}
```

이 때 해당 콜백 함수 타입의 첫 번째 매개변수에 있는 `this`는 실제론 사용 시점에는 매개변수로 여겨지지 않으며, 특별하게 처리됩니다.
만약 해당 콜백 함수를 `this` 바인딩 없이 그냥 실행하려고 하는 경우에 에러를 출력하게끔 하여, 바인딩을 강제하도록 합니다.

```ts
function addKeyListener(
  el: HTMLElement,
  fn: (this: HTMLElement, e: KeyboardEvent) => void
) {
  el.addEventListener('keydown', e => {
    fn(e); // The 'this' context of type 'void' is not assignable to method's 'this' of type 'HTMLElement'.(2684)
    fn(el, e); // Expected 1 arguments, but got 2.(2554)
    fn.call(el, e); // OK.
  });
}
```

또, `this`를 사용하는 해당 콜백함수를 작성하는 시점에 `this`에 대한 타입이 명확하게 추론되기 때문에 타입 안정성을 확보할 수 있습니다.

```ts
const el = document.getElementById('div')!;
addKeyListener(el, function(e) {
  // 앞선 타입 선언에 덕분에 알아서 `this`에 대한 타입을 추론합니다.
  this.innerHTML; // this는 HTMLElement 입니다.
});
```

만약 화살표 함수를 사용하여 `this`를 참조하려고 하는 경우, 해당 `this`는 다른 것을 가리킬 것이기 때문에 적절히 에러를 출력해냅니다.

```ts
class Foo {
  registerHandler(el: HTMLElement) {
    addKeyListener(el, e => {
      // 여기서의 `this`는 Foo의 인스턴스가 됩니다.
      this.innerHTML;
        // ~~~~~~~~~ Property 'innerHTML' does not exist on type 'Foo'
    });
  }
}
```
