# This is a demo project for libp2p @ typescript

versions
```
node --version
v19.0.0

tsc -v
Version 4.9.5

npm -v
8.19.2

libp2p
    "@chainsafe/libp2p-noise": "^11.0.0",
    "@libp2p/bootstrap": "^6.0.0",
    "@libp2p/mplex": "^7.1.1",
    "@libp2p/tcp": "^6.1.2",
    "it-to-buffer": "^3.0.0",
    "libp2p": "^0.42.2"
```

before running the code example (grabbed here https://github.com/libp2p/js-libp2p/blob/master/examples/transports/2.js)
```
rm -rf node_modules/ 
npm update
```

## This works (pure js)

``` 
tsconfig.json:

"compilerOptions": {
"module": "commonjs",
"target": "es2017",
```

```
node src/p2p-discovery.js
 
node 1 is listening on:
/ip4/127.0.0.1/tcp/56281/p2p/12D3KooWFchYTMfZJEjjYtwXoy226ryifCxHFcUKXTZyPCNMc7Fu
/ip4/192.168.1.104/tcp/56281/p2p/12D3KooWFchYTMfZJEjjYtwXoy226ryifCxHFcUKXTZyPCNMc7Fu
node 2 is listening on:
/ip4/127.0.0.1/tcp/56282/p2p/12D3KooWNpMbMXAGCXRyib4u239kW24cPdsh787MFNKvU42tntjv
/ip4/192.168.1.104/tcp/56282/p2p/12D3KooWNpMbMXAGCXRyib4u239kW24cPdsh787MFNKvU42tntjv
Hello p2p world!
```

## This also works (typescript + mocha test)
```
npm test

> test-p2p@0.1 test
> mocha --inspect=9229 -r ts-node/register tests/**/*.test.ts --require tests/root.ts --serial
Debugger listening on ws://127.0.0.1:9229/eefdf901-d29e-463a-852a-c2f19dd35480
For help, see: https://nodejs.org/en/docs/inspector
  hellotest
check that mocha is working
    ✔ test1
```

## This doesn't (ts-node @ typescript)

### attempt1
``` 
tsconfig.json:

"compilerOptions": {
"module": "es2020", 
"target": "es2017",
```

```
npm test

> snode@0.1 test
> mocha --inspect=9229 -r ts-node/register tests/**/*.test.ts --require tests/root.ts --serial

Debugger listening on ws://127.0.0.1:9229/cb1e93ce-94e4-46db-b278-f9d4b9f592cd
For help, see: https://nodejs.org/en/docs/inspector

✖ ERROR: /Users/igx/Documents/projects/test-p2p-fix/tests/root.ts:1
export const mochaHooks = {
^^^^^^

SyntaxError: Unexpected token 'export'
    at Object.compileFunction (node:vm:360:18)
    at wrapSafe (node:internal/modules/cjs/loader:1088:15)
    at Module._compile (node:internal/modules/cjs/loader:1123:27)
    at Module.m._compile (/Users/igx/Documents/projects/test-p2p-fix/node_modules/ts-node/src/index.ts:439:23)
    at Module._extensions..js (node:internal/modules/cjs/loader:1213:10)
    at Object.require.extensions.<computed> [as .ts] (/Users/igx/Documents/projects/test-p2p-fix/node_modules/ts-node/src/index.ts:442:12)
    at Module.load (node:internal/modules/cjs/loader:1037:32)
    at Function.Module._load (node:internal/modules/cjs/loader:878:12)
    at Module.require (node:internal/modules/cjs/loader:1061:19)
    at require (node:internal/modules/cjs/helpers:103:18)
    at exports.requireOrImport (/Users/igx/Documents/projects/test-p2p-fix/node_modules/mocha/lib/nodejs/esm-utils.js:60:20)
    at exports.handleRequires (/Users/igx/Documents/projects/test-p2p-fix/node_modules/mocha/lib/cli/run-helpers.js:94:28)
    at /Users/igx/Documents/projects/test-p2p-fix/node_modules/mocha/lib/cli/run.js:353:25
```



### attempt2

```
tsconfig.json:

"compilerOptions": {
"module": "es2020", 
"target": "es2020",
```


### attemp4
```
ts-node-dev --respawn --transpile-only ./src/index.ts

[INFO] 20:59:53 ts-node-dev ver. 2.0.0 (using ts-node ver. 10.9.1, typescript ver. 3.9.10)
Compilation error in /Users/igx/Documents/projects/test-p2p-fix/src/index.ts
Error: Must use import to load ES Module: /Users/igx/Documents/projects/test-p2p-fix/src/index.ts
    at Object.<anonymous> (/Users/igx/Documents/projects/test-p2p-fix/src/index.ts:1:7)
    at Module._compile (node:internal/modules/cjs/loader:1159:14)
    at Module._compile (/Users/igx/Documents/projects/test-p2p-fix/node_modules/source-map-support/source-map-support.js:568:25)
    at Module.m._compile (/private/var/folders/gh/qwb7hlgd74qg161hk7664p7m0000gn/T/ts-node-dev-hook-46611741353209.js:69:33)
    at Module._extensions..js (node:internal/modules/cjs/loader:1213:10)
    at require.extensions.<computed> (/private/var/folders/gh/qwb7hlgd74qg161hk7664p7m0000gn/T/ts-node-dev-hook-46611741353209.js:71:20)
    at Object.nodeDevHook [as .ts] (/Users/igx/Documents/projects/test-p2p-fix/node_modules/ts-node-dev/lib/hook.js:63:13)
    at Module.load (node:internal/modules/cjs/loader:1037:32)
    at Function.Module._load (node:internal/modules/cjs/loader:878:12)
    at Module.require (node:internal/modules/cjs/loader:1061:19)
[ERROR] 20:59:53 Error: Must use import to load ES Module: /Users/igx/Documents/projects/test-p2p-fix/src/index.ts

```