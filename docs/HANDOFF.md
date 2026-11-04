# vpinball handoff (Faris Pin cab: tables, ROMs, altsound)

This repo is only a working directory; it holds no code. The real work lives elsewhere (see Where things live).
Last updated 2026-11-04.

## What this is
Day-to-day care of the Faris Pin virtual pinball cab (`eric@100.88.10.29`, Tailscale `vpinball`, cmd.exe shell):
installing tables and their files, ROM and altsound setup, Popper cleanup. Load the `vpin` skill before cab work and
`vpin-tables` for installs.

## Where things live (all committed and pushed unless noted)
- `~/.claude/skills/vpin-tables/`: `SKILL.md`, `tablesync.py` (runs ON the cab), `recorder.py`, `test_tablesync.py`,
  `LESSONS.md`, `TODO.md`. Repo `~/.claude` -> `ericfaris/claude`, main.
- `~/.claude/skills/vpin/`: cab reference skill.
- `~/projects/launchpin`: the launcher (kill list for Popper leftovers lives in `launchpin/launcher.py`).
- Uncommitted and not mine: `~/.claude/settings.json` model switch `opus` -> `sonnet`. User said to leave it.

## Install workflow (vpin-tables)
1. Files come from THIS machine's Downloads: `/mnt/c/Users/ericf/Downloads` (not the cab's Downloads).
2. Copy to `C:\temp\tableinbox` on the cab with scp (skip `desktop.ini`, `*.exe`, `*.msi`), run
   `tablesync.py plan --src C:\temp\tableinbox`, decide with the user, then `apply --src ... --select N,N --rom "Title=rom"`.
3. Empty the inbox after a successful apply. `apply` needs launcher mode `menu` and no Popper/VPX running.
4. Rules cards (`*help`, `*rules`, `Quick Rules`, `Table Rules`) are always installed to `POPMedia\Visual Pinball X\GameHelp`.
5. Run tests: `python3 -m unittest discover -s ~/.claude/skills/vpin-tables -p 'test_*.py'` (43 pass).

## State of the cab (2026-11-04)
- Installed this session: Scared Stiff VPW v1.1.1 (GameID 271, ss_15), World Cup Soccer VPW v1.5 (272, wcs_l2),
  Street Fighter II extras (pin2dmd altcolor, 2.1 .ini and .directb2s), rules cards for those three.
- Popper DB backup: `PUPBackup\20261103133304_PUPDatabase.db`.
- Altsound: VPinMAME 3.6.0.1227 supports G-Sound. Per-ROM `sound_mode` (registry
  `HKCU\Software\Freeware\Visual PinMame\<rom>`) must be 1 for altsound; 0 plays the ROM sound.
  Set to 1 for `gldneye` and `sfight2`; `ft_l5` was already 1. All other ROMs are still 0.
  Registry backups: `C:\vPinball\VisualPinball\gldneye-pinmame-backup.reg`, `sfight2-pinmame-backup.reg`.
- `sfight2.zip` ROM was missing on the cab; copied from local Downloads to `VPinMAME\roms` (626 KB, 6 files).
- Launcher deployed with `PinUpDisplay*` + `PinUpDOF.exe` in the Popper-leftover kill list (launchpin `d8bf8f3`).

## Open items (priority order)
1. User to test: Goldeneye and Street Fighter II should now play altsound; SF2 should now run with the ROM present.
   If altsound still doesn't play, read `Tables\altsound.log` on the cab.
2. User to confirm the Popper-exit fix: after exiting a table, `tasklist` should show no PinUp* except PinVol-unrelated
   items. Any other survivor goes in `POPPER_LEFTOVERS` (never widen to `pin*`, it kills PinVol).
3. Decision pending: replace Knorr 1.4 World Cup Soccer with VPW v1.5? Would mean deleting Popper row 165, its playlist
   entry and the two Knorr files (`.vpx`, `.directb2s`). The installer cannot delete; do it by hand, back up the DB first,
   and only on explicit go-ahead. Both versions are currently in the playlist.
4. One sample missing from Goldeneye's pack: `0xFD8A-unlabeled_xadra_g.ogg` (listed in g-sound.csv).
5. Other ROMs with an altsound pack but `sound_mode = 0`: ask before changing any.
6. Pull cab-side `recordstart.bat` and the DMD settings into the vpin-tables skill folder (not versioned yet).
7. DMD recording still doesn't work (see the skill's DMD section).

## Decisions that may be revisited
- Source of installs is the local Downloads, not the cab's; the cab's own Downloads is no longer read.
- Rules cards are installed automatically; a plain `<table>.png` is not treated as one.

## Gotchas
- Never edit `tablesync.py` with a scripted slice-replace: use exact `count==1` asserts. The skill folder was untracked
  once; a bad edit deleted 190 lines and was recovered from `~/.claude/file-history/<session>/<hash>@vN`.
- Check mode before any launcher restart (`curl.exe -s http://127.0.0.1:8755/api/state`); a restart drops what the user is doing.
- `taskkill /im` wildcards really match. In cmd over ssh, `;` is not a separator.
- Run the cab's Python by full path; `python` is not on PATH.
- Commit messages for the user's repos: no `Co-Authored-By` lines (a trailer slipped into `1800a58` before this was caught; fixing it needs a force-push, which is not allowed).
