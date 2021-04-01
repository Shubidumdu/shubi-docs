# Serving Over HTTP

[여기](https://graphql-kr.github.io/learn/serving-over-http/)의 내용을 따라가보자.

## URIs, Routes

HTTP는 일반적으로 리소스를 핵심 개념으로 여기는 REST와 관련이 있다.

이와 반대로, GraphQL의 개념 모델은 엔티티 그래프로, 이는 URL로 식별되지 않는다.

GraphQL 서버는 단일 엔드포인트(일반적으로 `/graphql`)에서 작동하며, 주어진 서비스에 대한 모든 요청은 해당 엔드포인트에서 수행된다.

## HTTP Methods, Headers, and Body

GraphQL HTTP 서버는 HTTP GET / POST 메서드를 처리해야 한다.

### GET 요청

만약 다음과 같은 GraphQL 쿼리를 실행하려고 한다면,

```graphql
{
  me {
    name
  }
}
```

다음과 같이 HTTP GET을 통해 전송할 수 있다.

```
/graphql?query={me{name}}
```

여기에 더해 다음과 같은 추가 쿼리 파라미터를 가질 수 있다.

- `variables` : 쿼리 변수들을 넘기는 객체를 JSON Stringified 처리한 문자열
- `operationName` : 쿼리에 여러 개의 명명된 작업이 포함된 경우에, 어떤 쿼리를 실행하는지 제어

즉, `variables`과 함께 좀 더 복잡한 쿼리를 전달해보자면, 가령 아래와 같은 쿼리가 있다고 할 때,

```graphql
query gameInfo($id: ID!) {
  game(id: $id) {
    title
  }
}
```

```json
// variables
{
  "id": "1"
}
```

```json
// result
{
  "data": {
    "game": {
      "title": "Super Mario Bros"
    }
  }
}
```

이를 (굳이) HTTP GET 메서드로 요청해보겠다고 하면 아래와 같아진다. 만약 나머지 특수문자들도 인코딩한다면 훨씬 지저분해질 것이다.

```
/graphql?variables={"id":"1"}&query=query%20gameInfo($id:ID!){game(id:$id){title}}
```

`operationName`의 경우, 앞서 말했듯 여러 개의 쿼리 작업을 보유한 경우, 실행하길 원하는 작업명을 의미한다.

이를테면 아래와 같이 사용한다. 다음과 같은 쿼리가 있다고 하자.

```graphql
query query1 {
  game(id: "1") {
    title
  }
}

query query2 {
  game(id: "2") {
    title
  }
}
```

여기서, (굳이 또) HTTP GET 메서드로 `query2`에 해당하는 작업을 요구하려는 경우에는 다음과 같이 할 수 있다.

```
/graphql?operationName=query2&query=query%20query1{game(id:"1"){title}}%20query%20query2{game(id:"2"){title}}
```

### POST 요청

표준 GraphQL POST 요청은 `application/json` content-type을 사용해야하며, 아래 형식의 JSON 인코딩 처리된 Body를 포함해야한다.

```json
{
  "query": "...",
  "operationName": "...",
  "variables": { "myVariable": "someValue", ... }
}
```

`operationName`과 `variables`는 앞선 GET 메서드의 경우와 똑같은 역할을 한다.

위 내용 외에도 추가로 다음 두 가지 경우에 대해 지원하는 것이 좋다.

- 위의 GET 요청과 같은 방식으로 쿼리스트링 파라미터가 존재하는 경우, HTTP GET의 경우와 동일한 형식으로 처리
- `application/graphql` Content-Type header가 있는 경우, HTTP POST body의 내용을 GraphQL 쿼리스트링으로 처리 (Body에 넘겨진 텍스트 자체를 쿼리문으로 여겨야 한다는 듯)

### Response

쿼리와 변수가 전송된 방식과는 관계없이, 응답은 Body에 JSON 형태로 반환되어야 한다.
쿼리는 데이터 뿐만 아니라 오류 또한 유발할 수 있기 때문에, 다음과 같은 형태로 반환되어야 한다.

```
{
  "data": { ... },
  "errors": [ ... ]
}
```

오류가 없는 경우에는 `errors` 필드가 없어야 한다.
반면 데이터가 반환되지 않는 경우에는 실행 도중에 에러가 발생한 경우에 대해서만 `data` 필드가 포함된다.

### GraphiQL

GraphiQL이나 GraphQL Playground는 테스트 및 개발 중에 유용하게 쓰일 수 있지만, 기본적으로 프로덕션 환경에서는 사용하지 않도록 되어야 한다.

`express-graphql`의 경우는 다음과 같이 이를 구현할 수 있다.

```js
app.use(
  '/graphql',
  graphqlHTTP({
    schema: MySessionAwareGraphQLSchema,
    graphiql: process.env.NODE_ENV === 'development',
  }),
);
```
