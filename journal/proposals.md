# Proposals for the operator

## 2026-10-06T11:58Z — core scripts crash on this machine's cp949 locale (blocks every FULL cycle)

Evidence (this cycle, operator machine, Windows, Python 3.13):
`python3 core/lease.py acquire` and `python3 core/scan.py --hours 336 --limit 800`
both die at import with
`UnicodeDecodeError: 'cp949' codec can't decode byte 0xe2 in position 28`
reading `config/protected.json` via `Path.read_text()` with no encoding
(`core/scan.py:33`, `core/screen.py:109`, `core/ledger.py:30`; `lease.py` imports
`screen`). `resolve.py`, `score.py`, `ci.py` run fine. So step 0 lease, step 4
scan/screen and step 6 place cannot run: no candidate pool, no bets possible.
`forecast.py`/`ledger.py` also append with the locale encoding, so non-ASCII
notes would land in the journal as cp949 bytes.

Fix (either, operator-owned): set `PYTHONUTF8=1` in the runner environment
(loop.sh or the user env), or pass `encoding="utf-8"` to the `read_text()` /
`open()` calls in `core/`. The session's permission policy blocked running
`python3 -X utf8` / `PYTHONUTF8=1` myself, and I will not work around it.

Also: `strategy/schedule.json` watch_items still reference ledger/forecast ids
from the archived upstream journal (`archive/journal-upstream-20261006/`). They
cannot be graded against this empty ledger. I will prune them once a working
cycle confirms they're absent from the new journal, unless you want them kept.
