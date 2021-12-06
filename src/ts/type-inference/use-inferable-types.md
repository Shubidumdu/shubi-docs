# 추론 가능한 타입을 사용해 장황한 코드 방지하기

TS의 타입 추론은 생각보다 훨씬 정확해서, 일반적으로는 명시적인 타입 구문 자체가 필요하지 않습니다.
그렇기 때문에, 이러한 "불필요한 타입 구문"들은 가능한 줄이는 것이 좋습니다. 
tslint를 사용하고 있다면, [`no-inferrable-types`](https://palantir.github.io/tslint/rules/no-inferrable-types/) 옵션을 통해 작성된 모든 타입 구문이 정말로 필요한지에 대해 확인할 수 있습니다.

```ts
const person = {
  name: 'Sojourner Truth',
  born: {
    where: 'Swartekill, NY',
    when: 'c.1797',
  },
  died: {
    where: 'Battle Creek, MI',
    when: 'Nov. 26, 1883'
  }
};

// typeof person: {
//     name: string;
//     born: {
//         where: string;
//         when: string;
//     };
//     died: {
//         where: string;
//         when: string;
//     };
// }
```

## 함수와 메서드의 시그니처에는 타입 구문을 쓰세요.

타입스크립트에게 직접 타입을 명시적으로 지정해주어야 하는 경우는, 말 그대로 타입스크립트가 스스로 타입을 판단하기 어려운 경우입니다.
함수가 그 대표적인 예시가 되는데, **이상적인 TS 코드는 함수 및 메서드 시그니처에 타입 구문을 포함하지만, 그 내부의 지역 변수들에서는 타입 구문을 넣지 않습니다.**

```ts
interface Product {
  id: string;
  name: string;
  price: number;
}

function logProduct(product: Product) {
  const {id, name, price} = product;
  console.log(id, name, price);
}
```

물론 함수임에도 이러한 타입 구문이 필요하지 않은 경우도 있습니다.

### 매개변수에 대한 기본값을 통해 타입 추론이 가능한 경우

```ts
// `base`는 타입 추론에 따라 `number` 타입이 됩니다.
function parseNumber(str: string, base = 10) {
  // ...
}
```

### 콜백함수로 넘겨짐에 따라 타입 추론이 가능한 경우

```ts
// axios.get에 넘겨지는 콜백함수는 이미 본인의 매개변수 타입을 알고 있습니다.
app.get('/health', (request, response) => {
  response.send('OK');
});
```

## 타입 추론이 가능하더라도, 타입을 명시해야 하는 경우

타입이 추론 가능하더라도, 여전히 직접 타입을 명시하는게 좋은 상황이 있습니다.

### 객체 리터럴을 정의할 때

객체 리터럴을 정의할 때, 타입 구문이 없다면 앞선 아이템 11에서 살펴봤던 잉여 속성 체크가 동작하지 않고, 이 경우 속성에 대한 오타를 해당 시점에 잡아내지 못합니다.
이 경우, 추후 해당 객체가 직접 사용될 때 이르러서야 해당 객체를 사용한 곳에서 타입 에러가 발생하기에 혼동을 주기 쉽습니다.

```ts
interface Product {
  id: string;
  name: string;
  price: number;
}

function logProduct(product: Product) {
  const id: string = product.id;
  const name: string = product.name;
  const price: number = product.price;
  console.log(id, name, price);
}

const furby = {
  name: 'Furby',
  id: 630509430963,
  price: 35,
};

logProduct(furby);
        // ~~~~~ Argument .. is not assignable to parameter of type 'Product'
        //         Types of property 'id' are incompatible
        //         Type 'number' is not assignable to type 'string'
```

### 함수의 반환 타입

함수의 반환 타입 역시 알아서 추론이 가능하지만, 본인이 의도한 반환 타입과 다른 경우가 발생할 수 있으므로, 이를 미리 명시하고 제때 잡아내는 것이 필요합니다.
그렇지 않다면 구현 상의 문제가 [해당 함수를 사용하는 시점에서야 발견](https://github.com/grepp/hera-webapp/pull/2487/files)됩니다.

```ts
const cache: {[ticker: string]: number} = {};
function getQuote(ticker: string): Promise<number> {
  if (ticker in cache) {
    return cache[ticker];
        // ~~~~~~~~~~~~~ Type 'number' is not assignable to 'Promise<number>'
  }
  // COMPRESS
  return Promise.resolve(0);
  // END
}
```

함수의 반환 타입을 명시해야 하는 두 번째 이유는 명명된 타입(Named Type)을 사용하기 위해서입니다. 구조 상으로는 동일하더라도, 타입이 일관적이지 않으면 당황스러울 수 있기 때문이죠.

```ts
interface Vector2D { x: number; y: number; }
function add(a: Vector2D, b: Vector2D) {
  return { x: a.x + b.x, y: a.y + b.y };
}
// function add(a: Vector2D, b: Vector2D): {
//     x: number;
//     y: number;
// }
```