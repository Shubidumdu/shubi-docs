[여기](https://graphql-kr.github.io/learn/schema/) 문서에 따라, GraphQL 스키마 및 타입에 관해 직접 실습해보려고 한다.

실습을 위해 Typescript 기반으로 Apollo를 이용한 임의의 GraphQL 서버를 생성했다.

# Schema & Type

다음과 같은 쿼리를 받았다고 생각해보자.

```graphql
# query
{
  game {
    title
    genre
    tags
  }
}
```

```json
// result
// 임의로 작성됨
{
  "data": {
    "title": "Super Mario Bros",
    "genre": "adventure",
    "developer": "Nintendo",
    "publisher": "Nintendo",
    "tags": ["2d", "famicom"]
  }
}
```

기본적으로 GraphQL 쿼리의 형태가 결과와 거의 일치하기 때문에, 서버에 대해 모르는 상태에서도 쿼리가 어떤 형태의 값을 반활할지에 대해 어느 정도 예측할 수 있다.

하지만, 어떤 필드를 선택할 수 있는지, 어떤 종류의 객체를 반환할 수 있는지, 하위 객체에서 사용할 수 있는 필드가 무엇인지 등에 대한 정보를 얻기 위해 스키마가 필요하다.

모든 GraphQL 서비스는 해당 서비스에서 쿼리 가능한 데이터들을 완벽하게 설명하는 타입들을 정의하고, 쿼리가 들어오면 해당 스키마에 대한 유효성이 검사된 후에 실행된다.

## Type language

GraphQL은 어떤 언어로든 작성될 수 있으며, 해당 문서에서는 TypeScript에 기반하여 내용을 따라갈 예정이다.

## Object types and fields

GraphQL 스키마의 가장 기본적인 구성 요소는 객체 타입으로, 이는 서비스에서 가져올 수 있는 객체 종류와 그 객체의 필드를 나타낸다.

```graphql
type Game {
  title: String!
  developer: Developer!
  tags: [Tag]!
}

type Developer {
  name: String!
  games: [Game]!
}
```

- `Game`, `Developer` 등은 **GraphQL Object 타입**이다. 다시 말해, 필드가 존재하는 타입이란 의미이며, 스키마에서 대부분의 타입은 여기에 해당한다.
- `title`, `name` 등은 `Character` 타입 내에 존재하는 **Field**이다. 즉, 쿼리에서 `title`는 `Game` 타입 내에서, `name`은 `Developer` 타입 내에서 어디서든 사용할 수 있는 필드이다.
- `String`은 내장된 스칼라(scalar) 타입 중 하나다. 이는 단일 스칼라 객체로 해석된다.
- `String!`은 필드가 non-nullable함을 의미한다. 즉, 해당 필드를 쿼리하는 경우 항상 GraphQL 서비스는 해당 값을 반환한다는 것을 의미한다.
- `[Game]!`은 `Game` 객체의 배열을 나타내는데, 이 또한 non-nullable하여 무조건 배열을 반환함을 의미한다. (배열 자체는 길이가 0이어도 상관이 없다.)

## Arguments(인자)

GraphQL 객체 타입의 모든 필드는 0개 이상의 인수를 가질 수 있다.

```graphql
type Game {
  title(language: Language = KOREAN): String!
  developer: Developer!
  tags: [Tag]!
}
```

모든 인자에는 이름이 있다. 위의 예시에서는 `title` 필드가 `language`라는 매개변수를 갖는다.

인자는 required일수도, optional할수도 있다. 인자가 optional인 경우 기본값을 정의할 수 있으며, 이에 대해서는 쿼리에 관한 문서에서도 설명한 바가 있다.

## Query Type & Mutation Type

스키마 대부분의 타입은 일반 객체 타입이지만, 두 가지 특수한 타입이 존재한다.

```
schema {
  query: Query
  mutation: Mutation
}
```

모든 GraphQL 서비스는 `query` 타입을 가지며, `mutation` 타입은 가질 수도, 가지지 않을 수도 있다. 전반적인 취급은 동일하지만, 모든 GraphQL 쿼리의 진입점(**Entry Point**)를 정의하는 것이므로 이는 특별하다.

## Scalar Type

GraphQL 객체 타입은 이름과 필드를 가지지만, 결국 그 끝에는 구체적인 데이터로 해석되어야 하는데, 이것이 스칼라 타입이 필요한 이유다.

쿼리를 요청할 때, 필드에 하위 필드가 존재하지 않는 경우 그것이 스칼라 타입임을 알 수 있다. 기본적으로 존재하는 스칼라 타입에는 다음과 같은 것들이 있다.

- `Int` : 부호가 있는(Signed) 32비트 정수
- `Float` : 부호가 있는 부동소수점(double-precision floating-point) 값
- `String` : UTF-8 문자열
- `Boolean` : `true` 또는 `false`
- `ID` : ID 스칼라 타입은 객체를 다시 요청하거나 캐시 키로써 종종 사용되는 고유 식별자다. String과 같은 형태로 Serialized 되지만, `ID`로 정의하는 것은 사람들이 읽기 위한 용도가 아님을 의미한다.

별도로 커스텀 스칼라 타입을 지정할 수도 있는데, 이는 어떤 언어와 라이브러리를 활용하느냐에 따라 조금씩 다른 형태가 될 것이다. 아래는 JS와 apollo를 활용한 기준.

```js
const { ApolloServer, gql } = require('apollo-server');
const { GraphQLScalarType, Kind } = require('graphql');

const typeDefs = gql`
  scalar Date

  type Event {
    id: ID!
    date: Date!
  }

  type Query {
    events: [Event!]
  }
`;

const dateScalar = new GraphQLScalarType({
  name: 'Date',
  description: 'Date custom scalar type',
  serialize(value) {
    return value.getTime(); // Convert outgoing Date to integer for JSON
  },
  parseValue(value) {
    return new Date(value); // Convert incoming integer to Date
  },
  parseLiteral(ast) {
    if (ast.kind === Kind.INT) {
      return new Date(parseInt(ast.value, 10)); // Convert hard-coded AST string to integer and then to Date
    }
    return null; // Invalid hard-coded value (not an integer)
  },
});

const resolvers = {
  Date: dateScalar,
  // ...other resolver definitions...
};

const server = new ApolloServer({
  typeDefs,
  resolvers,
});
```

## Enum Type

Enum 타입은 특정 값들로 제한되는 특별한 종류의 스칼라다.

```graphql
enum Genre {
  action
  puzzle
  adventure
}
```

위의 예시에서 `Genre` 타입을 사용하면, 이는 정확히 `action`, `puzzle`, `adventure` 중에 하나일 것임을 보장한다.

## Lists & Non-Null

object, scalar, enum 타입은 GraphQL에서 정의할 수 있는 타입의 전부다.

하지만, 스키마의 다른 부분이나 쿼리 변수 선언에서 타입을 사용해 해당 값의 유효성 검사를 할 수 있는 타입 수정자를 적용할 수 있다.

```graphql
type Game {
  title: String!
  tags: [String]!
}
```

`String`타입의 뒤에 느낌표 `!`를 추가해 Non-Null임을 나타냈다. 이제 서버는 해당 필드에 대해 항상 `null`이 아닐 것이라 기대하며, 만약 `null`이 반환되면 오류를 발생시킨다.

Non-null 타입 수정자는 매개 변수를 정의할 때도 사용할 수 있는데, 이를 쿼리 시에 충족시키지 않는 경우 유효성 검사 오류를 반환하게끔 한다.

```graphql
# query
query GameInfo($id: ID!) {
  game(id: $id) {
    title
    tags
  }
}
```

```json
// variables
{
  // 아무것도 넘기지 않는다.
}
```

```json
// result
// variables에 아무것도 넘기지 않아 에러가 발생.
{
  "errors": [
    {
      "message": "Variable \"$id\" of required type \"ID!\" was not provided.",
      "locations": [
        {
          "line": 1,
          "column": 17
        }
      ]
    }
  ]
}
```

## Interface

여러가지 타입 시스템과 마찬가지로 GraphQL 역시 인터페이스를 지원한다.

예를 들면, 다음과 같은 식으로 작성할 수 있다.

```graphql
interface Game {
  id: ID!
  title: String!
  tags: [Tag]!
}

type ActionGame implements Game {
  id: ID!
  title: String!
  tags: [Tag]!
  genre: ActionGenre!
}

enum ActionGenre {
  Fighting
  Platformer
  ARPG
}
```

이런 식으로 `Game`을 implement하는 모든 타입이 해당 인자와 리턴 타입을 가진 정확한 필드를 가져야함을 명시해줄 수 있다.

만약, 앞선 쿼리의 형태에서 특정 타입에만 존재하는 필드를 가져오고자 하는 경우, 단순히 아래와 같은 형태는 에러가 발생한다.

```graphql
query GameInfo($id: ID!) {
  game(id: $id) {
    title
    genre # ActionGame 타입에만 존재
  }
}
```

```json
{
  "id": "id1234"
}
```

```json
{
  "errors": [
    {
      "message": "Cannot query field \"genre\" on type \"Game\". Did you mean to use an inline fragment on \"ActionGame\"?",
      "locations": [
        {
          "line": 4,
          "column": 5
        }
      ]
    }
  ]
}
```

이러한 경우에 아래와 같은 형태로 인라인 프래그먼트를 사용하여 특정 객체 타입일 경우의 필드를 요청할 수 있다.

```graphql
query GameInfo($id: ID!) {
  game(id: $id) {
    title
    ... on ActionGame {
      genre
    }
  }
}
```

```json
// variables
{
  "id": "id1234"
}
```

```json
// result
// 임의로 작성됨
{
  "data": {
    "title": "Super Mario Bros",
    "genre": "Platformer"
  }
}
```

## Union Type

유니온 타입은 인터페이스와 유사하지만, 타입 간의 공통 필드를 정의하지 않는다는 차이점이 있다.

```graphql
union SearchResult = ActionGame | PuzzleGame
```

이런 경우, SearchResult의 결과가 어떤 타입이더라도 쿼리할 수 있도록 조건부 프래그먼트를 사용해야 한다.

```graphql
search(text: "ma") {
  ... on ActionGame {
    # ActionGame 타입에서 존재하는 필드
  }
  ... on PuzzleGame {
    # PuzzleGame 타입에서 존재하는 필드
  }
}
```

## Input Type

지금껏 매개변수에 전달하는 인자가 간단한 스칼라 값인 경우에 대해서만 나타냈는데, 좀 더 복잡한 객체도 쉽게 전달할 수 있다. 이는 뮤테이션 타입에서 특히 유용하다.

이 때 활용하는 타입이 **Input Type**이며, 일반 객체 타입과 완전히 동일하지만, `type` 대신에 `input`을 사용한다는 차이점이 있다.

```graphql
input ReviewInput {
  stars: Int!
  comment: String
}
```

```graphql
# mutation
mutation CreateReviewForGame($gameId: ID!, $review: ReviewInput!) {
  createReview(gameId: $gameId, review: $review) {
    stars
    commentary
  }
}
```

```json
// variables
{
  "gameId": "mario1234",
  "review": {
    "stars": 5,
    "comment": "The begin of legend. :)"
  }
}
```

```json
// result
{
  "data": {
    "createReview": {
      "stars": 5,
      "commentary": "The begin of legend. :)"
    }
  }
}
```
