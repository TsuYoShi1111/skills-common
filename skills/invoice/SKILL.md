---
name: invoice
description: 請求書を自動生成するスキル。「請求書」「invoice」「請求書作って」「/invoice」で発動する。解決ガイド・司法書士ガイド・司法書士集客の3種類に対応。ExcelテンプレートにAppleScript経由で値を書き込み、PDFではなくxls/xlsx形式で出力する。
---

# 請求書自動生成スキル

既存スクリプト `~/invoice-generator/generate_invoice.py` を使って請求書を生成する。

## ワークフロー

### Step 1: タイプ選択

AskUserQuestionで以下の3つから選ばせる：

| タイプ | コマンド引数 | 金額 | 支払期限 |
|--------|-------------|------|----------|
| 解決ガイド | `kaiketsu` | 固定 1,143,630円 | 対象月末日 |
| 司法書士ガイド | `guide` | 固定 1,143,630円 | 対象月末日 |
| 司法書士集客 | `syukyaku` | 変動（件数×単価） | 翌月10日 |

### Step 2: 対象月の確認

AskUserQuestionで対象月を確認する。デフォルトは当月（YYYY-MM形式）。

### Step 3: 件数入力（司法書士集客のみ）

`syukyaku` を選んだ場合のみ、AskUserQuestionで各クライアントの件数を聞く。

対象クライアント一覧：
- ウイズユー司法書士事務所（単価2,000円）
- アストレックス司法書士事務所（単価2,000円）
- 司法書士法人リエゾン（単価2,000円）
- SAO司法書士法人（単価2,000円）
- グリフィン法務事務所（単価1,500円）
- ふくだ総合法務事務所（単価2,000円）
- 六本木総合法務務所成果（単価5,000円）
- SAO成果（単価5,000円）

件数入力後、以下のようにパイプでスクリプトに渡す：
```bash
echo -e "件数1\n件数2\n件数3\n件数4\n件数5\n件数6\n件数7\n件数8" | python3 ~/invoice-generator/generate_invoice.py syukyaku --month YYYY-MM
```

### Step 4: スクリプト実行

解決ガイド・司法書士ガイドの場合：
```bash
python3 ~/invoice-generator/generate_invoice.py {kaiketsu|guide} --month YYYY-MM
```

### Step 5: 結果報告

実行結果をユーザーに報告する。以下を含める：
- 生成されたファイルパス
- 請求日（実行日）
- 支払期限
- 御請求額

## 前提条件
- Microsoft Excelがインストール済みであること
- `jpholiday` Pythonパッケージがインストール済みであること
- 出力先 `~/Desktop/全体まとめ/請求書/【請求書】アンデリアル/` が存在すること
