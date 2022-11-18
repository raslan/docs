## Install dependencies

```
yarn add typescript ts-node @types/node prisma typegraphql-prisma ts-node-dev -D

yarn add @prisma/client graphql@^15.3.0 class-validator type-graphql reflect-metadata graphql-scalars graphql-fields @types/graphql-fields tslib @apollo/server
```

## Initialize prisma

```
yarn prisma init --datasource-provider mongodb

yarn prisma db pull
```

## Add mongodb connection to .env

```
mongodb+srv://username:password@address.mongodb.net/tablename?retryWrites=true&w=majority
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
    listen: { port: 4000 },
  });
};

init();
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
