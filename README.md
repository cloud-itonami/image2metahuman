# image2metahuman

**A declaration of a photo-to-MetaHuman avatar generator, without the
generator.** `kotodama.jsonld` describes an autonomous actor that takes a
photograph and produces a MetaHuman DNA calibration with FACS facial
animation, LOD0–LOD3 meshes and multi-layer skin SSS. What is actually in this
repo is that description, a licence notice, extraction provenance, and a Svelte
scaffold whose entire rendered output is a heading and the sentence *"Vite entry
scaffold after SvelteKit cleanup."*

There is **no photo analysis, no DNA calibration, no mesh, and no renderer
here** — not an incomplete one, none. Read this repo as a *claim staked*, not as
software.

It declares itself `kind :app` (`README.edn`) and was extracted verbatim from
`etzhayyim/root` at `60-apps/etzhayyim-project-image2metahuman`
(`migration.edn`, source revision `c3a74d20d7`, 13 files / 46,413 bytes). The
extraction is the repo's only commit.

## Status: nothing it declares is reachable — measured 2026-08-13

Measured against tip `c68a7e9` — the extraction commit, the only content this
repo had before this README. Re-take the measurement rather than trusting the
table: **`nbb docs/check-declared.cljs`** (see
[the quickstart](docs/operator-quickstart.md)).

| declared thing | declared in | exists? |
|---|---|---|
| `image2metahuman.etzhayyim.com` | `kotodama.jsonld` `routes` | **NXDOMAIN** |
| `im2mh8n1.etzhayyim.com` | `kotodama.jsonld` `routes`, `@id` | **NXDOMAIN** |
| `kami.etzhayyim.com` | `kotodama.jsonld` RACI *consulted* | **NXDOMAIN** |
| `yoro.etzhayyim.com` | `kotodama.jsonld` RACI *informed* | resolves |
| `@etzhayyim/kami-engine-sdk` | `svelte/package.json` `workspace:*` | **not in this repo** |
| `@etzhayyim/kotodama-host-sdk` | `appview/…/package.json` `workspace:*` | **not in this repo** |

A control lookup against `registry.npmjs.org` resolved in the same run, so the
NXDOMAINs are genuine absences and not a broken resolver here.

Two consequences, and they are different in kind:

- **The actor is not deployed.** Neither route host exists, and there is no
  deploy config in the repo (no `wrangler.jsonc`, no workflow) that would put
  one there.
- **The repo cannot be installed as it stands.** Both `package.json` files
  depend on `workspace:*` siblings that the extraction left behind. `pnpm
  install` fails:

  ```
  ERR_PNPM_WORKSPACE_PKG_NOT_FOUND  "@etzhayyim/kami-engine-sdk@workspace:*" is in
  the dependencies but no package named "@etzhayyim/kami-engine-sdk" is present in
  the workspace
  ```

  This is not a stale error message. It is the current state of the repo for
  anyone who clones it.

## The dangling dependency is the interesting part

Nothing under `src/` imports either SDK — `grep -r kami-engine-sdk` over the
TypeScript and Svelte sources returns nothing. So the dependency is unused, and
deleting it would make `pnpm install` succeed in about ten seconds.

**Do not delete it.** It is the only thing in the repo that records what this
app was supposed to be built out of. Removing it would turn a visibly unfinished
app into an apparently finished one that renders a placeholder — the failure
would stop being legible. A throwaway copy with the dependency removed builds
fine and is how the quickstart lets you see the scaffold render; that copy is a
probe, not a fix.

The lockfile says the same thing from the other side. `svelte/pnpm-lock.yaml`
(1,222 lines) predates the SvelteKit cleanup and disagrees with the
`package.json` next to it:

| | `package.json` declares | `pnpm-lock.yaml` records |
|---|---|---|
| `@etzhayyim/kami-engine-sdk` | `workspace:*` | `file:../../../../../packages/engine/kami-engine/kami-engine-sdk` |
| `@pixiv/three-vrm` | — | `^3.3.3` → 3.5.1 |
| `three` | — | `^0.170.0` → 0.170.0 |

That `file:` path climbs five directories above this repo's root. It is the old
monorepo layout, fossilised. The three.js pair it locks looks equally dated:
the sibling repo `cloud-itonami/image2vrm` records in its `CLAUDE.md` that
`@etzhayyim/kami-engine-sdk` removed all three.js code paths on 2026-05-26,
leaving the KAMI Engine wgpu path as the sole renderer. That is a second-hand
citation — it names an ADR (`ADR-2605264300`) that is **not** in this
workspace's ADR set — so treat it as a lead, not as established fact.

**Recorded, not repaired.** Reconciling them means deciding where
`@etzhayyim/kami-engine-sdk` comes from now — vendored, published to a registry,
or replaced — and that decision determines what this app will be. It is not a
documentation change.

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

### 2. `appview/…/svelte/` — a Vite + Svelte 5 scaffold

`App.svelte` is 32 lines, 27 of which are a `<style>` block. It renders one
`<h1>` and one `<p>`. `main.ts` mounts it. There is no router, no route table,
and no second view; every path serves the same document.

It does build, once the dangling dependency is worked around: 135 modules,
`dist/` at 0.40 kB HTML + 0.32 kB CSS + 2.70 kB JS. That number is the honest
size of what exists.

`package.json` also has a peer mismatch that `pnpm` reports on install:
`@sveltejs/vite-plugin-svelte` 4.0.4 wants `vite@^5.0.0`, while the same file
asks for `vite@^6.4.2`. The build succeeds anyway.

### 3. `README.edn` / `migration.edn` / `NOTICE` — identity and provenance

All three parse. Two disagreements with reality worth knowing before you rely
on them:

- `migration.edn` names the destination as
  `etzhayyim/com-etzhayyim-app-image2metahuman`; the repo actually lives at
  **`cloud-itonami/image2metahuman`**. The move happened; the record did not
  follow.
- `README.edn` declares `:kind :app`, but this workspace's classifier reads the
  repo as *unclassified*. It calls something an app when a root `src/` sits next
  to a UI marker; here it finds the UI marker (`index.html`, recorded as the
  `:ui` trait) but no root `src/`, because the sources are five levels down under
  `appview/`. The declaration and the observation disagree; neither has been
  corrected in favour of the other.

### 4. `docs/` — this README's evidence

`docs/check-declared.cljs` re-measures the status table above.
`docs/operator-quickstart.md` walks the repo end to end.

## If someone implements this

Two constraints already apply to this repo and are easy to miss:

- **ADR-2607052000** (`90-docs/adr/…-kami-engine-sdk-svelte-retirement-cljs-migration.edn`)
  retires Svelte from `kami-engine-sdk` in favour of ClojureScript, and names
  `image2metahuman` explicitly among the consumer apps that are **out of scope** —
  they "keep using the current published Svelte package at their pinned version
  until they separately choose to migrate". Note the assumption in that
  sentence: a *pinned published* version. This repo pins nothing and depends on
  a workspace sibling, so it is not in the state the ADR assumes its consumers
  are in.
- **3D in this workspace goes through `kami-engine`**, WebGPU first with a
  WebGL 2.0 fallback, consuming the canonical EDN render-IR. A second renderer
  — three.js, Babylon, or a hand-rolled one — is not an option here, whatever
  the fossilised lockfile suggests.

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
