# dominium

> Deterministic mean-field simulator with vector agents — civilization, faith, war and money **emerge** from the arithmetic — in Rust, with a [Llimphi](https://github.com/tawasuyu/llimphi) UI.

`dominium` is a bit-for-bit deterministic simulator: thousands of vector agents (lemmings) live on a dense numeric substrate of **five physical layers** (`materia`, `psique`, `poder`, `oro`, `degradacion`), acting through six atomic actions (move, take, drop, transmit, attack, rest). Nobody scripts a society — the engine only adds floats, and civilization, belief, conflict and trade come out of the coupling. **Concepts** (church, bank, commune, lab) are metaprogrammable field emitters loaded as JSON at runtime: rewrite the domain without recompiling.

<p align="center">
  <img src="docs/dominium_showreel.gif" alt="dominium showreel — an isometric procedural continent of blue seas, green plains and sienna ranges, populated by thousands of lemmings, with Concepts emitting fields and ψ-cluster recoloring over a live mean-field run" width="900">
  <br>
  <sub>a live mean-field run — the isometric continent, thousands of lemmings, Concepts emitting their fields, and ψ-cluster recoloring</sub>
</p>

## Run

```sh
# deterministic headless CLI
cargo run --release -p dominium-cli -- run --seed 42 --ticks 1000

# the live Llimphi app — isometric canvas + control panel (play/pause/reseed, sliders)
cargo run --release -p dominium-app-llimphi
```

## Install

```sh
cargo build --release
# binaries land in target/release/
```

- **Linux / macOS / Windows** — Llimphi UI.
- **Wawa / WASM** — `dominium-core`, `dominium-physics`, `dominium-iso`, `dominium-render-plan` compile with zero graphical deps.
- **Web** — via `dominium-notebook-kernel` (a pluma notebook kernel).

## Crates

| Crate | Role |
|---|---|
| `dominium-core` | Data + the six atomic actions + Concepts + worldgen. SoA grid of 5 layers, vector lemmings. Only `serde` + `libm`. |
| `dominium-physics` | The 6-phase tick: field diffusion + entropy + transitions, actions, aging. Bit-exact. |
| `dominium-sim` | Frontend-agnostic simulation session: World + clock (tick/epoch/seed) + snapshot ring (rewind) + trails + k-means clusters. |
| `dominium-iso` | CPU pseudo-3D isometric projection: fixed iso matrix + composite Z from 5 layers + analytic Lambert shadows. |
| `dominium-render-plan` | World → depth-sorted `Vec<Quad>`, ready for any backend. |
| `dominium-canvas-llimphi` | Llimphi/vello backend: a `RenderPlan` → `View<Msg>` with `paint_with`. |
| `dominium-app-llimphi` | The app — canvas + stats panel + play/pause/reseed, tick loop on its own thread re-entering via `Handle::dispatch`. |
| `dominium-cli` | Headless runner: cross-platform determinism checks, per-tick stats CSV, Concept packs. |
| `dominium-notebook-kernel` | Notebook kernel running cells over a World (5 languages: world/seed/tick/stats/param) on shared state. |

The chain `core → physics → iso → render-plan` is fully graphics-free; only the `*-llimphi` crates pull the GPU UI stack.

## How dependencies work

dominium is a full app. Llimphi (the sovereign GPU engine: Elm loop, theme, widgets) and every foundational dependency are pulled as **git dependencies** from the [`tawasuyu`](https://github.com/tawasuyu/tawasuyu) monorepo — the suite's source of truth. The simulation crates (`dominium-core/physics/sim/iso/render-plan`) are pure compute and depend only on `serde` + `libm`; the UI crates pull the GPU stack via git-dep.

## Considerations

- **Inviolable rule:** zero graphical deps in `core` / `physics` / `iso` / `render-plan`. Only `serde` and `libm`. Graphics live in `canvas-llimphi` / `app-llimphi`.
- **Bit-for-bit deterministic** given the same seed and the same version.
- **Concepts load at runtime** (`id + pos + radius + mods + hack`); the engine stays dumb, external AI is optional. They let you rewrite the domain without recompiling.
- **Everything non-fixed is data.** `SimParams` / `ZWeights` are serializable and live-editable from panel sliders; a whole scenario (params + relief + Concepts) serializes as one.

## License

MIT. Builds on [Llimphi](https://github.com/tawasuyu/llimphi) and the [tawasuyu](https://github.com/tawasuyu/tawasuyu) suite.
