# TypeScript Order Platform -- Microservices (TS_V12_VITE_BUN_MICRO)

Tool-evaluation repository for **Node 12**, bundled with **vite**,
managed with **bun**, in a **Microservices** layout.

This is branch **TS_V12_VITE_BUN_MICRO** of the consolidated `typescript-corpus` repository, which holds all 216 TypeScript branches across every Node version, bundler, package manager and architecture combination in this corpus.

## Project type

- **Language:** TypeScript 5.0.4
- **Runtime:** Node 12 (verified against 12.22.12)
- **Scenario:** 2 - Microservices
- **Architecture:** Microservices
- **Module layout:** workspace
- **Bundler:** Vite 2.9.18 (Rollup linker + its own bundled esbuild transform)
- **Package manager:** bun 1.4.0
- **Source root:** `packages/domain/src`

Node 12 is end-of-life, and that is deliberate: it pins the entire toolchain to
the last release of each tool that still supports it. Every version in this
repository was resolved against the live npm registry and then executed on a
real Node 12.22.12 interpreter. None was written from memory.

## Build

```bash
# bun is a standalone binary, not an npm package -- see Tool Triggering (Synthetic Data)/npm-audit/run_npm_audit.sh
bun install --frozen-lockfile
bun run build
bun run bundle
```

`bun run build` type-checks with `tsc --noEmit` and emits CommonJS + declarations to
`dist/`; `bun run bundle` then bundles with **Vite 2.9.18 (Rollup linker + its own bundled esbuild transform)** and **executes the bundle**.
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
bun run test        # mocha over tests/
bun run coverage    # c8 (primary) AND nyc + ts-node (cross-check)
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

* **16 tools read source**, so they get their own minimal program, for node12 -- one class or one function, no branching, no
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
