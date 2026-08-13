# Operator quickstart

Get from a fresh clone to *"I have seen exactly how much of this repo exists"*
in about five minutes.

**Every command below was executed against tip `c68a7e9` on 2026-08-13**, from
a clean checkout, in this order. The output shown is the output observed. Two
steps are *expected to fail* — that is stated where it happens, and a failure
there is the correct result, not a broken quickstart.

Read [`../README.md`](../README.md) first if you have not. The short version:
this repo declares a photo-to-MetaHuman avatar actor and contains none of it,
its two route hosts do not exist, and it cannot be installed as it stands.

## 0. Prerequisites

Versions used for the run recorded here. Nothing is pinned by the repo, so
newer versions are likely fine and older ones untested.

```
node   v26.3.0
pnpm   10.26.2
nbb    (on PATH)
jq     (step 2 only; any recent version)
```

No Cloudflare tooling is needed. There is no deploy config in this repo, so
there is nothing to deploy and no step 5 to skip.

The app directory is long and every step needs it:

```bash
APP=appview/etzhayyim-wasm-image2metahuman-im2mh8n1
```

## 1. Does anything this repo declares exist? (≈5s)

From the repo root:

```bash
nbb docs/check-declared.cljs
```

Observed — **exit 1**, and exit 1 is the expected result today:

```
SCANNED	11 files	DECLARED-HOSTS	4	WORKSPACE-DEPS	2	PACKAGES-HERE	2
control	registry.npmjs.org	resolves

       im2mh8n1.etzhayyim.com  NXDOMAIN  <- appview/…/kotodama.jsonld
image2metahuman.etzhayyim.com  NXDOMAIN  <- appview/…/kotodama.jsonld
           kami.etzhayyim.com  NXDOMAIN  <- appview/…/kotodama.jsonld
           yoro.etzhayyim.com  resolves  <- appview/…/kotodama.jsonld

   @etzhayyim/kami-engine-sdk  NOT-IN-WORKSPACE  <- appview/…/svelte/package.json
 @etzhayyim/kotodama-host-sdk  NOT-IN-WORKSPACE  <- appview/…/package.json

3 of 4 declared hosts do not exist; 2 of 2 workspace dependencies are not in this repo.
```

Three exit codes, and they mean different things:

| exit | meaning |
|---|---|
| `0` | everything declared is backed — the README's status table is stale, fix it |
| `1` | something declared is absent — **today's expected result** |
| `3` | **could not measure.** The control host failed (no DNS here), or zero hosts *and* zero workspace deps were extracted (wrong directory). A pass is never reported from this state. |

If you get `3`, nothing has been learned. Do not read it as good news.

The one host that resolves, `yoro.etzhayyim.com`, is the RACI *informed* party.
The two hosts the actor would actually be served on do not exist.

## 2. What does it claim, and what implements the claim? (≈5s)

```bash
jq -r '"capabilities: \(.profile.capabilities|join(", "))
routes      : \(.routes|map(.pattern)|join(", "))
compliance  : \(.governance.complianceFrameworks|join(", "))"' $APP/kotodama.jsonld
```

Observed:

```
capabilities: metahuman-generation, 3d-rendering, facial-animation, photo-analysis
routes      : image2metahuman.etzhayyim.com/*, im2mh8n1.etzhayyim.com/*
compliance  : MetaHuman-license, biometric-data-minimization, ISO27001
```

Now ask what implements any of it:

```bash
grep -rlE 'metahuman|MetaHuman|DNA|FACS|LOD' $APP/svelte/src || echo '(nothing)'
wc -l $APP/svelte/src/*
```

Observed: `(nothing)`, and

```
       6 main.ts
       5 svelte.d.ts
      32 App.svelte
      43 total
```

43 lines across three files, 27 of which are the `<style>` block inside
`App.svelte`. The entire rendered output is:

```bash
grep -oE '<h1>[^<]*</h1>|<p>[^<]*</p>' $APP/svelte/src/App.svelte
```

```
<h1>image2metahuman</h1>
<p>Vite entry scaffold after SvelteKit cleanup.</p>
```

The compliance frameworks in the descriptor are about handling photographs of
faces. Nothing here handles a photograph.

## 3. Try to install it — this is expected to fail (≈10s)

```bash
cd $APP/svelte
pnpm install
```

Observed — **exit 1**, no `node_modules` created:

```
ERR_PNPM_WORKSPACE_PKG_NOT_FOUND  In : "@etzhayyim/kami-engine-sdk@workspace:*" is in
the dependencies but no package named "@etzhayyim/kami-engine-sdk" is present in the
workspace

This error happened while installing a direct dependency of …/svelte
Packages found in the workspace:
```

The outer package fails the same way on `@etzhayyim/kotodama-host-sdk`:

```bash
cd $APP && pnpm install     # same error, different package name
```

This is the current state of the repo for anyone who clones it. The extraction
from `etzhayyim/root` took the app out of the pnpm workspace that provided both
siblings, and nothing replaced them.

**The obvious fix is a trap.** Nothing under `src/` imports either SDK, so
deleting the dependency would make this step pass — and would erase the only
record of what the app was meant to be built out of, leaving a repo that
installs, builds, and renders a placeholder while looking finished. See the
README. Step 4 works around it in a throwaway copy instead.

## 4. See the scaffold actually render (≈30s)

Copy the app out, drop the unsatisfiable dependency **in the copy only**, and
build that:

```bash
cd /path/to/image2metahuman        # repo root
rm -rf /tmp/i2mh-probe
cp -R $APP/svelte /tmp/i2mh-probe
cd /tmp/i2mh-probe
rm pnpm-lock.yaml
node -e 'const fs=require("fs"),p=JSON.parse(fs.readFileSync("package.json","utf8"));
         delete p.dependencies;
         fs.writeFileSync("package.json",JSON.stringify(p,null,2)+"\n")'
pnpm install
```

`pnpm-lock.yaml` has to go too: it predates the SvelteKit cleanup and still
records `three` and `@pixiv/three-vrm` plus a `file:` link five directories
above the repo root. Keeping it makes the install fail for a second, unrelated
reason.

Observed: installs `svelte 5.56.9`, `vite 6.4.3`, `typescript 5.9.3`, warns
that the build script for `esbuild` was ignored (harmless — the build below
succeeds without approving it), and reports a **peer mismatch that is the
repo's own**: `@sveltejs/vite-plugin-svelte` 4.0.4 wants `vite@^5.0.0` while
`package.json` asks for `vite@^6.4.2`.

Builds in this workspace are serialised repo-wide (CLAUDE.md, resource
governor). Do **not** call `pnpm build` directly:

```bash
node /path/to/com-junkawasaki/scripts/resource-guard.mjs run build -- pnpm build
```

If another session holds the build lock you get
`resource-guard: build is already running (pid=…)` and a non-zero exit. That is
the guard working — wait and retry. The run recorded here was blocked several
times by unrelated repos and succeeded on retry.

Observed on success — **exit 0**:

```
vite v6.4.3 building for production...
[plugin vite:resolve] Module "node:async_hooks" has been externalized for browser
compatibility, imported by ".../svelte/src/internal/server/render-context.js".
✓ 135 modules transformed.
dist/index.html                 0.40 kB │ gzip: 0.27 kB
dist/assets/index-D4mPpWLs.css  0.32 kB │ gzip: 0.24 kB
dist/assets/index-Cf5srVUg.js   2.70 kB │ gzip: 1.31 kB
✓ built in 182ms
```

The `node:async_hooks` line is Svelte's server renderer being externalised for
the browser; it is expected and not an error. The wall-clock figure is not —
two runs of this exact sequence took 182 ms and 911 ms on the same machine.
What *is* reproducible is everything else: both runs transformed 135 modules
and emitted byte-identical assets, down to the content hashes above.

3.4 kB is the honest size of this app. Serve it:

```bash
pnpm preview --port 4327
```

In another shell:

```bash
curl -sS http://localhost:4327/ | grep -oE '<title>[^<]*</title>'
grep -o 'Vite entry scaffold[^<"]*' dist/assets/*.js
```

Observed:

```
<title>image2metahuman</title>
Vite entry scaffold after SvelteKit cleanup.
```

There is no router, so every path returns the same document — `/nope` answers
`200` as readily as `/`. Nothing is missing from the build; there is nothing
else to reach.

Stop the preview:

```bash
pkill -f "preview --port 4327"
```

Not `pkill -f "vite preview"` — `pnpm` execs `…/vite/bin/vite.js preview --port
4327`, so `vite` and `preview` are not adjacent on the command line and the
obvious pattern silently matches nothing.

## Afterwards

Step 3 creates nothing (the install fails before writing). Step 4 works in
`/tmp`, so the repo stays clean — `rm -rf /tmp/i2mh-probe` when done.

If you instead build **inside** the repo, know that `dist/` is not ignored:
`svelte/.gitignore` lists `node_modules`, `.svelte-kit` and `build`, while
`vite.config.ts` sets `outDir: 'dist'`. The build output will show up in
`git status` as untracked. That mismatch is left as it is rather than papered
over with an ignore rule, because which of the two is wrong depends on whether
this app keeps its Vite config — an open question (see the README).

## What this quickstart does not cover

- **Making the app do what it says.** There is no photo analysis, DNA
  calibration, mesh or renderer in this repo to run.
- **Deploying.** No `wrangler.jsonc`, no workflow, and both route hosts are
  NXDOMAIN. There is nothing here that could be published.
- **Resolving where `@etzhayyim/kami-engine-sdk` comes from.** Vendor it,
  publish it, or replace it — the choice determines what this app becomes, so
  it is recorded in the README rather than decided here.
