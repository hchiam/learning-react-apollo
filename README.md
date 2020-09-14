# Learning React + Apollo (+ GraphQL)

Just one of the things I'm learning. <https://github.com/hchiam/learning>

My notes based off of <https://codeartistry.io/the-react-apollo-2020-cheatsheet>

Apollo = "frontend+backend connector"

Apollo gives us custom hooks to do this:

    React (UI) <-> Apollo <-> GraphQL (database + files)

Core Apollo hooks:

- `useQuery` = "get x1"
- `useLazyQuery` = "await get x1"
- `useMutation` = "change"
- `useSubscription` = "get whenever updates"

You can use Apollo hooks to:

- manually set the `fetch` policy, to override old Apollo local cache
- "manually" update the cache upon mutation, to override old Apollo local cache
- refetch queries with `useQuery`'s `refetch` when `useMutation` `onCompleted`

  - (like when you want to update UI in the FE after deleting some DB data in the BE)

- access the Apollo client with `useApolloClient` or `new ApolloClient` (both give you a promise that you need to `.then` and `.catch`)
