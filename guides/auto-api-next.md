## Create a NextJS app with TypeScript

#### Answer the questions as you like but when asked if you want to use the new `/app` directory, say no.

```
yarn create next-app --typescript
```

## Install dependencies

```
yarn add prisma typegraphql-prisma @types/micro-cors -D

yarn add apollo-server-micro micro-cors micro @prisma/client graphql@^15.3.0 class-validator type-graphql reflect-metadata graphql-scalars graphql-fields @types/graphql-fields tslib
```

## Initialize prisma

```
yarn prisma init --datasource-provider mongodb
```

## Add mongodb connection to .env

```
mongodb+srv://username:password@address.mongodb.net/tablename?retryWrites=true&w=majority
```

## Pull the database schema into Prisma

```
yarn prisma db pull
```

## Add generator to prisma

```prisma
generator typegraphql {
  provider = "typegraphql-prisma"
  output   = "../prisma/generated/type-graphql"
  simpleResolvers = true
}
```

## Create tsconfig.json or modify it at the NextJS root

```json
{
  "compilerOptions": {
    "allowJs": true,
    "forceConsistentCasingInFileNames": true,
    "noEmit": true,
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["./*"]
    },
    "sourceMap": true,
    "outDir": "dist",
    "strict": true,
    "lib": ["esnext", "esnext.asynciterable", "dom", "dom.iterable"],
    "target": "es2018",
    "module": "commonjs",
    "esModuleInterop": true,
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "skipLibCheck": true
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx"],
  "exclude": ["node_modules"]
}


```

## Create /pages/api/graphql.ts

```ts
import 'reflect-metadata';
import { resolvers } from 'prisma/generated/type-graphql';
import { PrismaClient } from '@prisma/client';
import { ApolloServer } from 'apollo-server-micro';
import Cors from 'micro-cors';
import * as tq from 'type-graphql';

const prisma = new PrismaClient();

const cors = Cors();

export default cors(async function handler(req, res) {
  const schema = await tq.buildSchema({
    resolvers,
    validate: false,
  });

  const context = () => {
    return {
      prisma,
    };
  };

  const apolloServer = new ApolloServer({ schema, context });

  if (req.method === 'OPTIONS') {
    res.end();
    return false;
  }
  await apolloServer.start();

  await apolloServer.createHandler({
    path: '/api/graphql',
  })(req, res);
});

export const config = {
  api: {
    bodyParser: false,
  },
};

```

## Generate the prisma schema

```
yarn prisma generate
```

## Add this line to the top of `_app.tsx`

```tsx
import 'reflect-metadata';
```

## Start your Next app

```
yarn dev
```
Access the server at [http://localhost:3000/api/graphql](http://localhost:3000/api/graphql)

## Enabling CORS to use outside Vercel

Add the following to `next.config.js`.

```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: true,
  async headers() {
    return [
      {
        source: '/api/:path*',
        headers: [
          { key: 'Access-Control-Allow-Credentials', value: 'true' },
          { key: 'Access-Control-Allow-Origin', value: '*' },
          {
            key: 'Access-Control-Allow-Methods',
            value: 'GET,OPTIONS,PATCH,DELETE,POST,PUT',
          },
          {
            key: 'Access-Control-Allow-Headers',
            value:
              'X-CSRF-Token, X-Requested-With, Accept, Accept-Version, Content-Length, Content-MD5, Content-Type, Date, X-Api-Version',
          },
        ],
      },
    ];
  },
};

module.exports = nextConfig;
```

## Automatically generate types for the client

Install graphql-code-generator
```
yarn add -D ts-node @graphql-codegen/cli @graphql-codegen/client-preset
```

Create `codegen.ts`

```ts
import { CodegenConfig } from '@graphql-codegen/cli';

const config: CodegenConfig = {
  schema: process.env.NEXT_PUBLIC_API_URL as string,
  documents: ['pages/**/*.tsx'],
  ignoreNoDocuments: true, // for better experience with the watcher
  generates: {
    './gql/': {
      preset: 'client',
      plugins: [],
    },
  },
};

export default config;

```

Install concurrently
```
yarn add -D concurrently
```

Modify your `dev` script in `package.json`

```
"dev": "concurrently \"yarn next dev\" \"yarn graphql-codegen --watch\"",
```

## Communicating with the API within Next

Add graphql-request and react-query

```
yarn add graphql-request react-query
```

Add React query provider and a global graphql-request client to `_app.tsx`

```tsx
import 'reflect-metadata';
import type { AppProps } from 'next/app';
import { QueryClient, QueryClientProvider } from 'react-query';

const queryClient = new QueryClient();

export const client = new GraphQLClient(process.env.NEXT_PUBLIC_API_URL as string, {
  headers: {},
});

export default function App({ Component, pageProps }: AppProps) {
  return (
    <QueryClientProvider client={queryClient}>
      <Component {...pageProps} />
    </QueryClientProvider>
  );
}

```
You're now ready to use react-query with your types, here's an example `index.tsx` with a database of players

```tsx
import Head from 'next/head';
import { graphql } from 'gql';
import { useQuery } from 'react-query';
import React from 'react';
import { client } from './_app';

const findManyPlayers = graphql(`
  query FindManyPlayers($take: Int) {
    findManyPlayers(take: $take) {
      age
      hits
      name
      id
      nationality
      overall
      player_id
      position
      potential
      team
    }
  }
`);

export default function Home() {
  const { data } = useQuery(['players'], async () =>
    client.request(findManyPlayers, {
      take: 5,
    })
  );
  if (!data) return <></>;
  return (
    <>
      <Head>
        <title>Create Next App</title>
        <meta name='description' content='Generated by create next app' />
        <meta name='viewport' content='width=device-width, initial-scale=1' />
        <link rel='icon' href='/favicon.ico' />
      </Head>
      <main>
        <div>
          {data.findManyPlayers.map((player) => (
            <React.Fragment key={player.id}>
              <p>{player.name}</p>
            </React.Fragment>
          ))}
        </div>
      </main>
    </>
  );
}
```
