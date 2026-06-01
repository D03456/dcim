# 画面設計

本フォルダは、DCIM の画面設計書とHTMLモックを管理する。

## 方針

- 基本設計 `docs/03_basic_design/01_screen_list.md` の SCR-001〜SCR-041 を対象にする。
- HTMLモックは `mocks/` 配下に配置する。
- 共通レイアウトは、前回マッチングプロジェクトの Vaadin レイアウトCSS（左メニュー、ヘッダー、メイン、フッター、モーダル、モバイル下部メニュー）を参考にした。
- CSSは `mocks/styles/dcim-vaadin-mock.css` に集約し、Vaadin Lumo風のCSS変数を使う。
- モックは静的HTMLであり、実装時はVaadin View / Dialog / Layoutへ置き換える。

## 成果物

| ファイル | 内容 |
|---|---|
| `00_common_layout.md` | 共通レイアウト・CSS利用方針 |
| `01_screen_mock_list.md` | SCR別HTMLモック一覧 |
| `02_screen_io_definition.md` | 画面別の表示項目・入力項目・ボタン・遷移・権限制御定義 |
| `mocks/index.html` | モック入口 |
| `mocks/styles/dcim-vaadin-mock.css` | 共通CSS |
| `mocks/SCR-*.html` | 各画面モック |

## 現時点の対象画面数

44画面
