# 象棋棋谱库

从 GitHub 等开源渠道下载的免费棋谱集合，供 ChineseChessUltra 项目使用。
整理日期：2026-10-01

## 目录结构与来源

| 目录 | 格式 | 数量 | 来源 |
|------|------|------|------|
| `XQF-象棋谱大全/` | XQF | 8,557 局 | [weiyinfu/xqp](https://github.com/weiyinfu/xqp)（象棋谱大全） |
| `ChessQ-gamebooks/gamebooks/` | XQF | 249 局 | [walker8088/ChessQ](https://github.com/walker8088/ChessQ) 的 `gamebooks/`（象棋残局杀势） |
| `CGLemon-PGN/wxf/ICCS/` | PGN (ICCS) | 41,743 局 | 世界象棋联合会（WXF）对局，[CGLemon/chinese-chess-PGN](https://github.com/CGLemon/chinese-chess-PGN) README 中给出的 Google Drive 链接 |
| `CGLemon-PGN/dpxq/ICCS/` | PGN (ICCS) | 99,771 局 | 东萍象棋网（dpxq）对局，来源同上 |

合计：8,806 个 XQF 文件 + 141,514 盘 PGN 对局（约 200 MB）。

## XQF-象棋谱大全 分类

全局（古谱全局：梅花谱、反梅花谱、吴氏梅花谱、事林广记等）、大师专集（胡荣华集、
阎文清专集等）、比赛对局（象甲联赛、全国个人赛/团体赛等）、布局（顺炮全集、布局
陷阱 100 局等）、残局（适情雅趣 550 局、象棋实用残局等）、实战中局、让子局、近代
国手名局、未分类。

## 格式说明（与项目解析器对接时注意）

### XQF（象棋演播室二进制格式）
- 文件头魔数为前两字节 `XQ`（0x58 0x51），第 3 字节是**版本号**，本批文件版本分布
  在 `0x0A`–`0x12`（10–18）。旧版规范只认 `0x01/0x02` 的解析器会误判，注意兼容。
- 版本 >= 12 的文件对棋子布局和走子数据做了**位置加密**，需按 KeyMask 解密
  （参考实现：[walker8088/cchess](https://github.com/walker8088/cchess) 的
  `src/cchess/io_xqf.py`，已验证可解析本批全部抽样文件）。
- 字符串字段编码为 GB18030。

### PGN（ICCS 坐标变体）
- 标签：`[Game "Chinese Chess"]`、`[Event]`、`[Red]`、`[Black]`、`[FEN]`、
  `[Format "ICCS"]` 等，UTF-8 编码。
- 走法为 ICCS 坐标（如 `H2-E2`，列 a–i、行 0–9），一步一个 token，非中文纵线记谱。
- 与项目 assets 中 `sample_pgn.pgn` 的**中文纵线记谱**（"炮二平五"）不同：项目的
  PGN 解析器（目前是空壳桩）落地时需同时支持 ICCS 与中文纵线两种走法文本，
  或选一条转换管道。
- 两个 `.pgns` 文件是多局合一的大文件（41.7 MB / 101 MB），按空行分隔多局，
  流式解析时不要整读入内存。

## 检验情况
- XQF：随机抽样 200 个文件，用 `cchess` 库（PyPI）`read_from_xqf` 全部解析成功。
- PGN：两个大文件尾部结果符（`1-0`/`0-1`/`1/2-1/2`）完整，事件数与官方声明一致
  （41,743 / 99,771）；单局用 `cchess.read_from_pgn` 解析成功，走法 token 经正则
  校验全部符合 ICCS 模式。

## 备用资源（未下载，格式不符）
- [chasoft/community-xiangqi-games-database](https://github.com/chasoft/community-xiangqi-games-database)：
  24,634 个古谱残局（适情雅趣、桔中秘、百变象棋谱等），但为东萍 DhtmlXQ 文本格式，
  非 XQF/PGN；如需要可写转换器转成 PGN（其 `binit`/`movelist` 字段可还原局面与走法）。

## 版权
均为开源/免费流通的棋谱数据，来源仓库分别采用各自声明的许可证（见各仓库）。

## 与 ChineseChessUltra 项目对接（2026-10-01）

- 项目通过**目录联接**引用本目录（不复制棋谱）：项目根执行
  `mklink /J corpus E:\ssy_proj\qp`，代码与测试统一经 `corpus/...` 访问。
- 解析器与测试实现见项目内 `docs/qp_parse.md`；XQF 魔数 `XQ`+版本号、
  密钥公式、32 子回文序等实证结论已记录在该文档。
- PGN 子目录：`CGLemon-PGN\wxf\ICCS\`（世象联 4.2 万盘）、
  `CGLemon-PGN\dpxq\ICCS\`（东萍 9.98 万盘），均为多局合一 ICCS PGN。
