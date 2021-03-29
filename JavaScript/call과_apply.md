# call, apply, decorator, call forwarding

## 데코레이터

**데코레이터(Decorator)는 특정 함수를 인수로 받아, 반환하는 값 자체는 동일하지만 그 외의 로직을 추가적으로 커스텀하는 함수를 의미한다.**

로직을 함수와 함수로 분리시킬 수 있기 때문에 한결 보기 편하다는 장점이 있다.

```js
function slow(x) {
  // CPU 집약적인 작업이 여기에 올 수 있습니다.
  alert(`slow(${x})을/를 호출함`);
  return x;
}

function cachingDecorator(func) {
  let cache = new Map();

  return function (x) {
    if (cache.has(x)) {
      // cache에 해당 키가 있으면
      return cache.get(x); // 대응하는 값을 cache에서 읽어옵니다.
    }

    let result = func(x); // 그렇지 않은 경우엔 func를 호출하고,

    cache.set(x, result); // 그 결과를 캐싱(저장)합니다.
    return result;
  };
}

slow = cachingDecorator(slow);

alert(slow(1)); // slow(1)이 저장되었습니다.
alert('다시 호출: ' + slow(1)); // 동일한 결과

alert(slow(2)); // slow(2)가 저장되었습니다.
alert('다시 호출: ' + slow(2)); // 윗줄과 동일한 결과
```

## `func.call`로 컨텍스트 지정하기

이러한 데코레이터는, 객체 메서드에서 사용하기에 적합하지 않다는 문제점이 있는데, 메서드가 기존 객체를 가리키던 `this`에 대한 컨텍스트를 잃어버리기 때문이다.

```js
// worker.slow에 캐싱 기능을 추가해봅시다.
let worker = {
  someMethod() {
    return 1;
  },

  slow(x) {
    // CPU 집약적인 작업이라 가정
    alert(`slow(${x})을/를 호출함`);
    return x * this.someMethod(); // (*)
  },
};

// 이전과 동일한 코드
function cachingDecorator(func) {
  let cache = new Map();
  return function (x) {
    if (cache.has(x)) {
      return cache.get(x);
    }
    let result = func(x); // (**)
    cache.set(x, result);
    return result;
  };
}

alert(worker.slow(1)); // 기존 메서드는 잘 동작합니다.

worker.slow = cachingDecorator(worker.slow); // 캐싱 데코레이터 적용

alert(worker.slow(2)); // 에러 발생!, Error: Cannot read property 'someMethod' of undefined
```

`func.call`은 이때 사용할 수 있는데, 이는 `this`를 명시적으로 고정해 함수를 호출할 수 있게 해주는 내장 함수 메서드다. 기본적으로는 아래와 같은 형태다.

```js
func.call(context, arg1, arg2, ...)
```

실제로 적용한다면 아래와 같아진다.

```js
func(1, 2, 3);
func.call(obj, 1, 2, 3);
```

위의 두 실행은 거의 동일하다. 유일한 차이점은 `func.call`에서의 `this`는 `obj`로 고정된다는 점이다.

## func.apply도 동일한 역할을 한다.

`func.apply`를 사용해도 된다. 이는 아래와 같은 형태다.

```js
func.apply(context, args);
```

`apply`는 `func`의 `this`를 `context`로 고정시켜주고, 유사 배열 객체 `args`를 인수로 사용할 수 있게 해준다. `call`과의 문법적 차이는 `call`이 여러 개의 인수들을 따로따로 받는 대신 `apply`는 배열을 인수로 받는 다는 점 뿐이다.

따라서 아래의 두 코드는 거의 같은 역할을 한다.

```js
func.call(context, ...args);
func.apply(context, args);
```

이렇게 인수 전체를 다른 함수에 전달하는 것을 **콜 포워딩(Call Forwarding)** 이라고 한다.
