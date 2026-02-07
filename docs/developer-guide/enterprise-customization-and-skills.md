# 企業向けカスタマイズとスキル蓄積

**対象**: SuperClaude を fork して自社向けに展開する組織、またはプロジェクトごとにルール・スキルを貯めたいチーム。

---

## 1. なぜこの体制か

- **Claude Code の使い方**はまだ業界標準化の黎明期であり、企業ごとに「計画の立て方」「changelog の扱い」「社内スキルの蓄積」を決めておくと再現性が高い。
- SuperClaude を**ベースレイヤー**として入れ、その上に**企業・プロジェクト固有の設定**を重ねるハイブリッド構成にすると、upstream の更新を取り込みつつ自社最適化を維持できる。

---

## 2. レイヤー構成

下がベース、上が上書きです。

| レイヤー | 場所 | 内容例 |
|---------|------|--------|
| 1. SuperClaude ベース | `~/.claude/`（install 先） | 30 コマンド、エージェント、MCP プリセット |
| 2. プロジェクト共通 | リポジトリ直下 | `CLAUDE.md`（UV・構成・開発フロー）、`AGENTS.md` |
| 3. Cursor ルール | `.cursor/rules/*.mdc` | コーディング規約、changelog/計画ルール、企業固有ルール |
| 4. スキル | `.claude/skills/` | 共通スキル（例: confidence-check）＋企業別サブディレクトリ |

推奨ディレクトリ例（リポジトリ内）:

```
各リポジトリ/
├── .claude/
│   ├── commands/          ← SuperClaude のコマンド（install で同期）
│   ├── settings.json      ← SuperClaude + 自社の権限・設定
│   └── skills/
│       ├── confidence-check/   ← 共通スキル
│       └── <company>/          ← 企業別スキル（例: tkbase/, sighted/）
├── .cursor/
│   └── rules/
│       ├── changelog-and-planning.mdc   ← 共通（changelog・計画）
│       ├── enterprise-skills.mdc        ← 共通（スキルレイヤー説明）
│       └── <company>-*.mdc              ← 企業固有ルール
├── CLAUDE.md               ← SuperClaude ベース + 自社共通ルール
├── CHANGELOG.md            ← Keep a Changelog 形式
├── TASK.md                 ← 現在のタスク・優先度
├── PLANNING.md             ← アーキテクチャ・絶対ルール（任意）
└── <project>-claude.md     ← プロジェクト固有（Sighted / Magne / PMI 等）
```

---

## 3. Changelog と計画のルール

- **CHANGELOG.md**: [Keep a Changelog](https://keepachangelog.com/en/1.0.0/) に準拠。notable な変更ごとに `[Unreleased]` に Added/Changed/Fixed/Removed を追記する。
- **TASK.md**: 現在のタスク・優先度・バックログ。作業開始時に読み、完了したらステータスを更新する。
- **PLANNING.md**: アーキテクチャと絶対ルール。実装判断時に参照する。

これらは `.cursor/rules/changelog-and-planning.mdc` に要約されており、Cursor が常に参照する。

---

## 4. 企業ごとのスキルを貯める

### スキル置き場

- **`.claude/skills/<company>/`**  
  企業・チーム固有のスキルを置く。例: `tkbase/`, `sighted/`。  
  各スキルは `SKILL.md`（YAML frontmatter + 説明・手順）と、必要ならスクリプトを同梱する。

### Cursor ルールで企業ルールを追加

- **`.cursor/rules/<company>-*.mdc`**  
  企業固有のコーディング規約・ドメインルール。  
  - `alwaysApply: true` で常時適用  
  - または `globs: **/*.ts` などでファイル種別ごとに適用  

Fork 運用時は、upstream には `changelog-and-planning.mdc` と `enterprise-skills.mdc` のような共通ルールだけをコミットし、`<company>-*.mdc` と `.claude/skills/<company>/` は自社リポジトリだけで管理すると、upstream とのマージが楽になる。

---

## 5. 運用の流れ（例: TKBase）

1. **ベース導入**  
   `pipx install superclaude && superclaude install` を 1 リポジトリ（例: Sighted）で試し、重さや利用コマンドを確認する。
2. **共通ルールの整備**  
   `CLAUDE.md` に SuperClaude のベース＋TKBase 共通ルールを記載。`.cursor/rules/` に changelog・計画・スキルレイヤーのルールを置く。
3. **企業スキル・ルールの追加**  
   `.claude/skills/tkbase/` や `.cursor/rules/tkbase-*.mdc` で、Sighted / AEO / PMI 向けのドメイン知識や手順を貯めていく。
4. **changelog の習慣化**  
   機能追加・修正時に CHANGELOG の `[Unreleased]` を更新し、リリース時にバージョン見出しに移す。

---

## 6. 関連ドキュメント

- [Extending the Framework](technical-architecture.md#extending-the-framework) - コマンド・エージェント・モードの追加
- [Contributing Code](contributing-code.md) - 開発フローと PR
- [Documentation Index](documentation-index.md) - 開発者向けドキュメント一覧
