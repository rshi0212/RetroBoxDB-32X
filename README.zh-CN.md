# RetroBoxDB 32X

[English](README.md) | 中文

世嘉 32X的单文件 SQLite 保存库。公开的 Catalog 只含元数据（校验值、DAT 与来源记录、头部字段、打包配方和程序），不含 ROM 数据，不能独立恢复文件；完整库保留在本地。

| 项目 | 数值 |
| --- | --- |
| 原始大小 | 源 ZIP 414 个，593.1 MiB（No-Intro 375 个，RetroAchievements 集合 39 个）；解压后 ROM 414 个，1.09 GiB |
| 入库后大小 | 完整库 87.6 MiB；公开 Catalog 3.7 MiB（不含 ROM 数据） |
| 比例 | 完整库为原 ZIP 的 14.8%，为解压后 ROM 总量的 7.8% |
| 使用的技术 | 存储 v4：32 KiB 块按 SHA256 去重，按 No-Intro 游戏族顺序装入最大 256 MiB 的 LZMA2 实体组（字典 256 MiB）；逐块 SHA256、逐对象 CRC32／MD5／SHA1／SHA256 校验；源 ZIP 由 TorrentZip 配方逐字节重建 |
| 导出性能 | Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz，空闲负载，Python 3.14.4，含全部校验。按最新 DAT 整套导出（`export_set.py`，219 个文件，逐个按 DAT 哈希校验）：69.9 MiB/s，平均 38 毫秒／个；单个文件冷缓存（每次清空缓存，需解压所在组的前段）：ROM 平均 1.524 秒，TorrentZip 平均 1.707 秒 |

## 下载与说明

| 文件／文档 | 内容 |
| --- | --- |
| [RetroBoxDB.32X.Catalog.sqlite](https://github.com/rshi0212/RetroBoxDB-32X/releases/latest/download/RetroBoxDB.32X.Catalog.sqlite) | 公开 Catalog（Release 附件，附 `SHA256SUMS`） |
| [存储 v4 说明](RetroBoxDB.Storage-v4.zh-CN.md)／[English](RetroBoxDB.Storage-v4.en.md) | 各平台的存储评估、内容、RA、中文名与维护 |
| [Technical design](RetroBoxDB.Storage-v4.Technical-Design.en.md) | 存储格式、平台适配、增量更新、校验 |
| [RA 清单](reports/ra-32x-games.csv)／[汇总](reports/ra-32x.json)、[构建报告](reports/32x-build-report.json)、[审计处理](reports/audit-resolution-20261004.md) | 逐项数据 |

## 本平台的存储选择与特殊情况

全部本地收藏实测 17 种块／组组合（`assessment/data/storage-experiment-32x.json`）：最小为 128 KiB / 256 MiB 83.58 MiB；按规则（最小值 0.5% 以内选块最小、再选组最小）采用 32 KiB / 256 MiB 83.85 MiB。ZIP 593.11 MiB，逐文件 LZMA 281.98 MiB。

- ROM 为 1–4 MiB，许多 dump 之间大部分数据相同（修订版、地区版、原型和测试卡带）：1.09 GiB 的 ROM 文件按 32 KiB 块去重后只剩 346 MiB，压缩后 80 MiB，完整库约为源 ZIP 的 15%。
- 32X 卡带使用 Mega Drive 头部（存入 `md_hardware`）。正式版卡带的系统字符串常写作 `SEGA MEGA DRIVE` 或 `SEGA GENESIS`，声明校验和为 0，这记为“未声明校验和”而不是不一致。另检查 0x3C0 的 MARS 安全头。
- RetroAchievements 主机 10（整文件 MD5）。

## 内容

| 项目 | 数值 |
| --- | --- |
| ROM 记录／游戏组／发行版本 | 228／61／227 |
| 各版 DAT 覆盖 | 20260317-140429：219/227 |
| 不在任何 DAT 的本地 ROM | 9 |
| RetroAchievements 集合中的 ROM 文件 | DAT 中有 31，仅 RA 收录 7，哈希不在最新 RA 快照 1（[清单](reports/ra-32x-collection-unknown.csv)）；仍缺本地 ROM 的 RA 游戏见 [缺口清单](reports/ra-32x-missing.csv) |
| No-Intro DB Export＋Dump Log unknown | 228 个档案、239 个文件身份、35 条有文档的硬件声明；Dump Log Verified 23 |
| RetroAchievements（console 10） | 有成就的游戏 36 个：本地有 ROM 35（39 个 ROM），ROM 在兄弟库中 0，仅 DAT 有 0，仅 DB 文件 0，无 No-Intro 对应 1 |
| 中文名 | 213 条记录中 213 条有中文（54 个唯一名）；本地 ROM 203 个有中文名 |
| 完整库审计 | 231 个对象、2 个组、387 个 ZIP 配方，全部通过 |

源 ZIP 均可由 TorrentZip 配方逐字节重建（`v_file_checksums.exported_bytes_equal_source`）。

## 使用

```bash
# 用 Catalog 内嵌引擎做只读审计（stats、checksums FILE_ID、help 同理）
python3 -B -c 'import sqlite3,sys; c=sqlite3.connect(sys.argv[1]); s=c.execute("SELECT content FROM resources WHERE name=?",("engine.py",)).fetchone()[0]; c.close(); exec(compile(s,"RetroBoxDB:engine.py","exec"))' ./RetroBoxDB.32X.Catalog.sqlite audit
# 完整库：按 DAT 版本、1G1R、RA 成就、TorrentZip／裸 ROM 组合导出
python3 -B tools/export_set.py RetroBoxDB.32X.sqlite OUT --set 1g1r --ra achievements --container torrentzip --layout ra-category
# 增量加入新 DAT、DB Export／Dump Log、ROM 与 RA 快照
python3 -B tools/update_db.py RetroBoxDB.32X.sqlite --discover --ra --catalog RetroBoxDB.32X.Catalog.sqlite
```

只需 Python 3.10+ 标准库。`resources` 中的 `engine.py` 等是可执行代码，只应从自己构建或 SHA256 已核对的 Release 附件中执行。发布由 `.github/workflows/publish-catalog.yml` 完成：工作流从 `release/catalog-release.json` 固定的基础 Catalog 出发，注入本仓库提交中的引擎与文档，核对全部数据表摘要、运行测试与审计后发布。
