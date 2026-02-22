---
name: find-skills
description: ユーザーが「○○する方法は？」「○○のスキルある？」と聞いたときに、skills.shエコシステムからスキルを検索・提案する。セキュリティのため、自動インストールは行わず、内容確認後に手動導入する安全版。
---

# スキル検索（安全版）

skills.sh（Vercel運営のスキルマーケットプレイス）から、必要なスキルを検索・提案するスキルです。

## セキュリティポリシー

**重要：以下のルールを必ず守ること**

1. `npx skills` コマンドは使用しない（リモートコード実行を避けるため）
2. スキルの自動インストール（`-y`フラグ）は絶対に行わない
3. スキルの導入前に必ず**ソースコードの内容をユーザーに提示**して承認を得る
4. インストールは GitHub からソースを確認した上で**ローカルにファイルをコピー**する方式で行う

## このスキルを使うタイミング

ユーザーが以下のような質問をしたとき：

- 「○○する方法は？」（既存スキルで対応できそうな一般的なタスク）
- 「○○のスキルを探して」
- 「○○ができるスキルはある？」
- エージェントの機能を拡張したいとき

## スキル検索の手順

### ステップ1: ニーズを理解する

ユーザーの質問から以下を特定する：

1. 分野（例：React、テスト、デザイン、デプロイ）
2. 具体的なタスク（例：テスト作成、アニメーション、PRレビュー）
3. 既存スキルで対応できそうか

### ステップ2: skills.shで検索する

WebSearch または WebFetch ツールを使って skills.sh で検索する：

```
https://skills.sh で「[キーワード]」を検索
```

例：
- 「Reactアプリを高速化したい」→ skills.sh で「react performance」を検索
- 「PRレビューを手伝って」→ skills.sh で「pr review」を検索
- 「チェンジログを作りたい」→ skills.sh で「changelog」を検索

### ステップ3: ユーザーに提案する

スキルが見つかったら、以下の情報を提示する：

1. スキル名と機能の説明
2. skills.shの詳細ページURL
3. **GitHubのソースコードURL**（内容確認用）

提示例：

```
「vercel-react-best-practices」というスキルが見つかりました。
VercelエンジニアリングによるReact/Next.jsパフォーマンス最適化ガイドラインです。

詳細: https://skills.sh/vercel-labs/agent-skills/vercel-react-best-practices
ソースコード: https://github.com/vercel-labs/agent-skills

導入する場合、まずソースコードの内容を確認しますか？
```

### ステップ4: 安全にインストールする

ユーザーが導入を希望した場合：

1. GitHubからSKILL.mdの内容を取得して表示する
2. ユーザーの承認を得る
3. ローカルの `~/.claude/skills/user/` にファイルをコピーする

```bash
# ローカルにスキルディレクトリを作成
mkdir -p ~/.claude/skills/user/[スキル名]

# SKILL.mdを手動で配置（内容確認済みのもの）
```

**絶対にやらないこと：**
- `npx skills add` による自動インストール
- `-y` フラグによる確認スキップ
- ソースコード未確認のままのインストール

## よく検索されるカテゴリ

| カテゴリ | 検索キーワード例 |
|---------|----------------|
| Web開発 | react, nextjs, typescript, css, tailwind |
| テスト | testing, jest, playwright, e2e |
| デプロイ | deploy, docker, kubernetes, ci-cd |
| ドキュメント | docs, readme, changelog, api-docs |
| コード品質 | review, lint, refactor, best-practices |
| デザイン | ui, ux, design-system, accessibility |
| 生産性 | workflow, automation, git |

## スキルが見つからない場合

1. 該当するスキルが見つからなかったことを伝える
2. エージェントの一般的な能力で直接サポートする
3. 必要であれば独自スキルの作成を提案する
