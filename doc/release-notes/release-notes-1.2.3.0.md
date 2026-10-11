# MasterNoder2 v1.2.3.0

**Release date:** 2026-06-17 (draft — build binaries before tagging)

## Summary

Maintenance release focused on **masternode sync reliability** on small networks (including single-masternode chains) and **RPC compatibility** for tooling that expects PIVX-style `getstakinginfo`.

## Changes

### P0 — Masternode sync deadlock fix

- **`src/masternode-sync.cpp`:** When the sync is in the MNW (winners) stage and no winner broadcasts arrive (`lastMasternodeWinner == 0`), advance to the next stage after the existing stage timeout (`MASTERNODE_SYNC_TIMEOUT * 5`, ~25s) **before** iterating peers.
- **Why:** On chains with few masternodes, every peer can be marked fulfilled before the in-loop timeout runs, leaving `mnsync` stuck at asset 3 and blocking staking until manual `--mnsync-finish` peer injection.

### P1 — RPC alias

- **`getstakinginfo`** — alias for `getstakingstatus` (same JSON shape). Registered in wallet RPC table.

### Version

- `configure.ac`: `CLIENT_VERSION_REVISION` → **3** (1.2.3.0)

## Upgrade (production daemon)

1. Stop `masternoder2d` (`systemctl stop masternoder2d`).
2. Backup `/var/www/html/config/wallet.dat` (and `masternoder2.conf`).
3. Replace binary from release asset `masternoder2d.tar.gz` — **no reindex** required.
4. Start daemon; unlock staking: `walletpassphrase "<pass>" 0 true`.
5. Verify: `getstakingstatus` → `"mnsync": true`, `"staking status": true`.
6. Confirm app health: `GET /api/mn2/health` → `daemon_staking.status: active`.

## Build assets (maintainer)

From Linux build host (same as v1.2.2.0 process):

```bash
./autogen.sh && ./configure && make -j$(nproc)
# package: masternoder2d.tar.gz, MasterNoder2-qt-linux.tar.gz, MasterNoder2-qt-win.zip
gh release create v1.2.3.0 --title v1.2.3.0 --notes-file doc/release-notes/release-notes-1.2.3.0.md \
  masternoder2d.tar.gz MasterNoder2-qt-linux.tar.gz MasterNoder2-qt-win.zip
```
