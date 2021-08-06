[여기](https://graphql-kr.github.io/learn/queries/) 문서에 따라, GraphQL의 쿼리를 직접 실습해보려고 한다.

여기서는 실습을 위해 Github의 [GraphQL API](https://docs.github.com/en/graphql/overview/explorer)를 활용했다.

## Fields

GraphQL의 핵심은 쿼리와 결과가 거의 동일한 형태를 보인다는 것이다. 덕분에 항상 클라이언트가 기대한 결과값을 얻을 수 있다.

```graphql
# query
{
  viewer {
    email
    interactionAbility {
      origin
    }
  }
}
```

```json
// result
{
  "data": {
    "viewer": {
      "email": "",
      "interactionAbility": {
        "origin": "USER"
      }
    }
  }
}
```

아래 예제에서 `licenses`는 배열을 반환하며, 배열 안 각각의 Item에 대해 `name`만을 가져온다.

쿼리문 자체는 모두 동일해보이지만, GraphQL 스키마를 기반으로 예상되는 결과를 알 수 있다.

```graphql
# query
{
  licenses {
    name
  }
}
```

```json
// result
{
  "data": {
    "licenses": [
      {
        "name": "GNU Affero General Public License v3.0"
      },
      {
        "name": "Apache License 2.0"
      },
      {
        "name": "BSD 2-Clause \"Simplified\" License"
      },
      ...
    ]
  }
}
```

## Arguments

필드에 인자를 전달할 수도 있다.

```graphql
# query
{
  user(login: "Shubidumdu") {
    name
    location
  }
}
```

```json
// result
{
  "data": {
    "user": {
      "name": "Won Gyo Seo",
      "location": "Seoul, South Korea"
    }
  }
}
```

## Aliases

만약, 여러 결과 객체 필드가 동일한 이름을 갖는 경우(위에서는 `user`), 충돌이 일어난다. 아래는 닉네임을 통해 여러 유저의 정보를 가져오는 예시인데, 아래대로라면 에러가 발생한다.

```graphql
# query
{
  user(login: "Shubidumdu") {
    name
    location
  }
  user(login: "adam-p") {
    name
    location
  }
}
```

```json
// result
// 에러 발생!
{
  "errors": [
    {
      "path": [],
      "extensions": {
        "code": "fieldConflict",
        "fieldName": "user",
        "conflicts": "{login:\"\\\"Shubidumdu\\\"\"} or {login:\"\\\"adam-p\\\"\"}"
      },
      "locations": [
        {
          "line": 2,
          "column": 2
        },
        {
          "line": 6,
          "column": 3
        }
      ],
      "message": "Field 'user' has an argument conflict: {login:\"\\\"Shubidumdu\\\"\"} or {login:\"\\\"adam-p\\\"\"}?"
    }
  ]
}
```

이러한 상황에서 Alias를 사용할 수 있다. 각각의 `user` 결과에 대해 이름을 지정해주자.

```graphql
# query
{
  me: user(login: "Shubidumdu") {
    name
    location
  }
  not_me: user(login: "adam-p") {
    name
    location
  }
}
```

```json
// result
{
  "data": {
    "me": {
      "name": "Won Gyo Seo",
      "location": "Seoul, South Korea"
    },
    "not_me": {
      "name": "Adam Pritchard",
      "location": "Toronto, Canada"
    }
  }
}
```

## Fragments

상대적으로 복잡한 페이지의 경우, Fragment라는 **재사용 가능한 단위**가 사용될 수 있다. 이를 사용하면 미리 필드셋을 구성한 다음 쿼리에 포함시킬 수 있다.

앞서 여러 유저들의 정보를 가져오는 쿼리를 Fragment를 통해 다시 만들어보면 아래와 같아진다.

```graphql
# query
{
  me: user(login: "Shubidumdu") {
    ...userInfo
  }
  not_me: user(login: "adam-p") {
    ...userInfo
  }
}

fragment userInfo on User {
  name
  location
}
```

```json
// result
{
  "data": {
    "me": {
      "name": "Won Gyo Seo",
      "location": "Seoul, South Korea"
    },
    "not_me": {
      "name": "Adam Pritchard",
      "location": "Toronto, Canada"
    }
  }
}
```

결과는 동일하지만, 쿼리 시에 일일이 객체 필드를 작성해줄 필요가 없게 되었다.

### Fragment 안에서 매개변수(variables) 사용하기

쿼리 및 뮤테이션에다 선언한 변수는 Fragment를 통해서도 접근할 수 있다.

아래는 기존의 `userInfo`에서 `$avatarSize` 변수를 통해 임의의 사이즈를 가진 avatar 이미지를 추가로 쿼리한 것이다.

```graphql
# query
query UserInfos($avatarSize: Int = 100) {
  me: user(login: "Shubidumdu") {
    ...userInfo
  }
  not_me: user(login: "adam-p") {
    ...userInfo
  }
}

fragment userInfo on User {
  name
  location
  avatarUrl(size: $avatarSize)
}
```

```json
// result
{
  "data": {
    "me": {
      "name": "Won Gyo Seo",
      "location": "Seoul, South Korea",
      "avatarUrl": "https://avatars.githubusercontent.com/u/54790378?s=100&u=9fa9c08aa2c952a873633a1baf3ea342a4c45855&v=4"
    },
    "not_me": {
      "name": "Adam Pritchard",
      "location": "Toronto, Canada",
      "avatarUrl": "https://avatars.githubusercontent.com/u/425687?s=100&v=4"
    }
  }
}
```

## Operation name (작업명)

지금껏 `query` 키워드와 이름을 모두 생략한 채 `{ ... }`와 같은 형태로 쿼리를 요청했다.

하지만 실제로 애플리케이션에 GraphQL을 적용하고자 할 때는 코드를 최대한 덜 헷갈리게 만드는 편이 좋다.

바로 위의 쿼리에서는 `UserInfos`와 같은 식으로 이름을 지정했다.

작업 타입은 `query`, `mutation`, `subscription`이 될 수 있으며, 해당 작업이 어떤 형태의 작업인지를 나타낸다.

작업명은 명시적인 작업의 **이름**인데, 디버깅 및 로깅에 있어 매우 유용하다. 임의의 쿼리 결과를 찾아내는 것보다, 직접 쿼리명을 찾아내는 것이 훨씬 쉽기 때문이다.

## Variables (변수)

지금껏 앞의 모든 예시에서 인자들은 쿼리 문자열에 함께 작성되었다. 허나, 대부분 필드에 대한 인자는 동적이다.

클라이언트 측에서는 쿼리 문자열을 런타임 시점에 동적으로 조작하고, 이를 GraphQL의 특정 포맷으로 Serialize해야 한다.

그렇기 때문에 동적 인자들을 쿼리 문자열에 직접 전달하는 것은 좋은 방법이 아니다. 그래서 GraphQL은 동적 값을 쿼리에서 없애고 이를 별도로 전달하는 방법을 제공하는데 이를 Variables(변수)라고 한다.

```graphql
{
  user(login: "Shubidumdu") {
    name
    location
  }
}
```

위의 쿼리를 Variables를 활용한 형태로 바꾸려면 다음과 같은 작업들이 필요하다.

1. 쿼리 내의 정적인 값을 `$variableName` 형태로 변경한다.
2. `$variableName`를 쿼리에서 받아오는 변수의 타입으로 선언한다.
3. 별도의 전송규약(일반적으로 JSON) 변수에 `variableName: value`를 전달한다.

변수를 이용해 위의 쿼리를 재작성하면 아래와 같은 형태가 된다.

```graphql
# query
query MyInfo($nickname: String!) {
  user(login: $nickname) {
    name
    location
  }
}
```

```json
// variables
{ "nickname": "Shubidumdu" }
```

```json
// result
{
  "data": {
    "user": {
      "name": "Won Gyo Seo",
      "location": "Seoul, South Korea"
    }
  }
}
```

이제, 클라이언트 측에서는 완전히 새로운 쿼리를 작성하지 않고 손쉽게 다른 변수를 전달할 수 있다.

한편, 이런 방식은 쿼리의 어떤 Argument가 동적인 형태를 띠는지 나타내는 좋은 방법이기도 하다.

### 변수 정의

변수 정의는 위 예시 쿼리에서 `($nickname: String!)`에 해당하는 부분이다. 정적타입 언어의 함수에 대한 인자 정의와 동일하다.

**모든 변수는 scalars, enum, 또는 input object type 이어야 한다.** 복잡한 객체를 필드에 전달하려면 서버에서 일치하는 입력 타입을 알아야 하며, 이에 대해서는 문서를 통해 더 알아보자.

변수 정의는 required 혹은 optional일 수 있다. 위에서는 `String!`으로 `!`가 붙었으므로 required scalar type에 해당한다. 반대로, `!`가 붙지 않았다면 이는 optional한 값이 된다.

### 변수 기본값

타입 선언 다음에 기본값을 할당할 수도 있다.
이 경우에는 별도로 Variable을 전달하지 않더라도 올바르게 동작한다.

```graphql
# query
query MyInfo($nickname: String = "Shubidumdu") {
  user(login: $nickname) {
    name
    location
  }
}
```

**여기에, `$nickname: String!`과 같이 required 변수를 요구하는 경우에는 기본값을 가질 수 없다는 점을 유의하자.**

## Directives (지시어)

Directives는 GraphQL의 기능으로, 필드나 프래그먼트 안에 삽입되어, 쿼리 실행에 영향을 줄 수 있다.

- `@include(if: Boolean)`: 인자가 `true`인 경우에만 이 필드를 결과에 포함한다.
- `@skip(if: Boolean)`: 인자가 `true`인 경우에만 이 필드를 건너뛴다.

이를 이용해 앞서 작성한 유저 정보 쿼리에서 `$withAvatar` 변수가 `true`인 경우에만 이미지를 함께 가져오게끔 해보자.

```graphql
# query
query MyInfo($nickname: String!, $withAvatar: Boolean = false) {
  user(login: $nickname) {
    name
    location
    avatarUrl @include(if: $withAvatar)
  }
}
```

```json
// variables
{
  "nickname": "Shubidumdu",
  "withAvatar": true
}
```

```json
{
  "data": {
    "user": {
      "name": "Won Gyo Seo",
      "location": "Seoul, South Korea",
      "avatarUrl": "https://avatars.githubusercontent.com/u/54790378?u=9fa9c08aa2c952a873633a1baf3ea342a4c45855&v=4"
    }
  }
}
```

물론, 필드가 객체를 참조하는 경우에도 활용할 수 있다.

```graphql
# query
query MyInfo(
  $nickname: String = "Shubidumdu"
  $withItemShowcase: Boolean = false
) {
  user(login: $nickname) {
    name
    location
    company
    itemShowcase @include(if: $withItemShowcase) {
      items {
        totalCount
      }
    }
  }
}
```

```json
// variables
{
  "withItemShowcase": true
}
```

```json
// result
{
  "data": {
    "user": {
      "name": "Won Gyo Seo",
      "location": "Seoul, South Korea",
      "company": "The Mong, Inc.",
      "itemShowcase": {
        "items": {
          "totalCount": 6
        }
      }
    }
  }
}
```

## Mutation

지금까지는 전부 데이터 가져오기(`fetch`)에만 초점을 뒀다.

REST의 경우, 사실 상 모든 요청이 사이드 이펙트를 일으킬 수 있지만, 데이터 수정에 있어서는 `GET`을 사용하지 않는다는 규칙이 정해져 있다.

이는 GraphQL 역시 마찬가지다. 기술적으로는 어떤 형태의 쿼리든 데이터에 수정을 가할 수 있으나, 사이드 이펙트를 유발하는 작업의 경우에는 Mutation을 통해 전송되어야 한다는 규칙이 있다.

아래는 내 `shubi-docs` repo에 star를 추가하는 예시 Mutation이다.

```graphql
# mutation
mutation MyMutation($repoId: ID!) {
  __typename
  addStar(input: { starrableId: $repoId, clientMutationId: "Star added!" }) {
    clientMutationId
  }
}
```

```json
// variables
{ "repoId": "MDEwOlJlcG9zaXRvcnkzMDYzNjgwMDY" }
```

```json
// result
// 실제로 repo에 star가 추가된다.
{
  "data": {
    "__typename": "Mutation",
    "addStar": {
      "clientMutationId": "Star added!"
    }
  }
}
```

### 다중 필드 Mutation

Mutation은 쿼리와 마찬가지로 여러 필드를 포함할 수 있는데, 둘 사이에 중요한 차이점이 있다.

**쿼리 필드는 병렬로 실행되지만 뮤테이션 필드는 하나씩 차례대로 실행된다**는 점이다.

덕분에, 아래와 같이 여러 개의 뮤테이션을 요청하면, 순서가 보장되기 때문에 결국 추가한 star는 다시 사라진다.

```graphql
mutation MyMutation($repoId: ID!) {
  __typename
  addStar(input: { starrableId: $repoId, clientMutationId: "Star added!" }) {
    clientMutationId
  }
  removeStar(
    input: { starrableId: $repoId, clientMutationId: "Star removed!" }
  ) {
    clientMutationId
  }
}
```

```json
// variables
{ "repoId": "MDEwOlJlcG9zaXRvcnkzMDYzNjgwMDY" }
```

```json
// result
{
  "data": {
    "__typename": "Mutation",
    "addStar": {
      "clientMutationId": "Star added!"
    },
    "removeStar": {
      "clientMutationId": "Star removed!"
    }
  }
}
```

## Inline Fragments

다른 여러 타입과 마찬가지로 GraphQL 스키마에는 인터페이스와 유니온 타입을 정의하는 기능이 포함되어 있다.

만약, 인터페이스나 유니언 타입을 반환하는 필드를 쿼리하는 경우, Inline Fragement를 사용할 수 있는데, 다음과 같은 형태다.

```graphql
# query
{
  node(id: "MDQ6VXNlcjU0NzkwMzc4") {
    id
    ... on User {
      name
    }
    ... on Organization {
      email
    }
  }
}
```

```json
// result
{
  "data": {
    "node": {
      "id": "MDQ6VXNlcjU0NzkwMzc4",
      "name": "Won Gyo Seo"
    }
  }
}
```

위의 인자로 입력한 `id`를 통해 반환되는 값은 Node이자 User 타입이다.

User를 반환받는 경우, `id`와 `name` 필드를 가져오도록 Inline Fragment (`... on User`)를 활용했기 때문에, `... on Organization {...}`은 완전히 무시된다.

### Meta fields

만약, GraphQL 상에서 리턴될 타입을 모르는 상황인 경우, 클라이언트에서 해당 데이터를 처리할 방법을 결정하기 위해 타입이 요구되는 경우가 있다.

GraphQL은 쿼리의 어느 지점에서건 메타 필드인 `__typename`을 요청해 그 시점에서의 객체 타입의 이름을 가져올 수 있다.

```graphql
# query
{
  node(id: "MDQ6VXNlcjU0NzkwMzc4") {
    __typename
    id
    ... on User {
      name
    }
    ... on Organization {
      email
    }
  }
}
```

```json
// result
{
  "data": {
    "node": {
      "__typename": "User",
      "id": "MDQ6VXNlcjU0NzkwMzc4",
      "name": "Won Gyo Seo"
    }
  }
}
```

위 쿼리에서 `__typename`을 추가해 클라이언트 측에서 타입을 구분할 수 있게끔 해주었다.

GraphQL은 이 외에도 몇 가지 메타필드를 제공하며, 이들은 **introspection**의 일부다. 이에 대해서는 다른 문서를 통해 설명하겠다.
