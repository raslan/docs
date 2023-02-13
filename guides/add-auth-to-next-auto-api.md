# Use Auth0 to Protect a TypeGraphQL Prisma API in Next.JS

In my previous guide I showed you how to generate a full GraphQL API inside a Next.js app using Prisma, TypeGraphQL and several other tools. In this guide I'll show you how to protect this API with Auth0.

One unfortunate step to take is we will have to replace Apollo with Graphql-Yoga, this is due to a current issue where Apollo Studio doesn't run on localhost and therefore makes the development process very uncomfortable once we protect the route.