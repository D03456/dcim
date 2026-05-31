# 共通画面レイアウト設計

## 1. 目的

DCIMの各画面で共通利用する、Vaadin実装前提の画面レイアウトを定義する。

## 2. 基本構成

```text
.dcim-layout
├── .dcim-body
│   ├── .dcim-side-menu        左メニュー
│   └── .dcim-main
│       ├── .dcim-header       画面タイトル・概要・主要操作
│       ├── .dcim-content      一覧、フォーム、カード、詳細領域
│       └── .dcim-footer       補助情報、最終更新、バージョン等
└── .dcim-bottom-menu          スマートフォン用下部メニュー
```

## 3. マッチングプロジェクトから流用する考え方

- 左側メニューは `admin-side-menu` / `user-side-menu` 相当の構成をDCIM向けに `dcim-side-menu` として再定義する。
- ヘッダーはタイトル、補足説明、状態バッジ、主要ボタンを横並びで配置する。
- メイン領域はカード、検索条件、テーブル、フォームを組み合わせる。
- フッターは最終更新日時、表示件数、設計注記などを表示する。
- 詳細確認・編集補助・履歴確認はモーダルを基本にする。
- モバイル時は左メニューを隠し、主要4メニューを下部ナビに表示する。

## 4. Vaadin実装時の対応イメージ

| HTML/CSS | Vaadin想定 |
|---|---|
| `.dcim-layout` | `AppLayout` または共通 `MainLayout` |
| `.dcim-side-menu` | `SideNav` / `VerticalLayout` |
| `.dcim-header` | `HorizontalLayout` + `H1` + 操作ボタン |
| `.dcim-card` | `VerticalLayout` / `FormLayout` / `Grid` の外枠 |
| `.dcim-modal` | `Dialog` |
| `.dcim-bottom-menu` | モバイル用 `HorizontalLayout` |

## 5. モーダル利用方針

以下はモーダル利用を基本とする。

- 一覧からの詳細確認
- 履歴・通知・エラー詳細
- 削除確認
- 関連付け/解除確認
- CSV取込結果・行エラー確認

大きな登録・編集フォームは通常画面遷移または同一View内フォームを基本とし、軽微な状態更新・詳細確認はモーダルで扱う。
