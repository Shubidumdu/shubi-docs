# Class

## 클래스와 기본 문법

클래스는 다음과 같은 문법을 통해 만들 수 있다.

```js
class MyClass {
  constructor() {}
  method1() {}
  method2() {}
  ...
}
```

이렇게 클래스를 만들고, `new MyClass()`를 호출하면 내부에서 정의한 메서드가 들어 있는 객체가 생성된다.

객체의 기본 상태를 설정해주는 생성자 메서드 `constructor()`는 `new`에 의해 자동으로 호출되므로, 별다른 절차 없이 객체를 초기화할 수 있다.

```js
class User {
  constructor(name) {
    this.name = name;
  }

  sayHi() {
    alert(this.name);
  }
}

// 사용법:
let user = new User('John');
user.sayHi();
```

위에서 `new User("John")`을 호출하면 다음과 같은 일이 일어난다.

1. 새로운 객체가 생성된다.
2. 넘겨받은 인수와 함께 `constructor`가 자동으로 실행된다. 이 때, 인수 `"John"`이 `this.name`에 할당된다.

이런 과정을 거친 후에 `user.sayHi()` 같은 객체 메서드를 호출할 수 있다.

### 그래서 클래스란??

`class User {...}` 문법 구조가 진짜 하는 일은 다음과 같다.

1. `User`라는 이름을 가진 함수를 만든다. 함수 본문은 생성자 메서드 `constructor`에서 가져온다. 생성자 메서드가 없으면 비워진 채로 함수가 만들어진다.
2. `sayHi`같은 클래스 내에서 정의한 메서드를 `User.prototype`에 저장한다.

`new User`를 호출해 객체를 만들고, 이후 객체의 메서드가 호출되면 메서드를 프로토타입에서 가져온다. 이 덕분에 객체에서 클래스 메서드에 접근할 수 있다.

```js
class User {
  constructor(name) {
    this.name = name;
  }
  sayHi() {
    alert(this.name);
  }
}

// 클래스는 함수입니다.
alert(typeof User); // function

// 정확히는 생성자 메서드와 동일합니다.
alert(User === User.prototype.constructor); // true

// 클래스 내부에서 정의한 메서드는 User.prototype에 저장됩니다.
alert(User.prototype.sayHi); // alert(this.name);

// 현재 프로토타입에는 메서드가 두 개입니다.
alert(Object.getOwnPropertyNames(User.prototype)); // constructor, sayHi
```

### 클래스는 **단순 Syntactic Sugar가 아니다.**

이건 나도 잘못 알고있었던 내용이다. 아래처럼 `prototype`을 이용하면 순수 함수로도 클래스 역할을 하는 함수를 만들 수 있다. 때문에 `class`를 단순한 Syntactic Sugar로 여기고 있었다.

```js
// class User와 동일한 기능을 하는 순수 함수를 만들어보겠습니다.

// 1. 생성자 함수를 만듭니다.
function User(name) {
  this.name = name;
}
// 모든 함수의 프로토타입은 'constructor' 프로퍼티를 기본으로 갖고 있기 때문에
// constructor 프로퍼티를 명시적으로 만들 필요가 없습니다.

// 2. prototype에 메서드를 추가합니다.
User.prototype.sayHi = function () {
  alert(this.name);
};

// 사용법:
let user = new User('John');
user.sayHi();
```

근데 둘 사이에는 중요한 차이가 몇 가지 있다.

1. `class`로 만든 함수엔 특수 내부 프로퍼티인 `[[FunctionKind]]: "classConstructor"`가 이름표처럼 붙는다. 이것만으로도 두 방법엔 차이가 있다. 이런 검증 과정 때문에, **클래스 생성자는 `new`와 함께 호출하지 않으면 에러가 발생한다.**

더불어, 대부분의 JS엔진에서 클래스 생성자를 문자열로 표현할 때 `class ...`로 시작하게 된다는 차이도 생긴다.

```js
class User {
  constructor() {}
}

alert(typeof User); // function
User(); // TypeError: Class constructor User cannot be invoked without 'new'
```

```js
class User {
  constructor() {}
}

alert(User); // class User { ... }
```

2. 클래스 메서드는 열거할 수 없다.(non-enumerable) 클래스의 `prototype` 프로퍼티에 추가된 메서드 전체의 `enumerable` 플래그는 `false`이고, `for..in`으로 객체를 순회할 때, 이는 순회 대상에서 제외된다. 보통 메서드는 순회 대상에서 제외하고자 하므로, 이 특징은 제법 유용하다.

3. 클래스는 항상 엄격모드로 실행된다.(`use strict`) 클래스 생성자 안 코드 전체엔 자동으로 엄격모드가 적용된다.

### 클래스 표현식

함수처럼 클래스도 다른 표현식 내부에서 정의, 전달, 반환, 할당할 수 있다.
먼저 클래스 표현식을 만들어보자.

```js
let User = class {
  sayHi() {
    alert('Hello');
  }
};
```

기명 함수 표현식(Named Function Expression)과 유사하게 클래스 표현식에도 이름을 붙일 수 있다.
클래스 표현식에 이름을 붙이면, 이 이름은 오직 클래스 내부에서만 사용할 수 있다.

```js
// 기명 클래스 표현식(Named Class Expression)
// (명세서엔 없는 용어이지만, 기명 함수 표현식과 유사하게 동작합니다.)
let User = class MyClass {
  sayHi() {
    alert(MyClass); // MyClass라는 이름은 오직 클래스 안에서만 사용할 수 있습니다.
  }
};

new User().sayHi(); // 제대로 동작합니다(MyClass의 정의를 보여줌).

alert(MyClass); // ReferenceError: MyClass is not defined, MyClass는 클래스 밖에서 사용할 수 없습니다.
```

필요에 따라 동적인 생성 역시 가능하다.

```js
function makeClass(phrase) {
  // 클래스를 선언하고 이를 반환함
  return class {
    sayHi() {
      alert(phrase);
    }
  };
}

// 새로운 클래스를 만듦
let User = makeClass('Hello');

new User().sayHi(); // Hello
```

### getter와 setter

리터럴을 사용해 만든 객체처럼 클래스도 getter나 setter, 계산된 프로퍼티(computed property)를 포함할 수 있다.

```js
class User {
  constructor(name) {
    // setter를 활성화합니다.
    this.name = name;
  }

  get name() {
    return this._name;
  }

  set name(value) {
    if (value.length < 4) {
      alert('이름이 너무 짧습니다.');
      return;
    }
    this._name = value;
  }
}

let user = new User('John');
alert(user.name); // John

user = new User(''); // 이름이 너무 짧습니다.
```

이런 방법으로 클래스를 선언하면 `User.prototype`에 getter와 setter가 만들어지는 것과 동일하다.

### 계산된 메서드명 [...]

대괄호 `[...]`을 이용해 계산된 메서드 이름(computed method name)을 만드는 예시도 있다.

```js
class User {
  ['say' + 'Hi']() {
    alert('Hello');
  }
}

new User().sayHi();
```

### 클래스 필드

이는 비교적 최근에 생겨난 기능으로, 구식 브라우저에서는 폴리필이 필요할 수도 있다.

지금까지 살펴본 예시엔 메서드가 하나만 있었다.

'클래스 필드(Class Field)'라는 문법을 사용하면 어떤 종류의 프로퍼티도 클래스에 추가할 수 있다.

클래스 `User`에 `name` 프로퍼티를 추가해보자.

```js
class User {
  name = 'John';

  sayHi() {
    alert(`Hello, ${this.name}!`);
  }
}

new User().sayHi(); // Hello, John!
```

클래스를 정의할 때 `<프로퍼티명> = <값>`을 써주면 간단히 클래스 필드를 만들 수 있다.]

클래스 필드의 중요한 특징 중 하나는 `User.prototype`이 아닌 개별 객체에**만** 클래스 필드가 설정된다는 점이다.

```js
class User {
  name = 'John';
}

let user = new User();
alert(user.name); // John
alert(User.prototype.name); // undefined
```

클래스 필드는 생성자가 그 역할을 다 한 이후에 처리된다. 따라서 복잡한 표현식이나 함수 호출 결과를 사용할 수 있다.

```js
class User {
  name = prompt('이름을 알려주세요.', '보라');
}

let user = new User();
alert(user.name); // 보라
```

### 클래스 필드로 바인딩 된 메서드 만들기

JS의 함수는 알다시피 동적인 `this`를 갖는다.

따라서 객체 메서드를 여기저기 전달해 다른 컨텍스트에서 호출하게 되면 `this`는 원래 객체를 참조하지 않는다.

관련 예시를 살펴보자. 예시를 실행하면 `undefined`가 출력된다.

```js
class Button {
  constructor(value) {
    this.value = value;
  }

  click() {
    alert(this.value);
  }
}

let button = new Button('hello');

setTimeout(button.click, 1000); // undefined
```

이렇게 `this`의 컨텍스트를 알수 없게 되어버리는 문제를 **Losing this**라고 한다.

문제를 해결하기 위해선 두 개의 방법이 있다.

1. `setTimeout(() => button.click(), 1000)`같이 래퍼 함수를 전달
2. 생성자 안 등에서 메서드를 객체에 바인딩

여기에 더해, 클래스 필드는 또 다른 훌륭한 방법을 제공한다.

```js
class Button {
  constructor(value) {
    this.value = value;
  }
  click = () => {
    alert(this.value);
  };
}

let button = new Button('hello');

setTimeout(button.click, 1000); // hello
```

클래스 필드 `click = () => {...}`는 각 `Button` 객체마다 독립적인 함수를 만들고 함수의 `this`를 해당 객체에 바인딩시켜준다. 따라서 개발자는 `button.click`과 같은 메서드를 아무 곳에나 전달할 수 있고, `this`에는 항상 의도한 값이 들어가게 된다.

대체로 클래스 필드의 이런 기능은 **브라우저 환경에서 메서드를 이벤트 리스너로 설정해야 할 때 특히 유용하다.**
