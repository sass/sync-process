# `sync-process`

This package exposes a `SyncProcess` class that allows Node.js to run
a subprocess synchronously *and* interactively.

## Usage

Use `new SyncProcess()` to start running a subprocess. This supports the same
API as [`child_process.spawn()`] other than a few options. You can send input to
the process using `process.stdin`, and receive events from it (stdout, stderr,
or exit) using `process.next()`. This implements [the iterator protocol], but
*not* the iterable protocol because it's intrinsically stateful.

[the iterator protocol]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols#the_iterator_protocol
[`child_process.spawn()`]: https://nodejs.org/api/child_process.html#child_processspawncommand-args-options

```js
import {SyncProcess} from 'sync-process';
// or
// const {SyncProcess} = require('sync-process');

const node = new SyncProcess('node', ['--interactive']);

for (;;) {
  if (node.next().value.data.toString().endsWith('> ')) break;
}

node.stdin.write("41 * Math.pow(2, 5)\n");
console.log((node.next().value.data.toString().split("\n")[0]));
node.stdin.write(".exit\n");
console.log(`Node exited with exit code ${node.next().value.code}`);
```

## Why synchrony?

See [the `sync-message-port` documentation] for an explanation of why running
code synchronously can be valuable even in an asynchronous ecosystem like
Node.js

[the `sync-message-port` documentation]: https://github.com/sass/sync-message-port?tab=readme-ov-file#why-synchrony

### Why not `child_process.spawnSync()`?

Although Node's built-in [`child_process.spawnSync()`] function does run
synchronously, it's not interactive. It only returns once the process has run to
completion and exited, which means it's not suitable for any long-lived
subprocess that interleaves sending and receiving data, such as when using the
[embedded Sass protocol].

[`child_process.spawnSync()`]: https://nodejs.org/api/child_process.html#child_processspawnsynccommand-args-options
[embedded Sass protocol]: https://github.com/sass/sass/blob/main/spec/embedded-protocol.md

---

Disclaimer: this is not an official Google product.
