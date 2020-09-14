# Learning React + Apollo (+ [GraphQL](https://github.com/hchiam/learning-graphql))

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

- manually set the `fetchPolicy` option to `"network-first"`, to override old Apollo local cache
- "manually" update the local cache query upon mutation, to override old Apollo local cache
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

<details>
<summary>Apollo client</summary>

## Apollo client

You can directly use the Apollo client, but it gives you a promise that you have to `.then` and `.catch`:

```jsx
import { ApolloProvider } from "@apollo/react-hooks";
// ...
return <ApolloProvider client={client}>{/* ... */}</ApolloProvider>;
```

```js
// import GET_POST, GET_POSTS, CREATE_POST

client
  .query({ query: GET_POSTS, variables: { limit: 5 } })
  .then((res) => console.log(res.data))
  .catch((err) => console.error(err));

client
  .mutate({ mutation: CREATE_POST, variables: { title: "...", body: "..." } })
  .then((res) => console.log(res.data))
  .catch((err) => console.error(err));

client
  .subscribe({ subscription: GET_POST, variables: { id: "..." } })
  .then((res) => console.log(res.data))
  .catch((err) => console.error(err));
```

```js
import { gql } from "apollo-boost";
// or
import gql from "graphql-tag";

const GET_POSTS = gql`
  query GetPosts($limit: Int) {
    posts(limit: $limit) {
      id
      body
      title
      createdAt
    }
  }
`;

const CREATE_POST = gql`
  mutation CreatePost($title: String!, $body: String!) {
    insert_posts(objects: { title: $title, body: $body }) {
      affected_rows
    }
  }
`;

const GET_POST = gql`
  subscription GetPost($id: uuid!) {
    posts(where: { id: { _eq: $id } }) {
      id
      body
      title
      createdAt
    }
  }
`;

// btw, "!" means non-nullable; GraphQL types are nullable by default.
```

</details>

## Apollo convenience hooks:

<details>
<summary>useQuery</summary>

## `useQuery` = "get x1"

```jsx
const { loading, error, data } = useQuery(GET_POSTS, {
  variables: { limit: 5 },
});
// loading: boolean
// error: boolean
// data: data
// ...: (other destructured variables)

if (loading) return <div>Loading...</div>;
if (error) return <div>Error...</div>;
return data.posts.map((post) => <div key={post.id}>post.text</div>);
```

</details>

<details>
<summary>useLazyQuery</summary>

## `useLazyQuery` = "await get x1"

```jsx
const [searchPosts, { data }] = useLazyQuery(SEARCH_POSTS, {
  variables: { query: `%${query}%` },
});
// (first item in array): query function
// {...}: object that contains { loading, error, data, called, ... }
useEffect(() => {
  // ...
  searchPosts();
  if (data) setResults(data.posts);
}, [query, data, searchPosts]);
if (called && loading) return <div>Loading...</div>;
return results.map((result) => <div key={result.id}>result.text</div>);
```

</details>

<details>
<summary>useMutation</summary>

## `useMutation` = "change"

`import { useMutation } from '@apollo/react-hooks';` and then:

```js
const [createPost, { loading, error, data }] = useMutation(CREATE_POST);
// (first item in array): mutation function
// {...}: object that contains { loading, error, data, ... }
// you can use "loading" variable to debounce
```

or

```js
const [createPost, { loading, error, data }] = useMutation(CREATE_POST, {
  onCompleted: (data) => console.log(data),
  onError: (error) => console.error(error),
});
```

</details>

<details>
<summary>useSubscription</summary>

## `useSubscription` = "get whenever updates"

`import { useSubscription } from '@apollo/react-hooks';` and then:

```js
const [createPost, { loading, error, data }] = useSubscription(GET_POST, {
  variables: { id },
  shouldResubscribe: true, // run query GET_POST when props change (default is false)
  onSubscriptionData: (data) => console.log("new data", data), // when subscription hook gets new data
  fetchPolicy: "network-only", // (default is 'cache-first')
});
```

</details>

## Overriding the local Apollo cache when it's old:

(The Apollo cache is only on the client side.)

<details>
<summary>Set `fetchPolicy` option to `"network-first"`</summary>

```js
const { loading, error, data } = useQuery(GET_POSTS, {
  variables: { limit: 5 },
  fetchPolicy: "network-first", // instead of the default 'cache-first'
});
```

Options for `fetchPolicy`:

- `cache-and-network`
- `cache-first` (default)
- `cache-only`
- `network-only`
- `no-cache`
- `standby`

</details>

<details>
<summary>Update local cache query upon mutation</summary>

Notice `cache.writeQuery` inside the `update` option inside `useMutation`:

```js
function EditPost({ id }) {
  const [updatePost] = useMutation(UPDATE_POST, {
    update: (cache, data) => {
      const { posts } = cache.readQuery(GET_POSTS);
      const newPost = data.update_posts.returning;

      // old posts -> new posts:
      const updatedPosts = posts.map((post) =>
        post.id === id ? newPost : post
      );

      // update local cache query:
      cache.writeQuery({ query: GET_POSTS, data: { posts: updatedPosts } });
    },
    onCompleted: () => history.push("/"),
  });
}
```

</details>
