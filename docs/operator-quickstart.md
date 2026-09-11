# Operator quickstart

Get from a fresh clone to *"I have seen exactly how much of this repo exists"*
in about five minutes.

**Every command below was executed against this branch on 2026-08-26**, from
a clean checkout, in this order, right after the Svelte → ClojureScript
frontend migration (see the README). The output shown is the output
observed. One step used to be *expected to fail* for the frontend; since the
migration it isn't — the frontend now installs and builds clean. The backend
install still fails, unchanged, and that is noted where it happens.

Read [`../README.md`](../README.md) first if you have not. The short version:
this repo declares a photo-to-MetaHuman avatar actor and contains none of it,
its two route hosts do not exist, and its backend cannot be installed as it
stands (its frontend now can).

## 0. Prerequisites

Versions used for the run recorded here. Nothing is pinned by the repo, so
newer versions are likely fine and older ones untested.

```
node      v26.3.0
npm       (bundled with node)
clojure   (the CLI — shadow-cljs shells out to it via deps.edn's :cljs alias)
nbb       (on PATH)
jq        (step 2 only; any recent version)
```

No Cloudflare tooling is needed. There is no deploy config in this repo, so
there is nothing to deploy and no separate deploy step to skip.

The app directory is long and every step needs it:

```bash
APP=appview/etzhayyim-wasm-image2metahuman-im2mh8n1
```

## 1. Does anything this repo declares exist? (≈5s)

From the repo root:

```bash
nbb docs/check-declared.cljk
```

Observed — **exit 1**, and exit 1 is the expected result today:

```
SCANNED	9 files	DECLARED-HOSTS	4	WORKSPACE-DEPS	1	PACKAGES-HERE	2
control	registry.npmjs.org	resolves

       im2mh8n1.etzhayyim.com  NXDOMAIN  <- appview/…/kotodama.jsonld
image2metahuman.etzhayyim.com  NXDOMAIN  <- appview/…/kotodama.jsonld
           kami.etzhayyim.com  NXDOMAIN  <- appview/…/kotodama.jsonld
           yoro.etzhayyim.com  resolves  <- appview/…/kotodama.jsonld

 @etzhayyim/kotodama-host-sdk  NOT-IN-WORKSPACE  <- appview/…/package.json

3 of 4 declared hosts do not exist; 1 of 1 workspace dependencies are not in this repo.
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

The `WORKSPACE-DEPS` count dropped from 2 to 1 in this migration:
`@etzhayyim/kami-engine-sdk`, declared by the now-deleted
`svelte/package.json`, is gone along with `svelte/`. The remaining one,
`@etzhayyim/kotodama-host-sdk`, is the backend Worker entry point's own
manifest — untouched by this migration, still unresolved.

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
grep -rlE 'metahuman|MetaHuman|DNA|FACS|LOD' $APP/cljs/src || echo '(nothing)'
wc -l $APP/cljs/src/image2metahuman/app.cljs $APP/cljs/test/image2metahuman/app_test.cljs
```

Observed — and this is a genuine, small regression in the check's precision
that this migration introduced, not a hidden implementation:

```
appview/etzhayyim-wasm-image2metahuman-im2mh8n1/cljs/src/image2metahuman/app.cljk
      88 appview/etzhayyim-wasm-image2metahuman-im2mh8n1/cljs/src/image2metahuman/app.cljk
      29 appview/etzhayyim-wasm-image2metahuman-im2mh8n1/cljs/test/image2metahuman/app_test.cljk
```

The old Svelte scaffold's grep here returned `(nothing)`, because Svelte
doesn't have namespaces — `App.svelte`/`main.ts` never spelled out the app's
name. ClojureScript does: the namespace is `image2metahuman.app`, so the
literal substring `metahuman` appears in the one place a namespace form has
to appear, whether or not anything is implemented. The docstring adds three
more matches (`MetaHuman` ×2, `DNA` ×1) because it explains, in prose, what
this scaffold does *not* do — quoting the product claim to say what's
missing has the same substring as making the claim. **Read the match, don't
just count it**: open the file and see that every occurrence is either the
namespace declaration or a sentence about absence, never a mesh, a DNA
calibration routine, or a FACS/LOD implementation. Most of `app.cljs`'s line
count is that explanatory docstring — the actual state/view/mount code is
about 20 lines. The entire rendered output is still:

```bash
grep -n ':page/heading\|:page/description' $APP/cljs/src/image2metahuman/app.cljs
```

```
:page/heading "image2metahuman"
:page/description "Vite entry scaffold after SvelteKit cleanup."
```

The compliance frameworks in the descriptor are about handling photographs of
faces. Nothing here handles a photograph.

## 3. Install and build the frontend — this now succeeds (≈15s)

```bash
cd $APP/cljs
npm install
```

Observed: `added 129 packages, and audited 130 packages` — clean, no
`workspace:*` dependency to fail on (unlike the deleted `svelte/package.json`,
`cljs/package.json` depends only on `react`/`react-dom`/`shadow-cljs` from the
public npm registry; reagent/re-frame/clojurescript/shadow-cljs's *JVM-side*
classpath comes from `deps.edn`'s `:cljs` alias, resolved by the `clojure`
CLI, not by npm).

Builds in this workspace are serialised repo-wide (CLAUDE.md, resource
governor). Do **not** call `shadow-cljs` directly:

```bash
node /path/to/com-junkawasaki/scripts/resource-guard.mjs run build -- npx shadow-cljs compile app
```

If another session holds the build lock you get
`resource-guard: build is already running (pid=…)` and exit code `2`. That is
the guard working — wait and retry; several unrelated builds share this lock
across the fleet, so more than one `exit 2` in a row is normal.

Observed on success — **exit 0**:

```
[:app] Compiling ...
[:app] Build completed. (111 files, 110 compiled, 0 warnings, 41.84s)
```

Then the test build, the same way:

```bash
node /path/to/com-junkawasaki/scripts/resource-guard.mjs run build -- npx shadow-cljs compile test
```

```
[:test] Compiling ...
[:test] Build completed. (112 files, 111 compiled, 0 warnings, 9.65s)
```

Then run the compiled tests directly (no lock needed — this is `node`, not a
compile):

```bash
node out/tests.js
```

```
Testing image2metahuman.app-test
re-frame: Subscribe was called outside of a reactive context.
 https://day8.github.io/re-frame/FAQs/UseASubscriptionInAnEventHandler/
[... repeated 3 more times ...]

Ran 4 tests containing 6 assertions.
0 failures, 0 errors.
```

The repeated re-frame warning is expected: the tests `deref` a subscription
directly outside a Reagent render, which is exactly what a unit test does.
It is a warning about *how the test observes state*, not a test failure — the
result line under it is the one that matters, and it's clean.

**The backend still fails, unchanged, and that is out of scope here:**

```bash
cd $APP && pnpm install     # backend Worker entry point's own package.json
```

```
 ERR_PNPM_WORKSPACE_PKG_NOT_FOUND  In : "@etzhayyim/kotodama-host-sdk@workspace:*" is in
the dependencies but no package named "@etzhayyim/kotodama-host-sdk" is present in the
workspace

This error happened while installing a direct dependency of $APP
```

Observed — **exit 1**, no `node_modules` created (checked with the shell's own
`$?` on `pnpm install` itself, not on whatever it's piped through — piping the
output to `tail` and reading `$?` afterward reports `tail`'s exit code, not
`pnpm`'s, and would silently read this failure as a success).

This is the same failure the README's status table reports (§1 above). It
predates this migration and is not touched by it — `kotoba/src/registry.ts`
(if present) and `src/app.ts`, the Worker-side code, are unaffected.

## 4. See the scaffold actually render (≈10s)

Unlike the old Vite/pnpm setup, there is no separate preview server step —
`public/index.html` is a complete, self-contained document (DADS CSS inlined
by `jp-go-dds.page/->page` at authoring time) that loads `js/app.js`, produced
by step 3's `shadow-cljs compile app`, via a plain relative `<script>` tag.
Open it directly:

```bash
open $APP/cljs/public/index.html          # macOS; any browser works
```

Observed: a centered heading reading "image2metahuman" and, below it, the
paragraph "Vite entry scaffold after SvelteKit cleanup." — same text, same
layout intent (`main { display: grid; place-content: center; }` in the old
Svelte CSS, `dds-ext-hero dds-ext-center` in the new hiccup), styled with the
デジタル庁デザインシステム instead of the old hand-written dark theme in
`svelte/src/app.css`. There is no router, so this is the only view there is
to reach — same as before.

If you want a live-reloading dev loop instead of a one-shot build:

```bash
npx shadow-cljs watch app
```

then open (or reload) the same `public/index.html`. `Ctrl-C` to stop.

## Afterwards

`public/js/` (the build output) and `node_modules/` are both gitignored —
`git status` stays clean after building. `out/tests.js` (the test build
output) is gitignored too.

If you build **inside** the repo (as above, rather than in a scratch copy),
know that this is the intended way to build it now — there is no more
`dist/` vs. tracked-files mismatch to worry about the way the old
`svelte/.gitignore`/`vite.config.ts` pair had (see the pre-migration history
of this file in `git log` if you want that story).

## What this quickstart does not cover

- **Making the app do what it says.** There is no photo analysis, DNA
  calibration, mesh or renderer in this repo to run.
- **Deploying.** No `wrangler.jsonc`, no workflow, and both route hosts are
  NXDOMAIN. There is nothing here that could be published.
- **Fixing the backend's dangling dependency.** `@etzhayyim/kotodama-host-sdk`
  is still `workspace:*` and still unresolved. Vendor it, publish it, or
  replace it — that choice determines what the Worker side of this app
  becomes, and it is a separate decision from the frontend migration recorded
  here.
