
# Snapshot & Rollback Strategy

Every exercise in `/redteam/playbooks/` can leave the Target VLAN in a broken or compromised state by design, and  that's the point. This guide defines how to make that safe and repeatable, and directly supports the reversibility requirement in `../ROE.md`.

## Why this matters

Without a snapshot strategy, a destructive playbook (privilege escalation, lateral movement, or anything involving persistence) either can't be run safely, or forces a full VM rebuild afterward, which kills the ability to re-run the same playbook later to confirm a detection fix actually worked. Snapshots make the whole purple-team loop (exercise → gap found → fix → re-run → confirm) practical instead of a one-shot exercise.

## Snapshot points

Take a snapshot at these points for every target VM involved in a playbook:

1. **Baseline** — immediately after the VM is built/patched and before any exercise has ever touched it. Never overwrite this one.
2. **Pre-exercise** — right before running a specific playbook. This is what you roll back to after the exercise, or after a detection fix, to re-test from a clean state.
3. **Post-exercise** (optional) — after the exercise, before rollback, if you want to preserve the compromised state for further manual analysis later.

## Naming convention

```
<vm-name>-baseline-YYYY-MM-DD
<vm-name>-pre-<playbook-id>-YYYY-MM-DD
<vm-name>-post-<playbook-id>-YYYY-MM-DD
```

Example: `dvwa-pre-01-example-playbook-2026-07-08`

Consistent naming matters here specifically because snapshots accumulate fast once you're running the same playbook repeatedly to confirm fixes — an unclear name means guessing which one is actually clean.

## VMware steps

**Taking a snapshot** (ESXi or Workstation):

1. Right-click the target VM → **Snapshot → Take Snapshot**
2. Name it per the convention above
3. Confirm the VM is powered on and in a known-good state before snapshotting (a snapshot mid-boot or mid-scan is not a clean baseline)

**Rolling back**:

1. Right-click the target VM → **Snapshot → Revert to Snapshot**
2. Select the relevant `pre-<playbook-id>` snapshot
3. Confirm the VM boots cleanly and services are responsive before re-running any exercise

## When to roll back vs. rebuild

- **Roll back** for anything reversible via VM state — most exploitation, credential theft, and web attack playbooks in this repo.
- **Full rebuild** only if a playbook is explicitly testing something below the VM snapshot layer (e.g. persistence that survives a revert isn't something this lab currently tests) — none of the current playbooks require this, but it's worth stating the boundary.

## Where this fits

- This file: `/redteam/setup/02-snapshot-rollback-strategy.md`
- Referenced by: `../ROE.md` (reversibility requirement), every playbook that modifies target state
- Snapshot names used in a given exercise should be recorded in the corresponding `../reports/` file, so a report shows exactly which snapshot state the results came from