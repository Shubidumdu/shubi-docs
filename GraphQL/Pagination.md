[여기](https://graphql-kr.github.io/learn/pagination/)의 내용을 Github GraphQL API로 따라가보자.

# Pagination

## Plurals

여러 개의 객체를 가져오기 위한 가장 간단한 방법은 Plurals(복수형) 타입을 반환하는 필드를 사용하는 것이다.

```graphql
licenses {
  name
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
      {
        "name": "BSD 3-Clause \"New\" or \"Revised\" License"
      },
      {
        "name": "Boost Software License 1.0"
      },
      {
        "name": "Creative Commons Zero v1.0 Universal"
      },
      {
        "name": "Eclipse Public License 2.0"
      },
      {
        "name": "GNU General Public License v2.0"
      },
      {
        "name": "GNU General Public License v3.0"
      },
      {
        "name": "GNU Lesser General Public License v2.1"
      },
      {
        "name": "MIT License"
      },
      {
        "name": "Mozilla Public License 2.0"
      },
      {
        "name": "The Unlicense"
      }
    ]
  }
}
```

## Slicing

헌데, 여기에 클라이언트가 가장 앞의 둘, 혹은 가장 뒤의 둘과 같은 식으로 Slicing을 원한다면, 아래와 같은 형태가 이루어질 수 있다.

```graphql
{
  search(query: "react", type: REPOSITORY, first: 2) {
    nodes {
      ... on Repository {
        name
        owner {
          ... on User {
            name
          }
          ... on Organization {
            name
          }
        }
      }
    }
  }
}
```

```json
// result
{
  "data": {
    "search": {
      "nodes": [
        {
          "name": "react",
          "owner": {
            "name": "Facebook"
          }
        },
        {
          "name": "react",
          "owner": {
            "name": "TypeScript Cheatsheets"
          }
        }
      ]
    }
  }
}
```

## Pagination and Edges

페이지네이션을 할 수 있는 방법은 여러 가지가 있다.

- `field(first: 2, offset: 2)` : 리스트로 다음 두 개를 요청

- `field(first: 2, after: $id)` : 앞서 가져온 마지막 item의 id값을 통해 그 다음 두개를 요청

- `field(first: 2, after: $fieldCursor)` : 마지막 항목으로부터 커서를 가져와 사용

이 중 가장 기능이 강력한 것은 마지막의 **커서 기반 페이지네이션(cursor-based pagination)**이며, 커서를 사용하면 향후 페이지네이션 모델이 변경될 경우에 추가적인 유연성이 제공된다.

다만, 또 여기서 문제가 발생하는데, 객체에서 어떻게 커서를 가져오느냐 하는 것이다.

기본적으로, 커서는 연결(`connection`)을 위한 필드이므로 이것이 객체 속성에 포함되는 것은 부적절해보인다.

때문에 `edge`라고 하는 별도의 필드를 가지며, 이는 객체와 관련된 정보가 아닌 엣지와 관련된 자체 정보가 있는 경우에 유용하다.

```graphql
# query
{
  search(query: "react", type: REPOSITORY, first: 3) {
    edges {
      cursor
    }
    nodes {
      ... on Repository {
        id
        name
        owner {
          id
          ... on User {
            name
          }
          ... on Organization {
            name
          }
        }
      }
    }
  }
}
```

```json
// result
{
  "data": {
    "search": {
      "edges": [
        {
          "cursor": "Y3Vyc29yOjE="
        },
        {
          "cursor": "Y3Vyc29yOjI="
        },
        {
          "cursor": "Y3Vyc29yOjM="
        }
      ],
      "nodes": [
        {
          "id": "MDEwOlJlcG9zaXRvcnkxMDI3MDI1MA==",
          "name": "react",
          "owner": {
            "id": "MDEyOk9yZ2FuaXphdGlvbjY5NjMx",
            "name": "Facebook"
          }
        },
        {
          "id": "MDEwOlJlcG9zaXRvcnkxMzU3ODYwOTM=",
          "name": "react",
          "owner": {
            "id": "MDEyOk9yZ2FuaXphdGlvbjUwMTg4MjY0",
            "name": "TypeScript Cheatsheets"
          }
        },
        {
          "id": "MDEwOlJlcG9zaXRvcnk3NTM5NjU3NQ==",
          "name": "react",
          "owner": {
            "id": "MDQ6VXNlcjMyNDk2NTM=",
            "name": "肚皮"
          }
        }
      ]
    }
  }
}
```

## End-of-list, counts, and Connections

그렇다면 이런 식으로 pagination을 반복하다가 언제 `connection`이 끝났는지를 알 수 있을까?? 또한, 총 몇 개의 item이 존재하는지 어떻게 알 수 있을까??

이를 위해 필드는 `connection` 객체를 반환할 수 있다.

`connection` 객체에는 엣지에 대한 필드 뿐만 아니라 다른 정보(ex. item 갯수, 다음 페이지 존재 여부)등을 담고 있다.

이를 활용한다면, 다음과 같은 형태로 이용할 수 있다.

```graphql
{
  search(query: "react", type: REPOSITORY, first: 3) {
    nodes {
      ... on Repository {
        id
        name
        owner {
          id
          ... on User {
            name
          }
          ... on Organization {
            name
          }
        }
      }
    }
    repositoryCount
    pageInfo {
      startCursor
      endCursor
      hasPreviousPage
      hasNextPage
    }
  }
}
```

```json
// result
{
  "data": {
    "search": {
      "nodes": [
        {
          "id": "MDEwOlJlcG9zaXRvcnkxMDI3MDI1MA==",
          "name": "react",
          "owner": {
            "id": "MDEyOk9yZ2FuaXphdGlvbjY5NjMx",
            "name": "Facebook"
          }
        },
        {
          "id": "MDEwOlJlcG9zaXRvcnkxMzU3ODYwOTM=",
          "name": "react",
          "owner": {
            "id": "MDEyOk9yZ2FuaXphdGlvbjUwMTg4MjY0",
            "name": "TypeScript Cheatsheets"
          }
        },
        {
          "id": "MDEwOlJlcG9zaXRvcnk3NTM5NjU3NQ==",
          "name": "react",
          "owner": {
            "id": "MDQ6VXNlcjMyNDk2NTM=",
            "name": "肚皮"
          }
        }
      ],
      "repositoryCount": 1979845,
      "pageInfo": {
        "startCursor": "Y3Vyc29yOjE=",
        "endCursor": "Y3Vyc29yOjM=",
        "hasPreviousPage": false,
        "hasNextPage": true
      }
    }
  }
}
```

`pageInfo`내의 `startCursor`, `endCursor`를 통해 페이지네이션에 필요한 커서를 얻을 수 있으며, 더 이상 `edge`를 쿼리할 필요가 없어졌다.

## Complete Connection Model

이는 별도로 `~Connection`과 같은 필드를 추가하는 방식이다.

단순히 복수 타입을 갖도록 하는 형태보다 훨씬 더 복잡하지만, 이러한 디자인을 채택함으로써 클라이언트를 위한 다양한 기능을 사용할 수 있게 된다.

- 리스트의 페이지네이션 기능
- `totalCount` 또는 `pageInfo`와 같은 연결 자체에 대한 정보를 요청하는 기능
- `cursor` 등 엣지 자체에 대한 정보를 요청하는 기능
- 백엔드 측에서 페이지네이션 방식 변경이 가능 (사용자가 불투명(`opaque`) 커서만을 사용하기 때문에)

아래는 예시.

```graphql
{
  hero {
    name
    friends {
      name
    }
    friendsConnection(first: 3) {
      totalCount
      edges {
        cursor
      }
      pageInfo {
        endCursor
        hasNextPage
      }
    }
  }
}
```

```json
// result
{
  "data": {
    "hero": {
      "name": "R2-D2",
      "friends": [
        {
          "name": "Luke Skywalker"
        },
        {
          "name": "Han Solo"
        },
        {
          "name": "Leia Organa"
        }
      ],
      "friendsConnection": {
        "totalCount": 3,
        "edges": [
          {
            "cursor": "Y3Vyc29yMQ=="
          },
          {
            "cursor": "Y3Vyc29yMg=="
          },
          {
            "cursor": "Y3Vyc29yMw=="
          }
        ],
        "pageInfo": {
          "endCursor": "Y3Vyc29yMw==",
          "hasNextPage": false
        }
      }
    }
  }
}
```
