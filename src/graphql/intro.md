# Introspection

[여기](https://graphql-kr.github.io/learn/introspection/)의 내용을 Github Graphql API로 따라가보자.

타입 시스템을 사용하기 때문에, 우리는 현재 유효한 타입이 무엇인지 알 수 있으나, 그렇지 않은 경우 Query의 루트에서 사용할 수 있는 `__schema` 필드를 쿼리하여 GraphQL에 요청할 수 있다.

```graphql
# query
{
  __schema {
    types {
      name
    }
  }
}
```

```json
// result
{
  "data": {
    "__schema": {
      "types": [
        {
          "name": "AcceptEnterpriseAdministratorInvitationInput"
        },
        {
          "name": "AcceptEnterpriseAdministratorInvitationPayload"
        },
        {
          "name": "AcceptTopicSuggestionInput"
        },
        // ...
        {
          "name": "__Schema"
        },
        {
          "name": "__Type"
        },
        {
          "name": "__TypeKind"
        }
      ]
    }
  }
}
```

직접 해보면 알겠지만, 엄청 많이 뜬다. 이를 몇개로 그룹화해볼 수 있다.

- `Query`, `User` 등 : 타입 시스템을 통해 정의한 것
- `String`, `Boolean` 등 : 타입 시스템이 제공하는 내장 스칼라
- `__Schema`, `__Type` 등 : 이들 앞에는 `__`가 붙어있는데, 이는 이것이 Introspection 시스템의 일부임을 나타낸다.

```graphql
# query
query {
  __schema {
    queryType {
      name
    }
  }
}
```

```json
// result
{
  "data": {
    "__schema": {
      "queryType": {
        "name": "Query"
      }
    }
  }
}
```

최상단의 `Query` 타입에서 위와 같이 요청하면, 다음과 같이 `queryType`을 통해 우리가 `__schema`를 요청한 지점이 `Query` 타입에 해당함을 확인할 수 있다.

보통은 특정 타입 내에서 검사하는 작업이 유용한 경우가 많으며, 아래에서 `User` 타입에 대해 살펴보자.

```graphql
query {
  __type(name: "User") {
    name
    description
    kind
  }
}
```

```json
// result
{
  "data": {
    "__type": {
      "name": "User",
      "description": "A user is an individual's account on GitHub that owns repositories and can make new content.",
      "kind": "OBJECT"
    }
  }
}
```

위와 같은 식으로 특정 타입(위에서는 `User`)에 대한 상세한 정보를 얻을 수 있다. 여기에 더 나아가 해당 타입이 보유한 필드들에 어떤 것들이 있는지 찾아보자.

```graphql
query {
  __type(name: "User") {
    name
    description
    kind
    fields {
      name
      type {
        name
        kind
        ofType {
          name
          kind
        }
      }
    }
  }
}
```

```json
{
  "data": {
    "__type": {
      "name": "User",
      "description": "A user is an individual's account on GitHub that owns repositories and can make new content.",
      "kind": "OBJECT",
      "fields": [
        {
          "name": "anyPinnableItems",
          "type": {
            "name": null,
            "kind": "NON_NULL",
            "ofType": {
              "name": "Boolean",
              "kind": "SCALAR"
            }
          }
        },
        {
          "name": "avatarUrl",
          "type": {
            "name": null,
            "kind": "NON_NULL",
            "ofType": {
              "name": "URI",
              "kind": "SCALAR"
            }
          }
        },
        {
          "name": "bio",
          "type": {
            "name": "String",
            "kind": "SCALAR",
            "ofType": null
          }
        },
        // ...
        {
          "name": "watching",
          "type": {
            "name": null,
            "kind": "NON_NULL",
            "ofType": {
              "name": "RepositoryConnection",
              "kind": "OBJECT"
            }
          }
        },
        {
          "name": "websiteUrl",
          "type": {
            "name": "URI",
            "kind": "SCALAR",
            "ofType": null
          }
        }
      ]
    }
  }
}
```

(어지럽다..)

유의할만한 내용으로는 위에서 볼수 있는 `anyPinnableItems` 와 같은 필드의 경우에는 `NON_NULL wrapper` 타입에 해당하기 때문에, 타입에 대한 이름(`name`)이 존재하지 않는다.

이 경우, 해당 필드에서 `ofType`을 쿼리해 추가로 정보를 얻어보면, 해당 타입이 `Boolean!`에 해당함을 확인할 수 있다. (아래 일부)

```json
// fields에 반환되는 내용 중 일부
{
  "name": "anyPinnableItems",
  "type": {
    "name": null,
    "kind": "NON_NULL",
    "ofType": {
      "name": "Boolean",
      "kind": "SCALAR"
    }
  }
},
```

이는 `LIST wrapper` 타입의 경우도 마찬가지이며, 아래와 같이 깊숙한 정보를 요구할 수도 있다.

```graphql
query {
  __type(name: "Repository") {
    name
    description
    kind
    fields {
      name
      type {
        name
        kind
        ofType {
          name
          kind
          ofType {
            name
            kind
          }
        }
      }
    }
  }
}
```

```json
// fields에 반환되는 내용 일부
{
  "name": "viewerPossibleCommitEmails",
  "type": {
    "name": null,
    "kind": "LIST",
    "ofType": {
      "name": null,
      "kind": "NON_NULL",
      "ofType": {
        "name": "String",
        "kind": "SCALAR"
      }
    }
  }
}
```

위의 `viewerPossibleCommitEmails`는 `[String!]`에 해당함을 확인할 수 있다.

앞서 봤듯이, Introspection 기능을 통해 타입 시스템의 문서에 접근할 수 있고, 문서 탐색기 및 풍부한 IDE 환경을 만들 수 있다.

이는 Introspection 시스템의 극히 일부에 해당하며, [여기](https://github.com/graphql/graphql-js/blob/main/src/type/introspection.js)에 GraphQL의 Introspection 시스템을 구현하는 코드가 있으니 추후에 따로 확인해보자.
