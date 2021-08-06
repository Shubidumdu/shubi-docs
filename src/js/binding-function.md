# 함수 바인딩

`setTimeout`에 메서드를 전달할 때처럼, 객체 메서드를 콜백으로 전달할 때는 `this`가 사라지는 문제가 생긴다.

## 사라진 `this`

앞서 다양한 예제를 통해 `this` 정보가 사라지는 문제를 경험했다. 객체 메서드가 객체 내부가 아닌 다른 곳에 전달되어 호출되면 `this`가 사라진다.

대표적인 예는 `setTimeout`등에서 콜백함수로 넘겨지는 경우에 발생하는 것이다.

```js
let user = {
  firstName: 'John',
  sayHi() {
    alert(`Hello, ${this.firstName}!`);
  },
};

setTimeout(user.sayHi, 1000); // Hello, undefined!
```

이런 문제는 콜백함수를 전달 할 때, 객체에서 메서드가 분리(`user.sayHi`)되어 하나의 함수로써 전달되기 때문이다.

### 해결 1 : 래퍼(Wrapper) 함수

가장 간단한 해결책은 래퍼 함수를 사용하는 것이다.

```js
let user = {
  firstName: 'John',
  sayHi() {
    alert(`Hello, ${this.firstName}!`);
  },
};

setTimeout(function () {
  user.sayHi(); // Hello, John!
}, 1000);
```

위 예시가 의도대로 동작하는 이유는, 외부 렉시컬 환경에서 `user`를 받아 보통 때와 똑같이 메서드를 호출하기 때문이다.

단, 이 경우 약간의 취약성이 생기는데, `setTimeout`이 트리거 되기 전, `user`에 변경이 가해지면, 변경된 상태의 객체 메서드를 호출한다는 점이다.

```js
let user = {
  firstName: 'John',
  sayHi() {
    alert(`Hello, ${this.firstName}!`);
  },
};

setTimeout(() => user.sayHi(), 1000);

// 1초가 지나기 전에 user의 값이 바뀜
user = {
  sayHi() {
    alert('또 다른 사용자!');
  },
};

// setTimeout에 또 다른 사용자!
```

이런 문제는 두 번째 방법을 사용함으로써 방지할 수 있다.

### 방법 2 : bind

모든 함수는 `this`를 수정하게 해주는 내장 메서드 `bind`를 제공한다.

기본 문법은 다음과 같다.

```js
// 더 복잡한 문법은 뒤에 나옵니다.
let boundFunc = func.bind(context);
```

`func.bind(context)`는 함수처럼 호출 가능한 '특수 객체(exotic object)'를 반환한다. 이 객체를 호출하면 `this`가 `context`로 고정된 함수 `func`이 반환된다.

이제 `boundFunc`를 호출하면 `this`가 고정된 `func`를 호출하는 것과 동일한 효과를 본다.

아래 `funcUser`에는 `this`가 `user`로 고정된 `func`이 할당된다.

이제 객체 메서드에 `bind`를 적용해보자.

```js
let user = {
  firstName: 'John',
  sayHi() {
    alert(`Hello, ${this.firstName}!`);
  },
};

let sayHi = user.sayHi.bind(user); // (*)

// 이제 객체 없이도 객체 메서드를 호출할 수 있습니다.
sayHi(); // Hello, John!

setTimeout(sayHi, 1000); // Hello, John!

// 1초 이내에 user 값이 변화해도
// sayHi는 기존 값을 사용합니다.
user = {
  sayHi() {
    alert('또 다른 사용자!');
  },
};
```

### 부분 적용 (Partial Application)

지금껏 `this`에 대해서만 이야기했지만, `this`뿐만 아니라 인수에 대해서도 바인딩이 가능하다. 이는 자주 쓰이진 않지만 가끔 유용하다.

```bind`의 전체 문법은 다음과 같다.

```js
let bound = func.bind(context, [arg1], [arg2], ...);
```

`bind`는 컨텍스트를 `this`로 고정하는 것 뿐만 아니라 함수의 인수도 고정해준다.

곱셈을 해주는 함수 `mul(a, b)`를 예시로 들어보고, `bind`를 통해 새로운 함수 `double`을 만들어본다.

```js
function mul(a, b) {
  return a * b;
}

let double = mul.bind(null, 2);

alert(double(3)); // = mul(2, 3) = 6
alert(double(4)); // = mul(2, 4) = 8
alert(double(5)); // = mul(2, 5) = 10
```

이처럼, `context`는 따로 넘겨주지 않고 인수에 대해서만 값을 전달해 고정해주는 방식을 **부분 적용(Partial Application)**이라고 한다.

이는 매우 포괄적인 함수를 기반으로 덜 포괄적인 변형 함수를 만들어 낼 수 있다는 점에서 유용하다.
