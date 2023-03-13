# UPDATED
last fix works \
also you can check this https://github.com/libp2p/js-libp2p/issues/1583#issuecomment-1463666072


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
# Initial state
before running the code example (grabbed here https://github.com/libp2p/js-libp2p/blob/master/examples/transports/2.js)
```
rm -rf node_modules/ 
npm install
```

## This works (pure js)

``` 
tsconfig.json:

"compilerOptions": {
"module": "commonjs",
"target": "es2017",
```

```
node src/p2p-discovery.mjs
 
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


src/index.ts 
```
console.log('Hello ');
```
also works
```
npm start

> test-p2p@0.1 start
> nodemon

[nodemon] 2.0.20
[nodemon] to restart at any time, enter `rs`
[nodemon] watching path(s): *.*
[nodemon] watching extensions: ts,json
[nodemon] starting `ts-node src/index.ts`
Hello 
```

# Let's examine error cases
### now lets put libp2p code into index.ts

src/index.ts
```
console.log('Hello ');


import { createLibp2p } from 'libp2p'
import { tcp } from '@libp2p/tcp'
import { noise } from '@chainsafe/libp2p-noise'
import { mplex } from '@libp2p/mplex'
import { toString as uint8ArrayToString } from 'uint8arrays/to-string'
import { fromString as uint8ArrayFromString } from 'uint8arrays/from-string'
import { pipe } from 'it-pipe'
import toBuffer from 'it-to-buffer'


run();

async function run() {
    const createNode = async () => {
        const node = await createLibp2p({
            addresses: {
                // To signal the addresses we want to be available, we use
                // the multiaddr format, a self describable address
                listen: ['/ip4/0.0.0.0/tcp/0']
            },
            transports: [tcp()],
            connectionEncryption: [noise()],
            streamMuxers: [mplex()]
        })

        return node
    }

    function printAddrs (node, number) {
        console.log('node %s is listening on:', number)
        node.getMultiaddrs().forEach((ma) => console.log(ma.toString()))
    }

    ;(async () => {
        const [node1, node2] = await Promise.all([
            createNode(),
            createNode()
        ])

        printAddrs(node1, '1')
        printAddrs(node2, '2')

        node2.handle('/print', async ({ stream }) => {
            const result = await pipe(
                stream,
                async function * (source) {
                    for await (const list of source) {
                        yield list.subarray()
                    }
                },
                toBuffer
            )
            console.log(uint8ArrayToString(result))
        })

        await node1.peerStore.addressBook.set(node2.peerId, node2.getMultiaddrs())
        const stream = await node1.dialProtocol(node2.peerId, '/print')

        await pipe(
            ['Hello', ' ', 'p2p', ' ', 'world', '!'].map(str => uint8ArrayFromString(str)),
            stream
        )
    })();

}
```
results in
```
npm start

> test-p2p@0.1 start
> nodemon

[nodemon] 2.0.20
[nodemon] to restart at any time, enter `rs`
[nodemon] watching path(s): *.*
[nodemon] watching extensions: ts,json
[nodemon] starting `ts-node src/index.ts`
Error: No "exports" main defined in /Users/igx/Documents/projects/p2p-test/node_modules/libp2p/package.json
    at new NodeError (node:internal/errors:393:5)
    at throwExportsNotFound (node:internal/modules/esm/resolve:292:9)
    at packageExportsResolve (node:internal/modules/esm/resolve:546:7)
    at resolveExports (node:internal/modules/cjs/loader:529:36)
    at Function.Module._findPath (node:internal/modules/cjs/loader:569:31)
    at Function.Module._resolveFilename (node:internal/modules/cjs/loader:981:27)
    at Function.Module._load (node:internal/modules/cjs/loader:841:27)
    at Module.require (node:internal/modules/cjs/loader:1061:19)
    at require (node:internal/modules/cjs/helpers:103:18)
    at Object.<anonymous> (/Users/igx/Documents/projects/p2p-test/src/index.ts:2:1)
[nodemon] app crashed - waiting for file changes before starting...
^C

```

so lets switch to new modules section according to https://github.com/ipfs/js-ipfs/blob/master/docs/upgrading/v0.62-v0.63.md#typescript-and-esm

```
tsconfig.json:

"compilerOptions": {
   "module": "es2020", // ensures output is ESM
    "target": "es2020", // support modern features like private identifiers
```
results in
```
igx@igxmbp p2p-test % npm start

> test-p2p@0.1 start
> nodemon

[nodemon] 2.0.20
[nodemon] to restart at any time, enter `rs`
[nodemon] watching path(s): *.*
[nodemon] watching extensions: ts,json
[nodemon] starting `ts-node src/index.ts`

/Users/igx/Documents/projects/p2p-test/node_modules/ts-node/src/index.ts:261
    return new TSError(diagnosticText, diagnosticCodes)
           ^
TSError: ⨯ Unable to compile TypeScript:
src/index.ts(4,30): error TS2307: Cannot find module 'libp2p' or its corresponding type declarations.
src/index.ts(5,21): error TS2307: Cannot find module '@libp2p/tcp' or its corresponding type declarations.
src/index.ts(6,23): error TS2307: Cannot find module '@chainsafe/libp2p-noise' or its corresponding type declarations.
src/index.ts(7,23): error TS2307: Cannot find module '@libp2p/mplex' or its corresponding type declarations.
src/index.ts(8,48): error TS2307: Cannot find module 'uint8arrays/to-string' or its corresponding type declarations.
src/index.ts(9,52): error TS2307: Cannot find module 'uint8arrays/from-string' or its corresponding type declarations.
src/index.ts(10,22): error TS2307: Cannot find module 'it-pipe' or its corresponding type declarations.
src/index.ts(11,22): error TS2307: Cannot find module 'it-to-buffer' or its corresponding type declarations.

    at createTSError (/Users/igx/Documents/projects/p2p-test/node_modules/ts-node/src/index.ts:261:12)
    at getOutput (/Users/igx/Documents/projects/p2p-test/node_modules/ts-node/src/index.ts:367:40)
    at Object.compile (/Users/igx/Documents/projects/p2p-test/node_modules/ts-node/src/index.ts:558:11)
    at Module.m._compile (/Users/igx/Documents/projects/p2p-test/node_modules/ts-node/src/index.ts:439:43)
    at Module._extensions..js (node:internal/modules/cjs/loader:1213:10)
    at Object.require.extensions.<computed> [as .ts] (/Users/igx/Documents/projects/p2p-test/node_modules/ts-node/src/index.ts:442:12)
    at Module.load (node:internal/modules/cjs/loader:1037:32)
    at Function.Module._load (node:internal/modules/cjs/loader:878:12)
    at Function.executeUserEntryPoint [as runMain] (node:internal/modules/run_main:82:12)
    at Object.<anonymous> (/Users/igx/Documents/projects/p2p-test/node_modules/ts-node/src/bin.ts:157:12)
[nodemon] app crashed - waiting for file changes before starting...
```
if we try tsconfig.json: "moduleResolution": "node", same error
``` 
tsconfig.json:

"compilerOptions": {
   "module": "es2020", // ensures output is ESM
    "target": "es2020", // support modern features like private identifiers
    "moduleResolution": "node",
```
error will be
```
 npm start

> test-p2p@0.1 start
> nodemon

[nodemon] 2.0.20
[nodemon] to restart at any time, enter `rs`
[nodemon] watching path(s): *.*
[nodemon] watching extensions: ts,json
[nodemon] starting `ts-node src/index.ts`
(node:90591) Warning: To load an ES module, set "type": "module" in the package.json or use the .mjs extension.
(Use `node --trace-warnings ...` to show where the warning was created)
/Users/igx/Documents/projects/p2p-test/src/index.ts:2
import { createLibp2p } from 'libp2p';
^^^^^^

SyntaxError: Cannot use import statement outside a module
    at Object.compileFunction (node:vm:360:18)
    at wrapSafe (node:internal/modules/cjs/loader:1088:15)
    at Module._compile (node:internal/modules/cjs/loader:1123:27)
    at Module.m._compile (/Users/igx/Documents/projects/p2p-test/node_modules/ts-node/src/index.ts:439:23)
    at Module._extensions..js (node:internal/modules/cjs/loader:1213:10)
    at Object.require.extensions.<computed> [as .ts] (/Users/igx/Documents/projects/p2p-test/node_modules/ts-node/src/index.ts:442:12)
    at Module.load (node:internal/modules/cjs/loader:1037:32)
    at Function.Module._load (node:internal/modules/cjs/loader:878:12)
    at Function.executeUserEntryPoint [as runMain] (node:internal/modules/run_main:82:12)
    at Object.<anonymous> (/Users/igx/Documents/projects/p2p-test/node_modules/ts-node/src/bin.ts:157:12)
[nodemon] app crashed - waiting for file changes before starting...


```
and if we try package.json: "type":"module"
```
 npm start

> test-p2p@0.1 start
> nodemon

[nodemon] 2.0.20
[nodemon] to restart at any time, enter `rs`
[nodemon] watching path(s): *.*
[nodemon] watching extensions: ts,json
[nodemon] starting `ts-node src/index.ts`
TypeError: Unknown file extension ".ts" for /Users/igx/Documents/projects/p2p-test/src/index.ts
    at new NodeError (node:internal/errors:393:5)
    at Object.getFileProtocolModuleFormat [as file:] (node:internal/modules/esm/get_format:75:9)
    at defaultGetFormat (node:internal/modules/esm/get_format:114:38)
    at defaultLoad (node:internal/modules/esm/load:81:20)
    at nextLoad (node:internal/modules/esm/loader:161:28)
    at ESMLoader.load (node:internal/modules/esm/loader:593:26)
    at ESMLoader.moduleProvider (node:internal/modules/esm/loader:445:22)
    at new ModuleJob (node:internal/modules/esm/module_job:63:26)
    at ESMLoader.#createModuleJob (node:internal/modules/esm/loader:468:17)
    at ESMLoader.getModuleJob (node:internal/modules/esm/loader:422:34)
[nodemon] app crashed - waiting for file changes before starting...
```



# Option 2 (start compiled js from dist/src/index.js on typescript 4.9) 

I've edited package.json:

```
  "main": "dist/src/index.js",
  "module": "ESNext",
  "type": "module",
   "scripts": {
    "build": "rimraf ./build && tsc",
    "dev2": "ts-node-dev --transpile-only ./tests/p2p-message.test.ts",
  ...
      "typescript": "^4.9.5"  
```

##  we can run dist/src/index.js (works)

```
rm -rf node_modules/
npm i
npm run build

> test-p2p@0.1 build
> rimraf ./build && tsc


npm start    

> test-p2p@0.1 start
> nodemon
[nodemon] 2.0.20
[nodemon] to restart at any time, enter `rs`
[nodemon] watching path(s): *.*
[nodemon] watching extensions: js,mjs,json
[nodemon] starting `node dist/src/index.js`
Hello 
node 1 is listening on:
/ip4/127.0.0.1/tcp/54447/p2p/12D3KooWGkm8u8fvnTBpA3L3BYjNoFQuYZ1FrWFiWGZTSgjiScVt
/ip4/192.168.1.39/tcp/54447/p2p/12D3KooWGkm8u8fvnTBpA3L3BYjNoFQuYZ1FrWFiWGZTSgjiScVt
/ip4/10.110.62.251/tcp/54447/p2p/12D3KooWGkm8u8fvnTBpA3L3BYjNoFQuYZ1FrWFiWGZTSgjiScVt
node 2 is listening on:
/ip4/127.0.0.1/tcp/54448/p2p/12D3KooWDTCvd7qaeYNQJujVHnxQ4jnmud5Pmsyn4vBpE28EUYZr
/ip4/192.168.1.39/tcp/54448/p2p/12D3KooWDTCvd7qaeYNQJujVHnxQ4jnmud5Pmsyn4vBpE28EUYZr
/ip4/10.110.62.251/tcp/54448/p2p/12D3KooWDTCvd7qaeYNQJujVHnxQ4jnmud5Pmsyn4vBpE28EUYZr
Hello p2p world!
^C

```
## src/index.ts fails (!)

Let's revert main back to index.ts

```
"main": "src/index.ts",
```
and start it (fails)
```
igx@igxmbp p2p-test % npm start  

> test-p2p@0.1 start
> nodemon

[nodemon] 2.0.20
[nodemon] to restart at any time, enter `rs`
[nodemon] watching path(s): *.*
[nodemon] watching extensions: ts,json
[nodemon] starting `ts-node src/index.ts`
TypeError: Unknown file extension ".ts" for /Users/igx/Documents/projects/p2p-test/src/index.ts
    at new NodeError (node:internal/errors:393:5)
    at Object.getFileProtocolModuleFormat [as file:] (node:internal/modules/esm/get_format:75:9)
    at defaultGetFormat (node:internal/modules/esm/get_format:114:38)
    at defaultLoad (node:internal/modules/esm/load:81:20)
    at nextLoad (node:internal/modules/esm/loader:161:28)
    at ESMLoader.load (node:internal/modules/esm/loader:593:26)
    at ESMLoader.moduleProvider (node:internal/modules/esm/loader:445:22)
    at new ModuleJob (node:internal/modules/esm/module_job:63:26)
    at ESMLoader.#createModuleJob (node:internal/modules/esm/loader:468:17)
    at ESMLoader.getModuleJob (node:internal/modules/esm/loader:422:34)
[nodemon] app crashed - waiting for file changes before starting...

```


##  let's try to run code as a test (fails)
```
igx@igxmbp p2p-test % npm run dev2

> test-p2p@0.1 dev2
> ts-node-dev --transpile-only ./tests/p2p-message.test.ts

[INFO] 13:27:54 ts-node-dev ver. 2.0.0 (using ts-node ver. 10.9.1, typescript ver. 4.9.5)
Compilation error in /Users/igx/Documents/projects/p2p-test/tests/p2p-message.test.ts
Error: Must use import to load ES Module: /Users/igx/Documents/projects/p2p-test/tests/p2p-message.test.ts
    at Object.<anonymous> (/Users/igx/Documents/projects/p2p-test/tests/p2p-message.test.ts:1:7)
    at Module._compile (node:internal/modules/cjs/loader:1159:14)
    at Module._compile (/Users/igx/Documents/projects/p2p-test/node_modules/source-map-support/source-map-support.js:568:25)
    at Module.m._compile (/private/var/folders/gh/qwb7hlgd74qg161hk7664p7m0000gn/T/ts-node-dev-hook-30600465553697687.js:69:33)
    at Module._extensions..js (node:internal/modules/cjs/loader:1213:10)
    at require.extensions..jsx.require.extensions..js (/private/var/folders/gh/qwb7hlgd74qg161hk7664p7m0000gn/T/ts-node-dev-hook-30600465553697687.js:114:20)
    at require.extensions.<computed> (/private/var/folders/gh/qwb7hlgd74qg161hk7664p7m0000gn/T/ts-node-dev-hook-30600465553697687.js:71:20)
    at Object.nodeDevHook [as .ts] (/Users/igx/Documents/projects/p2p-test/node_modules/ts-node-dev/lib/hook.js:63:13)
    at Module.load (node:internal/modules/cjs/loader:1037:32)
    at Function.Module._load (node:internal/modules/cjs/loader:878:12)
[ERROR] 13:27:54 Error: Must use import to load ES Module: /Users/igx/Documents/projects/p2p-test/tests/p2p-message.test.ts
^C

```

Summary ts-node/ts-node-dev cannot run libp2p code without errors.
However, if we run a typescript compiler by hand - the resulting js works.


#### Useful links
https://github.com/libp2p/js-libp2p/issues/1286


https://github.com/TypeStrong/ts-node/issues/1007


https://github.com/ipfs/js-ipfs/blob/master/docs/upgrading/v0.62-v0.63.md#typescript-and-esm
