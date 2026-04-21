# rdl-bun-dual-lockfiles

Test fixture for the `remove_duplicate_lockfiles` maintenance task.

## Scenario

Bun produces two lockfile names during its migration from the legacy binary format to the new text format. **Both map to the same manager** (`bun`), so their coexistence is **not** a conflict.

Files:
- `package.json` (no `packageManager`)
- `bunfig.toml`
- `bun.lock` (new text format)
- `bun.lockb` (legacy binary format, stub bytes)

## Expected MT behaviour

- `scan_lockfiles` groups both files under the `bun` manager for the `node` ecosystem.
- `find_conflicts` requires **2+ distinct managers** per (directory, ecosystem) → sees only `bun` → emits no conflict.
- `analyze()` sets `compatibility.can_proceed = False` with blocker `"No duplicate lockfiles found"`.

This guards against treating legacy + new bun lockfiles as a conflict, which would be a false positive.
