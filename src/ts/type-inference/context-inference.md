# 타입 추론에 문맥이 어떻게 사용되는지 이해하기

TS는 타입을 추론할 때 값 뿐만 아니라 문맥도 고려합니다. 함수로 값을 넘기는 경우가 대표적입니다.

## string 리터럴의 경우

```ts
type Language = 'JavaScript' | 'TypeScript' | 'Python';
function setLanguage(language: Language) { /* ... */ }

setLanguage('JavaScript');  // OK
// 'JavaScript' 리터럴은 `Language` 타입에 부합합니다.

let language = 'JavaScript'; // string으로 추론됩니다.
setLanguage(language);
// ~~~~~~~~ Argument of type 'string' is not assignable
//          to parameter of type 'Language'
```

이러한 문제를 해결하기 위해서는 크게 두가지 방법이 있습니다. 이는 당장의 string 리터럴 외에도 범용적으로 활용될 수 있습니다.

### 타입 선언

하나는 직접 타입을 선언해서 해당 `language` 변수에 가능한 값을 제한시키는 방법입니다.

```ts
let language: Language = 'JavaScript'; // type = Language
```

### 상수로 만들기

다른 하나는 `const` 키워드를 통해 `language` 변수가 변경 가능성이 없음을 타입체커에게 알려줘 더 정확한 타입을 유추할 수 있도록 해주는 방법입니다.

```ts
const language = 'JavaScript'; // type = `JavaScript`
```

## 튜플의 경우

튜플의 경우에도 이러한 문제가 발생할 수 있어 주의해야 합니다.

```ts
type Language = 'JavaScript' | 'TypeScript' | 'Python';
function setLanguage(language: Language) { /* ... */ }
// Parameter is a (latitude, longitude) pair.
function panTo(where: [number, number]) { /* ... */ }

panTo([10, 20]);  // OK
// [10, 20]은 [number, number] 타입에 부합합니다.

const loc = [10, 20]; // number[]로 추론됩니다.
panTo(loc);
//    ~~~ Argument of type 'number[]' is not assignable to
//        parameter of type '[number, number]'
```

이 경우에도 마찬가지로 타입 선언을 통해 해결할 수 있습니다.

```ts
const loc: [number, number] = [10, 20];
panTo(loc);  // OK
```

또는 해당 매개변수가 **정말로 상수인 경우**에는 `const` 단언을 사용할 수 있습니다. `const` 단언을 사용하면 해당 참조가 깊은(deeply) 상수라는 정보를 TS에 전달할 수 있습니다. 다만, 이 경우 해당 값이 `readonly`가 되어 전혀 변경할 수 없는 상태가 되기 때문에, 해당 값을 매개변수로 사용하는 함수 측에도 `readonly` 타입 정보를 추가해야 합니다.

```ts
function panTo(where: readonly [number, number]) { /* ... */ }
const loc = [10, 20] as const; // type = readonly [number, number]
panTo(loc);  // OK
```

다만, 해당 방식으로 문제를 해결하는 경우, `loc`에서 값을 할당하는 시점에 실수가 있었더라도, 정작 에러는 함수를 호출하는 곳에서 발생하기 때문에, 추후 혼란을 줄 수 있다는 문제가 있습니다.

```ts
function panTo(where: readonly [number, number]) { /* ... */ }
const loc = [10, 20, 30] as const;  // error is really here.
panTo(loc);
//    ~~~ Argument of type 'readonly [10, 20, 30]' is not assignable to
//        parameter of type 'readonly [number, number]'
//          Types of property 'length' are incompatible
//            Type '3' is not assignable to type '2'
```

## 객체의 경우

객체의 경우에도 이러한 문제가 동일하게 발생할 수 있습니다.

```ts
type Language = 'JavaScript' | 'TypeScript' | 'Python';
interface GovernedLanguage {
  language: Language;
  organization: string;
}

function complain(language: GovernedLanguage) { /* ... */ }

complain({ language: 'TypeScript', organization: 'Microsoft' });  // OK

const ts = {
  language: 'TypeScript', // type = string
  organization: 'Microsoft', // type = string
};
complain(ts);
//       ~~ Argument of type '{ language: string; organization: string; }'
//            is not assignable to parameter of type 'GovernedLanguage'
//          Types of property 'language' are incompatible
//            Type 'string' is not assignable to type 'Language'
```

이를 해결하고자 하는 경우에도 앞선 경우들과 마찬가지로 1)타입 선언을 추가하거나, 2)상수로 만들어주는 방법이 있습니다.

```ts
// 1) 타입 선언을 추가하거나
const ts: GovernedLanguage = {
  language: 'TypeScript',
  organization: 'Microsoft',
};

// 2) 상수로 만드세요.
const ts = {
  language: 'TypeScript' as const,
  organization: 'Microsoft',
};
```

## 콜백의 경우

TS는 콜백 함수의 매개변수를 유추하는 경우에도 문맥이 고려됩니다. 따라서 해당 콜백 함수를 따로 분리하는 경우에도 문제가 발생합니다.

```ts
function callWithRandomNumbers(fn: (n1: number, n2: number) => void) {
  fn(Math.random(), Math.random());
}

// 콜백함수의 매개변수에 타입 명시를 하지 않더라도, 문맥으로 타입을 유추해냅니다.
callWithRandomNumbers((a, b) => {
  a;  // number
  b;  // number
});

// 하지만 아래의 경우는 문맥이 유실되어 타입 추론이 불가능합니다.
const fn = (a, b) => {
  // ~    Parameter 'a' implicitly has an 'any' type
  //    ~ Parameter 'b' implicitly has an 'any' type
}
callWithRandomNumbers(fn);
```

이 경우에도 타입 선언을 통해 해결해줄 수 있습니다.

```ts
// 1) 매개변수에 타입을 명시하거나
const fn = (a: number, b: number) => {
  // ...
}

// 2) 함수 자체에 타입을 명시하세요.
type CallbackFn = (n1: number, n2: number) => void;
const fn: CallbackFn = (a, b) => {
  a // number
  b // number
}
```
