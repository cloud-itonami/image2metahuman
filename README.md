# image2metahuman

**A declaration of a photo-to-MetaHuman avatar generator, without the
generator.** `kotodama.jsonld` describes an autonomous actor that takes a
photograph and produces a MetaHuman DNA calibration with FACS facial
animation, LOD0–LOD3 meshes and multi-layer skin SSS. What is actually in this
repo is that description, a licence notice, extraction provenance, and a
reagent + re-frame ClojureScript scaffold whose entire rendered output is a
heading and the sentence *"Vite entry scaffold after SvelteKit cleanup."* —
yes, still that sentence; see [below](#2026-08-26-svelte--clojurescript)
for why it survived a framework migration unchanged.

There is **no photo analysis, no DNA calibration, no mesh, and no renderer
here** — not an incomplete one, none. Read this repo as a *claim staked*, not as
software.

It declares itself `kind :app` (`README.edn`) and was extracted verbatim from
`etzhayyim/root` at `60-apps/etzhayyim-project-image2metahuman`
(`migration.edn`, source revision `c3a74d20d7`, 13 files / 46,413 bytes). That
extraction is unaffected by anything below — `migration.edn` is provenance for
*that* event, not a description of the current tree, and is not rewritten
when the tree changes.

## Status: nothing it declares is reachable — measured 2026-08-26

Measured on a clean checkout of this branch, after the Svelte → ClojureScript
frontend migration described below. Re-take the measurement rather than
trusting the table: **`kbb --backend sci docs/check-declared.cljk`** (see
[the quickstart](docs/operator-quickstart.md)).

| declared thing | declared in | exists? |
|---|---|---|
| `image2metahuman.etzhayyim.com` | `kotodama.jsonld` `routes` | **NXDOMAIN** |
| `im2mh8n1.etzhayyim.com` | `kotodama.jsonld` `routes`, `@id` | **NXDOMAIN** |
| `kami.etzhayyim.com` | `kotodama.jsonld` RACI *consulted* | **NXDOMAIN** |
| `yoro.etzhayyim.com` | `kotodama.jsonld` RACI *informed* | resolves |
| `@etzhayyim/kotodama-host-sdk` | `appview/…/package.json` `workspace:*` | **not in this repo** |

A control lookup against `registry.npmjs.org` resolved in the same run, so the
NXDOMAINs are genuine absences and not a broken resolver here. The row for
`@etzhayyim/kami-engine-sdk` (`svelte/package.json` `workspace:*`) that used
to be in this table is gone because `svelte/` is gone — see below.

Two consequences, and they are different in kind:

- **The actor is not deployed.** Neither route host exists, and there is no
  deploy config in the repo (no `wrangler.jsonc`, no workflow) that would put
  one there.
- **The frontend now installs and builds cleanly; the backend still does
  not.** `appview/…/cljs/` (below) has no dangling dependency and its build is
  verified green. The outer `appview/…/package.json` — the Worker entry
  point's own manifest, untouched by this migration — still depends on the
  `workspace:*` sibling `@etzhayyim/kotodama-host-sdk` that the original
  extraction from `etzhayyim/root` left behind:

  ```
   ERR_PNPM_WORKSPACE_PKG_NOT_FOUND  In : "@etzhayyim/kotodama-host-sdk@workspace:*" is
  in the dependencies but no package named "@etzhayyim/kotodama-host-sdk" is present in
  the workspace
  ```

  That is a backend concern (the XRPC/Worker entry point), out of scope for a
  frontend migration, and this migration did not touch it.

## 2026-08-26: Svelte → ClojureScript

`appview/…/svelte/` — the Vite + Svelte 5 scaffold described below until this
date — has been replaced by `appview/…/cljs/`: the identical scaffold
(same heading, same paragraph, same layout), ported to reagent + re-frame,
rendered with `jp-go-dds` (デジタル庁デザインシステム) hiccup, as part of this
workspace's repo-wide Svelte retirement (`svelte-cljs-wave`).

**On the dangling dependency.** The deleted `svelte/package.json` carried an
unused `@etzhayyim/kami-engine-sdk` `workspace:*` dependency — nothing under
`svelte/src/` ever imported it (`grep -r kami-engine-sdk` returned nothing,
same as before). The previous revision of this README warned against deleting
that dependency on its own, on the grounds that it was "the only thing in the
repo that records what this app was supposed to be built out of," and that
removing it would make an unfinished app look finished. That warning was
about a *narrow* fix — dropping the one line to make `pnpm install` pass while
leaving everything else as-is. What happened instead is a **wholesale
frontend replacement**, workspace-wide and not specific to this repo, and the
replacement is exactly as much of a placeholder as what it replaced (same
rendered text, no capability implemented). The intent the dangling dependency
recorded — that a real implementation here would need a 3D avatar
renderer — is not lost: this workspace's repo-wide rule ("3D はすべて
kami-engine を使う") already requires that of *any* future MetaHuman/3D work
in this workspace, dangling `package.json` line or not. `cljs/src/image2metahuman/app.cljs`
says so explicitly in its own docstring, so the record persists in prose
instead of in an unresolvable dependency declaration.

**Verified**, on this branch, before merge:

```
$ npm install                       # appview/…/cljs — no workspace:* dependency, installs clean
$ amu compile --target wasm32-browser app       # [:app] Build completed. (111 files, 110 compiled, 0 warnings, 41.84s)
$ amu compile --target wasm32-browser test      # [:test] Build completed. (112 files, 111 compiled, 0 warnings, 9.65s)
$ node out/tests.js                 # Ran 4 tests containing 6 assertions. 0 failures, 0 errors.
```

`svelte/pnpm-lock.yaml` — which predated the SvelteKit cleanup and still
recorded `three`/`@pixiv/three-vrm` and a `file:` path five directories above
this repo's root — is gone with the rest of `svelte/`. That fossil is no
longer this repo's problem to reconcile.

## What is actually in here

Four pieces, none of which implement the product.

### 1. `appview/…/kotodama.jsonld` — the actor descriptor

`did:web:im2mh8n1.etzhayyim.com`, `runtimeType: worker`, `uiType: appview`. It
advertises four capabilities (`metahuman-generation`, `3d-rendering`,
`facial-animation`, `photo-analysis`), a `convoSystemPrompt`, two route
patterns, and a `subscribeRepos` trigger on four `app.bsky.feed|graph`
collections.

It also carries real governance intent: `classification: restricted`, and
compliance frameworks `MetaHuman-license`, `biometric-data-minimization`,
`ISO27001`. Those frameworks are about handling photographs of faces. **Nothing
in this repo processes a photograph**, so nothing implements them either — they
describe obligations that would attach the moment someone does.

### 2. `appview/…/cljs/` — a reagent + re-frame scaffold, rendered with jp-go-dds

`src/image2metahuman/app.cljs` holds the same two strings the old
`App.svelte` rendered as bare markup literals — a heading and one paragraph —
as re-frame app-db data instead (`:page/heading`, `:page/description`), with
a `reg-event-db`/`reg-sub` pair and `4` tests / `6` assertions covering them.
There is no router, no route table, and no second view; every path serves the
same document, same as before.

It builds clean: `amu compile --target wasm32-browser app` reports `111 files, 110 compiled, 0
warnings` (measured 2026-08-26). `deps.edn` keeps reagent/re-frame/
clojurescript/shadow-cljs under the `:cljs` alias rather than top-level
`:deps`, per this workspace's `jvm-new-surface-guard` PreToolUse hook
(ADR-2608201300, which denies new top-level JVM runtime deps) — only
`org.clojure/clojure` and the `jp-go-digital-design-system` git dependency are
top-level. `public/index.html`'s inlined `<style>` was generated once, at
authoring time, via `jp-go-dds.page/->page` on the JVM (see the regeneration
recipe in `app.cljs`'s docstring) — the browser bundle itself only needs
`jp-go-dds.core`.

### 3. `README.edn` / `migration.edn` / `NOTICE` — identity and provenance

All three parse. Two disagreements with reality worth knowing before you rely
on them:

- `migration.edn` names the destination as
  `etzhayyim/com-etzhayyim-app-image2metahuman`; the repo actually lives at
  **`cloud-itonami/image2metahuman`**. The move happened; the record did not
  follow.
- `README.edn` declares `:kind :app`, but this workspace's classifier reads the
  repo as *unclassified*. It calls something an app when a root `src/` sits next
  to a UI marker; here it finds the UI marker (`kotodama.jsonld`'s
  `uiType: appview`, recorded as the `:ui` trait) but no root `src/`, because
  the sources are five levels down under `appview/…/cljs/src/`. The declaration
  and the observation disagree; neither has been corrected in favour of the
  other. This did not change with the Svelte → ClojureScript migration — the
  Svelte sources were equally deep, under `appview/…/svelte/src/`.

### 4. `docs/` — this README's evidence

`docs/check-declared.cljk` re-measures the status table above.
`docs/operator-quickstart.md` walks the repo end to end.

## If someone implements this

Two constraints already apply to this repo and are easy to miss:

- **ADR-2607052000** (`90-docs/adr/…-kami-engine-sdk-svelte-retirement-cljs-migration.edn`)
  retires Svelte from `kami-engine-sdk` in favour of ClojureScript, and named
  `image2metahuman` explicitly among the consumer apps that were, at the time,
  **out of scope** — they could "keep using the current published Svelte
  package at their pinned version until they separately choose to migrate."
  That carve-out assumed a *pinned published* Svelte version; this repo never
  had one (it depended on an unresolvable `workspace:*` sibling instead), so
  the assumption the ADR's exemption rests on never actually held here. The
  2026-08-26 migration above resolves that mismatch by migrating anyway, as
  part of a workspace-wide wave rather than a per-repo decision.
- **3D in this workspace goes through `kami-engine`**, WebGPU first with a
  WebGL 2.0 fallback, consuming the canonical EDN render-IR. A second renderer
  — three.js, Babylon, or a hand-rolled one — is not an option here, whatever
  the fossilised `svelte/pnpm-lock.yaml` used to suggest (that lockfile is now
  gone along with the rest of `svelte/`).

## Boundary with the nearest repos

- **`cloud-itonami/image2vrm`** — the sibling with the same shape and the same
  extraction, targeting **VRM** avatars instead of MetaHuman. Unlike this one it
  has a written design (`docs/character-maker-design.md`) and a CLAUDE.md
  describing a working KAMI Engine wgpu pipeline. If you are looking for how
  this repo was *meant* to work, read that one.
- **`kotoba-lang/character`** — the portable `.cljc` character model. Engine
  side, no app surface, no MetaHuman specifics.
- **`kotoba-lang/kami-gen-procedural`** — deterministic EDN-parameter character
  generation. It composes characters from parameters; this repo's claim is to
  derive them from a photograph. Same output category, opposite input.

## Operator entry point

**[`docs/operator-quickstart.md`](docs/operator-quickstart.md)** — every step in
it was executed against this tip. It says which commands succeed, which one is
expected to fail, and what each prints.

## Licence

Apache 2.0 with the etzhayyim Charter Compliance Rider v3.1 — see `NOTICE`.
