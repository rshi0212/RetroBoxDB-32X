32X Catalog, storage v4 (32 KiB blocks, 2 solid LZMA2 groups of up to 256 MiB). Metadata only: **no ROM payloads are published**; `compression_groups`, `chunks` and `object_chunks` are empty.

- First release: 32X cartridges (Mega Drive header; many retail cartridges declare checksum 0) from the No-Intro folders including Aftermarket, and the RetroAchievements 32X set; 1 Parent-Clone DAT version, DB Export and Dump Log unknown, RetroAchievements snapshot and Chinese names.
- Storage measured on the whole collection: 32 KiB blocks, 256 MiB groups (`assessment/data/storage-experiment-32x.json`).
- Source: 414 ZIPs (nointro 375, retroachievements 39), 593.1 MiB (414 ROM files, 1.09 GiB uncompressed). Populated database: 87.6 MiB (14.8% of the ZIPs). All source ZIPs are reproduced byte-for-byte.
- Contents: 228 ROM records, 61 games, 227 releases; DAT versions: 20260317-140429.
- RetroAchievements: 35 of 36 games with achievements have a local ROM.
- Chinese names: 213 of 213 CSV rows translated; 203 local ROMs with Chinese names.
- Export (Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz, idle, all checks): whole newest-DAT set with export_set.py 69.9 MiB/s (219 files); single file with a cold cache 1.524 s (ROM) / 1.707 s (TorrentZip) on average.
- Full audit of the populated database: 231 objects, 2 groups, 387 archive plans, no errors.

The release workflow starts from the base Catalog pinned by SHA256 in `release/catalog-release.json`, injects the engine and documents of the tagged commit, checks every data-table digest, SQLite integrity and foreign keys, runs the Catalog audit and the repository tests. Verify the download with `SHA256SUMS`.

[中文说明](https://github.com/rshi0212/RetroBoxDB-32X/blob/main/README.zh-CN.md)
