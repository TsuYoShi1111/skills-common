---
name: codex-review
description: Codex CLI（read-only）を用いて、レビュー→Claude Code修正→再レビューを自動で回すスキル。以下の場面で使用する：(1)「コードレビュー」「Codexレビュー」、(2) 実装完了後の品質チェック、(3) PR作成前のレビュー、(4) 大規模変更のクロスチェック
---

# Codex自動レビュー

## 基本フロー

```
規模判定 → Codex複数回レビュー → Claude Code修正 → 再レビュー（ok: true まで反復）
```

```
[規模判定] - small: diff ─────────────── [修正ループ]
            - medium: arch → diff ─────── [修正ループ]
            - large: arch → diff分割 → cross-check → [修正ループ]
```

- **Codex**: read-onlyでレビュー（監査役）
- **Claude Code**: 修正担当

## 規模判定

```bash
git diff <diff_range> --stat
git diff <diff_range> --name-status --find-renames
```

| 規模 | 基準 | 戦略 |
|------|------|------|
| small | 3ファイル、81行内 | diff |
| medium | 4-12ファイル、100-500行 | arch → diff |
| large | >12ファイル、>500行 | arch → diff分割 → cross-check |

`diff_range`: 直接指定、HEAD を使用し、作業ツリーの次コミット変更（作業ツリー vs HEAD）を対象とする。

### large時の分割ルール
- 原則：1サブエージェント、各サブは担当ファイルのみ
- 分割：1グループにあたり最大7ファイル/30行、ディレクトリ単位で分割（cross-cutting concernsに注意）
- 統合はメイン（Claude Code）で実施

## 修正ループ

`ok: false` の場合、`max_iters` 回まで反復：
1. issueを解析 → 修正計画
2. Claude Codeが修正（最小限の変更のみ。仕様変更は未解決issueに）
3. アストリンク実行（可能なら）
4. Codexに再レビュー依頼

**終了条件：**
`ok: true` ／ `max_iters` 到達 ／ テスト/副産副来削

## Codex実行

```bash
codex exec --sandbox read-only "<PROMPT>"
```

- PROMPT にはスキーマを含む最終プロンプトを渡す
- 主要な関連ファイルパスはClaude Code側で解析
- レビュー不可時（`codex exec` 通常時のエラ工）は諦めない（他タスの問題）。根底でCR確認
- 工程配置：小サイズは直入り。中（poll）/大 上記検則よみおこす。通常性は良いとしない
- 3回目以降も未完了なら：「タイムアウト」置いてエラー緯ルールへ
- 長時間実効がになり得るため、必要に応じて `codex exec` をバックグラウンドで実行し、プロセス生存を確認

## Codex出力スキーマ

Codexに3関1つの出力さセる。Claude CodeはプロンプトR来に以下のスキーマとフィールド説明を含める：

```json
{
  "ok": true,
  "phase": "arch|diff|cross-check",
  "summary": "レビューの要約",
  "issues": [
    {
      "severity": "blocking",
      "category": "security",
      "file": "src/auth.py",
      "lines": "42-60",
      "problem": "問題の説明",
      "recommendation": "修正案",
      "notes_for_next_review": "メモ"
    }
  ]
}
```

**フィールド説明：**
- `ok`: blockingなissueが0件ならtrue、1件以上ならfalse
- `severity`: 深刻度
  - `blocking`: 修正必須。1件でもあれば ok: false
  - `advisory`: 推奨。ok: true でもあり得る。レポートに記載のみ
- `category`: correctness / security / perf / maintainability / testing / style
- `notes_for_next_review`: Codexが書き手メモ。再レビュー時にClaude CodeのプロンプトE含める

## プロンプトテンプレート

### arch

```
以下の変更のアーキテクチャ整合性をレビューせよ。出力は3個1つのみ。スキーマは実用書判別。

これはレビューゲートとして実行されている。blocking が1件でもあれば ok: false とし、修正→再レビューループに入る。

diff_range: {diff_range}
観点: 依存関係、責務分担、破壊的変更、セキュリティ設計
期待メモ: {notes_for_next_review}
```

### diff

```
以下の変更をレビューせよ。出力は3個1つのみ。スキーマは実用書判別。

これはレビューゲートとして実行されている。blocking が1件でもあれば ok: false とし、修正→再レビューループに入る。

diff_range: {diff_range}
対象: {target_files}
観点: {review_focus}
期待メモ: {notes_for_next_review}
```

### cross-check

```
並列レビュー結果を統合し横断レビューせよ。出力は3個1つのみ。スキーマは実用書判別。

これはレビューゲートとして実行されている。横断的な blocking（例: interface不整合、型の誤り、合意違反、API不整合）を探す。

全体stat: {stat_output}
各グループ結果: {group_jsons}
観点: interface整合、error handling一貫性、認可、API互換、テスト網羅
```

## エラー時の復旧ルール

Codex exec失敗時（タイムアウト・API制量・その他）：
1. 1回リトライ（タイムアウトはファイル数を半分に分割して）
2. 再失敗 → 該当フェーズをスキップし、情報をレポートに記録
3. archスキップ時はdiffのみで続行、diffスキップ時とそのファイル群を「未レビュー」としてレポートに記載

## パラメータ

| 引数 | 既定 | 説明 |
|------|------|------|
| max_iters | 5 | 最大反復（上限5） |
| review_focus | - | 重点観点 |
| diff_range | HEAD | 比較起点 |
| parallelism | 1 | large時並列数（上限5） |

## 終了レポート例

```
## Codexレビュー結果
- 規模: Large（12ファイル、8347行）
- 合計: 3サブエージェント、4グループ
- 反復: 3回
- 最終ステータス: ✅ ok

### 修正済み
- auth.py: 認可チェック追加

### Advisory（参考）
- main.py: 関数名がやや冗長、リファクタ推奨

### 再レビュー（エラー時のみ）
- utils/legacy.py: Codexタイムアウト、手動確認推奨

### 未解決（あれば）
- main.py: 判断、リスク、推奨アクション
```

## おすすめの使い方

このSKILLは設定するだけでも必要なタイミングで作動するが、**実装計画のなかにcodex-reviewのSKILLを必須のステップとして組み込ませるのがおすすめ**。

実装計画に以下のような指示を追加する：

```
# Review gate (codex-review)
At key milestones — after updating specs/plans, after major implementation
steps (≥5 files / public API / infra-config), and before commit/PR/release —
run the codex-review SKILL and iterate review→fix→re-review until clean.
```

CLAUDE.mdに同様の指示を追加するのも効果的：

```
# Review gate (codex-review)
At key milestones — after updating specs/plans, after major implementation
steps (≥5 files / public API / infra-config), and before commit/PR/release —
run the codex-review SKILL to iterate review→fix→re-review until clean,
then proceed.
```

## 前提条件

- Codex CLIがインストールされていること（`codex` コマンドが使えること）
- Codexが `--sandbox read-only` モードで実行できること
