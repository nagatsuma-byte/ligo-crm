# LIPON プロジェクト — 引き継ぎドキュメント

> **このファイルは yuujin さん（および yuujin さんのClaude Code）が前任者から引き継ぐためのものです。**
> Claude Codeで本フォルダを開いたら、**最初にこのファイルを読んでください。**

最終更新: 2026-05-17

---

## 🎯 プロジェクトゴール

株式会社LIGOの**顧客管理ツールを内製する**。

- 現状はSalesforceを利用しているが多機能すぎて使いこなせていない
- 「シンプル × インフォグラフ × 文字入力最小」の独自CRMを構築
- 社内Webで30名（営業/CS/管理者）が利用
- GCP上で構築

---

## 📌 確定済み要件

| 項目 | 内容 |
|---|---|
| **ホスティング** | GCP（Cloud Run + Cloud SQL + IAP + Cloud Storage + Cloud Scheduler） |
| **同時利用者** | 30名 |
| **権限** | 管理者 / メンバー（営業・CS） / 閲覧 の3層 |
| **データ更新** | Indeed: 毎日APIで自動同期 ／ Salesforce: 週次CSV取込 |
| **デザイン** | 白ベース × ピンクアクセント、Slack風UI |
| **モバイル戦略** | 営業の総合用 + 入力専用アプリ の2種 |
| **既存データ移行** | Salesforce Data Loader経由でCSV → 新DBへ |

---

## 🏢 LIGOの事業構造（Salesforce調査から把握）

**運送・物流業界向けドライバー採用支援**
- 商品：Indeed広告運用代行、doda、求人BOX、ISパス（インサイドセールス）、採用代行、ハコプロ
- 営業フロー：IS（インサイドセールス）→ FS（フィールドセールス）→ BO（バックオフィス）→ 運用
- 営業所：拠点ベース（関東/関西/中部/九州）

---

## 🗂 Salesforce 主要オブジェクト（既存）

| オブジェクト | 役割 | 新CRMでの扱い |
|---|---|---|
| Account（取引先） | 顧客企業 | → Customer |
| Contact（取引先責任者） | 担当者 | → Person |
| Opportunity（商談） | 案件＋ヨミ管理 | → Deal |
| Task / Event | 接点記録 | → Touchpoint |
| IndeedCampaign__c | Indeed運用 | → Deal派生 or 別テーブル |
| Amount_spent_c__c | 広告費 | → AdSpend |
| 採用活動 / 応募者 / スカウト | 採用ファネル | → RecruitmentStatus（簡略化） |
| 案件 / 請求書 / 契約 | 受注・請求 | → Revenue（月次集計） |
| branch__c | 営業所 | → Branch |
| User | ユーザー | → User |

Salesforceの商談は約30+フィールド（ヒアリング・採用課題まで含む）あり、**新CRMでは「使う/見る」項目に絞って入力必須7〜10項目に削減**する方針。

---

## 💾 新CRMのコアデータモデル

```
User（営業/CS）          ─ id, name, role, branch_id
Branch（営業所）         ─ id, name
Customer（顧客）         ─ id, name, owner_id, cs_id, status, industry, branch_id, signal
Person（担当者）         ─ id, customer_id, name, role, phone, email
Touchpoint（接点）       ─ id, customer_id, person_id, user_id, date, channel, content(提案/振返り/フォロー), result, tags[], note
Deal（商談=ヨミ）        ─ id, customer_id, owner_id, product, stage(C/B/A/成約/失注), amount, close_date
Revenue（売上計上）       ─ id, customer_id, deal_id, year, month, amount
AdSpend（広告費消）       ─ id, customer_id, year, month, amount, source(Indeed/doda)
RecruitmentStatus（採用状態） ─ id, customer_id, status(A/B/C/D/E), updated_at, updated_by_id
```

### 採用ステータス定義（重要）
- **A**: 採用有充足
- **B**: 採用有未充足
- **C**: 面接有未採用
- **D**: 応募有未面接
- **E**: 応募無

→ 状態と「**前回ステータス記入日**」を蓄積。45日以上経過は ⚠️マーク + フィルタで抽出可能。

---

## 🎨 モック（3アプリ）

`/Users/nagatsumajun/Downloads/ligo-crm/` 配下に格納：

### 1. `index.html` — 🖥 PC管理者ダッシュボード
- 全社ダッシュボード / マイダッシュボード / カスタムダッシュボード（ドラッグ&ドロップ）
- 顧客一覧 / 顧客詳細
- ヨミ管理（カンバン）
- 営業/CS分析、傾向分析
- CSV取込、設定
- **採用ステータス分布ウィジェット**（社数 / 90日推移 / 平均滞在日数 / 自動示唆コメント）

### 2. `mobile.html` — 📱 営業の総合用（スマホ）
- iPhone風フレーム
- 下部タブ + 中央フローティング「＋接点」ボタン
- ホーム / 顧客 / 接点 / ヨミ管理 / マイ
- 顧客詳細に **採用ステータスバー** ＋履歴タイムライン

### 3. `input.html` — 📲 入力専用「LIPON Tap」（スマホ）
- 4タップで終わる接点記録フロー
- 音声入力ファースト
- ホームから採用ステータス更新も可能（**前回更新日表示・45日超⚠️マーク・フィルタ付き**）
- 紙吹雪🎉アニメ、ストリーク🔥、今日のカウント
- 挨拶：**「熱狂様です！🔥」**（LIGOカルチャー反映済み）

### ブラウザで開く
```bash
open /Users/nagatsumajun/Downloads/ligo-crm/index.html
open /Users/nagatsumajun/Downloads/ligo-crm/mobile.html
open /Users/nagatsumajun/Downloads/ligo-crm/input.html
```

---

## ⚙️ 推奨技術スタック（GCP）

| 層 | 技術 | 補足 |
|---|---|---|
| フロント/API | Next.js（App Router）on Cloud Run | TypeScript + Tailwind + shadcn/ui |
| DB | Cloud SQL (PostgreSQL) | db-f1-micro〜small で十分 |
| 認証 | Identity-Aware Proxy + Google Workspace SSO | 30名なら最適 |
| ストレージ | Cloud Storage | CSVアップロード保管 |
| バッチ | Cloud Scheduler + Cloud Run Jobs | Indeed APIを毎朝5時に同期 |
| シークレット | Secret Manager | Indeed APIキー等 |
| グラフ | Recharts | インフォグラフィック向き |
| 監視 | Cloud Logging / Error Reporting | — |
| ドラッグ&ドロップ | Sortable.js / dnd-kit | カスタムダッシュボード用 |

**月額イメージ**：5,000〜15,000円程度

---

## 📦 Salesforce移行計画

### Phase 1（設計検証）— サンプルCSV
Data Loaderで各オブジェクト10〜30件をエクスポート → `sf-export/` フォルダに配置 →
Claudeがフィールドマッピング設計書を作成。

### Phase 2（本番移行）— 全件
新DBスキーマ確定後、移行スクリプト（Python or Node）で投入。

### 必要なオブジェクト一覧
1. Account（20件）
2. Contact（20件）
3. Opportunity（30件、フェーズ多様）
4. Task（50件）
5. Event（30件）
6. IndeedCampaign__c（全件・11件）
7. Amount_spent_c__c（50件）
8. User（全件・IsActive=true）
9. branch__c（全件）

**保存先**: `/Users/nagatsumajun/Downloads/ligo-crm/sf-export/`

---

## ⏳ 進行中・保留中のタスク

### ✅ 完了
- [x] Salesforce調査・データ構造把握
- [x] 3モック作成（PC/モバイル/入力専用）
- [x] 採用ステータス機能の組み込み
- [x] 採用ステータス前回更新日 + 45日超マーク + フィルタ

### 🔄 ユーザー（菅原さん）が確認中
- [ ] **Indeed API仕様** — 応募数と利用金額をどう取得しているか？（API公開されてない可能性、レポートCSV方式かも）
- [ ] Salesforce Data Loader → 利用可能と確認済み（次は実エクスポート）

### ⏭ 次のステップ
1. **Salesforce CSVサンプル受領 → フィールドマッピング設計書作成**
2. Indeed API仕様確定
3. GCPプロジェクト初期化（Cloud Run + Cloud SQL）
4. DBスキーマDDL作成（Prisma推奨）
5. 認証（IAP）設定
6. フロント実装着手
7. 移行スクリプト作成

---

## 🚨 確認した重要な制約・前提

- **個人情報の取り扱い**：顧客企業データを扱うため、GCPのIAM・データ暗号化・監査ログ設定が必要
- **Indeed APIキー**：Secret Managerで管理。クライアント環境変数に絶対漏らさない
- **ピンクのトーン**：強すぎないバランス（白80% / ピンク15% / その他5% 程度）
- **挨拶文化**：「おはようございます」ではなく **「熱狂様です！🔥」** を使う
- **フィールド営業の現場**：スマホ片手で10秒以内に記録できることを最優先

---

## 💬 yuujinさんへ — このまま続けるには

1. **このフォルダ全体を受け取る**（Google Drive / Dropbox / メール添付ZIP）
2. **Claude Codeで開く**：
   ```bash
   cd path/to/ligo-crm
   claude
   ```
3. **最初のメッセージで**：
   > 「HANDOFF.md を読んで、現在のプロジェクト状況を理解してから、次に何をすべきか提案して」
4. これでClaudeが文脈を把握 → 続きから議論・実装を始められます

### このプロジェクトでよく使う指示例
- 「3つのモックを開いてレビューして」
- 「sf-export/ にCSVを置いたから、マッピング設計書を作って」
- 「GCPプロジェクトのセットアップ手順を出して」
- 「Prismaスキーマを書いて」

---

## 📁 ファイル構成

```
ligo-crm/
├── HANDOFF.md          ← このファイル
├── index.html          ← PC管理者ダッシュボード
├── mobile.html         ← スマホ総合（営業）
├── input.html          ← スマホ入力専用（LIPON Tap）
└── sf-export/          ← Salesforce CSV置き場（空、Data Loaderで投入予定）
```

---

## 🤝 連絡先・体制

- **発起人**：菅原（株式会社LIGO）
- **引き継ぎ**：yuujin@li-go.jp
- **AIアシスタント**：Claude Code（各自のローカルClaudeが本ドキュメントを文脈源として動作）

何か不明点があれば、菅原さんに直接確認してください。
