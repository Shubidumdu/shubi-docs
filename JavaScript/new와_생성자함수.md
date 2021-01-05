# new와 생성자 함수

## 1. 생성자 함수
생성자 함수 (Contructor function)와 일반 함수 사이의 기술적인 차이는 없다. 다만 생성자 함수는 아래의 두 관례를 따른다.

1) 함수 이름 첫 글자는 대문자
2) 반드시 `new` 연산자를 붙여 사용한다.

```js
function User(name) {
  this.name = name;
  this.isAdmin = false;
}

let user = new User("Jack");
```

위와 같은 관례를 따라 `new User(...)`를 써서 함수를 실행하면 아래와 같은 알고리즘이 동작한다.

1) 빈 객체를 만들어 `this`에 할당한다.
2) 함수 본문을 실행하고 `this`에 새로운 프로퍼티를 추가해 `this`를 수정한다.
3) `this`를 반환한다.

즉, 내부적으로는 아래와 같은 일이 동작하는 것이다.

```js
function User(name) {
  // this = {};  (빈 객체가 암시적으로 만들어짐)

  // 새로운 프로퍼티를 this에 추가함
  this.name = name;
  this.isAdmin = false;

  // return this;  (this가 암시적으로 반환됨)
}
```

결국 생성자의 의의는 **재사용할 수 있는** 객체 생성 코드를 구현하기 위한 것이다. 

모든 함수는 생성자 함수가 될 수 있다. `new`를 붙여 실행한다면 어떤 함수라도 위와 같은 일이 벌어진다. 첫 글자가 대문자인 함수는 `new`를 붙여 실행하는 것이 일종의 관례라는 점도 기억하자.


### new function() {...}
생성자를 이용해 함수에 **캡슐화**를 적용할 수도 있다.
```js
let user = new function() {
  this.name = "John";
  this.isAdmin = false;

  // ...
};
```

위의 생성자 함수는 익명함수이기 때문에, 어디에도 저장되지 않으며, 단 한번만 호출될 목적으로 만들어져 재사용이 불가능하다.

### ()이 없어도 된다.
다만 좋은 코드 스타일은 아니다.

```js
let user = new User; // <-- 괄호가 없음
// 아래 코드는 위 코드와 똑같이 동작합니다.
let user = new User();
```

## 2. `new Function()`

함수 표현식과 함수 선언문 이외에 함수를 만들 수 있는 방법이 하나 더 있다. 

이는 자주 사용하는 방법은 아니지만, 마땅한 대안이 없을 때 사용될 수 있다.

`new Function`을 이용은 다음과 같다.

```js
let func = new Function ([arg1, arg2, ...argN], functionBody);
```

`new Function`을 이용하는 방법의 가장 큰 차이는 런타임 시점에 받는 문자열을 사용해 함수를 만들 수 있다는 점이다.

```js
let sum = new Function('a', 'b', 'return a + b');

alert( sum(1, 2) ); // 3
```

### 클로저와의 미묘한 관계

클로저를 떠올려보자, 반환받은 중첩함수는 `[[Environment]]` 프로퍼티 덕분에 본인이 생성된 렉시컬 외부 환경을 기억할 수 있었다.

그런데, `new Function`을 이용해 함수를 만들게 되면 함수의 `[[Environment]]` 프로퍼티가 현재의 렉시컬 환경이 아닌 전역 렉시컬 환경을 참조하게 된다.

따라서, `new Function`을 통해 만든 함수는 외부 블록의 변수에 접근할 수 없고, 오직 전역 변수에만 접근할 수 있다.

```js
function getFunc() {
  let value = "test";

  let func = new Function('alert(value)');

  return func;
}

getFunc()(); // ReferenceError: value is not defined
```

이러한 특징은, 특정 함수 내부에서 이름이 겹치는 변수들을 사용해도 충돌을 하지 않는다는 이점이 있다.