commit: 4ffed9b

# The backup folder is now `backups/` (lowercase), not `Backups/`

## What broke

**Instruction contract and on-disk state.** The backup folder is renamed from `Backups/`
to `backups/` in commit `4ffed9b`. Linux treats the two names as different folders, so:

- `~/.claude/CLAUDE.md` still tells the agent to write to `.../Backups/`, which is no
  longer where the script writes.
- Existing dumps in `Backups/` are not found by `scripts/restore.py` or by
  `recall.py --candidates`, which now look in `backups/`.

macOS is case-insensitive and may treat the two names as one folder. Check which
folders exist before moving anything.

## How to resolve

1. Pull `master` (or the merge of `backups-lowercase`) so the repo has the new paths.
2. If an old `Backups/` folder still exists and no `backups/` folder does, rename it.
   The `-e` check keeps this a no-op on a case-insensitive filesystem where both names
   resolve to the same folder:

   ```bash
   cd "<repo>"
   [ -d Backups ] && [ ! -e backups ] && mv Backups backups
   ```

3. Re-render the managed block in `~/.claude/CLAUDE.md`:

   ```bash
   bash scripts/install-claude-md.sh
   ```

   It keeps this machine's settings (`EPISODIC_MODE`, `BASE_PATH`, the DB path) from the
   existing block, and replaces only the managed block.
4. Restart Claude Code so it reads the new instructions.

## Verification

```bash
grep -n '"backup"' ~/.claude/CLAUDE.md     # must end in .../backups/
ls -d backups && ls -d Backups 2>/dev/null  # backups exists; Backups should be absent
grep -o 'synced-at [0-9a-f]*' ~/.claude/CLAUDE.md
git rev-parse HEAD                          # the synced-at sha must match this
```

The `synced-at` sha must equal the repo's `HEAD`, and the `"backup"` line must mention
`backups/`, not `Backups/`.
