# Skills - Common

どの環境でも使える汎用Claude Codeスキルです。

## スキル一覧

### Commands（スラッシュコマンド）
| コマンド | 説明 |
|----------|------|
| `/email` | ビジネスメールをフォーマル・スタンダードの2パターン作成 |
| `/fix` | 問題の調査・修正を段階的に実行 |

### Skills（高機能スキル）
| スキル | 説明 |
|--------|------|
| codex-review | コードレビュー |
| issue-workflow | Issue管理ワークフロー |
| skill-creator | 新しいスキルの作成 |
| accounting-helper | 会計・経理サポート |
| competitive-analysis | 競合分析 |
| manual-creator | マニュアル作成 |
| asset-strategy | 資産戦略 |
| psychology-notes | 心理学ノート |

## インストール方法

```bash
git clone https://github.com/TsuYoShi1111/skills-common.git
cp skills-common/commands/*.md ~/.claude/commands/
cp -r skills-common/skills/* ~/.claude/skills/user/
```
