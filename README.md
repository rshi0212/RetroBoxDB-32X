# RetroBoxDB 32X

English | [中文说明](README.zh-CN.md)

Single-file SQLite preservation database for Sega 32X. The public Catalog holds metadata only (checksums, DAT and provenance records, header fields, archive recipes and the processing code); it contains no ROM data and cannot restore files. The populated database stays local.

| Item | Value |
| --- | --- |
| Original size | 414 source ZIPs, 593.1 MiB (No-Intro 375, RetroAchievements sets 39); 414 ROM files, 1.09 GiB uncompressed |
| Stored size | populated database 87.7 MiB; public Catalog 3.8 MiB (no ROM data) |
| Ratio | 14.8% of the source ZIPs, 7.8% of the uncompressed ROM files |
| Technology | storage v4: SHA256-deduplicated 32 KiB blocks packed in No-Intro family order into solid LZMA2 groups of up to 256 MiB (256 MiB dictionary); per-block SHA256 and per-object CRC32/MD5/SHA1/SHA256 verification; source ZIPs reproduced byte-for-byte from TorrentZip plans |
| Export performance | Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz, idle, Python 3.14.4, all checks included. whole newest-DAT set with `export_set.py` (219 files, each checked against the DAT hashes): 69.9 MiB/s, 38 ms per file on average; single file with a cold cache (the group is decoded up to the file): ROM 1.524 s, TorrentZip 1.707 s on average |

## Downloads and documents

| File / document | Content |
| --- | --- |
| [RetroBoxDB.32X.Catalog.sqlite](https://github.com/rshi0212/RetroBoxDB-32X/releases/latest/download/RetroBoxDB.32X.Catalog.sqlite) | Public Catalog (Release asset with `SHA256SUMS`) |
| [Storage v4 guide](RetroBoxDB.Storage-v4.en.md) / [中文](RetroBoxDB.Storage-v4.zh-CN.md) | Storage evaluation, contents, RA, names and maintenance for every platform |
| [Technical design](RetroBoxDB.Storage-v4.Technical-Design.en.md) | Storage format, platform adapters, incremental updates, verification |
| [RA list](reports/ra-sega32x-games.csv) / [summary](reports/ra-sega32x.json), [build report](reports/sega32x-build-report.json), [audit resolution](reports/audit-resolution-20261004.md) | Detailed data |

## Storage choice and platform specifics

17 block/group combinations measured on the whole local collection (`assessment/data/storage-experiment-sega32x.json`): smallest 128 KiB / 256 MiB at 83.58 MiB; by the rule (within 0.5% of the smallest, the smallest block, then the smallest group) 32 KiB / 256 MiB at 83.85 MiB. ZIPs 593.11 MiB, per-file LZMA 281.98 MiB.

- ROMs are 1–4 MiB, and many dumps share most of their data (revisions, regional versions, prototypes and test cartridges): 1.09 GiB of ROM files become 346 MiB of unique 32 KiB blocks, then 80 MiB after compression, so the populated database is about 15% of the source ZIPs.
- 32X cartridges carry a Mega Drive header (`md_hardware`). Retail cartridges often name the system `SEGA MEGA DRIVE` or `SEGA GENESIS` and declare checksum 0; this is reported as 'no checksum declared' rather than a mismatch. The MARS security block at 0x3C0 is checked.
- RetroAchievements console 10 (whole-file MD5).

## Contents

| Item | Value |
| --- | --- |
| ROM records / games / releases | 228 / 61 / 227 |
| DAT coverage per version | 20260317-140429: 219/227 |
| Local ROMs in no DAT | 9 |
| ROM files of the RetroAchievements set | in a No-Intro DAT 31, RA only 7, hash not in the latest RA snapshot 1 ([list](reports/ra-sega32x-collection-unknown.csv)); RA games still without a local ROM: [gap list](reports/ra-sega32x-missing.csv) |
| No-Intro DB Export + Dump Log unknown | 228 archives, 239 file identities, 35 documented hardware assertions; Dump Log Verified 23 |
| RetroAchievements (console 10) | 36 games with achievements: 35 with a local ROM (39 ROMs), 0 with the ROM in a sibling database, 0 DAT only, 0 DB file only, 1 without a No-Intro counterpart |
| Chinese names | 213 of 213 rows translated (54 unique); 203 local ROMs have a Chinese name |
| Populated-database audit | 231 objects, 2 groups, 387 archive plans, all passed |

Every source ZIP is reproduced byte-for-byte from its TorrentZip plan (`v_file_checksums.exported_bytes_equal_source`).

## Usage

```bash
# Query-only audit with the Catalog's embedded engine (also: stats, checksums FILE_ID, help)
python3 -B -c 'import sqlite3,sys; c=sqlite3.connect(sys.argv[1]); s=c.execute("SELECT content FROM resources WHERE name=?",("engine.py",)).fetchone()[0]; c.close(); exec(compile(s,"RetroBoxDB:engine.py","exec"))' ./RetroBoxDB.32X.Catalog.sqlite audit
# Populated database: export by DAT version, 1G1R, RA achievements, TorrentZip or plain ROMs
python3 -B tools/export_set.py RetroBoxDB.32X.sqlite OUT --set 1g1r --ra achievements --container torrentzip --layout ra-category
# Add new DATs, DB Export / Dump Log snapshots, ROMs and an RA snapshot incrementally
python3 -B tools/update_db.py RetroBoxDB.32X.sqlite --discover --ra --catalog RetroBoxDB.32X.Catalog.sqlite
```

Python 3.10+ standard library only. `engine.py` and the other `resources` entries are executable code; run them only from a database you built or a Release asset whose SHA256 you verified. Releases are produced by `.github/workflows/publish-catalog.yml`: it starts from the base Catalog pinned in `release/catalog-release.json`, injects the engine and documents of this commit, checks every data-table digest, runs the tests and the audit, then publishes.
