[여기](https://blog.bitsrc.io/understanding-execution-context-and-execution-stack-in-javascript-1c9ea8642dd0)의 문서를 번역하여, 임의로 정리한 내용입니다.

# 실행 컨텍스트 (Execution Context)

JS에서의 호스팅, 스코프, 클로저와 같은 개념들을 이해하기 위해서는 실행 컨텍스트(Execution Context)와 실행 스택(Execution Stack)에 대해서 이해해야 합니다.

## 실행 컨텍스트란?

단순히 말해서, 실행 컨텍스트는 JS 코드가 평가되고 실행되는 환경의 추상적인 개념입니다. 어떤 코드가 JS에서 실행될 때, 이는 실행 컨텍스트 내에서 실행됩니다.

## 실행 컨텍스트의 종류

다음 세 종류의 실행 컨텍스트가 있습니다.

- **전역 실행 컨텍스트** : 기본 실행 컨텍스트로서, 어떤 함수에도 포함되어 있지 않은 코드의 경우, 전역 실행 컨텍스트에 포함됩니다. 브라우저를 기준으로, `window`에 해당하는 글로벌 객체를 생성하고, 해당 글로벌 객체를 `this`로 설정합니다. 하나의 프로그램에는 하나의 전역 실행 컨텍스트만 존재할 수 있습니다.

- **함수 실행 컨텍스트** : 함수가 실행될 때마다, 해당 함수에 대한 새로운 실행 컨텍스트가 생성됩니다. 각 함수들은 자신의 실행 컨텍스트를 보유하지만, 이는 해당 함수가 실행될 때에 생성됩니다. 함수 실행 컨텍스트는 여러개가 될 수 있습니다.

- **Eval 함수 실행 컨텍스트** : `eval`함수 내에서 실행되는 코드들도 자신의 실행 컨텍스트를 보유합니다. 다만, `eval`은 JS 개발자들 사이에 자주 사용되지 않으므로, 여기에서 언급하지 않겠습니다.

## 실행 스택

**호출 스택**이라고도 불리는 실행 스택은, LIFO(후입선출) 스택 구조로 이루어진 하나의 스택입니다. 실행 스택은 코드 실행 중에 생성되는 모든 실행 컨텍스트를 담고 있습니다.

JS 엔진이 처음으로 스크립트에 마주치면, 전역 실행 객체를 생성한 후, 이를 현재 실행 스택에 추가(push)합니다. 이후 JS 엔진이 함수 실행을 발견할 때마다, 새로운 실행 컨텍스트를 생성하여 스택의 최상단에 추가합니다.

엔진은 실행 컨텍스트가 스택의 최상단에 있는 함수를 실행합니다. 해당 함수가 완료되면, 실행 스택은 스택으로부터 제거(pop)되며, 그 다음 컨텍스트로 넘어가게 됩니다.

```js
let a = 'Hello World!';
function first() {
  console.log('Inside first function');
  second();
  console.log('Again inside first function');
}
function second() {
  console.log('Inside second function');
}
first();
console.log('Inside Global Execution Context');
```

<img src="https://miro.medium.com/max/2000/1*ACtBy8CIepVTOSYcVwZ34Q.png" />

## 실행 컨텍스트의 생성

이제 JS에서 실행 컨텍스트가 어떻게 생성도는지에 다루어보겠습니다.

실행 컨텍스트는 두 단계를 통해 생성됩니다. **첫째는 생성 단계(Creation Phase)이고, 두번째는 실행 단계(Execution Phase)입니다.**

## 생성 단계 (Creation Phase)

실행 컨텍스트는 생성 단계에서 생성됩니다. 해당 단계에서는 아래와 같은 일들이 일어납니다.

1. 렉시컬 환경(Lexical Environment)이 생성됩니다.
2. 변수 환경(Variable Environment)이 생성됩니다.

따라서, 실행 컨텍스트는 개념적으로 아래와 같이 나타낼 수 있습니다.

```
ExecutionContext = {
  LexicalEnvironment = <ref. to LexicalEnvironment in memory>,
  VariableEnvironment = <ref. to VariableEnvironment in  memory>,
}
```

### 렉시컬 환경(Lexical Environment)

렉시컬 환경은, 식별자-변수(Identifier-Variable) 매핑을 하는 자료구조입니다. (여기서, 식별자란 변수 또는 함수의 이름을 가리키며, 변수는 실제 객체에 대한 참조 또는 원시값을 가리킵니다.)

예를 들어, 아래의 코드를 살펴봅시다.

```js
var a = 20;
var b = 40;
function foo() {
  console.log('bar');
}
```

위의 코드에서 렉시컬 환경은 다음과 같아질 것입니다.

```
lexicalEnvironment = {
  a: 20,
  b: 40,
  foo: <ref. to foo function>
}
```

각각의 렉시컬 환경은 다음의 셋으로 구성되어 있습니다.

1. 환경 레코드 (Environment Record)
2. 외부 환경에 대한 참조 (Reference to the outer environment)
3. This 바인딩

#### 환경 레코드

환경 레코드는 렉시컬 환경 내에서 변수와 함수의 선언이 보관되는 장소입니다. 환경 레코드에는 두가지 타입이 있습니다.

- 선언 환경 레코드(Declaration Environment Record) : 변수 및 함수 선언을 저장합니다. 함수 코드에 대한 렉시컬 환경은 선언 환경 레코드를 포함합니다.

- 객체 환경 레코드 (Object Environment Record) : 전역 코드에 대한 렉시컬 환경은 객체 환경 레코드를 포함합니다. 이는 변수와 함수 선언 외에도, 전역 바인딩 객체(Global binding object: 브라우저 상에서는 `window`)도 담고 있습니다. 따라서 레코드 내에 해당 바인딩 객체의 각 프로퍼티에 대한 새로운 항목이 생성됩니다.

참고로, 함수 코드를 위해, 환경 레코드는 `arguments` 객체를 포함하고 있습니다. 이 `arguments`객체에는 인덱스와 함수에 전달되는 인수(arguments) 간의 매핑과, 함수에 넘겨지는 인수의 갯수(length)가 담겨있습니다. 예를 들어, 아래 함수에서 `argument` 객체는 다음과 같은 형태일 것입니다.

```js
function foo(a, b) {
  var c = a + b;
}
foo(2, 3);
// argument object
Arguments: {0: 2, 1: 3, length: 2},
```

#### 외부 환경에 대한 참조

"외부 환경에 대한 참조"는 외부 렉시컬 환경에 대한 접근을 의미합니다. 이는 JS 엔진이 현재 렉시컬 환경에서 원하는 변수를 찾지 못하면, 외부 환경으로 뻗어나가 해당 변수를 찾을 수 있다는 것을 의미합니다.

#### This 바인딩

전역 실행 컨텍스트에서, `this`의 값은 전역 객체에 바인딩됩니다. (브라우저 상에서 `window` 객체)

함수 실행 컨텍스트에서, `this`의 값은 어떻게 해당 함수가 호출되느냐에 따라 달라집니다. 객체 참조에 의해서 호출되는 경우 `this`의 값은 해당 객체가 됩니다. 그렇지 않은 경우에 `this`는 전역 객체가 되며, strict 모드 상에서는 `undefined`가 됩니다.

```js
const person = {
  name: 'peter',
  birthYear: 1994,
  calcAge: function () {
    console.log(2018 - this.birthYear);
  },
};
person.calcAge();
// `this`는 `person`이 됩니다. `calcAge`가 `person` 객체 참조에 의해 호출되었기 때문입니다.
const calculateAge = person.calcAge;
calculateAge();
// `this`는 전역 객체에 해당하는 `window`입니다. 별도로 넘겨받은 객체 참조가 없기 때문입니다. strict 모드라면, `undefined`가 됩니다.
```

지금까지의 내용을 되짚어보자면, 렉시컬 환경은 추상적으로 다음과 같이 생겼을겁니다.

```
GlobalExectionContext = {
  LexicalEnvironment: {
    EnvironmentRecord: {
      Type: "Object",
      // Identifier bindings go here
    }
    outer: <null>,
    this: <global object>
  }
}
FunctionExectionContext = {
  LexicalEnvironment: {
    EnvironmentRecord: {
      Type: "Declarative",
      // Identifier bindings go here
    }
    outer: <Global or outer function environment reference>,
    this: <depends on how function is called>
  }
}
```

### 변수 환경 (Variable Environment)

이 또한 실행 컨텍스트 내에서 `VariableStatements`를 통해 생성된 바인딩을 환경 레코드에 보관하는 렉시컬 환경입니다.

위에서 말했듯, 변수 환경 또한 렉시컬 환경입니다. 따라서 앞서 말했던 렉시컬 환경이 갖고 있는 모든 구성요소와 프로퍼티를 갖고 있습니다.

ES6 상에서, 렉시컬 환경과 변수 환경의 한가지 차이는, 렉시컬 환경이 함수 선언과 `let`, `const` 변수 바인딩에 사용되는 반면, 변수 환경은 오직 `var` 변수의 바인딩에만 사용된다는 것입니다.

## 실행 단계 (Execution Phase)

이 단계에서 모든 변수에 대한 할당이 완료되고, 코드가 마침내 실행됩니다.

아래 예시 코드를 살펴봅시다.

```js
let a = 20;
const b = 30;
var c;

function multiply(e, f) {
  var g = 20;
  return e * f * g;
}

c = multiply(20, 30);
```

위의 코드가 실행될 때, JS 엔진은 전역 코드를 실행하기 위해 전역 실행 컨텍스트를 먼저 생성합니다. 따라서 전역 실행 컨텍스트는 생성 단계를 거쳐 아래와 같아질 것입니다.

```
GlobalExectionContext = {
  LexicalEnvironment: {
    EnvironmentRecord: {
      Type: "Object",
      // Identifier bindings go here
      a: < uninitialized >,
      b: < uninitialized >,
      multiply: < func >
    }
    outer: <null>,
    ThisBinding: <Global Object>
  },
  VariableEnvironment: {
    EnvironmentRecord: {
      Type: "Object",
      // Identifier bindings go here
      c: undefined,
    }
    outer: <null>,
    ThisBinding: <Global Object>
  }
}
```

실행 단계에서는 변수 할당이 수행됩니다. 따라서 전역 실행 컨텍스트 실행 단계를 거쳐 아래와 같아질 것입니다.

```
GlobalExectionContext = {
  LexicalEnvironment: {
      EnvironmentRecord: {
        Type: "Object",
        // Identifier bindings go here
        a: 20,
        b: 30,
        multiply: < func >
      }
      outer: <null>,
      ThisBinding: <Global Object>
    },
  VariableEnvironment: {
      EnvironmentRecord: {
        Type: "Object",
        // Identifier bindings go here
        c: undefined,
      }
      outer: <null>,
      ThisBinding: <Global Object>
    }
}
```

이후, `multiply(20, 30)` 함수의 호출을 마주치면, 해당 함수 코드를 실행하기 위해 새로운 함수 실행 컨텍스트가 생성됩니다. 따라서, 생성 단계를 거쳐 다음과 같이 새로운 함수 실행 컨텍스트가 만들어집니다.

```
FunctionExectionContext = {
  LexicalEnvironment: {
      EnvironmentRecord: {
        Type: "Declarative",
        // Identifier bindings go here
        Arguments: {0: 20, 1: 30, length: 2},
      },
      outer: <GlobalLexicalEnvironment>,
      ThisBinding: <Global Object or undefined>,
    },
  VariableEnvironment: {
      EnvironmentRecord: {
        Type: "Declarative",
        // Identifier bindings go here
        g: undefined
      },
      outer: <GlobalLexicalEnvironment>,
      ThisBinding: <Global Object or undefined>
    }
}
```

그 다음, 실행 컨텍스트는 실행 단계를 거쳐 아래와 같이 변수 할당이 완수됩니다. 따라서, 함수 실행 컨텍스트는 실행 단계를 거쳐 다음과 같은 형태가 됩니다.

```
FunctionExectionContext = {
  LexicalEnvironment: {
      EnvironmentRecord: {
        Type: "Declarative",
        // Identifier bindings go here
        Arguments: {0: 20, 1: 30, length: 2},
      },
      outer: <GlobalLexicalEnvironment>,
      ThisBinding: <Global Object or undefined>,
    },
  VariableEnvironment: {
      EnvironmentRecord: {
        Type: "Declarative",
        // Identifier bindings go here
        g: 20
      },
      outer: <GlobalLexicalEnvironment>,
      ThisBinding: <Global Object or undefined>
    }
}
```

이후 함수의 실행이 완료되면, 반환 값이 `c`에 보관됩니다. 따라서 전역 렉시컬 환경이 갱신됩니다. 이후, 전역 코드가 모두 완료되고, 프로그램은 종료됩니다.

**참고** - 아마 위의 과정을 따라가면서, 최초에 `let`과 `const`에는 아무런 값도 할당되지 않은 것을 확인했을 것입니다. 반면에 `var`에는 `undefined`로 값이 할당됩니다.

이런 일이 발생하는 이유는, 생성 단계를 통해 코드의 변수 및 함수 선언에 대해 스캔하는 동안, 함수 선언은 환경에 전체적으로 저장되는 반면, 각 변수는 `undefined`로 설정되거나(`var`의 경우), 초기화되지 않은 상태(initialized)로 설정되기 때문입니다.(`let` 또는 `const`의 경우)

이것이 `var`를 사용할 때, `var`가 선언되기도 전에 상단에서 접근할 수 있게되는 이유입니다. 반면에 `let` 또는 `const`의 경우 이들이 선언되기 전에는 참조 에러를 발생시킬 것입니다.

그리고, `var`에 나타나는 이러한 현상을 우리는 **호이스팅**이라고 합니다.

**참고** - 실행 단계에서, JS 엔진이 `let` 키워드로 선언된 변수의 값을 찾지 못하는 경우, `undefined`로 이를 할당합니다.
