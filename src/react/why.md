# 왜 React인가?

> [여기](https://medium.com/javascript-scene/the-missing-introduction-to-react-62837cb2fd76)의 내용을 의역 및 일부 편집한 내용입니다.

**_컴포넌트는 비즈니스 로직, 애플리케이션 상태, 네트워크와 무관하게 있을 때 가장 이상적입니다. 동일한 props가 있다면, 동일한 형태로 렌더링되어야 하죠._**

다른 프레임워크들이 MVC, MVVM 과 같은 패턴을 따를 때, React는 View에 대한 렌더링을 Model과 완전히 떼어놓으려는 시도를 했습니다. 그 노력이 바로 Flux 패턴입니다.

그렇다면 왜 이것이 MVC보다도 낫다고 여겨졌을까요?

2013년에 페이스북은 채팅 기능을 통합하기 위해 많은 노력을 기울였습니다. 애플리케이션 환경 전반에 걸쳐 라이브가 가능하고, 사이트의 모든 페이지에 통합된 기능이었죠. 이미 복잡한 애플리케이션 내에서의 새로운 복잡한 앱이었고, DOM의 비제어(uncontrolled) 변경과 더불어 수많은 이용자들의 병렬적이고 비동기적인 I/O도 페이스북 팀에게 어려운 과제였습니다.

예를 들어, 그 무엇이든지 간에 DOM을 멋대로 조작하고, 그것을 마음대로 변형할 수 있다면, 과연 적절한 화면이 렌더링된 것인지 어떻게 알 방도가 없습니다.

React 이전에는 이러한 "올바른 화면"에 대한 보장을 그 어떤 프레임워크도 할 수 없었습니다. DOM의 경쟁 상태는 이전 웹 애플리케이션의 가장 흔한 버그 중 하나였습니다.

## 비결정적 = 병행 처리 + 변형가능한 상태

React 팀이 가장 먼저 하고자 했던 것은 이러한 문제를 고치는 것이었고, 그러기 위해 두 가지 혁신이 필요했습니다.

1. Flux 구조를 이용한 단방향 데이터 바인딩
2. Immutable한 컴포넌트 상태 : 일단 설정 되고 나면, 컴포넌트의 상태는 변하지 않습니다. 상태의 변화는 현재 View의 상태를 변경하는 것이 아니라, **새로운 상태에 대한 새로운 View 렌더링을 유발합니다.**

Flux 패턴을 통해, React는 통제 불가능한 변형의 문제를 다룰 수 있었습니다. 수많은 DOM의 업데이트를 위해 수많은 이벤트 리스너를 추가하는 대신에, React는 컴포넌트의 상태 조작을 위해서 유일한 방법을 사용합니다. 바로 액션을 **Dispatch**하는 것입니다. 이를 통해 Store의 상태가 변경되면, Store는 해당 컴포넌트를 리렌더링합니다.

<img src="https://miro.medium.com/max/700/1*lNLcKqywLkrHadcA-zhgBA.png" />

그래서, "React를 왜 써야 하나요?"에 대한 대답은 심플합니다. 바로 **결정론적인(deterministic) View를 손쉽게 렌더링할 수 있기 때문**입니다.

> _주의_ : 따라서, 우리가 VanillaJS를 다루는 것 처럼, DOM에 데이터를 보관하거나 조작하는 것은 안티패턴입니다. 이 경우 React를 쓰는 의미가 없어집니다.

결정론적인 렌더링 방식은 React의 유일한 트릭이었음에도, 이는 이미 엄청난 혁신이었습니다. 그럼에도 불구하고, React는 계속해서 더 뛰어난 기능들을 선보이고 있습니다.

## JSX

JSX는 선언적인 형태로 커스텀 UI 컴포넌트를 생성할 수 있는 JS의 확장입니다. JSX는 다음과 같은 장점을 갖습니다.

- 쉽고, 선언적인 마크업
- 컴포넌트와 함께 배치
- 관심사를 구분할 수 있음 (ex. UI vs 상태 로직 vs 사이드이펙트)
- DOM의 차이를 추상화
- 내부적인 기술에 대한 추상화

단, JSX에서는 명심해야할 부분이 몇가지 있습니다.

- `class` 어트리뷰트는 JSX에서 `className`이 됩니다.
- List item 형태의 요소들은 반드시 `key` 어트리뷰트를 가져야 합니다. (여기에 대해서는 이 [문서](https://ko.reactjs.org/docs/reconciliation.html#recursing-on-children)를 읽어보세요!)

## Synthetic Events (합성 이벤트)

React에서는 DOM 이벤트에 대한 래퍼를 제공하는데, 이는 Synthetic Events라고 합니다. 이는 다음과 같은 장점을 갖습니다.

1. 이벤트 핸들링 시에 플랫폼 간의 차이를 완화해줍니다.
2. 자체적으로 메모리 관리가 자동으로 이루어집니다. 이를테면, 무한 스크롤 리스트를 만들 때, 메모리 누수를 방지하기 위해 이벤트 위임이 필요할 것입니다. 한편, Synthetic Event는 자동으로 최상단 부모 노드에 이벤트 위임을 적용하기 때문에, 이벤트 메모리 관리에 신경쓰지 않아도 됩니다.

## 컴포넌트 생명주기 (Component Lifecycle)

React의 컴포넌트 생명주기는 컴포넌트의 상태를 보호하기 위해 존재합니다. 컴포넌트 상태는 React가 컴포넌트를 그려내는 동안에는 변경되지 않아야 하기 때문입니다.

생명주기에 대한 이해는 곧 React가 동작하는 방식에 대한 이해입니다.

React의 컴포넌트 생명주기는 다음과 같이 나누어 볼 수 있습니다.

<img src="https://miro.medium.com/max/336/1*xRzCfozCPTWXp8wgnZrXiA.png" />

그리고, Update 시점은 다음과 같은 형태로 이루어집니다.

<img src="https://miro.medium.com/max/360/1*9wk48udC9l884fOZydImiw.png"/>

- **Render** - `render` 함수는 결정론적이며, 사이드이펙트를 포함해서는 안됩니다. 이것을 props를 가져와 JSX를 반환하는 순수 함수로 이해할 수 있습니다.

- **Pre-Commit** - `getSnapShotBeforeUpdate` 생명주기 메서드를 이용해 DOM으로부터 데이터를 가져올 수 있습니다. 만약 스크롤 위치나 렌더링된 요소의 크기를 파악하고자 할 때 유용합니다.

- **Commit** - DOM과 ref들에 대한 갱신을 수행합니다. `componentDidUpdate` 또는 `useEffect` 훅을 통해 이를 이용할 수 있습니다. 여기에서는 사이드 이펙트를 유발하거나 DOM을 조작해도 괜찮습니다.

다음 그림이 React 컴포넌트의 전반적인 흐름을 파악할 수 있게끔 도와 줄 겁니다.

<img src="https://miro.medium.com/max/700/1*cEWErpe-oY-_S1dOaT1NtA.jpeg" />

말했다시피, React에서의 컴포넌트는, "변형"을 가하는 것이 아니라, 상태의 변화에 따라 리렌더링 단계를 거쳐 새롭게 "대체"를 한다고 보는 것이 맞습니다. 이러한 단계를 통해 React의 "결정론적인 View 렌더링"을 쉽게 해줍니다.

다시 말해, React 컴포넌트의 대부분은 앞서 말한 것 처럼, props를 받아 JSX를 반환하는 순수함수로 생각될 수 있습니다.

## Hooks

React Hooks는 클래스형 컴포넌트가 아닌 경우에도 React 컴포넌트 생명주기를 활용하기 위한 함수들입니다. Hooks의 사용은 일반적으로 사이드 이펙트의 유발을 일으킵니다. 여기서 사이드 이펙트란 함수의 반환값 외에 발생하는 함수 외부 값의 상태 변화를 의미힙니다.

Hooks는 결국 다음과 같은 것들을 가능하게 합니다.

- 클래스형 컴포넌트가 아니더라도 함수형 컴포넌트에서 생명주기 로직을 처리할 수 있습니다.
- 코드를 더 잘 정리할 수 있습니다.
- 다른 컴포넌트 간에 재사용할 수 있는 로직을 공유할 수 있습니다.
- 스스로 임의의 커스텀 훅을 만들 수 있습니다.

## 컨테이너 vs 프레젠테이션 컴포넌트

컴포넌트의 모듈화와 더 나은 재사용성을 위해 대체로 다음의 두 형태로 컴포넌트를 구분지을 수 있습니다.

- **컨테이너 컴포넌트**는 데이터 스토어와 연결되어, 여러 사이드이펙트를 유발할 수 있습니다.
- **프레젠테이션 컴포넌트**는 _대부분_ 순수 컴포넌트이며, 동일한 컨텍스트 내 동일한 props에 대해서는 항상 동일한 JSX를 반환합니다.

프레젠테이션 컴포넌트는 다음과 같은 특징을 지닙니다.

- 네트워크와 접촉하지 않습니다.
- 로컬 스토리지에 저장 또는 불러오지 않습니다.
- 랜덤 데이터를 생성하지 않습니다.
- 현재 시스템 시간을 가져오지 않습니다.(`Date.now()`)
- 데이터 스토어에 직접 접근하지 않습니다.
- 한편, form input과 같은 로컬 컴포넌트 상태를 사용할 수는 있습니다. 물론 이 경우, 최초 상태에서부터 결정론적인 유닛 테스트가 이루어져야 합니다.

컨테이너 컴포넌트는 다음과 같은 특징을 지닙니다.

- 상태 관리, I/O, 그 외의 사이드 이펙트를 유발합니다.
- 스스로에 대한 마크업을 렌더링하지 않아야 합니다.
- 프레젠테이션 컴포넌트의 래퍼로서 사용됩니다.