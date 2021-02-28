
## Installing ESLint with Airbnb, a11y and React/Next.

### Download Packages

```shell
# add the basic requirements not provided by airbnb
yarn add -D eslint @babel/eslint-parser eslint-config-prettier eslint-plugin-prettier
# install airbnb config and all its deps
npx install-peerdeps -D eslint-config-airbnb
```

### Add .eslintrc

```js
// .eslintrc
{
  "env": {
    "browser": true,
    "commonjs": true,
    "es6": true,
    "node": true
  },
  "parser": "@babel/eslint-parser",
  "extends": [
    "eslint:recommended",
    "airbnb",
    "airbnb/hooks",
    "plugin:react/recommended",
    "plugin:import/errors",
    "plugin:import/warnings",
    "plugin:jsx-a11y/recommended",
    // always put prettier at last
    "prettier"
  ],
  "globals": {
    "Atomics": "readonly",
    "SharedArrayBuffer": "readonly"
  },
  "parserOptions": {
    "ecmaFeatures": {
      "jsx": true // enable linting for jsx files
    },
    "ecmaVersion": 2021,
    "sourceType": "module"
  },
  "settings": {
    "react": {
      "version": "detect"
    }
  },
  "plugins": ["react", "react-hooks"],
  "rules": {
    // NextJs specific fix: suppress errors for missing 'import React' in files for nextjs
    "react/react-in-jsx-scope": "off",
    // NextJs specific fix: allow jsx syntax in js files
    "react/jsx-filename-extension": [1, { "extensions": [".js", ".jsx"] }], //should add ".ts" if typescript project
    "react/display-name": 1,
    "react/prop-types": "off",
    "react/no-danger": "off",
    "react/jsx-props-no-spreading": [
      1,
      {
        "html": "enforce",
        "custom": "ignore"
      }
    ]
  }
}
```

### Add .eslintignore
```shell
# don't ever lint node_modules
node_modules
# don't lint build output (make sure it's set to your correct build folder name)
dist
# don't lint nyc coverage output
coverage
```
