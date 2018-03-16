# 0x

<img alt=0x src=assets/0x-logo.png width=350>

🔥 single-command flamegraph profiling 🔥

Discover the bottlenecks and hot paths in your code, with flamegraphs.

## Visualize Stack Traces

`0x` can profile and generate an interactive flamegraph for a Node process with a single command,
on any platform which Node runs on (macOs, Linux, Windows, Android...).

## Support

* Node v8.5.0 and above
* Default usage supports any Operating System that Node runs on!
* Chrome
  * Other browsers may open flamegraphs in a degraded, but functional form 

## Legacy

Older versions of Node are supported via previous 0x versions:

| 0x | Node       | macOS/SmartOS | Linux | Windows |
|----|------------|-------|-------|---------|
| v4 | v8.5.0+    | ☑️    | ☑️     | ☑️      |
| v3 | v6 – v8.4.0| ☑️    | ☑️     | ⤬      |
| v2 | v4         | ☑️    | ☑️     | ⤬      |

## Demo

An example interactive flamegraph can be viewed at <http://davidmarkclements.github.io/0x-demo/>

## Install

```sh
npm install -g 0x
```

## Basic Usage

Use `0x` to run a script:

```sh
0x my-app.js
```

Open the flamegraph automatically in the browser with:

```sh
0x -o my-app.js
```

Using a custom node executable:

```sh
0x -- /path/to/node my-app.js
```

Passing custom arguments to node:

```sh
0x -- node --zero-fill-buffers my-app.js
```

## Generating

When ready to generate a flamegraph, send a SIGINT.

The simplest way to do this is pressing CTRL+C.

When `0x` catches the SIGINT, it process the stacks and
generates a profile folder (`<pid>.0x`), containing `flamegraph.html`.

## The UI

The `flamegraph.html` file contains the 0x UI, which is explained in 
[docs/ui.md](docs/ui.md).

## Production Servers

A lightweight, production server friendly, approach to generating a 
flamegraph is described in [docs/production-servers.md](docs/production-servers.md).

## The Profile Folder

By default, a Profile Folder will be created and named after the PID, e.g.
`3866.0x` (we can set this name manually using the `--output-dir` flag).

The Profile Folder can contain the following files

* flamegraph.html - the interactive flamegraph
* isolate-0x103000600-3866-v8.log - a v8 profiling log file
* isolate-0x103000600-3866-v8.json - the profile log file processed into JSON
* meta.json - additional meta data captured via processing
* stacks.3866.json - only with `--tree-debug` flag: a JSON tree representing the captured stacks 

The hex address in the isolate log files (0x103000600) is set according
to V8 internals, and will (most likely) differ each time a process is profiled. 

When used with the [`--kernel-tracing`](#--kernel-tracing) flag the Profile Folder won't
contain the isolate V8 log or JSON files, but will contain the following:

* .stacks.3866.out - pre-processed captured stacks via kernel tracing 
* stacks.3866.out - post-processed captured stacks via kernel tracing

## Example

Clone this repo, run `npm i -g` and from the repo root run

```sh
0x examples/rest-api
```

In another tab run

```sh
npm run stress-rest-example
```

To put some load on the rest server, once that's done
use ctrl + c to kill the server.

## Command Line API

### --help | -h

Print usage info.

### --open | -o

Open the flamegraph on your browser using `open` or `xdg-open` (see
https://www.npmjs.com/package/open for details).

### --name

The name of the HTML file, without the .html extension
Can be set to - to write HTML to STDOUT (note
due to the nature of CLI argument parsing, this must be set using `=`, 
e.g. `--name=-`).

If either this flag or `--output-html-file` is set to `-` 
then the HTML will go to STDOUT.

Default: flamegraph

### ---title 

Set the title to display in the flamegraph UI.

### --output-dir | -D

Specify artifact output directory. This can be specified in template
form with possible variables being `{pid}`, `{timestamp}`, `{name}` 
(based on the `--name` flag) and `{outputDir}`(variables
must be specified without whitespace, e.g. `{ pid }` is not supported).

Default: `{pid}.0x`

### --output-html | -F

Specify destination of the generated flamegraph HTML file. 
This can be specified in template form with possible variables 
being `{pid}`, `{timestamp}`, `{name}` (based on the `--name` flag) and 
`{outputDir}` (variables must be specified without whitespace, 
e.g. `{ pid }` is not supported). It can also be set to `-` to 
send the HTML output to STDOUT (note
due to the nature of CLI argument parsing, this must be set using `=`, 
e.g. `--output-html=-`).

If either this flag or --name is set to - 
then the HTML will go to STDOUT.

Default: `{outputDir}/{name}.html`

### --kernel-tracing

Use an OS kernel tracing tool (perf on Linux or 
dtrace on macOS and SmartOS). This will capture 
native stack frames (C++ modules and Libuv I/O), 
but may result in missing stacks on Node 8.

See [docs/kernel-tracing](docs/kernel-tracing.md) for more information.

Default: false 

### --phase

Stage in initialization to begin aggregating stacks. 

**Phase 0** visualizes from the very start, this includes bootstrapping 
stacks and loading the application module tree (these can dominate the flamegraph). 

**Phase 1** excludes core bootstrapping stacks, except the end of the boostrapping process 
where the application module tree is loaded from the entry point. 

**Phase 2** excludes all initialization, this renders the most pragmatic flamegraph for most 
use cases.

Default: 2

### --quiet | -q 

Limit output, the only output will be fatal errors or 
the path to the `flamegraph.html` upon successful generation.

Default: false

### --silent | -s

Suppress all output, except fatal errors.

Default: false

### --collect-only

Don't generate the flamegraph, only create the stacks
output. 

Default: false

### --visualize-only 

Supply a path to a profile folder to build or rebuild visualization 
from original stacks. Similar to --gen flag, except specify containing folder
instead of stacks file.

Default: ''  

### --kernel-tracing-debug

Show output from dtrace or perf tools

Default: false

### --tree-debug

Save the intermediate tree representation of captured trace output to a JSON
file.

Default: false

## Programmatic API 

0x can also be required as a Node module and scripted:

```js
const zeroEks = require('0x')
const path = require('path')

async function capture () {
  const opts = {
    argv: [path.join(__dirname, 'my-app.js'), '--my-flag', '"value for my flag"'],
    workingDir: __dirname
  }
  try {
    const file = await zeroEks(opts)
    console.log(`flamegraph in ${file}`)
  } catch (e) {
    console.error(e)
  }
}

capture()

```

### `require('0x')(opts) => Promise -> assetPath`

The `opts` argument is an object, with the following properties:

#### `argv` (array) – required

Pass the arguments that the spawned Node process should receive. 

#### `workingDir` (string)

The base directory where profile folders will be placed. 

Default: process.cwd()

#### `pathToNodeBinary` (string)

The path to any node binary executable. This will be used to run execute 
the script and arguments supplied in `argv`.

Default: Node executable according to the `PATH` environment variable.

#### `name` (string) 

The name of the flamegraph HTML output file, without the extension.

Default: flamegraph

#### `title` (string)

See [`--title`](#--title)

#### `visualizeOnly` (string)

See [`--visualize-only`](#--visualize-only)

#### `collectOnly` (boolean)

See [`--collect-only`](#--collect-only)

#### `phase` (number)

See [`--phase`](#--phase)

#### `mapFrames` (function)

Will override phase. A custom mapping function that receives 
an array of frames and an instance of the Profiler (see [stacks-to-json-stack-tree](http://github.com/davidmarkclements/stacks-to-json-stack-tree)).

Takes the form `(frames, profiler) => Array|false`. 

The `frames` parameter is an array of objects containing a `name` property.

Return `false` to remove the whole stack from the output, or return a 
modified array to change the output. 

#### `quiet` (boolean)

See [`--quiet`](#--quiet---q)

#### `silent` (boolean)

See [`--silent`](#--silent---s)

### `kernelTracing`

See [`--kernel-tracing`](#--kernel-tracing)

#### `outputDir` (string)

See [`--output-dir`](#--output-dir---d)

#### `outputHtml` (string)

See [`--output-html`](#--output-html---f)

#### `open` (boolean)

See [`--open`](#--open---o)

#### `treeDebug` (boolean)

See [`--tree-debug`](#--tree-debug)

#### `kernelTracingDebug` (boolean)

See [`--kernel-tracing-debug`](#--kernel-tracing-debug)

## Troubleshooting

### Memory Issues

Very complex applications with lots of stacks may hit memory issues. 

The `--stack-size` flag can be used to set the memory to the maximum 8GB 
in order to work around this when profiling:

```
node --stack-size=8024 $(which 0x) my-app.js
```

There may still be a problem opening the flamegraph in Chrome. The same work
around can be used by opening Chrome from the command line (platform dependent)
and nesting the `--stack-size` flag within the `--js-flags` flag: 
`--js-flags="--stack-size 8024"`.

## Debugging

`DEBUG=0x* 0x my-app.js`

## Alternatives

* <https://github.com/brendangregg/FlameGraph> (perl)
* <https://www.npmjs.com/package/stackvis> (node)
* <https://www.npmjs.com/package/d3-flame-graph> (node)

## Acknowledgements

Sponsored by [nearForm](http://nearform.com)

This tool is inspired from various info and code sources
and would have taken much longer without the following people and 
their Open Source/Info Sharing efforts:

* Thorsten Lorenz (<http://thlorenz.com/>)
* Dave Pacheco (<http://dtrace.org/blogs/dap/about/>)
* Brendan Gregg (<http://www.brendangregg.com/>)
* Martin Spier (<http://martinspier.io/>)

## License

MIT
