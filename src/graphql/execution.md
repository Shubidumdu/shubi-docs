이 [문서](https://graphql-kr.github.io/learn/execution/)의 내용을 실습하며 따라가보려고 한다. (TypeScript를 사용)

GraphQL 쿼리의 각 필드는 특정한 타입의 값을 반환하는 함수로 생각할 수 있으며, 이는 사실 실제 GraphQL의 작동방식이기도 하다.

타입의 각 필드는 GraphQL 서버 측의 `resolver` 함수에 대응되며, 해당 필드가 `string`이나 `number` 같은 스칼라 값을 반환하게 되면 실행이 완료된다.

반면, 필드가 객체를 반환하는 경우, 쿼리는 해당 객체에 적용되는 다른 필드들을 포함하게 되며, 이는 스칼라 값에 도달할 때까지 반복된다.

즉, GraphQL 쿼리의 끝은 항상 스칼라 값이어야 한다.

## Root fields & resolvers

모든 GraphQL 서버의 최상위 레벨은 GraphQL API에서 사용 가능한 모든 진입점을 나타내는 타입이며, 이는 `Root` 타입 혹은 `Query` 타입으로 불린다.

```js
// resolver
const resolvers = {
  Query: {
    game: (obj: any, { id }: GameArgs, context: Context) => {
      return context.db.games.find(({ id: gameId }) => id === gameId);
    },
    // ...
  },
  // ...
};
```

각 필드의 `resolver` 함수는 네 개의 매개변수를 받는데, 다음과 같다.

- `obj` : **부모 객체**, 위에서는 이것이 `Query` Type에 해당하므로 거의 쓰일 일이 없다.
- `args` : GraphQL 쿼리의 필드에 제공된 인수. 이를테면 `game(id: '1') {...}` 과 같은 경우에는 args가 `{ id: '1' }`이 된다.
- `context` : 모든 `resolver` 함수들에 동일하게 전달되며, 데이터베이스 접근이나 로그인 세션 등에 활용될 수 있다.
- `info` : 현재의 쿼리, 스키마 정보와 관련된 필드별 정보를 보유하며, 자세한 내용은 [여기](https://graphql.org/graphql-js/type/#graphqlobjecttype)를 참조하자.

## Async Resolvers

```js
// 임의로 작성됨
const resolver = async (obj, args, context) {
  const result = await context.db.gameInfo(args.id);
  return result.data;
};
```

위와 같이 임의로 작성된 비동기 resolver의 경우에도 정상적으로 동작한다.

하지만, 여기서는 실제 DB에 접근하지 않고 임의의 객체로 만든 Mocking DB를 활용할 것이므로, 편의상 일반적인 함수를 통해 resolver를 구현하겠다.

## Trivial resolvers

앞서 `Game` 객체에 대해 접근하는 resolver를 작성했으므로, 이제 이 `Game` 객체 내 각 필드를 구체화해보자.

```js
const resolvers = {
  Query: {
    game: (obj: any, { id }: GameArgs, context: Context) => {
      return context.db.games.find(({ id: gameId }) => id === gameId);
    },
  },
  Game: {
    // id의 resolver 첫번째 파라미터는 이제 Game 객체가 된다.
    id: (game: Game) => game.id,
    // ...
  },
};
```

아래와 같은 구성으로 타입을 지정했다고 하자.

```graphql
const typeDefs = gql`
  enum Score {
    good
    normal
    bad
  }

  type Query {
    game(id: ID!): Game
    developer(id: ID!): Developer
  }

  type Game {
    id: ID!
    title: String!
    developer: Developer!
    score: Score!
  }

  type Developer {
    id: ID!
    name: String!
    games: [Game]!
  }
`;
```

이에 대해 객체 타입에 대한 resolver 작성을 한꺼번에 해보면 이런 식이다.

```js
const resolvers = {
  Query: {
    game: (_: any, { id }: GameArgs, context: Context) => {
      return context.db.games.find(({ id: gameId }) => id === gameId);
    },
    developer: (_: any, { id }: DeveloperArgs, context: Context) => {
      return context.db.developers.find(
        ({ id: developerId }) => id === developerId,
      );
    },
  },
  Game: {
    id: (game: Game) => game.id,
    title: (game: Game) => game.title,
    developer: ({ developer: id }: Game, _: any, context: Context) => {
      return context.db.developers.find(
        ({ id: developerId }) => id === developerId,
      );
    },
    score: (game: Game) => {
      return game.score;
    },
  },
  Developer: {
    id: (developer: Developer) => developer.id,
    name: (developer: Developer) => developer.name,
    games: ({ games }: Developer, _: any, context: Context) => {
      return games.map((gameId) =>
        context.db.games.find(({ id }) => id === gameId),
      );
    },
  },
};
```
