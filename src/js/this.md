# this와 메서드

## what is `this`?

JS에서 `this`는 이따금씩 개발자를 헷갈리게 만드는 존재다. 이는 쓰이는 상황에 따라 각각 다른 것을 가리키게 되는데, 일반적으로는 다음과 같은 상황들이 있다.

## 1. 메서드에서

메서드 내부에서 사용하는 `this`는 해당 메서드가 선언된 객체를 가리킨다.

```js
let user = {
  name: 'John',
  age: 30,

  sayHi() {
    alert(this.name);
  },
};

user.sayHi(); // John
```

## 2. 일반 함수에서

다른 언어와 달리 JS는 모든 함수에서 `this`를 사용할 수 있는데, 이 경우는 런타임 시점에 `this`가 가리키는 것이 결정된다. 즉, **컨텍스트**에 따라 달라진다.

```js
// 같은 함수라도 다른 객체에서 호출한다면 `this`가 달라진다.

let user = { name: 'John' };
let admin = { name: 'Admin' };

function sayHi() {
  alert(this.name);
}

// 별개의 객체에서 동일한 함수를 사용함
user.f = sayHi;
admin.f = sayHi;

// 'this'는 '점(.) 앞의' 객체를 참조하기 때문에
// this 값이 달라짐
user.f(); // John  (this == user)
admin.f(); // Admin  (this == admin)

admin['f'](); // Admin (점과 대괄호는 동일하게 동작함)
```

## 3. 화살표 함수에서

화살표 함수는 일반 함수와 달리 고유한 `this`를 가지지 않는다. 화살표에서 `this`를 사용하면 외부 컨텍스트를 통해 `this`를 가져온다. 때문에 별도로 `this`를 만들기는 원치 않은 반면, 외부 컨텍스트의 `this`를 이용하고자 하는 경우는 화살표 함수를 이용하면 된다.

```js
let user = {
  firstName: '보라',
  sayHi() {
    let arrow = () => alert(this.firstName);
    arrow();
  },
};

user.sayHi(); // 보라
```
