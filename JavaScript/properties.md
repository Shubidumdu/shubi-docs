# Properties(프로퍼티)

알다시피, 객체에는 프로퍼티가 저장된다. 지금까지는 단순히 'key-value'의 관점에서 보일 수 있었겠지만, 사실 프로퍼티는 생각보다 더 유연하고 강력한 자료구조다.

프로퍼티 플래그를 사용하면 손쉽게 `getter`나 `setter`함수를 구현할 수 있다.

## 프로퍼티 플래그 (Property flag)

- 객체 프로퍼티는 **값(value)** 뿐만 아니라, **플래그(flag)**라 불리는 특별한 속성 세 가지를 갖는다.

> **writable** - `true`라면 값을 수정할 수 있다. 그렇지 않다면 읽기 전용이 된다.
> **enumarable** - `true`라면 반복문을 통해 나열될 수 있다. 그렇지 않다면 나열되지 않는다.
> **configurable** - `true`라면 프로퍼티 삭제나 플래그 수정이 가능하다. 그렇지 않다면 프로퍼티 삭제와 플래그 수정이 불가능하다.

프로퍼티 플래그는 특별한 경우가 아니라면 쓰이지 않는다. 평범한 방식으로 프로퍼티를 만들면 해당 프로퍼티 플래그는 모두 `true`가 되고, 이렇게 설정된 플래그는 언제든 수정할 수 있다.


먼저 플래그를 얻는 방법에 대해 알아보자.

`Object.getOwnPropertyDescriptor` 메서드는 특정 프로퍼티에 대한 정보를 모두 얻을 수 있게 해준다.

```js
let descriptor = Object.getOwnPropertyDescriptor(obj, propertyName);
```

해당 메서드를 호출하면 프로퍼티 설명자(descriptor)라고 불리는 객체가 반환되며, 여기에는 프로퍼티 값과 세 플래그에 대한 정보가 모두 담겨있다.

```js
let user = {
  name: "John"
};

let descriptor = Object.getOwnPropertyDescriptor(user, 'name');

alert( JSON.stringify(descriptor, null, 2 ) );
/* property descriptor:
{
  "value": "John",
  "writable": true,
  "enumerable": true,
  "configurable": true
}
*/
```

`Object.defineProperty`를 사용하면 플래그를 변경할 수 있다.

```js
Object.defineProperty(obj, propertyName, descriptor)
```

이는 해당 프로퍼티가 이미 존재한다면, 해당 프로퍼티를 인자로 넘긴 플래그에 따라 변경해주고, 프로퍼티가 없으면 인수로 넘겨받은 정보를 통해 새로운 프로퍼티를 만든다. 플래그 정보가 따로 없는 경우는 자동으로 `false`가 된다.

`Object.defineProperties`는 앞선 프로퍼티 정의 여러개를 한꺼번에 할 수 있다.

```js
Object.defineProperties(user, {
  name: { value: "John", writable: false },
  surname: { value: "Smith", writable: false },
  // ...
});
```

`Object.getOwnPropertyDescriptors`는 프로퍼티 설명자를 전부 한꺼번에 가져올 수 있게 한다. `Object.defineProperties`와 함께 사용하면 객체 복사 시 플래그도 함께 복사할 수 있다.

```js
let clone = Object.defineProperties({}, Object.getOwnPropertyDescriptors(obj));
```

## 접근자 프로퍼티 (Getter & Setter)

객체 프로퍼티는 두 종류로 나뉜다.

> 1. **데이터 프로퍼티(data property)** - 지금껏 사용한 모든 프로퍼티는 데이터 프로퍼티다.
> 2. **접근자 프로퍼티(accessor property)** - 접근자 프로퍼티의 본질은 함수인데, 이 함수는 값을 획득(get)하고, 설정(set)하는 역할을 담당한다. 그러나 외부 코드에서는 함수가 아닌 일반적인 프로퍼티처럼 **보인다**.

객체 리터럴 안에서 `getter`와 `setter` 메서드는 `get`과 `set`으로 나타낼 수 있다.

```js
let obj = {
  get propName() {
    // getter, obj.propName을 실행할 때 실행되는 코드
  },

  set propName(value) {
    // setter, obj.propName = value를 실행할 때 실행되는 코드
  }
};
```

`getter`를 구현하면, 마치 일반 프로퍼티인것처럼 동작한다. 이는 함수처럼 **호출**하지 않으며, 일반 프로퍼티에 접근하듯 평범히 `user.fullName`을 통해 값을 얻어올 수 있다. 실질적으로는 메서드를 호출하는 것이지만.

```js
let user = {
  name: "John",
  surname: "Smith",

  get fullName() {
    return `${this.name} ${this.surname}`;
  }
};

alert(user.fullName); // John Smith

user.fullName = "Test"; // Error (프로퍼티에 getter 메서드만 있어서 에러가 발생합니다.)
```

또한, `getter`만 있는 경우는 값을 직접 할당할 수 없어 위와 같은 에러가 발생한다.

여기에 `setter`도 추가로 구현한다면 다음과 같아진다.

```js
let user = {
  name: "John",
  surname: "Smith",

  get fullName() {
    return `${this.name} ${this.surname}`;
  },

  set fullName(value) {
    [this.name, this.surname] = value.split(" ");
  }
};

// 주어진 값을 사용해 set fullName이 실행됩니다.
user.fullName = "Alice Cooper";

alert(user.name); // Alice
alert(user.surname); // Cooper
```

`getter`와 `setter` 메서드를 구현하면 객체에는 `fullName`이라는 가상 프로피터가 생기며, 이는 읽고 쓸수는 있지만 실제로 존재하진 않는다.

### 접근자 프로퍼티의 설명자(descriptor)

데이터 프로퍼티의 설명자와 접근자 프로퍼티의 설명자는 다르다.
접근자 프로퍼티에는 `value`와 `writable` 대신에 `get`과 `set`이 있다.

> **get** – 인수가 없는 함수로, 프로퍼티를 읽을 때 동작함
> **set** – 인수가 하나인 함수로, 프로퍼티에 값을 쓸 때 호출됨
> **enumerable** – 데이터 프로퍼티와 동일함
> **configurable** – 데이터 프로퍼티와 동일함