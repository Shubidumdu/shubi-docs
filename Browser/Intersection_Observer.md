# Intersection Observer

Intersection Observer는 브라우저의 뷰포트와 요소의 교차점을 관찰하여, 요소가 현재 뷰포트 상으로 **보이는 상태인지**를 체크하는 기능을 제공한다.

대표적인 사용은 무한 스크롤, 레이지 로딩 등이다.

앞선 예처럼, 종종 `scroll` 이벤트를 대체하는 용도로 사용되는데 큰 이유는 두가지에서다. 

1. `scroll` 이벤트의 경우, 말그대로 스크롤을 하는 내내 이벤트가 발생하기 때문에, 핸들러가 무수하게 많이 호출된다. 이는 결국 불필요한 호출들을 일으키고, 이 때문에 **Debouncing**, **Throttling**과 같은 호출 제한 테크닉이 요구되어 왔다. Intersection Observer는 특정 요소가 화면에 보이는 시점에만 한번 이벤트가 동작하기 때문에, 이벤트 호출 빈도를 확실히 줄일 수 있다.

2. `scroll` 이벤트에서는 현재 높이 값을 얻기 위해 `offsetTop` 값을 확인하는데, 이를 얻기 위해선 매번 layout을 새로 그리게 된다. 이를 reflow라고 하며, 해당 과정을 반복함에 따라 렌더링 상의 성능 이슈가 발생할 수 있다.

## 사용법

기본적으로는 MutationObserver와 얼추 비슷한 듯 다르다.

```js
const observer = new IntersectionObserver(callback, options);
observer.observe(element);
```

### callback
감시 타겟이 등록되거나, 가시성(visibility)에 변화가 생기면, 옵저버는 콜백을 실행한다. 이때 해당 콜백은 2개의 인수(`entries`, `observer`)를 갖는다.


#### entries
`entries`는 `IntersectionObserverEntry` 인스턴스의 배열이며, 각각의 인스턴스는 다음 일기 전용 프로퍼티들을 포함한다.

- `boundingClientRect` : target의 사각형 정보
- `intersectionRect` : target이 보여지는(교차한) 영역의 정보
- `intersectionRatio` : 뷰포트 기준 target영역의 백분율(교차한 영역의 백분율) `0.0 ~ 1.0`
- `isIntersecting` : target이 보여진 상태(교차한 상태)에 대한 boolean
- `rootBounds` : 지정 루트 요소의 사각형 정보
- `target` : target
- `time` : 변경이 발생한 시간 정보

#### observer
`observer`는 콜백을 실행시킨 해당 옵저버 자체다.

### options

#### root
target이 보여지는지를 검사할 때, 기본 설정인 뷰포트 대신 사용할 요소(루트 요소)를 지정한다. target보다 상위 요소여야 하고, 기본값은 `null`이다.

#### rootMargin
바깥의 margin을 이용해 Root 범위를 확장하거나 축소할 수 있다. CSS margin값과 똑같은 형태로 값을 받으며, 반드시 `px` 혹은 `%`의 단위를 입력해줘야 한다.

#### threshold
옵저버가 콜백을 실행시키려면 target 요소가 어느정도의 가시성(visibility)를 가져야하는지에 대한 설정이다. `0 ~ 1` 사이의 `Number` 배열값을 받는다. 기본값으로는 `[0]`이지만, `Number` 타입의 단일 값으로도 작성할 수 있다. 배열로 값을 받는 경우, 배열 각각의 가시성에 대해서 매번 콜백을 호출한다.

## 메서드
### .observe(element)
target 요소의 감시를 시작한다.

### .unobserve(element)
target 요소의 감시를 중지한다. 애초에 감시하던 요소가 아닌 경우 아무 일도 일어나지 않는다.

기본적으로 콜백 실행 시에 두번째 인수로 `observer`자체를 가져오므로, 이를 이용해, 한번 콜백을 실행한 후에 감시를 중지하도록 할수도 있다.

```js
const observer = new IntersectionObserver((entries, observer) => {
  entries.forEach(entry => {
    if (!entry.isIntersecting) {
      return
    }

    // ...

    observer.unobserve(entry.target)
  })
}, options)
```

### .disconnect()
해당 `observer`가 감시하고 있는 **모든 요소**의 감시를 중지한다.

### .takeRecords()
이는 MutationObserver에서도 있는 메서드와 비슷한데, 도중에 작동이 중지된 경우에 처리되지 않은 IntersectionObserverEntry 객체 배열을 가져온다.