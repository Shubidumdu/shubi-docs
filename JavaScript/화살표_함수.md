# 화살표 함수 다시보기

화살표 함수(Arrow Function)에 대해 다시 생각해보자.

화살표 함수는 단순히 함수를 짧게 쓰기 위해 쓰지 않는다. 화살표 함수는 몇 가지 독특하고 유용한 기능을 제공한다.

## 화살표 함수에는 `this`가 없다.

화살표 함수 본문에서 `this`에 접근하면 외부에서 값을 가져온다.

이런 특징은 객체의 메서드(`showList()`)안에서 동일 객체의 프로퍼티(`students`)를 대상으로 순회를 하는 데 사용할 수 있다.

```js
let group = {
  title: '1모둠',
  students: ['보라', '호진', '지민'],

  showList() {
    this.students.forEach((student) => alert(this.title + ': ' + student));
  },
};

group.showList();
```

예시의 `forEach`에서 화살표 함수를 사용했기 때문에 화살표 함수 본문에 있는 `this.title`은 화살표 함수 바깥에 있는 메서드인 `showList`가 가리키는 대상과 동일해진다. 즉 `this.title`은 `group.title`과 같다.

위 예시에서 화살표 함수 대신 일반 함수를 사용했다면 에러가 발생했을 것이다.

```js
let group = {
  title: '1모둠',
  students: ['보라', '호진', '지민'],

  showList() {
    this.students.forEach(function (student) {
      // TypeError: Cannot read property 'title' of undefined
      alert(this.title + ': ' + student);
    });
  },
};

group.showList();
```

이는 `forEach`에 전달되는 함수의 `this`가 `undefined`이기 때문에 발생한다. `alert` 함수에서 `undefined.title`에 접근하려 했기 때문에 에러가 발생한다.

반면, 화살표 함수에는 `this`라는 개념 자체가 없기 때문에 이런 에러가 발생하지 않는다.

또한, `this`가 없기 때문에 `new`와 함께 사용할 수 없기도 하다.

## 화살표 함수에는 `arguments`가 없다.

화살표 함수는 일반 함수와 다르게 모든 인수에 접근할 수 있게 해주는 유사 배열(Array) 객체 `arguments`를 지원하지 않는다.

이런 특징은 현재 `this`값과 `arguments` 정보를 함께 실어 호출을 포워딩해주는 데코레이터를 만들 때 유용하게 사용된다.

```js
function defer(f, ms) {
  return function () {
    setTimeout(() => f.apply(this, arguments), ms);
  };
}

function sayHi(who) {
  alert('안녕, ' + who);
}

let sayHiDeferred = defer(sayHi, 2000);
sayHiDeferred('철수'); // 2초 후 "안녕, 철수"가 출력됩니다.
```

이것을 화살표 함수 없이 구현하려 했다면 아래와 같이 가독성이 떨어지는 형태가 된다.

```js
function defer(f, ms) {
  return function (...args) {
    let ctx = this;
    setTimeout(function () {
      return f.apply(ctx, args);
    }, ms);
  };
}
```
