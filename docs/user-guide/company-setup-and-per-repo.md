# 会社向けSuperClaudeの作り方とリポジトリごとの設定

**対象**: 会社・チームでSuperClaudeを導入したい方、リポジトリごとに設定を切りたい方、自分が使っているスキルをSuperClaudeと一緒に活用したい方。

※ このドキュメントはデフォルトのREADMEを変えるものではなく、**会社向けの作り方**と**オン/オフ・スキル活用**のやり方だけをまとめています。

---

## 1. 会社向けSuperClaudeをどう作るか

### パターンA: Fork しない（オーバーレイで使う）

- **やり方**: そのまま `pipx install superclaude && superclaude install` でグローバルに導入し、**リポジトリ側**で会社用の設定を重ねる。
- **リポジトリに置くもの**:
  - `CLAUDE.md` … SuperClaudeのベース（[CLAUDE.md](https://github.com/SuperClaude-Org/SuperClaude_Framework/blob/master/CLAUDE.md) を参考）＋自社の開発ルール・ツール（UV など）
  - `.cursor/rules/*.mdc` … changelog・計画・会社固有のコーディング規約
  - `.claude/skills/<会社名>/` … 自社用スキル（Sighted / AEO / PMI などプロジェクト固有の手順・ドメイン知識）
- **メリット**: upstream の更新をそのまま取り込める。運用がシンプル。
- **向いているケース**: まずは全社で「同じSuperClaude」を使い、リポジトリごとにルールとスキルだけ差し替えたい場合。

### パターンB: Fork して自社版を育てる

- **やり方**: 本リポジトリを fork し、日本語ローカライズや自社用コマンド・エージェントを追加した「SuperClaude JP」のような形で運用する。
- **Fork 側でやること**:
  - upstream を定期的に merge
  - 自社用は **別ディレクトリ** にだけ追加（例: `.claude/skills/<会社名>/`, `.cursor/rules/<会社>-*.mdc`）して、upstream と衝突しないようにする。
- **メリット**: 自社ブランドで配布したい、日本語ドキュメントを本体に含めたい場合に有利。
- **詳細**: [企業向けカスタマイズとスキル蓄積](../developer-guide/enterprise-customization-and-skills.md) を参照。

**まとめ**: 基本的には **パターンA（オーバーレイ）** で十分なことが多いです。Fork は「日本向けパッケージとして配布したい」など、明確な理由があるときでよいです。

---

## 2. リポジトリごとに設定を「オン」にしたり「オフ」にしたりするには

### デフォルトの考え方: 基本的に「全部オン」でよい

- `superclaude install` は **ユーザーのホーム** の `~/.claude/commands/sc` にコマンドを入れます（**グローバル**）。
- そのため、**どのリポジトリを開いても** Claude Code からは同じ SuperClaude コマンド（`/sc:research` など）が使えます。
- 多くの場合は **全リポジトリでこのまま「全部オン」** で問題ありません。

### リポジトリ「だけ」SuperClaude を使いたい（他はオフにしたい）

- **方法1**: グローバルにはインストールせず、**SuperClaude を使うリポジトリのルートでだけ** インストールする。
  ```bash
  cd /path/to/only-repo-that-uses-superclaude
  pipx install superclaude
  superclaude install --target .claude/commands/sc
  ```
  - このリポジトリには `.claude/commands/sc` ができるので、**このリポだけ** SuperClaude のコマンドが有効になります。
  - 他のリポジトリには `.claude/commands/sc` がないので、SuperClaude のスラッシュコマンドは出てきません（**オフ**）。

- **方法2**: グローバルには入れたまま、**特定リポではSuperClaudeに頼らない**運用にする。
  - そのリポジトリの `CLAUDE.md` に「このプロジェクトでは `/sc:` コマンドは使わず、〇〇の手順に従う」と書いておく。
  - コマンドは一覧には出ますが、AI に「このリポでは使わない」と指示する形で実質オフにできます。

### リポジトリごとに「使うコマンド」を減らしたい（軽くしたい）

- コマンドの実体は `~/.claude/commands/sc/*.md` です。
- **グローバルで減らす**: 不要な `.md` を `~/.claude/commands/sc/` から削除またはリネーム（例: `research.md` → `research.md.bak`）すると、そのコマンドだけ「オフ」になります。
- **リポジトリでだけ減らす**: そのリポに `superclaude install --target .claude/commands/sc` で入れたあと、リポジトリ内の `.claude/commands/sc/` から不要な `.md` を削除すると、そのリポだけで使うコマンドを絞れます。

### まとめ（オン/オフ）

| 目的 | やり方 |
|------|--------|
| 全部のリポで SuperClaude を使う（基本） | `superclaude install` のまま（グローバル） |
| あるリポだけ SuperClaude を使う | そのリポでだけ `superclaude install --target .claude/commands/sc` |
| あるリポでは SuperClaude を使わない | そのリポでは install しない、または CLAUDE.md で「/sc: は使わない」と記載 |
| 使うコマンドを減らしたい | `commands/sc` 配下の不要な `.md` を削除 or リネーム |

---

## 3. 自分が使っているスキルを SuperClaude と一緒に活用する

SuperClaude は **スキル** を `~/.claude/skills/`（またはリポジトリの `.claude/skills/`）で読み込みます。ここに **自分用・会社用のスキル** を置くと、そのまま SuperClaude のコマンドやエージェントと一緒に使えます。

### 既存スキルを「そのまま」使う

- **置き場所**（どちらかでよい）:
  - **グローバル**: `~/.claude/skills/<スキル名>/`
  - **リポジトリだけ**: リポジトリルートの `.claude/skills/<スキル名>/`
- **中身**: SuperClaude 付属の `confidence-check` と同じ形でよいです。
  - `SKILL.md`（YAML frontmatter ＋ 説明・手順）を入れる。
  - 必要なら `.ts` / `.py` なども同梱してよい。
- スキル名はフォルダ名（例: `my-workflow`, `tkbase-sighted`）。Claude Code は `~/.claude/skills` およびプロジェクトの `.claude/skills` を参照するため、**追加のインストールコマンドは不要**で、置くだけで読まれます。

### パッケージ付きスキルを「インストール」する形で使う

- SuperClaude に同梱されているスキル（例: `confidence-check`）は、次のようにインストールできます。
  ```bash
  superclaude install-skill confidence-check
  # 任意: リポジトリだけに入れたい場合
  superclaude install-skill confidence-check --target .claude/skills
  ```
- **自分や会社で作ったスキル**をパッケージ化している場合は、そのスキルが `SKILL.md` を含むディレクトリであれば、`install_skill` のソースとして参照するか、上記の「置き場所」に手動でコピーすれば同じように使えます。

### 今後の SuperClaude に活かしたい場合

- **自社で便利だったスキル**を汎用化できそうなら、SuperClaude 本体に PR でスキルを追加することを検討できます（リポジトリの `skills/` または `src/superclaude/skills/` に同じ形式で追加）。
- 会社内だけで使うスキルは、`.claude/skills/<会社名>/` や `.cursor/rules/<会社>-*.mdc` に置いて蓄積し、[企業向けカスタマイズとスキル蓄積](../developer-guide/enterprise-customization-and-skills.md) のレイヤーに乗せると、今後の SuperClaude のアップデートと両立しやすくなります。

---

## 4. 関連ドキュメント

- [企業向けカスタマイズとスキル蓄積](../developer-guide/enterprise-customization-and-skills.md) … Fork 時のレイヤー構成・changelog・スキル蓄積
- [クイックスタート](../getting-started/quick-start.md) … 初回インストール
- [インストールガイド](../getting-started/installation.md) … 詳細なセットアップ
