# 객체 래퍼 타입 피하기

JS에는 객체 외에 일곱가지 기본형 값(Primitives)들이 있습니다.

- `string`
- `number`
- `boolean`
- `null`
- `undefined`
- `symbol`
- `bigint` -> `number`의 최대치인 2^53-1보다 큰 값을 표현할 때 사용

### 기본형 값들의 특징

- 불변(immutable)합니다.
- 메서드를 가지지 않습니다.

이를 듣고 봤을 때, 언뜻 아래 코드는 앞서 설명한 바와 말이 다른 것처럼 보입니다.

```js
'primitive'.charAt(3) // m
```

사실, `charAt`은 `string`의 메서드가 아닙니다. 
Primitive에 해당하는 `string`에는 메서드가 없는 대신, 이 `string`과 관련된 메서드를 지닌 `String` 객체 타입이 별도로 정의되어 있습니다.
여러모로 서로 간에 깊이 연관된 탓에, `string`과 `String`을 언뜻 동일한 것으로 이해하기 쉽지만, 둘 사이에는 분명한 차이가 있습니다.

대부분의 경우 사실 래퍼 객체들은 직접 사용할 일이 드뭅니다. 우리가 직접 사용하지 않더라도 내부적으로 필요한 경우에 사용되기 때문입니다.
이를테면 위에서 `string`에 `charAt`을 쓴 경우, 우리가 보지 못하는 다음의 일들이 일어납니다.

1. 기본값인 `string`을 `String` 객체로 래핑합니다.
2. 이후 래핑한 `String` 객체에서 메서드를 호출합니다.
3. 그리고나서 래핑한 객체를 버립니다.

이러한 과정을 거치는 탓에, 아래와 같이 이상한 코드를 작성했을 때 에러는 발생하지 않지만, 그렇다고 의도대로 동작하지도 않습니다.

```js
const x = 'hello';
x.language = 'English'
console.log(x.language) // undefined
```

### 래퍼 객체는 오직 자기 자신하고만 동일합니다.

그렇기 때문에 아래의 코드는 틀린 내용입니다.

```js
'hello' === new String('hello') // false
new String('hello') === new String('hello') // false
```

> **주의** : 단, `new` 없이 호출된 객체 래퍼들은 기본형을 생성합니다.

```js
String('abc') === 'abc' // true
BigInt(1) === BigInt(1) // true
```


### 래퍼 객체 타입보다는 기본형 타입을 사용하세요.

- string / String
- number / Number
- boolean / Boolean
- symbol / Symbol
- bigint / BigInt

위처럼 TS에서도 기본형과 객체 래퍼 타입은 별도로 모델링되며, 엄연히 다른 타입입니다.
TS에서 타입을 다룰 때는 모든 경우에 기본형 타입을 사용하는 것이 옳습니다. `기본형 -> 래퍼 객체`는 할당이 가능하지만, `래퍼 객체 -> 기본형`은 할당이 불가능하기 때문입니다.
이 탓에 에러가 발생할 가능성이 높습니다.