# LIPON — モック

株式会社LIGOの新CRMツール「LIPON」 3画面モック。

## 🚀 ライブプレビュー（GitHub Pages）

| アプリ | URL | 対象 |
|---|---|---|
| 🖥 **PC 管理者ダッシュボード** | [/index.html](./index.html) | 全社俯瞰・カスタムダッシュボード |
| 📱 **スマホ総合（営業用）** | [/mobile.html](./mobile.html) | フィールド営業の日常 |
| 📲 **入力専用 LIPON Tap** | [/input.html](./input.html) | 5タップで接点記録 |

## 📋 プロジェクト概要

詳細は [HANDOFF.md](./HANDOFF.md) を参照。

- **ゴール**：Salesforceから「シンプル × インフォグラフ × 文字入力最小」のCRMへ移行
- **対象**：30名（営業 / CS / 管理者）
- **インフラ**：GCP（Cloud Run + Cloud SQL + IAP）
- **デザイン**：白ベース × ピンクアクセント、Slack風UI

## 🤝 開発を続けるには

このリポジトリをcloneして、Claude Codeで開く：

```bash
git clone https://github.com/nagatsuma-byte/ligo-crm.git
cd ligo-crm
claude
```

最初のメッセージで「HANDOFF.md を読んで状況を把握して、次のステップを提案して」と打てば、文脈を踏まえて続きから動きます。

## 📁 ファイル構成

```
ligo-crm/
├── README.md       ← このファイル
├── HANDOFF.md      ← 引き継ぎドキュメント（全文脈）
├── index.html      ← PC管理者
├── mobile.html     ← スマホ総合
└── input.html      ← スマホ入力専用
```
