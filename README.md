# Bun: A Practical Guide

<p align="center">
  <img src="assets/bun.png" alt="Bun logo" width="160" />
</p>

A plain-language overview for developers who are new to the ecosystem, plus a short **senior take** at the end for trade-offs and decision-making.

---

## What is Bun?

**Bun** is an all-in-one toolkit for JavaScript and TypeScript on the server and in tooling. Think of it as several tools you might already know, rolled into one program:

| If you know… | Bun offers something similar |
|--------------|------------------------------|
| **Node.js** | A **runtime** that runs `.js` and `.ts` files |
| **npm / yarn / pnpm** | A **package manager** (`bun install`, `bun add`) |
| **esbuild / webpack / Vite** (for bundling) | A **bundler** (`bun build`) |
| **Jest / Vitest** | A **test runner** (`bun test`) |

You do **not** have to use every part. Many teams use Bun only as a package manager or only as a runtime, and keep Node or another bundler for the rest.

**One sentence for juniors:** Bun is a fast way to run JavaScript/TypeScript and manage packages, with extra tools (build and test) built in so you need fewer separate programs.

---

## What can Bun do?

### 1. Run code (runtime)

- Executes JavaScript and **TypeScript without a separate compile step** in the common case.
- Aims to be **largely compatible** with Node.js APIs (`fs`, `path`, `http`, etc.) so many existing scripts and libraries work with little or no change.
- Some packages that rely on **native addons** or deep Node/V8 behavior may need fixes or alternatives. Compatibility is strong but not 100%.

### 2. Manage dependencies (package manager)

- Reads `package.json` like npm.
- Uses a **lockfile** (`bun.lock`) for reproducible installs.
- Often **very fast** installs because of how it is implemented (see “How it is built” below).

### 3. Bundle front-end and back-end code (`bun build`)

- Turns many files into fewer output files for browsers or servers.
- Useful for shipping apps and libraries.

### 4. Run tests (`bun test`)

- Jest-style API in many cases, so tests can look familiar if you have used Jest.

### 5. Small conveniences

- Built-in **`.env` loading** in many workflows.
- **Watch mode** and scripting via `package.json` scripts with `bun run`.

---

## How is Bun built? (Simple version)

You do not need to memorize this to use Bun, but it helps explain *why* it feels different from Node.

### Language: Zig (and friends)

- The core of Bun is written in **[Zig](https://ziglang.org/)**, a systems language focused on clarity and performance.
- Parts of the stack also use **C++** where that fits (for example, around the JavaScript engine integration).

### JavaScript engine: JavaScriptCore (not V8)

- **Node.js** uses **Google’s V8** engine (same family as Chrome).
- **Bun** uses **JavaScriptCore (JSC)**, the engine from **WebKit** (Safari’s lineage).

Same language (JavaScript), **different engine** underneath. That is why:

- Performance profiles differ (some workloads faster on Bun, some edge cases behave differently).
- A tiny fraction of packages that assume V8-specific details may break or need updates.

### Design goals

- **Speed** for startup, installs, and many I/O-heavy tasks.
- **Batteries included**: one binary often replaces several tools.
- **Node compatibility** as a guiding principle, implemented over time rather than promised for every package on day one.

---

## The future of Bun

No one can predict the future perfectly, but these trends are useful to keep in mind:

1. **Maturing compatibility**: More npm packages and Node APIs work over time; gaps usually shrink with each release.
2. **Ecosystem choice**: Teams may standardize on “Bun everywhere,” “Bun for installs only,” or “Node in production, Bun locally.” All of these are common patterns while the ecosystem settles.
3. **Competition is healthy**: Node.js continues to evolve (performance, TypeScript story, built-in tools). Bun pushes the whole space forward; both can coexist for years.
4. **Production readiness**: Many companies run Bun in production; others stay on Node for policy, support, or native-module reasons. **Your** future depends on your org’s risk tolerance and what your dependencies support.

**For juniors:** It is reasonable to learn **Node concepts first** (modules, `package.json`, async I/O) because they transfer to Bun. Learning Bun afterward is mostly learning *new commands* and *small differences*, not a whole new world.

---

## Bun vs Node.js

### Side-by-side (junior-friendly)

| Topic | **Node.js** | **Bun** |
|--------|-------------|---------|
| **What it is** | The long-established JavaScript runtime; huge ecosystem. | A newer runtime + package manager + bundler + test runner in one. |
| **Engine** | V8 (Chrome’s JS engine). | JavaScriptCore (WebKit’s JS engine). |
| **TypeScript** | Run via `ts-node`, transpilers, or build step. | Often run **directly** with `bun run file.ts`. |
| **Package manager** | Separate tool: npm, yarn, pnpm, etc. | **`bun install`** built in (optional). |
| **Mindshare & jobs** | Extremely high; most tutorials assume Node. | Growing; fewer job postings say “Bun only.” |
| **Compatibility** | Reference for server-side JS; most libs target Node first. | Very good for many apps; check critical dependencies. |

### When juniors often reach for each

- **Default to Node** if your course, team, or deployment platform **standardizes on Node**; fewer surprises.
- **Try Bun** when you want **faster installs**, a **single tool**, or the team has **checked** that your stack works on Bun.

### Commands you might compare

```bash
# Node (examples)
node app.js
npm install
npm test

# Bun (examples)
bun run app.js
bun install
bun test
```

---

## Senior perspectives (trade-offs and decisions)

This section is for tech leads, staff engineers, and anyone choosing a platform for production.

### Compatibility and risk

- **Native addons** (`node-gyp`, N-API) and **deep V8 assumptions** remain the highest-risk areas when moving from Node to Bun.
- **Polyfills and subtle semantics** (timers, streams, error objects) can differ; invest in **integration tests** and **staging**, not only “it runs locally.”

### Operational and organizational factors

- **Support contracts, security reviews, and base images** may standardize on Node LTS; adopting Bun may require explicit approval paths.
- **Observability** (APM, profiling) is mature for Node everywhere; for Bun, verify your vendors and tooling.
- **Talent and onboarding**: Node skills are universal; Bun adds a second mental model (JSC vs V8, lockfile format, CLI flags).

### Performance claims

- Bun often wins on **cold start**, **install**, and **certain I/O patterns**; always **measure your workload**. Microbenchmarks rarely equal your monolith or your CI pipeline.
- **CPU-bound** or **allocation-heavy** JS may not show the same gap; profile before optimizing the runtime choice.

### Pragmatic strategies teams use

- **Bun as package manager only** (`bun install`) with Node running the app: low risk, quick win on install time.
- **Bun in dev, Node in prod** until confidence is high: common middle ground.
- **Full Bun** when dependencies are green and ops sign off: reasonable for many greenfield services.

### Bottom line

Bun is a **credible, fast, developer-focused** platform that competes with Node on ergonomics and speed. Node remains the **default compatibility and hiring baseline**. The best choice is usually **the one your dependencies, platform, and team can support**, with a path to revisit as both runtimes evolve.

---

## Further reading

- [Bun official documentation](https://bun.sh/docs)
- [Node.js documentation](https://nodejs.org/docs/latest/api/)

---
