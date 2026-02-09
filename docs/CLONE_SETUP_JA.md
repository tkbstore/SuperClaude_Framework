# SuperClaude リポジトリクローンからのセットアップマニュアル

このドキュメントでは、**Git でリポジトリをクローンした状態**から SuperClaude を使い始める手順と、`/sc:orchestrate` をはじめとするスラッシュコマンドの使い方をまとめています。

---

## 1. 前提条件

- **Python 3.10 以上**
- **Git**
- **UV パッケージマネージャ**（未導入の場合はインストールスクリプト内で導入を促されます）

```bash
# UV を手動で入れる場合（Windows: Git Bash や WSL で実行）
curl -LsSf https://astral.sh/uv/install.sh | sh
# インストール後、ターミナルを開き直すか PATH を反映
```

---

## 2. クローンとインストール

### 2.1 リポジトリのクローン

```bash
git clone https://github.com/SuperClaude-Org/SuperClaude_Framework.git
cd SuperClaude_Framework
```

> 自前の fork を使う場合も同様に `git clone <あなたのリポジトリURL>` でクローンし、`cd` で入ってから以下を実行します。

### 2.2 インストールスクリプトの実行

**Linux / macOS / WSL / Git Bash（Windows）:**

```bash
./install.sh
```

対話的に進めます。オプション付きで実行する場合:

```bash
./install.sh --yes    # 確認なしで実行
./install.sh --help   # ヘルプ表示
```

**Windows（PowerShell / CMD）で `install.sh` が使えない場合:**

次のように手動で実行します。

```bash
# 1) UV が無い場合は先にインストール（PowerShell 例）
#    https://github.com/astral-sh/uv を参照

# 2) リポジトリのルートで実行
cd SuperClaude_Framework
uv pip install -e ".[dev]"

# 3) スラッシュコマンドをインストール（30 コマンドが ~/.claude/commands/sc/ に入る）
superclaude install
```

### 2.3 インストール内容

- **SuperClaude パッケージ**: エディタブルモード（`-e`）でインストールされるため、クローン内の変更がそのまま反映されます。
- **スラッシュコマンド**: 約 30 個が `~/.claude/commands/sc/` に配置され、Claude Code から `/sc:xxx` で利用できます。**`/sc:orchestrate` もここに含まれます。**

---

## 3. インストール確認

```bash
# バージョン確認
superclaude --version

# ヘルスチェック
superclaude doctor

# インストール済みコマンド一覧（orchestrate が含まれることを確認）
superclaude install --list
```

**Claude Code 側:**  
Claude Code を**再起動**したあと、チャットで次のように入力してコマンドが使えるか確認します。

- `/sc` … 利用可能な SuperClaude コマンド一覧
- `/sc:help` … ヘルプ

---

## 4. `/sc:orchestrate` の使い方

`/sc:orchestrate` は **「1 コマンドでタスクのスコープを見積もり → 戦略を自動選択 → 実行 → 方向転換時にドリフト監視」** まで行うオーケストレーション用コマンドです。（「/auchestrate」ではなく **`/sc:orchestrate`** が正式なコマンド名です。）

### 4.1 基本構文

```
/sc:orchestrate [タスクの説明]
```

### 4.2 内部で行う 5 段階（自動）

1. **スコープ推定** … 影響範囲を軽く見積もり、small / medium / large に分類
2. **戦略候補の列挙** … 単一セッション / 同一セッション内並列 / エージェントチームの 3 候補
3. **自動選択** … コスト・再現性・拡張性を考慮して 1 つを自動選択（ユーザーは選ばない）
4. **実行** … 選択された戦略で実装（必要に応じて `/sc:implement` や `/sc:task` に委譲）
5. **ドリフト監視** … 「やり直し」「こっちの方がいい」など方向転換時に、元の計画との差分と手戻りリスクを出して確認してから続行

### 4.3 使用例

| タスク例 | 想定される戦略 |
|----------|----------------|
| `/sc:orchestrate "設定画面にダークモードトグルを追加"` | 単一セッションになりやすい |
| `/sc:orchestrate "認証モジュールをリファクタして単体テストを追加"` | 同一セッション内並列になりやすい |
| `/sc:orchestrate "チェックアウト: API、UI、E2E テスト"` | エージェントチームになりやすい |

### 4.4 エージェントチーム（大規模タスク）を使う場合

戦略として「エージェントチーム」が選ばれたときは、[Claude Code エージェントチーム](https://code.claude.com/docs/ja/agent-teams) が使われます。  
その前に以下を設定してください。

- **環境変数:** `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`
- または **`.claude/settings.json`** の `"env"` に上記を追加

---

## 5. その他の主なコマンド（抜粋）

| コマンド | 用途 |
|----------|------|
| `/sc` | 利用可能な SuperClaude コマンド一覧 |
| `/sc:help` | ヘルプ |
| `/sc:orchestrate "タスク"` | 段階的オーケストレーション（上記） |
| `/sc:implement "機能"` | 機能実装 |
| `/sc:research "テーマ"` | 並列検索による調査 |
| `/sc:index-repo` | リポジトリのインデックス作成 |
| `/sc:agent` | 専門エージェント |
| `/sc:recommend` | コマンド推奨 |

一覧は `superclaude install --list` および Claude Code で `/sc` を実行して確認できます。

---

## 6. オプション: MCP サーバー

パフォーマンス向上や深い調査のために、オプションで MCP サーバーを導入できます。

```bash
# 利用可能な MCP 一覧
superclaude mcp --list

# 対話式でインストール
superclaude mcp

# 特定サーバーを指定してインストール
superclaude mcp --servers tavily context7
```

詳細は [README-ja.md](../README-ja.md) の「パフォーマンス向上（オプションのMCP）」および `docs/mcp/` を参照してください。

---

## 7. ドキュメント・トラブルシューティング

- **クイックスタート:** `docs/getting-started/quick-start.md`
- **コマンド一覧（日本語):** `docs/user-guide-jp/commands.md`
- **ユーザーガイド:** `docs/user-guide/`（英語）、`docs/user-guide-jp/`（日本語）
- **インストール不調時:**  
  - `superclaude doctor` で状態確認  
  - `superclaude install --force` でコマンドの再インストール  

---

## まとめ

1. リポジトリを **クローン** し、**`./install.sh`**（または手動で `uv pip install -e ".[dev]"` + `superclaude install`）でセットアップする。
2. **Claude Code を再起動** し、`/sc` や `/sc:help` でコマンドが使えることを確認する。
3. オーケストレーションには **`/sc:orchestrate "タスク説明"`** を使う。スコープと戦略は自動で選ばれ、方向転換時はドリフト確認が行われる。
4. 大規模タスクでエージェントチームを使う場合は **`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`** を設定する。

以上が、このリポジトリをクローンして SuperClaude と `/sc:orchestrate` を使うためのマニュアルです。
