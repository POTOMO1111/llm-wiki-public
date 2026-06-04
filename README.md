# llm-wiki

**Claude Code を使った、LLM 駆動の研究知識ベース構築フレームワーク**

論文の PDF を読み込み、要約・概念ページ・エンティティページを自動生成しつつ、相互リンクされた wiki を持続的に維持・拡張するための **スケルトン + Claude 指示ファイル + カスタムスキル** を同梱したリポジトリ。学術研究の論文読解と知識統合を LLM に任せ、人間は「何を読ませるか」と「何を問うか」に集中するための仕組み。

このリポジトリ自体は **wiki の中身を含まない**。`git clone` 直後に `_skeleton/vault/` を `vault/` にコピーすれば、すぐに自分専用の wiki を組み上げられる。

---

## ディレクトリ構成

```
llm-wiki/
├── .claude/                 # Claude Code の設定とカスタムスキル
│   ├── settings.json        # 権限設定（全ユーザー共通）
│   └── skills/
│       ├── ingest-paper/SKILL.md   # /ingest-paper コマンド
│       ├── lint/SKILL.md           # /lint コマンド
│       └── query/SKILL.md          # /query コマンド
├── CLAUDE.md                # プロジェクト指示（wiki スキーマと運用ルール）
├── README.md                # 本ファイル
├── docs/
│   └── setup-guide.pdf      # 詳細セットアップガイド
├── _skeleton/               # 新規環境の初期構造テンプレ
│   └── vault/               # vault/ にコピーして使う
└── vault/                   # ユーザーのコンテンツ（.gitignore で除外）
    ├── raw/                 # 未 ingest の PDF を投入
    ├── ingested/            # ingest 済み PDF のアーカイブ（変更不可）
    └── wiki/
        ├── index.md         # コンテンツカタログ
        ├── log.md           # 操作ログ
        ├── overview.md      # 研究分野の俯瞰
        ├── sources/         # 論文要約ページ（1 論文 = 1 ファイル）
        ├── concepts/        # 概念ページ
        └── entities/        # 著者・モデル・データセット・ベンチマークページ
```

---

## クイックスタート

### 前提

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) の CLI または VSCode 拡張がインストール済み
- `pdftotext` が利用可能
  - Linux / macOS: `apt install poppler-utils` または `brew install poppler`
  - Windows: [Xpdf command-line tools](https://www.xpdfreader.com/download.html) の `pdftotext.exe` を PATH に追加

### セットアップ

```bash
# 1. リポジトリを clone
git clone git@github.com:POTOMO1111/llm-wiki-public.git my-wiki
cd my-wiki/

# 2. vault/ スケルトンをコピー
cp -r _skeleton/vault ./vault     # macOS / Linux

# Windows PowerShell の場合:
# Copy-Item -Recurse _skeleton/vault ./vault
```

`vault/` が直下にできれば準備完了。

### （任意）vault/ を独立 git リポジトリにする

スマホでの参照や、研究内容のみを別途バックアップ・同期したい場合：

```bash
cd vault/
git init
git remote add origin <自分の content 用 GitHub repo>
git add . && git commit -m "Initial commit" && git push -u origin main
cd ..
```

これで、外側の `llm-wiki-public` クローンと内側の `vault/` クローンの **2 リポジトリ構成** になる。Obsidian の Git プラグイン等と相性が良い。

---

## 使い方

Claude Code を開いた状態で以下のコマンドを実行する。

### 論文を ingest する

1. 論文 PDF を `vault/raw/` に置く
2. Claude Code で `/ingest-paper <filename>` を実行
   - Claude が PDF を読み、要約・概念・エンティティページを自動生成
   - 完了後、PDF は `<first-author>-<year>-<keyword>.pdf` に rename され `vault/ingested/` に移動
3. `ls vault/raw/` でいつでも未 ingest 一覧を確認可能（LLM トークン不要）

### wiki に問いを投げる

```
/query <自然言語の質問>
```

Claude が `vault/wiki/index.md` を参照して関連ページを読み、引用付きで回答を合成する。残しておく価値があれば synthesis page として保存提案も行う。

### wiki の健全性チェック

```
/lint
```

矛盾・孤立ページ・古い記述・概念のギャップ等を報告。5 件以上 ingest した後の実行が効果的。

---

## 設計思想

- **論文を読むのは LLM、何を読ませるかを決めるのは人間。** ingest の判断と方向付けが人間側の主要な作業。
- **`vault/raw/` と `vault/ingested/` は不変。** PDF 内容は読み取り専用、唯一の許可された書き込みは `/ingest-paper` 完了時の rename + 移動のみ。
- **`vault/raw/` がそのまま未処理キュー。** ファイル一覧を見るだけで残作業がわかるため、LLM トークンの消費なしに進捗確認できる。
- **wiki は append-only ではない。** 概念ページや overview は新規 ingest を経て継続的に更新される。`/lint` で陳腐化を検出。
- **外側 (framework) と内側 (content) を分離。** `vault/` は別 git リポジトリにできる（推奨）ことで、研究内容のみを独立してバックアップ・公開できる。

---

## 詳細仕様

- `CLAUDE.md` — wiki スキーマ・ページフォーマット・命名規則・LLM 行動規則・操作仕様の正式定義（必読）
- `.claude/skills/ingest-paper/SKILL.md` — `/ingest-paper` の 9 ステップワークフロー
- `.claude/skills/lint/SKILL.md` — `/lint` の検査項目
- `.claude/skills/query/SKILL.md` — `/query` の合成ロジック
- `docs/setup-guide.pdf` — スクリーンショット付きセットアップガイド（VSCode 拡張環境前提）

---

## ライセンス

このフレームワーク（`.claude/`, `CLAUDE.md`, `_skeleton/`, `README.md`, `docs/`）はテンプレートとして自由に複製・改変・利用してください。ユーザーが `vault/` 配下に生成する wiki コンテンツの権利は完全にユーザーに帰属します。
