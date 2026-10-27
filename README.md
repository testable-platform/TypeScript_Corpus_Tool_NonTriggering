# TypeScript Order Platform -- Microservices (TS_V26_ESBUILD_NPM_MICRO)

Tool-evaluation repository for **Node 26**, bundled with **esbuild**,
managed with **npm**, in a **Microservices** layout.

This is branch **TS_V26_ESBUILD_NPM_MICRO** of the consolidated `typescript-corpus` repository, which holds all 216 TypeScript branches across every Node version, bundler, package manager and architecture combination in this corpus.

## Project type

- **Language:** TypeScript 5.9.3
- **Runtime:** Node 26 (verified against 26.8.1)
- **Scenario:** 2 - Microservices
- **Architecture:** Microservices
- **Module layout:** workspace
- **Bundler:** esbuild 0.28.2 (esbuild transform and linker)
- **Package manager:** npm 12.0.2
- **Source root:** `packages/domain/src`

Each tool is pinned to the release that was resolved for Node 26; `dataset.json`
records the tools that were skipped and why. Every version in this
repository was resolved against the live npm registry and then executed on a
real Node 26.8.1 interpreter. None was written from memory.

## Build

```bash
npm i -g npm@12.0.2
npm ci
npm run build
npm run bundle
```

`npm run build` type-checks with `tsc --noEmit` and emits CommonJS + declarations to
`dist/`; `npm run bundle` then bundles with **esbuild 0.28.2 (esbuild transform and linker)** and **executes the bundle**.
Emitting is not proof; running it is.

## Run

```bash
node dist/packages/domain/src/index.js
```

Each service is independently runnable:

```bash
node dist/services/gateway-service/src/index.js
node dist/services/pricing-service/src/index.js
```

## Test

```bash
npm run test        # mocha over tests/
npm run coverage    # c8 (primary) AND nyc + ts-node (cross-check)
```

Both coverage tools must report non-zero. They deliberately disagree: c8 reads
V8 coverage of the emitted output and remaps it, while nyc instruments the
TypeScript AST directly through `ts-node/register`. Identical numbers would mean
one of them is not an independent second opinion.

`nyc` here never reports through a source-map remap. Doing so silently yields
0% -- the file is remapped to `.ts`, an `--include` written against `dist/**`
stops matching, and the report empties while the process still exits 0.

## Architecture

**Microservices.** Three independently deployable services over a shared domain
package, wired through a workspace root.

```
packages/
  domain/         @orderkit/domain     -- models, services, analysis fixtures
  contracts/      @orderkit/contracts  -- inter-service message types
services/
  gateway-service/    accepts payloads, emits order.submitted
  order-service/      validates and prices, emits order.priced / order.rejected
  pricing-service/    owns tier and volume rates (leaf -- calls no one)
```

Each service has its own `package.json`, its own `src/index.ts` entry point and
its own start script. The workspace root lists them under `workspaces`.

**Documented inter-service call path:**

```
http.request -> gateway-service -> order.submitted
                order-service   -> pricing.quote -> pricing-service
                pricing-service -> pricing.rate  -> order-service
                order-service   -> order.priced | order.rejected
```

This matters because of what the JavaScript corpus got wrong: FlintAtlas's sheet
said `Microservices` while the repository was a flat monolith with no
workspaces, and its own README and `dataset.json` both said "monolith". Nothing
in that repo compared the two. Here `Tool Triggering (Synthetic Data)/full_check.ts` fails the build if the
declared architecture and the actual layout disagree.


## Tools non triggering (Synthetic Data)

One of five per-branch data folders in the original combined `testable-platform/typescript-corpus` repo -- this repo carries only this one.

Clean makes each tool run and report nothing wrong. Invalid makes it run and
report something. This folder holds data with nothing in it for any tool to
catch -- and, where no program would reach the tool at all, nothing for it to
start on. It is the negative control that tells *correctly detected nothing*
apart from *the scan never ran*.

`Tools non triggering (Synthetic Data)/` holds 40 tool-named folders, matching the names in Clean
and Invalid so the data sets line up name-for-name:

* **16 tools read source**, so they get their own minimal program, for node26 -- one class or one function, no branching, no
  duplication, no dependency, no dead export, no magic number.
* **24 tools cannot be answered by a program** -- they read a lockfile,
  a coverage report, compiled bytecode or the commit history -- so they carry
  the subject matter as a plain record instead, with the reason stated in that
  folder's own README.

**No manifest, no lockfile, no tool configuration, no runner and no
`trigger.yaml` anywhere in it**, so the folder adds no discovered project and
no task to a run.

See `Tools non triggering (Synthetic Data)/README.md` for the per-tool table, the mechanism each tool is
inert by, and what was measured.
