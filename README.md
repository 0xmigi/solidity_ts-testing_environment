# Maximum-Solidity-Environment
[![CircleCI](https://circleci.com/gh/Luiserebii/Maximum-Solidity-Environment/tree/master.svg?style=svg)](https://circleci.com/gh/Luiserebii/Maximum-Solidity-Environment/tree/master)
The ultimate environment for Solidity development. The final choice. The end.

## Tools used:
  * Truffle
  * npm

## Reproduce:

#### Core: Truffle, Ganache, and npm
* Install Truffle
* Install npm: 
  * ```npm i --save-dev truffle```
* Generate truffle files: 
  * ```truffle init```
* Generate package.json:
  * ```npm init```
* Create .gitignore, and ignore:
  * Any temporary files created by text editor/IDE (e.g. `.swo` and `.swp` by vim)
  * `node_modules/` (dependency installations)
  * `build` (contract build folder generated by Truffle during compilation)
* Install ganache-cli (test blockchain): ```npm i --save-dev ganache-cli```
* Within `truffle-config.js`, uncomment the `development` object underneath `networks`
```javascript
    development: {
      host: "127.0.0.1",     // Localhost (default: none)
      port: 8545,            // Standard Ethereum port (default: none)
      network_id: "*",       // Any network (default: none)
    },
```
* Set up scripts in package.json: 
```JSON
  "scripts": {
    "compile": "npx truffle compile",
    "chain": "ganache-cli",
    "test": "echo \"Error: no test specified\" && exit 1"
  }
```
  * Runnable now via `npm run chain`

#### Testing, Linting, and Code Coverage: TypeScript, solhint, ESLint, solidity-coverage

##### Testing
* Mocha and Chai are included within Truffle, so technically, there should be no need to install them! This comes with a caveat, however...
  * TypeScript will yell at us about `error TS2304: Cannot find name 'assert'.`, so we will need to import chai into our tests and declare assert, and thus, install it.
  * `npm i --save-dev chai`
* TypeScript, however, a stricter superset of JavaScript, must be: ```npm i --save-dev typescript```
* Create a `src` and `bin` folder within `test`
  * `src` will be where we will place our TypeScript test files
  * `bin` will be reserved for our built Javascript test files, which will be consumed by Mocha for testing
    * Since `bin` will contain compiled files, let's add it to .gitignore as well
* We will also need to install some dependencies so TypeScript can pick up some of the types we'll be using:
```
npm i --save-dev @types/chai @types/mocha @types/node @machinomy/types-truffle @machinomy/types-truffle-contract @machinomy/types-truffle-resolver
```
  * The truffle-specific types have dependencies themselves, so we will also need install them:
```
npm i --save-dev truffle-contract truffle-resolver
```
* Now, we need to create a tsconfig.json file to pass to the Typescript compiler (`tsc`)
```
cd test
tsc --init
```
  * We can now modify .tsconfig according to our setup:
    * Modify target to `"ES2017"`
    * Uncomment outDir and set it to `"./bin"`
    * And, add our types to the `"compilerOptions"`:
```JSON
    "types": [
      "@types/node",
      "@types/mocha",
      "@types/chai",
      "@machinomy/types-truffle",
      "@machinomy/types-truffle-contract",
      "@machinomy/types-truffle-resolver"
    ],   
```
  * This should now look more or less like the following (many options commented out are shown in the generated file, but are excluded here for brevity):
```JSON
{
  "compilerOptions": {
    "target": "ES2017",                          /* Specify ECMAScript target version: 'ES3' (default), 'ES5', 'ES2015', 'ES2016', 'ES2017', 'ES2018', 'ES2019' or 'ESNEXT'. */
    "module": "commonjs",                     /* Specify module code generation: 'none', 'commonjs', 'amd', 'system', 'umd', 'es2015', or 'ESNext'. */
    "outDir": "./bin",                        /* Redirect output structure to the directory. */
    "strict": true,                           /* Enable all strict type-checking options. */
    "types": [
      "@types/node",
      "@types/mocha",
      "@types/chai",
      "@machinomy/types-truffle",
      "@machinomy/types-truffle-contract",
      "@machinomy/types-truffle-resolver"
    ],   
    "esModuleInterop": true                   /* Enables emit interoperability between CommonJS and ES Modules via creation of namespace objects for all imports. Implies 'allowSyntheticDefaultImports'. */
  }
}
```
* Finally, we can add our testing script:
```JSON
  "scripts": {
    ...
    "test": "rm -rf ./test/bin && tsc --project ./test/tsconfig.json && truffle test ./test/bin/*"
  }
```



##### Linting

* First, let's set up Solidity linting via `solhint`
  * Install via: `npm i --save-dev solhint`
  * Next, setup `.solhint.json`. This is a pretty good configuration that can be used:
```JSON
{
  "extends": "solhint:recommended",
  "rules": {
    "indent": ["error", 4],
    "func-order": "off",
    "bracket-align": "off",
    "compiler-fixed": "off",
    "no-simple-event-func-name": "off",
    "separate-by-one-line-in-contract": "off",
    "two-lines-top-level-separator": "off",
    "mark-callable-contracts": "off",
    "compiler-version": ["error", "^0.5.0"]
  }
}
```
  * Finally, add the following script:
```JSON
  "scripts": {
    ...
    "lint:sol": "solhint --max-warnings 0 \"contracts/**/*.sol\"",
    ...
  }
```
* Next, comes TypeScript linting via ESLint:
  * Install via: `npm i --save-dev eslint`
  * We're using Google's style guide plugin, so: `npm i --save-dev eslint-config-google`
  * We will also need the TypeScript ESLint parser and plugin: `npm install @typescript-eslint/parser @typescript-eslint/eslint-plugin --save-dev`
  * Finally, setup `.eslintrc.js`. The following is a pretty good configuration that can be used:
```javascript
module.exports = {
  'env': {
    'es6': true,
    'node': true,
    "node": true,
    "mocha": true,
  },
  'extends': [
    'google',
  ],
  'globals': {
    'Atomics': 'readonly',
    'SharedArrayBuffer': 'readonly',
  },
  'parser': '@typescript-eslint/parser',
  'parserOptions': {
    'ecmaVersion': 2018
  },
  'rules': {
    'max-len': ['error', 120, 2],
    "no-unused-vars": "off",
    "@typescript-eslint/no-unused-vars": ["error", { "vars": "all", "args": "after-used", "ignoreRestSiblings": false }]
  },
  "plugins": [
    "@typescript-eslint"
  ]
};

```
* Within package.json, edit the "lint" script and add the new lint scripts below:
```JSON
    "lint": "npm run lint:sol && npm run lint:js",
    "lint:fix": "npm run lint:js:fix",
    "lint:sol": "solhint --max-warnings 0 \"contracts/**/*.sol\"",
    "lint:js": "eslint ./test/src/**/*.ts",
    "lint:js:fix": "eslint --fix ./test/src/**/*.ts"
```

##### Code Coverage

* Install `solidity-coverage`: 
  * `npm i --save-dev solidity-coverage`
* Add this script to package.json: 
```
    "coverage": "npx solidity-coverage",
```
* And, add a configuration file `.solcover.js`:
```javascript
module.exports = {
    testCommand: 'npm t'
};
```

#### CI/CD Cloud Tools 

##### CircleCI

* CircleCI is pretty easy to set up. First, make an account on the CircleCI webpage.
* Next, add a .circleci folder at the root of the directory
  * `mkdir .circleci`
* And, add the following YAML file as config.yml: 
```yaml
# Recommended documentation on CircleCI and node applications: https://circleci.com/docs/2.0/language-javascript/

version: 2
# 2.1 does not yet support local run
# unless with workaround. For simplicity just use it.
# https://github.com/CircleCI-Public/circleci-cli/issues/79

aliases:
  # Alias for any default options
  - &defaults
    docker:
      - image: circleci/node:10.12 # Docker Node 10.12 image.
 
  # Alias for installing npm, if necessary (hence the name)
  - &npm_install_if_necessary
    run:
      name: Install npm dependencies
      command: |
        # If no node_modules directory
        if [ ! -d node_modules ]; then 
          # Perform a clean npm install
          npm ci
        fi
  # Alias for caching - caches node dependencies with a cache key
      # template for an environment variable,
      # see circleci.com/docs/2.0/caching/
  - &cache_key_node_modules
    key: v1-node_modules-{{ checksum "package-lock.json" }}

jobs:
  dependencies:
    <<: *defaults # << is used to inject an alias
    steps:
      - checkout
      - restore_cache: # special step to restore the dependency cache
          <<: *cache_key_node_modules
      - *npm_install_if_necessary
      - save_cache: # special step to save the dependency cache
          paths:
            - node_modules
          <<: *cache_key_node_modules # Insert our alias to specify cache key to save to

  lint:
    <<: *defaults
    steps:
      - checkout
      - restore_cache:
          <<: *cache_key_node_modules
      - *npm_install_if_necessary
      - run:
          name: Linter
          command: npm run lint
  test:
    <<: *defaults
    steps:
      - checkout
      - restore_cache:
          <<: *cache_key_node_modules
      - *npm_install_if_necessary
      - run: 
          name: Run development blockchain
          command: npm run chain
          background: true
      - run:
          name: Unit tests
          command: npm test
  coverage:
    <<: *defaults
    steps:
      - checkout
      - restore_cache:
          <<: *cache_key_node_modules
      - *npm_install_if_necessary
      - run:
          name: Unit tests with coverage report
          command: npm run coverage

workflows:
  version: 2
  everything:
    jobs: # For reference, all jobs run in parallel
      - dependencies
      - lint:
          requires:
            - dependencies # Wait for dependencies jobs to finish before running
      - test:
          requires:
            - dependencies
      - coverage:
          requires:
            - dependencies
```
* Push this to your repo. For example, if using master, this would be:
  * `git push origin master`
* Finally, enable CircleCI on this repository within the online app, and everything should be up and running!

#### Finishing touches

* Add a LICENSE (MIT recommended)
* Add the following to a new file `.gitattributes`. This will allow GitHub to recognize your .sol files as Solidity
```
*.sol linguist-language=Solidity
```
