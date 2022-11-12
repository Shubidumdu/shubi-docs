# 타입스크립트 도입 전에 @ts-check와 JsDoc으로 시험해 보기

## @ts-check

TS로 전환하기에 앞서, `@ts-check` 지시자를 사용하면 TS 전환 시에 어떤 문제가 발생하는지 JS 상에서 미리 시험해 볼 수 있습니다.
`@ts-check` 지시자를 통해 타입 체커가 파일을 분석하고, 발견된 오류를 보고하도록 지시합니다.

```js
// @ts-check
const person = {first: 'Grace', last: 'Hopper'};
2 * person.first
 // ~~~~~~~~~~~~ The right-hand side of an arithmetic operation must be of type
 //              'any', 'number', 'bigint', or an enum type
```

그러나 `@ts-check` 지시자는 매우 느슨한 수준의 타입 체크를 수행합니다. 심지어는 `noImplicitAny` 설정을 해제한 것보다 느슨하므로 이에 주의해야 합니다.

만약 기존에 JSDoc 스타일의 주석을 사용 중이었다면, `@ts-check` 지시자 설정 시 기존 주석에 대한 타입 체크가 동작합니다.

```js
// @ts-check
/**
 * Gets the size (in pixels) of an element.
 * @param {Node} el The element
 * @return {{w: number, h: number}} The size
 */
function getSize(el) {
  const bounds = el.getBoundingClientRect();
                 // ~~~~~~~~~~~~~~~~~~~~~ Property 'getBoundingClientRect'
                 //                       does not exist on type 'Node'
  return {width: bounds.width, height: bounds.height};
       // ~~~~~~~~~~~~~~~~~~~ Type '{ width: any; height: any; }' is not
       //                     assignable to type '{ w: number; h: number; }'
}
```

`@ts-check` 지시자와 `JsDoc` 주석을 통해 JS 환경에서도 TS와 유사한 경험으로 개발이 가능하기 때문에 마이그레이션 과정에 도움이 됩니다.
하지만, 장기적인 관점에서는 주석의 양이 늘어나 로직의 해석을 방해한다는 문제가 있습니다. 때문에 궁극적으로는 모든 코드가 TS 기반으로 전환될 수 있는 것을 목표로 삼아야 합니다.