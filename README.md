# karen_strikes_back

A 5-profile **Hermes Agent** fleet for adversarial & usability review of code,
software, and UI. Point it at a repo and get a combined, severity-ranked report
from five distinct, opinionated lenses.

> The name is a joke: this is the "review pod" that tears your project apart so
> your users don't have to. Karen is the orchestrator; the four workers are the
> ones who actually strike.

---

## What's in here

```
karen_strikes_back/
├── karen/        # ORCHESTRATOR — fans out, synthesizes
├── critic/       # Adversarial code + UI reviewer
├── bsneng/       # Junior UX critic (fresh-grad voice)
├── poweruser/    # Config perfectionist (insufferable PC enthusiast)
└── struggler/    # Struggling-user lens (low-fluency advocate)
```

Each folder contains `SOUL.md` (persona) and `config.yaml` (model + tools).

| Profile | Role | Voice |
|---------|------|-------|
| **karen** | Top orchestrator | Empathetic senior; routes + synthesizes; speaks for the user |
| **critic** | Worker | Blunt, evidence-led; cites file:line |
| **bsneng** | Worker | Fresh-grad Masters grad; "*that's heuristic #4!*" |
| **poweruser** | Worker | Insufferable PC enthusiast; *expose every knob* |
| **struggler** | Worker | Self-deprecating → fierce; what normies actually hit |

---

## Requirements

- [Hermes Agent](https://github.com/NousResearch/Hermes) installed.
- All five run on **`moonshotai/kimi-k3`** via the **`nous`** provider
  (note: `nous/k3` is *not* a valid slug — the correct id is `moonshotai/kimi-k3`).
- **`karen` must have the `delegation` tool** to spawn her workers. Her
  `config.yaml` ships with a `platform_toolsets.cli` block that includes
  `delegation`. If you hand-edit configs, do not drop it.

---

## Install

```bash
# from this repo's root
for p in karen critic bsneng poweruser struggler; do
  hermes profile create "$p" --no-skills
  cp "$p/SOUL.md"     ~/.hermes/profiles/"$p"/SOUL.md
  cp "$p/config.yaml"  ~/.hermes/profiles/"$p"/config.yaml
done
```

Seed the UX skills (the pod leans on `dogfood/adversarial-ux-test`,
`creative/*`, `software-development/requesting-code-review`). Copy them from any
profile that has them, or install via `hermes skills install`.

---

## Run a review

One command. Karen clones the repo, spawns the four workers in parallel via
`delegate_task` (each gets the absolute repo path in context — workers run in
isolated sessions), runs her own heuristic eval, then dedupes and
severity-ranks everything into a single report.

```bash
hermes -p karen chat -q "Review the GitHub repo <url> for adversarial code, UX, \
config/reproducibility, and onboarding friction. Fan out to critic, bsneng, \
poweruser, struggler via delegate_task; run your own heuristic eval; return one \
combined, severity-ranked report." --cli
```

Or run a single persona directly:

```bash
hermes -p critic    chat -q "Adversarial review of /abs/path/to/repo. Cite file:line." --cli
hermes -p bsneng    chat -q "UX review of /abs/path/to/repo: UI, README, error messages." --cli
hermes -p poweruser chat -q "Config/setup review of /abs/path/to/repo. Is every setting exposed?" --cli
hermes -p struggler chat -q "Struggling-user read of /abs/path/to/repo onboarding. Where does a newbie get stuck?" --cli
```

---

## How the lenses differ

- **critic** — SQLi, missing authz, no idempotency, code/UI defects. Cites line numbers.
- **bsneng** — layout, labels, error visibility, textbook Nielsen violations.
- **poweruser** — locked settings, no config file, telemetry-on-by-default, missing CLI/env overrides. Will `BLOCK` shipping.
- **struggler** — hidden steps, jargon, confusing labels, "just know" flows, unparseable errors.

Combined, they catch the bug *and* the onboarding wall *and* the setting you
forgot to expose.

---

## Notes

- Profiles are designed to live **outside** any single host profile (top-level
  peers under `~/.hermes/profiles/`).
- Model pinning matters: re-verify `model.default` after any profile mutation.
- This repo ships persona + config only. Skill trees are generic upstream
  Hermes skills and are not included.

*Brutal by design.*
