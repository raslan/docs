# Express.js, TypeScript, ESLint, Prettier, Airbnb and very fast reloading.

## Create an Express project

```bash
mkdir my-project
cd my-project
yarn init ## -y if you'd like to skip the init process
```

## Add dependencies

### Express
```bash
yarn add express
```

### TypeScript

```bash
yarn add -D typescript ts-node-dev @types/express @types/node
```

### ESLint

```bash
yarn add -D @typescript-eslint/eslint-plugin @typescript-eslint/parser eslint eslint-config-airbnb-base eslint-config-prettier eslint-plugin-import eslint-plugin-prettier
```

### Add .eslintrc
```json
{
  "env": {
    "browser": true,
    "es2021": true
  },
  "extends": ["airbnb-base", "prettier"],
  "parser": "@typescript-eslint/parser",
  "parserOptions": {
    "ecmaVersion": 12,
    "sourceType": "module"
  },
  "plugins": ["@typescript-eslint"],
  "rules": {}
}
```

### Add .eslintignore

```bash
# don't lint node_modules
node_modules
# don't lint build output (make sure it's set to your correct build folder name)
dist
build
out
```

### Add scripts in your package.json

```json
"scripts": {
    "dev": "ts-node-dev --respawn index.ts",
    "build": "tsc --project ./"
}
```

### Initialize TypeScript configuration

```bash
yarn tsc --init
```

### Create the file index.ts
```ts
import express from "express";
const app = express();
const PORT = 5000;

app.get("/", (req, res) => res.send("Express + TypeScript Server"));

app.listen(PORT, () => {
  console.log(`⚡️[server]: Server is running at https://localhost:${PORT}`);
});
```

### Run the project
```bash
yarn dev
```
