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
if we add moduleResolution = same error
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
and if 



#### Useful links
https://github.com/libp2p/js-libp2p/issues/1286
https://github.com/TypeStrong/ts-node/issues/1007
https://github.com/ipfs/js-ipfs/blob/master/docs/upgrading/v0.62-v0.63.md#typescript-and-esm
