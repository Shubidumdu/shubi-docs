# 프로토타입

## 1. 프로토타입 상속

개발을 하다보면 기존에 있는 기능을 가져와 확장을 해야하는 경우가 생긴다.

이는 자바스크립트의 고유 기능인 포로토타입 상속(Prototypal Inheritance)를 이용하면 실현할 수 있다.

### [[Prototype]]

자바스크립트의 객체는 `[[Prototype]]`이라는 숨김 프로퍼티를 갖는다. 이 숨김 프로퍼티는 `null`이거나, 다른 객체에 대한 참조가 되는데 (**그 외의 자료형은 무시된다.**), 이것이 참조하는 대상을 **프로토타입(prototype)**이라고 부른다.

JS는 객체에서 프로퍼티를 찾다가, 해당 프로퍼티가 없으면 **자동으로 프로토타입에서 프로퍼티를 찾는다.** 프로그래밍에서는 이런 동작 방식을 **프로토타입 상속**이라 부른다.

`[[Prototype]]` 프로퍼티는 내부 프로퍼티면서 숨김 프로퍼티지만, 다양한 방법을 사용해 개발자가 값을 설정할 수 있다.

그 중 하나는, `__proto__`를 사용해 값을 설정하는 것이다.

> 참고로, `__proto__`는 `[[Prototype]]`용 getter / setter라는 점을 이해하자.
> 요즘에는 `__proto__`를 직접 쓰는 경우는 드물고, `Object.getPrototypeOf`나 `Object.setPrototypeOf`를 써서 프로토타입을 획득 혹은 설정한다. 왜 `__proto__`를 요즘은 쓰지 않는지에 대해서는 추후에 다루자.

`obj.hasOwnProperty(key)`는 해당 객체의 `key`에 해당하는 프로퍼티가 상속받은 것이 아닌, 직접 구현된 프로퍼티일 경우 `true`를 반환한다. **프로토타입으로 부터의 프로퍼티인지**를 체크하는 역할을 한다고 보면 되겠다.

## 2. `prototype` 프로퍼티

`new F()`와 같은 생성자 함수를 사용하면 새로운 객체를 만들 수 있다는 것을 앞서 배웠다. 그런데, 이 `F.prototype`이 객체라면, `new` 연산자는 `F.prototype`을 사용해 새롭게 생성된 객체의 `[[Prototype]]`을 설정한다.

과거엔 프로토타입에 직접 접근할 방법이 없었고, 그나마 믿고 사용할 수 있는 방법이 해당 방법 뿐이었다. 여전히 이 문법이 남아있는 이유다.

여기서 `F.prototype`은 그저 일반 프로퍼티라는 점에 주의해야 한다.

```js
let animal = {
  eats: true,
};

function Rabbit(name) {
  this.name = name;
}

Rabbit.prototype = animal;

let rabbit = new Rabbit('White Rabbit'); //  rabbit.__proto__ == animal

alert(rabbit.eats); // true
```

## `contructor` 프로퍼티

사실, 개발자가 따로 할당하지 않더라도, 모든 함수는 `prototype` 프로퍼티를 갖는다. 기본 프로퍼티인 `prototype`은 `constructor` 프로퍼티 하나만 있는 객체를 가리키는데, 이 `constructor` 프로퍼티는 함수 자신을 가리킨다. 이 관계를 코드로 나타내면 다음과 같다.

```js
function Rabbit() {}

/* 기본 prototype
Rabbit.prototype = { constructor: Rabbit };
*/
```

```js
function Rabbit() {}
// 기본 prototype:
// Rabbit.prototype = { constructor: Rabbit }

let rabbit = new Rabbit(); // {constructor: Rabbit}을 상속받음

alert(rabbit.constructor == Rabbit); // true (프로토타입을 거쳐 접근함)
```

`constructor`프로퍼티를 사용하면, 기존에 있던 객체의 `constructor`를 사용해서 새로운 객체를 만들 수 있다.

```js
function Rabbit(name) {
  this.name = name;
  alert(name);
}

let rabbit = new Rabbit('White Rabbit');

let rabbit2 = new rabbit.constructor('Black Rabbit');
```

이 `constructor`는 객체가 있는데, 이 객체를 만드는데 어떤 생성자가 사용되었는지 알수 없는 경우에 사용된다. **단, 가장 중요한점은 JS가 알맞은 `constructor`값을 보장하진 않는다는 점이다.** 함수에 기본으로 `prototype`값이 설정되지만 그것이 전부다. `constructor`에 벌어지는 모든 일은 전적으로 개발자에게 맡겨지며, 만약 함수의 기본 `prototype`값을 다른 객체로 바꾼다면 이 객체엔 `constructor`가 없어진다.

이를 방지하고 알맞은 `constructor`를 유지하기 위해서는 `prototype` 전체를 덮어쓰지 말고 기본 `prototype`에 원하는 프로퍼티를 추가/제거해야 한다. (참조 관계를 끊지 않기 위해서)

```js
function Rabbit() {}

// Rabbit.prototype 전체를 덮어쓰지 말고
// 원하는 프로퍼티는 그냥 추가하세요.
Rabbit.prototype.jumps = true;
// 이렇게 하면 기본 Rabbit.prototype.constructor가 유지됩니다.
```

수동으로 `constructor` 프로퍼티를 다시 만들어주는 것도 대안이 된다.

```js
Rabbit.prototype = {
  jumps: true,
  constructor: Rabbit,
};

// 수동으로 추가해 주었기 때문에 알맞은 constructor가 유지됩니다.
```

## 3. 네이티브 프로토타입

`prototype` 프로퍼티는 JS 내부에서도 광범위하게 사용되는데, 모든 내장 생성자 함수에서 `prototype` 프로퍼티를 사용한다.

### Object.prototype

```js
let obj = {};
alert(obj); // "[object Object]" ?
```

여기서 `"[object Object]"`를 생성하는 코드는 대체 어디에 있을까?? `obj`는 비어있는데.

참고로 `obj = {}`는 `obj = new Object()`를 줄인 것이다. 여기서 `Object`는 내장 객체 생성자 함수인데, 이 객체의 `prototype`은 `toString`을 비롯해 다양한 메서드들이 구현된 거대한 객체를 참조한다. 따라서 `obj.toString()`을 호출하면 `Object.prototype`에서 해당 메서드를 찾아 가져오게 된다.

```js
let obj = {};

alert(obj.__proto__ === Object.prototype); // true

alert(obj.toString === obj.__proto__.toString); //true
alert(obj.toString === Object.prototype.toString); //true
```

단, 이때 `Object.prototype` 위에는 그 이상의 `[[Prototype]]`이 존재하지 않는다는 점을 주의하자.

```js
alert(Object.prototype.__proto__); // null
```

### **모든 것은 객체를 상속받는다**

`Array`, `Date`, `Function`을 비롯한 내장 객체들 역시 프로토타입에 메서드를 저장해놓는다.

명세서 상에서는 모든 내장 프로토타입의 꼭대기에는 `Object.prototype`이 있어야 한다고 규정한다. 이 때문에 **모든 것은 객체를 상속받는다**는 말을 하기도 한다.

체인 상의 프로토타입에는 중복 메서드가 있을 수도 있는데, 이 경우, 체인 상에서 가까운 메서드를 사용하며, `Array`의 경우, `Array.prototype`의 메서드가 `Object.prototype`의 메서드보다 가깝기 때문에 해당 메서드가 사용된다.

### 원시값(Primitive Value)

그럼 원시값은요?? 이들을 프로토타입을 통해 다루는 것은 상당히 까다롭다.

문자열과 숫자, 불린은 객체가 아니다. 그런데 이런 원시값들의 프로퍼티에 접근하려고 하면 내장 생성자 `String`, `Number`, `Boolean`을 사용하는 **임시 래퍼(Wrapper) 객체가 생성**된다. 이 래퍼 객체는 해당 메서드만 제공하고 나면 사라진다.

래퍼 객체는 보이지 않는 곳에서 만들어지고, 엔진에 의해 최적화된다.

참고로 `null`과 `undefined`에 대응하는 래퍼 객체는 없다. 떄문에 메서드와 프로퍼티는 물론, 당연히 프로토타입도 사용할 수 없다.

### 네이티브 프로토타입 변경

이런 네이티브 프로토타입을 직접 변경할 수도 있다.

```js
String.prototype.show = function () {
  alert(this);
};

'BOOM!'.show(); // BOOM!
```

다만, 이는 좋은 생각이 아닌데, 기본적으로 네이티브 프로토타입은 전역으로 영향을 미치기 때문이다. 때문에 이런식으로 네이티브 프로토타입을 수정하게 되면 다른 라이브러리의 메서드와 충돌할 가능성이 크다.

네이티브 프로토타입 변경이 허용되는 유일한 경우는 딱 하나인데, 바로 **폴리필을 만들 때**다.

폴리필은 JS 명세서에는 정의되어 있으나 특정 JS 엔진에서 해당 기능이 구현되지 않았을 경우 만들어 사용한다.

### 프로토타입에서 빌려오기

네이티브 프로토타입에 구현된 메서드를 빌려와서 사용할 수도 있다.

다음은 객체 `obj`에 `Array`의 `join` 메서드를 구현하는 내용이다.

```js
let obj = {
  0: 'Hello',
  1: 'world!',
  length: 2,
};

obj.join = Array.prototype.join;

alert(obj.join(',')); // Hello,world!
```

## **모던**하게 프로토타입을 다루기

`__proto__`는 브라우저를 대상으로 개발한다면 구식의 방법이기에 더는 사용하지 않는다.

이를 대체할 아래의 모던한 메서드들이 있다.

**Object.create(proto, [descriptors])** – [[Prototype]]이 proto를 참조하는 빈 객체를 만든다. 이때 프로퍼티 설명자({ value, enumarable, ...})를 추가로 넘길 수 있다.
**Object.getPrototypeOf(obj)** – obj의 [[Prototype]]을 반환한다.
**Object.setPrototypeOf(obj, proto)** – obj의 [[Prototype]]이 proto가 되도록 설정한다.

```js
let animal = {
  eats: true,
};

// 프로토타입이 animal인 새로운 객체를 생성합니다.
let rabbit = Object.create(animal);

alert(rabbit.eats); // true

alert(Object.getPrototypeOf(rabbit) === animal); // true

Object.setPrototypeOf(rabbit, {}); // rabbit의 프로토타입을 {}으로 바꿉니다.
```

앞서 말한것처럼 프로퍼티 설명자를 선택적으로 전달할 수도 있다.

```js
let animal = {
  eats: true,
};

let rabbit = Object.create(animal, {
  jumps: {
    value: true,
  },
});

alert(rabbit.jumps); // true
```

`Object.create`를 통해 객체를 효율적으로 (얕게) 복제할 수도 있다.
아래 코드는 `obj`의 모든 프로퍼티를 포함한 완벽한 사본을 만든다.

```js
let clone = Object.create(
  Object.getPrototypeOf(obj),
  Object.getOwnPropertyDescriptors(obj),
);
```

주의해야 할 점은, 앞선 메서드들로 객체의 `[[Prototype]]`을 수정하는데 기술적인 문제는 전혀 없으나, 이는 권장되는 사항이 아니다. 이는 객체 프로퍼티 접근 관련 최적화를 망치기 때문에, JS 엔진의 속도를 매우 느리게 한다. 때문에 `[[Prototype]]`은 객체를 처음 생성할 때만 설정하는 것이 일반적이다.

### 아주 단순한(Very plain) 객체

`Object.create()`는 인자의 `[[Prototype]]`을 상속받은 객체를 생성한다. 이 때, 상속받는 객체 자체가 없다면 어떻게 될까??

`Object.create(null)`은 `__proto__`를 상속받지 않는다. 때문에 `__proto__`가 키 값이 되어도 일반 데이터 프로퍼티처럼 처리하므로 버그가 발생하지 않는다.

이런 객체는 아주 단순한(Very plain), 혹은 순수 사전식(Pure dictionary) 객체라고 부른다. 일반 객체 `{...}`보다도 훨씬 단순하기 때문이다.

단, 이 단순한 객체는 프로토타입 자체가 없기 때문에 **내장 메서드조차 없다.**

```js
let obj = Object.create(null);

alert(obj); // Error: Cannot convert object to primitive value (toString이 없음)
```
