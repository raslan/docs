# How to create a GraphQL API from just a JSON file

## Initialize a node project

```
yarn init -y
```

## Install dependencies

```
yarn add typescript ts-node @types/node prisma typegraphql-prisma ts-node-dev -D

yarn add @prisma/client graphql@^15.3.0 class-validator type-graphql reflect-metadata graphql-scalars graphql-fields @types/graphql-fields tslib @apollo/server
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

```
generator typegraphql {
  provider = "typegraphql-prisma"
  output   = "../prisma/generated/type-graphql"
  simpleResolvers = true
}
```

## Create tsconfig.json

```json
{
  "compilerOptions": {
    "sourceMap": true,
    "outDir": "dist",
    "strict": true,
    "lib": ["esnext", "esnext.asynciterable"],
    "target": "es2018",
    "module": "commonjs",
    "esModuleInterop": true,
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "skipLibCheck": true
  }
}

```

## Create index.ts

```ts
import 'reflect-metadata';
import { ApolloServer } from '@apollo/server';
import { startStandaloneServer } from '@apollo/server/standalone';
import { PrismaClient } from '@prisma/client';
import { buildSchema } from 'type-graphql';
import { resolvers } from './prisma/generated/type-graphql';

const init = async () => {
  const prisma = new PrismaClient();

  const schema = await buildSchema({
    resolvers,
    validate: false,
  });

  const server = new ApolloServer({
    schema,
  });

  await startStandaloneServer(server, {
    context: async () => ({ prisma }),
    listen: { port: 4000 }, // change as needed
  });
};

init();
```

## Generate the prisma schema

```
yarn prisma generate
```

## Add scripts to package.json

```
 "scripts": {
    "dev": "ts-node-dev --respawn index.ts",
    "build": "tsc --project ./"
  },
```

## Run the server

```
yarn dev
```
Access the server at http://localhost:4000


# Bonus: REST

## Install sofa-api

```
yarn add sofa-api
```

## Add lines to index.ts

```ts
import http from 'http';
import { useSofa } from 'sofa-api';

...

const restServer = http.createServer(
    useSofa({
      basePath: '/api',
      schema,
      context: async () => ({ prisma }),
    })
  );

  const port = 4001; // change this as needed
  const host = 'localhost'; // change to where the application is hosted

  restServer.listen(port, host, () => {
    console.log(`Server is running on http://${host}:${port}`);
  });
```
