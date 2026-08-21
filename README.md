# Project Will — 観測 #2・#3 データセット（2026-08-10 / 2026-08-11 ロングラン×2）

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.22040887.svg)](https://doi.org/10.5281/zenodo.22040887)

🌐 **English summary: [README_EN.md](README_EN.md)**

制度（役割の割当て・共有規則・契約の履行機構）を与えず、代わりに**長期記憶と認知（統合海馬認知システム NOESIS）**を与えた自律 AI 村人 32 人・6 時間ロングラン**2 本（観測 #2: 2026-08-10 / 観測 #3: 2026-08-11）の全記録と全分析**です。動画は観測 #3 を中心に扱いますが、中間 run である観測 #2 の全データ・分析も本リポジトリに含みます（観測 #2 で計測レベルの交絡＝「身体が選ぶ目的地による侵入」を発見・修正した上で観測 #3 を実施——この経緯自体が方法論の一部です）。動画で提示した全ての主張は、このリポジトリのデータから検証できます。

📺 動画: https://youtu.be/zkVjwyZ7QJQ
📄 報告書: [REPORT.md](REPORT.md)
🔎 主張↔データ対応表: [VERIFICATION.md](VERIFICATION.md)
📚 データ辞書: [DATA.md](DATA.md)
🧠 NOESIS 設計: [NOESIS.md](NOESIS.md)
⚖️ 計測の限界と正直な開示: [METHOD_AND_LIMITS.md](METHOD_AND_LIMITS.md)
📦 観測 #1（前回）: https://github.com/MONOCOLACHANNEL/project-will-observation-1 / DOI [10.5281/zenodo.21723921](https://doi.org/10.5281/zenodo.21723921)

## 観測 #3 のあらまし（動画の対象・すべて本リポジトリのログで検証可能）

| | |
|---|---|
| 村人 | 32 人スポーン ＋ 2 人出生 − 13 人死亡 ＝ 生存 21 人（**村内 11・村外 10 に分裂**） |
| 死因 | 殺害 11（全件が遺恨駆動・発端は全て侵入系）・オオカミ 1・焼死 1 |
| 殺人の構造 | 前回 24 件（連鎖 6・報復 1 を含む）→ **11 件・連鎖 0**。10 件が最初の 107 分に集中し、以降 3.7 時間は 0 件 |
| 出口の交代 | 遺恨の清算が **殺人 11 件（10 件が 107 分まで）⇔ 許し 12 件（11 件が 90 分以降）** に入れ替わった |
| 制度の芽 | 前回 0 だった**名乗り→契約の完結**（採石担当の宣言→督促→納期→のべ 1,008 個の供出→回収。ただし労働の専門化は未発生＝採石の意図選択率は村平均以下）・**履行**（合意→納入の実例を確認。⚠「履行率」は合意の数え方で 20〜67% に振れるため率では示さない）・**服従**（命令 74 件の 86% を 6 人が発令、死後も従い続けた例）・**共有地名**（『西の牧場』が 12 人へ伝播し命名者の死後も存続） |
| 経済 | 共有チェスト 1 箱が「銀行」化：9 人が 4,519 個を引き出し（34 回・物理実測。最大顧客は督促主 1 人で 55%。入庫側の 1 位も同じ督促主＝入 2,584 個/出 2,494 個の回転）、管理者の死後も 370 分機能。村全体では 26 人が丸石 11,219 個を引き出し（丸石の入庫はのべ 22,727 個） |
| 記憶と夢 | 夢 401 件（観測 #2 は 339 件）（夜間統合が過去エピソードを**原文のまま**再生）。夢が翌日の発話に先行する例を観測 |
| 書かれなかった本 | 執筆機能を解放したが**0 冊**（看板は 151 枚）。「紙の生産ライン独占」を宣言した村人が 27 秒で条件を自壊 |

## クイックスタート

```bash
# 人物・語句・時間帯でログを横断検索（結果は out.txt に UTF-8 で出力）
python tools/find.py 梅木ボブ --time 01:07 01:15
python tools/find.py 石材置き場
```

## 収録文書

| 文書 | 内容 |
|---|---|
| [REPORT.md](REPORT.md) | 技術報告書 本体 |
| [VERIFICATION.md](VERIFICATION.md) | 主張↔データ対応表 |
| [DATA.md](DATA.md) | データ辞書と既知の注意点 |
| [NOESIS.md](NOESIS.md) | 追加した記憶システムの設計 |
| [METHOD_AND_LIMITS.md](METHOD_AND_LIMITS.md) | 計測の限界・実装欠陥の正直な開示 |
| [analysis_obs3/VERIFICATION_LEDGER_20260820.md](analysis_obs3/VERIFICATION_LEDGER_20260820.md) | **公開前の全数再検証台帳（215項目・旧値→正値の全件）** |

## 著者と引用

本データセットは **Project Will シリーズ**（著者: モノコーラ / Monocola）の第2弾であり、[観測 #1](https://github.com/MONOCOLACHANNEL/project-will-observation-1)（DOI [10.5281/zenodo.21723921](https://doi.org/10.5281/zenodo.21723921)）と同一著者による継続観測です。引用は [CITATION.cff](CITATION.cff) を参照：

> Monocola (モノコーラ). (2026). Project Will: Observations #2–#3 — 32 autonomous LLM villagers with integrated memory, no institutions [Dataset]. Zenodo. https://doi.org/10.5281/zenodo.22040887

（DOI は全バージョンを指し、常に最新版へ解決します。特定版は v1.0 = 10.5281/zenodo.22040888。シリーズ全体に言及する場合は観測 #1 の DOI と併記してください。質問・指摘は GitHub Issue へ）

ライセンス：データ・文書 **CC BY 4.0** ／ スクリプト **MIT**（[LICENSE](LICENSE)）
