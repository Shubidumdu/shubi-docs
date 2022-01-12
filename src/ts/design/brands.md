# 공식 명칭에는 상표를 붙이기

C++, Java, Swift와 같은 언어들에서는 Nominal Typing(명시적 타이핑, 명목적 타이핑) 체계를 활용합니다.
이 말인 즉슨 똑같은 구조를 가진다고 해도 동일한 타입으로 간주되지 않는 것을 의미합니다.

```ts
// 의사 코드입니다!

class Foo {
  method(input: string): number { ... }
}

class Bar {
  method(input: string): number { ... }
}

let foo: Foo = new Bar(); // ERROR!!
```

한편 앞선 [아이템 4](/ts/intro/typing.html)에서도 말한 것 처럼, TS는 구조적 타이핑(Structural Typing) 체계를 활용합니다.
이는 다시 말해, 해당 타입의 이름이 달라도 타입이 호환되기만 한다면 이용에 아무런 문제가 없음을 의미합니다.

```ts
class Foo {
  method(input: string): number { return 2; }
}
class Bar {
  method(input: string): number { return 4; }
}

let foo: Foo = new Bar(); // Okay.
```

이러한 특징은 기본적으로 JS의 덕 타이핑을 모델링하기 위해 존재하는 것입니다. 하지만 때로 이러한 특성이 문제를 일으킬 수도 있죠.

```ts

interface Vector2D {
  x: number;
  y: number;
}
function calculateNorm(p: Vector2D) {
  return Math.sqrt(p.x * p.x + p.y * p.y);
}

calculateNorm({x: 3, y: 4});  // 정상입니다.
const vec3D = {x: 3, y: 4, z: 1};
calculateNorm(vec3D);  // 놀랍게도 이 역시 정상입니다.
```

위의 코드는 구조적 타이핑 관점에서 아무런 문제가 없지만, 우리가 개념적으로 생각했을 때는 분명 문제가 있는 코드입니다.
3D 벡터는 2D 벡터 매개변수에 전달되어서는 안된다는 것이죠.

## TS에서 Nominal Typing을 흉내내는 법

아이러니하게도, TS에서 Nominal Typing 체계를 흉내내기 위해선 하나의 프로퍼티를 추가적으로 사용해야 합니다.
이를 상표 기법(Branding)이라고 하는데, `_brand` (일종의 컨벤션)라는 프로퍼티로 타입이 아닌 값의 관점에서 해당 타입이 `Vector2D`임을 나타내는 것이죠.

```ts
type Vector2D = {
  _brand: '2d';
  x: number;
  y: number;
}

function vec2D(x: number, y: number): Vector2D {
  return {x, y, _brand: '2d'};
}

function calculateNorm(p: Vector2D) {
  return Math.sqrt(p.x * p.x + p.y * p.y);
}

calculateNorm(vec2D(3, 4)); // OK, returns 5

const vec3D = {x: 3, y: 4, z: 1};
calculateNorm(vec3D);
           // ~~~~~ Property '_brand' is missing in type...
```

### 원시 타입에도 적용할 수 있습니다

해당 기법이 재미있는 이유는, 객체가 아닌 어느 타입이든 활용할 수 있다는 점 때문입니다.
이를테면, `string`이나 `number`같은 기본적으로 프로퍼티를 가질 수 없는 타입에도 적용할 수 있습니다.

```ts
type AbsolutePath = string & {_brand: 'abs'};

function listAbsolutePath(path: AbsolutePath) {
  // ...
}

function isAbsolutePath(path: string): path is AbsolutePath {
  return path.startsWith('/');
}

function f(path: string) {
  if (isAbsolutePath(path)) {
    // 이제 `path`는 AbsolutePath로 간주됩니다. 실제론 `_brand` 프로퍼티가 없지만요.
    listAbsolutePath(path);
  }
  listAbsolutePath(path);
                // ~~~~ Argument of type 'string' is not assignable
                //      to parameter of type 'AbsolutePath'
}
```

`number` 타입의 경우에도 상표를 붙일 수는 있으나, 추가적인 연산이 이루어지게 되면 상표가 사라지기 때문에 실제 이용은 어렵습니다.

```ts
type Meters = number & {_brand: 'meters'};
type Seconds = number & {_brand: 'seconds'};

const meters = (m: number) => m as Meters;
const seconds = (s: number) => s as Seconds;

const oneKm = meters(1000);  // Type is Meters
const oneMin = seconds(60);  // Type is Seconds

const tenKm = oneKm * 10; // Type is number
const v = oneKm / oneMin; // Type is number
```

### 표현하기 어려운 속성을 모델링할 수 있습니다

이를테면, Array 타입 자체만으로 해당 Array가 정렬 처리되었는지에 대한 여부를 나타내는 것은 상당히 어렵습니다.

```ts
type SortedList<T> = T[];
```

Array가 정렬되었음을 나타내려면 마찬가지로 상표 기법을 사용하면 됩니다.

```ts
type SortedList<T> = T[] & {_brand: 'sorted'};

function isSorted<T>(xs: T[]): xs is SortedList<T> {
  for (let i = 1; i < xs.length; i++) {
    if (xs[i] > xs[i - 1]) {
      return false;
    }
  }
  return true;
}

const list = [1, 2, 3];

if (isSorted(list)) {
  // 이제 `list`는 `SortedList` 타입입니다.
  // ...
}
```
