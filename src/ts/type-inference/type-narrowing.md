# 타입 좁히기

타입 넓히기의 반대는 타입 좁히기(Type narrowing)입니다.
타입 좁히기는 타입스크립트가 넓은 타입으로부터 좁은 타입으로 진행하는 과정을 말합니다.

### `null` 체킹

가장 일반적인 예시는 `null` 체킹입니다.
TS가 문맥 상 해당 변수가 `null`이 아님을 확신할 수 있을 경우, 타입에서 `null`이 제거됩니다.

```ts
const el = document.getElementById('foo'); // Type is HTMLElement | null
if (el) {
  el // Type is HTMLElement
  el.innerHTML = 'Party Time'.blink();
} else {
  el // Type is null
  alert('No element #foo');
}
```

```ts
const el = document.getElementById('foo'); // Type is HTMLElement | null
if (!el) throw new Error('Unable to find #foo');
el; // Now type is HTMLElement
el.innerHTML = 'Party Time'.blink();
```


### instanceof

```ts
function contains(text: string, search: string|RegExp) {
  if (search instanceof RegExp) {
    search  // Type is RegExp
    return !!search.exec(text);
  }
  search  // Type is string
  return text.includes(search);
}
```

### 프로퍼티 체크

```ts
interface A { a: number }
interface B { b: number }
function pickAB(ab: A | B) {
  if ('a' in ab) {
    ab // Type is A
  } else {
    ab // Type is B
  }
  ab // Type is A | B
}
```

### 내장함수 사용

```ts
function contains(text: string, terms: string|string[]) {
  const termList = Array.isArray(terms) ? terms : [terms];
  termList // Type is string[]
  // ...
}
```

### Tagged Union (Discriminated Union)

```ts
interface UploadEvent { type: 'upload'; filename: string; contents: string }
interface DownloadEvent { type: 'download'; filename: string; }
type AppEvent = UploadEvent | DownloadEvent;

function handleEvent(e: AppEvent) {
  switch (e.type) {
    case 'download':
      e  // Type is DownloadEvent
      break;
    case 'upload':
      e;  // Type is UploadEvent
      break;
  }
}
```

## User-Defined Type Guards (사용자 정의 타입 가드)

TS가 타입을 적절히 식별하도록 하기 위해, 커스텀 함수를 직접 작성하여 타입 좁히기에 관여할 수 있습니다.

```ts
// 해당 함수가 true를 반환한다면 `el`은 HTMLInputElement 타입으로 좁혀집니다.
function isInputElement(el: HTMLElement): el is HTMLInputElement {
  return 'value' in el;
}

function getElementContent(el: HTMLElement) {
  if (isInputElement(el)) {
    el; // Type is HTMLInputElement
    return el.value;
  }
  el; // Type is HTMLElement
  return el.textContent;
}
```

```ts
const jackson5 = ['Jackie', 'Tito', 'Jermaine', 'Marlon', 'Michael'];

function isDefined<T>(x: T | undefined): x is T {
  return x !== undefined;
}

const members = ['Janet', 'Michael'].map(
  who => jackson5.find(n => n === who) // Type is (string | undefined)[]
).filter(isDefined);  // Type is string[]
```