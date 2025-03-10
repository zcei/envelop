## `@envelop/live-query`

The easiest way of adding live queries to your GraphQL server!

Push new data to your clients automatically once the data selected by a GraphQL operation becomes stale by annotating your query operation with the `@live` directive.

```graphql
query UserProfile @live {
  me {
    id
    login
    bio
  }
}
```

The invalidation mechanism is based on GraphQL ID fields and schema coordinates. Once a query operation has been invalidated, the query is re-executed and the result is pushed to the client.

## Installation

```bash
yarn add @envelop/live-query @n1ru4l/in-memory-live-query-store
```

## Usage Example

### `makeExecutableSchema` from `graphql-tools`

```ts
import { envelop, useSchema, useExtendContext } from '@envelop/core';
import { useLiveQuery } from '@envelop/live-query';
import { InMemoryLiveQueryStore } from '@n1ru4l/in-memory-live-query-store';
import { makeExecutableSchema } from '@graphql-tools/schema';

const schema = makeExecutableSchema({
  typeDefs: [
    /* GraphQL */ `
      type Query {
        greetings: [String!]
      }
    `,
    GraphQLLiveDirectiveSDL,
  ],
  resolvers: {
    Query: {
      greetings: (_, __, context) => context.greetings,
    },
  },
});

const liveQueryStore = new InMemoryLiveQueryStore();

const greetings = ['Hello', 'Hi', 'Ay', 'Sup'];
// Shuffle greetings and invalidate queries selecting Query.greetings every second.
setInterval(() => {
  const firstElement = greetings.pop();
  greetings.unshift(firstElement);
  liveQueryStore.invalidate('Query.greetings');
}, 1000);

const getEnveloped = envelop({
  plugins: [
    useSchema(schema),
    useLiveQuery({ liveQueryStore }),
    useExtendContext(() => ({ greetings })),
    /* other plugins */
  ],
});
```

### `GraphQLSchema` from `graphql`

You need to pass the `GraphQLLiveDirective` to the list of directives:

```tsx
import { GraphQLSchema } from 'graphql';
import { GraphQLLiveDirective } from '@envelop/live-query';

const schema = new GraphQLSchema({
  directives: [...specifiedDirectives, GraphQLLiveDirective],
});
```

## Further information

This plugin requires you to use a graphql transports that supports usage of the `@defer` and `@stream` directives, as it is built upon the same concepts (return an `AsyncIterable` from `execute`).

The following transports have been successfully tested:

| Package                                                                                                                          | Transport                                                                                   | Version                                                                                                                                                                         | Downloads                                                                                                                                                                          |
| -------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [`@n1ru4l/socket-io-graphql-server`](https://github.com/n1ru4l/graphql-live-queries/blob/main/packages/socket-io-graphql-server) | GraphQL over Socket.io (WebSocket/HTTP Long Polling)                                        | [![npm version](https://badge.fury.io/js/%40n1ru4l%2Fsocket-io-graphql-server.svg)](https://github.com/n1ru4l/graphql-live-queries/blob/main/packages/socket-io-graphql-server) | [![npm downloads](https://img.shields.io/npm/dm/@n1ru4l/socket-io-graphql-server.svg)](https://github.com/n1ru4l/graphql-live-queries/blob/main/packages/socket-io-graphql-server) |
| [`graphql-helix`](https://github.com/danielrearden/graphql-helix)                                                                | [GraphQL over HTTP](https://github.com/graphql/graphql-over-http) (IncrementalDelivery/SSE) | [![npm version](https://badge.fury.io/js/graphql-helix.svg)](https://github.com/danielrearden/graphql-helix)                                                                    | [![npm downloads](https://img.shields.io/npm/dm/graphql-helix.svg)](https://github.com/danielrearden/graphql-helix)                                                                |
| [`graphql-ws`](https://github.com/enisdenjo/graphql-ws)                                                                          | [GraphQL over WebSocket](https://github.com/graphql/graphql-over-http/pull/140) (WebSocket) | [![npm version](https://badge.fury.io/js/graphql-ws.svg)](https://github.com/enisdenjo/graphql-ws)                                                                              | [![npm downloads](https://img.shields.io/npm/dm/graphql-ws.svg)](https://github.com/enisdenjo/graphql-ws)                                                                          |

For more details check out the [live query repository](https://github.com/n1ru4l/graphql-live-query) or the [introduction blog post](https://the-guild.dev/blog/subscriptions-and-live-queries-real-time-with-graphql).

[There is also a full schema example available.](https://github.com/n1ru4l/graphql-live-query/blob/main/packages/todo-example/server-ws/src/schema.ts)
