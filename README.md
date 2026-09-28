# Sprawler packs

First-party rules packs for [Sprawler](https://github.com/clankernative/sprawler). A pack is a TOML
file of architecture policy (layers, what may depend on what, rules with a `why`, views) that a
profile `extends`. Packs merge left to right; a rule with the same `id` replaces the earlier one.

```sh
sprawler pack add ddd-hexagonal cqrs --from path/to/sprawler-packs
# or: SPRAWLER_PACKS_SOURCE=path/to/sprawler-packs sprawler pack add ddd-hexagonal
sprawler pack list
```

These packs are also built into the `sprawler` binary, so `extends = ["ddd-hexagonal"]` works
without installing anything. Install from here to get a newer version than your binary ships.

## Architectures

| Pack | What it adds |
|---|---|
| [`ddd-hexagonal`](architectures/ddd-hexagonal.toml) | Layers core, port, application, driving and driven adapters, root; dependencies point inward |
| [`cqrs`](architectures/cqrs.toml) | Command and query layers; reads never import writes. Add after `ddd-hexagonal` |
| [`vertical-slices`](architectures/vertical-slices.toml) | Slices never import siblings; `shared` never knows its slices. Add after `cqrs` |

```toml
# sprawler.toml
extends = ["ddd-hexagonal", "cqrs", "vertical-slices"]
name = "shop"
root = "."
```

## Platforms

A platform pack describes one platform's layout on top of an architecture: where its tiers live,
the contract seams between its parts, and the views that make sense for it. It extends the
architecture packs and overrides rules where the platform is stricter.

| Pack | Platform |
|---|---|
| [`clankernative`](platforms/clankernative.toml) | [Clankernative](https://github.com/clankernative): Roc application SDK + Rust host. Tiers (apps, SDK, host, tooling, instances), SDK↔host protocol seams, the Operations and SDK ports views |

Sprawler recognises a Clankernative workspace automatically and uses this pack. To write a pack for
your own platform, start from `platforms/clankernative.toml`: `extends` the architectures you
follow, then add `[[tiers]]`, `[[platform_map]]` / `[[app_map]]` path templates, `[[seams]]` and
`[views]`. Every key is documented in Sprawler's
[docs/CONFIG.md](https://github.com/clankernative/sprawler/blob/main/docs/CONFIG.md).

## Languages

| Pack | Notes |
|---|---|
| [`dotnet`](languages/dotnet.toml) | C#/.NET projects: roles from the C# analyzer mapped to layers. Being folded into the C# plugin's defaults |

## Contributing

Every rule needs a `why`. Rules are applied in order and the first match wins, so when you change a
pack, check that overlapping rules keep their order, and compare an atlas before and after on a
real repo. Validate with `sprawler profile validate` on a profile that extends your pack.

MIT — see [LICENSE](LICENSE).
