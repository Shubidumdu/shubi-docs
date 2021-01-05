# `var`를 쓰지 않는 이유

## `var`에는 **블록 스코프**가 없다.

`var`로 선언한 변수의 스코프는 `function-scoped` 혹은 `global-scoped`다. 블록을 기준으로 스코프가 생기지 않기 때문에, 혼동을 일으키기 매우 좋다. 아래는 그 덕분에 작성가능한 괴상망측한 코드다.

```js
for (var i = 0; i < 10; i++) {
  // ...
}

alert(i);
```

코드 블럭이 함수 안에 있다면, `var`는 함수 스코프**만** 적용된 변수가 된다.

```js
function sayHi() {
  if (true) {
    var phrase = "Hello";
  }

  alert(phrase); // 제대로 출력됩니다.
}

sayHi();
alert(phrase); // Error
```

## `var`는 재선언이 되는 **척**을 한다

아래는 실행 상 아무 문제가 없다. 다만 중간의 `John`값으로 `user`를 다시 선언/할당한 내용은 무시된다.

## `var`는 선언도 하기 전에 사용할 수 있다. **(호이스팅: Hoisting)**

`var` 선언은 함수가 시작될 때 처리된다. 전역에서 선언한 변수라면 스크립트가 시작될 때 처리된다.

**때문에, `var`로 변수를 선언한다면, 그 위치랑 상관없이 함수 본문이 시작되는 지점에서 정의가 된다.**

이렇게 **`var`로 인하여 변수의 선언이 함수 최상위로 끌어올려지는** 현상을 호이스팅(Hoisting)이라고 한다.

아래 코드에서 `if`블럭 안의 코드는 절대 실행되지 않겠지만, 이는 호이스팅 자체에 전혀 영향을 주지 않기 때문에, 에러가 발생하지 않는다.

```js
function sayHi() {
  phrase = "Hello"; // (*)

  if (false) {
    var phrase;
  }

  alert(phrase);
}
sayHi();
```

다만, **선언**만 호이스팅되고 **할당**은 호이스팅 되지 않는다. 

```js
function sayHi() {
  alert(phrase);

  var phrase = "Hello";
}

sayHi();
```

위 예시는 `var`를 이용했기 때문에 사실상 아래와 같이 실행된다.

```js
function sayHi() {
  var phrase; // 선언은 함수 시작 시 처리됩니다.

  alert(phrase); // undefined

  phrase = "Hello"; // 할당은 실행 흐름이 해당 코드에 도달했을 때 처리됩니다.
}

sayHi();
```

## IIFE(즉시 실행 함수 표현식) : `var`가 남긴 폐해의 잔재 

과거에는 `var`만 쓸 수 있었고, 이를 쓰기 위해서 과거의 개발자들은 블록 레벨 스코프를 구현하기 위해 여러 방안을 고려했다. 그 결과 만들어진 것이 IIFE(Immediately Invoked Function Expressions)다.

요즘에는 쓰지 않으나, 오래된 스크립트에서 만나볼 수 있다.

```js
(function() {

  let message = "Hello";

  alert(message); // Hello

})();
```

## 결론
결국, 두가지 끔찍한 이유 때문에 `var`를 사용하지 않는다.

1) `var`는 함수 스코프를 갖는다.
2) `var`는 호이스팅을 유발한다.