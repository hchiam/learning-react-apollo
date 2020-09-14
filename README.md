# Learning React + Apollo (+ GraphQL)

Just one of the things I'm learning. <https://github.com/hchiam/learning>

My notes based off of <https://codeartistry.io/the-react-apollo-2020-cheatsheet>

## Big picture summary:

Apollo = "frontend+backend connector" = "FE+BE connector"

Apollo gives us custom hooks to do this:

    React (UI, FE) <-> Apollo <-> GraphQL (database + files, BE)

## The core Apollo hooks:

- `useQuery` = "get x1"
- `useLazyQuery` = "await get x1"
- `useMutation` = "change"
- `useSubscription` = "get whenever updates"

### You can use Apollo hooks to:

- manually set the `fetch` policy, to override old Apollo local cache
- "manually" update the cache upon mutation, to override old Apollo local cache
- refetch queries with `useQuery`'s `refetch` when `useMutation` `onCompleted`

  - (like when you want to update UI in the FE after deleting some DB data in the BE)

- access the Apollo client with `useApolloClient` or `new ApolloClient` (both give you a promise that you need to `.then` and `.catch`)

## Apollo setup:

If you only need to _"manually"_ trigger queries nad mutations in your app:

```bash
yarn add @apollo/react-hooks apollo-boost graphql
```

```js
import ApolloClient from "apollo-boost";

const client = new ApolloClient({
  uri: "https://your-graphql-endpoint.com/api/graphql",
});
```

Or if you _also_ need **_subscriptions_** to _"automatically"_ trigger queries and mutations in your app:

```bash
yarn add @apollo/react-hooks apollo-client graphql graphql-tag apollo-cache-inmemory apollo-link-ws
```

```js
import ApolloClient from "apollo-client";
import { WebSocketLink } from "apollo-link-ws";
import { InMemoryCache } from "apollo-cache-inmemory";

const client = new ApolloClient({
  link: new WebSocketLink({
    url: "https://your-graphql-endpoint.com/api/graphql",
    options: {
      reconnect: true,
      connectionParams: {
        headers: {
          Authorization: "Bearer yourauthtoken",
        },
      },
    },
  }),
  cache: new InMemoryCache(),
});
```
