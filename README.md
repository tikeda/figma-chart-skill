**日本語** | [English](./README.en.md)

# Figma Chart Skills for Claude Code

スプレッドシートのデータ（CSV/Excel/Google Sheets）をFigmaで作ったグラフのデザインテンプレートに可視化、またはテーブルに流し込む Claude Code プラグインです。

グラフもFigma上でデザインにこだわって綺麗に描画したい。でも数値の更新やグラフのバーの調整は超面倒くさい。それをAIがいい感じにやってくれます。例えば、決算資料やIR資料の数値、チームのメンバーリスト。用途はさまざまです。このプラグインは、スプレッドシートとFigma上のテンプレートコンポーネントを橋渡しし、データの流し込みからデザイン補正までを自動化するため、テンプレートのデザイン意図を読み取って補正までしてくれます。Figma側のテンプレートは最小限の構成で済み、データの値をそのまま忠実に流し込んでくれます。

<img width="2268" height="1455" alt="image" src="https://github.com/user-attachments/assets/610176d1-c5c6-4ed4-8268-c121f19086c0" />



## Skills

### `figma-bar-chart`

**CSV → 棒グラフ**

CSVデータをFigmaの棒グラフテンプレートに流し込みます。

- **Simple** — 単一値の棒グラフ
- **Stacked** — 積み上げ棒グラフ
- **Parallel** — 並列棒グラフ（2値）

チャートタイプはCSVの末尾行で指定するか、値の列数から自動判別します。

### `figma-table`

**CSV → テーブル**

CSVデータをFigmaのテーブルテンプレートに流し込みます。テンプレートのデザインは都度異なる可能性があるため、構造を動的に解析して対応します。

- 行数・列数はCSVに応じて可変
- ヘッダー階層（期 > 四半期、期のみ、等）を動的に解析
- テンプレートのスタイル領域を自動検出し、CSVの列グループとマッピング
- テンプレートのデザイン意図を読み取り、行列の追加時にデザインを自動補正

## 前提条件

- [Claude Code](https://claude.com/claude-code) がインストール済み
- [Figma MCP Server](https://help.figma.com/hc/en-us/articles/32132100833559-Guide-to-the-Figma-MCP-Server) が接続済み（`use_figma`, `get_screenshot` 等。Figma MCP Desktop ではありません）
- Figma上にデザインテンプレート（コンポーネント）を用意済み

## インストール

### クイックスタート（単一セッション）

```bash
git clone https://github.com/tikeda/figma-chart-skill.git
claude --plugin-dir ./figma-chart-skill
```

### 恒久インストール

Claude Code セッション内で：

```
/plugin marketplace add tikeda/figma-chart-skill
/plugin install figma-bar-chart@figma-chart-skill
```

## 使い方

```bash
# CSVファイルのパスとFigmaテンプレートのURLを伝える
figma-table samples/table.csv https://www.figma.com/design/xxx/yyy?node-id=47-658
```

CSVファイルのパスはFinderからターミナルにドロップでもOKです。テンプレートのURL、ガイド設定などは対話的に確認しながら進めます。

## 仕組み

### figma-bar-chart（11 steps）

| ステップ | 内容 | 確認 |
|------|------|:------:|
| 1 | CSVファイルの取得 | yes |
| 2 | CSVの読み込みとパース | yes |
| 3 | チャートタイプの判別 | yes |
| 4 | FigmaテンプレートのURL取得 | yes |
| 5 | ガイド設定の取得（最大値・刻み幅） | yes |
| 6 | テンプレートのインスタンス化 | |
| 7 | 全レベルのDetach | |
| 8 | ガイドの追加とラベル設定 | |
| 9 | データの流し込み（バー高さ計算） | |
| 10 | ラベルの設定（月/年の表示制御） | |
| 11 | スクリーンショットで確認 | yes |

### figma-table（10 steps）

| ステップ | 内容 | 確認 |
|------|------|:------:|
| 1 | CSVファイルの取得 | yes |
| 2 | CSVの読み込みとパース | yes |
| 3 | FigmaテンプレートのURL取得 | yes |
| 4 | テンプレート構造の解析 | yes |
| 5 | 列グループとスタイル領域のマッピング | |
| 6 | テンプレートのインスタンス化 | |
| 7 | 行数・列数の調整 | |
| 8 | データの流し込み | |
| 9 | デザイン自動補正 | |
| 10 | スクリーンショットで確認 | yes |

## 設計思想

### デザインの主導権は人間が持つ

すべてをAIに任せるのではなく、**自分が作ったテンプレートの上でAIが動く**ことを目指しています。デザインの主導権は人間が持ち、AIは退屈な作業を引き受けます。

### テンプレートは最小限、補正はSkillで

Figma側のテンプレートはデザインの骨格だけを定義します。行列数の違い、テキスト溢れ、ストロークの整合性といった調整はSkillが自動で補正します。テンプレートの既存セルのプロパティパターンを「デザイン意図」として読み取り、追加・変更したセルにそのパターンを適用します。

### CSVがデータの原本

データの加工や計算はCSV側で完結させます。Skill側ではCSVの値をそのまま流し込み、`▲` や `-` といった表記もCSVの表現をそのまま尊重します。

## サンプルデータ

| ファイル | 用途 |
|------|------|
| `samples/simple.csv` | シンプル棒グラフ |
| `samples/stacked.csv` | 積み上げ棒グラフ |
| `samples/parallel.csv` | 並列棒グラフ |
| `samples/table.csv` | テーブル（四半期マトリクス型） |
| `samples/table2.csv` | テーブル（事業セグメント型） |
| `samples/table3.csv` | テーブル（プロジェクトメンバーリスト） |
| `samples/table4.csv` | テーブル（料金プラン比較表） |
| `samples/Figma-Chart_Sample.fig` | Figmaデザインテンプレート |
