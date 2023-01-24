## Introduction

In this guide we'll be using a JSON or CSV file with some data in it to build a fully featured GraphQL API inside a NextJS application.

For this to work, we'll need some tools, here's links to all the tools used in this guide if you'd like to check any of them out or potentially explore replacements:

- [Prisma](https://prisma.io) The database connector where all the magic happens
- [Typegraphql-Prisma](https://prisma.typegraphql.com/) To generate the TypeGraphQL schema for Apollo server from Prisma
- [TypeGraphQL](https://typegraphql.com/)
- [Apollo Server](https://www.apollographql.com/)
- [graphql-code-generator](https://the-guild.dev/graphql/codegen) To build the types for the frontend
- [Next](https://nextjs.org/) The fullstack framework we'll be using
- [React](https://reactjs.org) The frontend part


## Create a MongoDB database and add your data (skip if you already have a MongoDB database with data in it)

Head to [mongodb.com](mongodb.com) and sign up for a free Atlas account or set up mongodb locally

Create your project

![image](https://user-images.githubusercontent.com/24810123/213875987-e529e32e-dac1-42a2-b2ad-b2fafbd0947d.png)

Create a cluster, select `SHARED` for the free tier.

![image](https://user-images.githubusercontent.com/24810123/213876016-cc50b07b-749a-4c5b-8b3c-b82697148727.png)

Create your user and add your IP address to the list of allowed IP addresses (or 0.0.0.0 to allow any connection).

![image](https://user-images.githubusercontent.com/24810123/213876088-1b4d42db-f6e1-4609-a6a2-563035cc97f9.png)

Press `connect` on your cluster on mongodb.com

![image](https://user-images.githubusercontent.com/24810123/213876204-558c3d87-9c95-4ed4-8cc0-cb2b21c9043f.png)

Click on Compass

![image](https://user-images.githubusercontent.com/24810123/213876216-82bfb6ce-b0e4-4ccc-834d-efcda13a4d5c.png)

Download Compass if you don't have it, and copy the connection string

![image](https://user-images.githubusercontent.com/24810123/213876259-1dc4bb37-f9c5-4692-87dc-cdc041297964.png)

Paste the connection string into Compass, replacing <password> with the password for the database user you created earlier and press connect
  ![image](https://user-images.githubusercontent.com/24810123/213876317-835ad3e3-e3b4-4fd2-aea3-901130dee0d8.png)

  Click databases
  ![image](https://user-images.githubusercontent.com/24810123/213876355-b0fccd85-7187-4038-99ef-02ec163c3f03.png)

  Click create
  ![image](https://user-images.githubusercontent.com/24810123/213876371-56f39960-4660-4b44-ba84-c6b1a6359837.png)

  In the database name and collection name fields, describe the data type your database will contain, for example a database of football players could be called `football_players` or `players`
  ![image](https://user-images.githubusercontent.com/24810123/213876423-ef89a240-8830-4b19-9792-4ee2e6c707bd.png)

  Once the database is created, click on it
  ![image](https://user-images.githubusercontent.com/24810123/213876476-edb8e9e0-8b2d-4403-85f6-07120e4b74b1.png)

  Click the collection
  ![image](https://user-images.githubusercontent.com/24810123/213876501-c3770aeb-8c53-4eae-a662-0acf44a03539.png)

  You can now upload your JSON or CSV file to the database to fill it
  ![image](https://user-images.githubusercontent.com/24810123/213876522-74caf063-cd06-440a-a6cb-f1710858c457.png)

  We now have a database with the data we need, you can close Compass.
  

## Create a NextJS app with TypeScript

#### Answer the questions as you like but when asked
  
'if you want to use the new `/app` directory' no
  
'if you want to use `src` directory' no

```
yarn create next-app --typescript
```

## Install dependencies

```
yarn add prisma typegraphql-prisma -D

yarn add @apollo/server @as-integrations/next @prisma/client graphql class-validator type-graphql@next reflect-metadata graphql-scalars graphql-fields @types/graphql-fields tslib 
```

## Initialize prisma

```
yarn prisma init --datasource-provider mongodb
```

## Add mongodb connection and the eventual graphql api url to .env

```
DATABASE_URL="mongodb+srv://username:password@address.mongodb.net/tablename?retryWrites=true&w=majority"
NEXT_PUBLIC_API_URL="http://localhost:3000/api/graphql"
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

## Generate the prisma schema

```
yarn prisma generate
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
  schema: "http://localhost:3000/api/graphql",
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
import { GraphQLClient } from 'graphql-request';

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
