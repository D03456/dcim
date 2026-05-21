# 画面遷移

## 1. 目的

本書は、Data Center Asset & Infrastructure Manager の主要画面遷移を定義する。

## 2. 全体方針

- ログイン後はダッシュボードへ遷移する。
- 左サイドメニューから主要機能へ遷移する。
- 各一覧画面から詳細画面、登録・編集画面へ遷移する。
- 詳細画面から関連情報へ遷移できるようにする。
- 削除は原則として一覧または詳細画面から確認ダイアログを経由して実行する。

## 3. 主要画面遷移図

```mermaid
flowchart TD
    Login[SCR-001 ログイン] --> Dashboard[SCR-002 ダッシュボード]

    Dashboard --> DCList[SCR-003 データセンター一覧]
    Dashboard --> DeviceList[SCR-012 機器一覧]
    Dashboard --> MaintenanceAlert[SCR-021 保守期限アラート一覧]
    Dashboard --> Usage[SCR-030 契約プラン・利用状況]

    DCList --> DCDetail[SCR-004 データセンター詳細]
    DCList --> DCEdit[SCR-005 データセンター登録・編集]
    DCDetail --> DCEdit
    DCDetail --> BuildingFloor[SCR-006 建物・フロア一覧]
    BuildingFloor --> RackRow[SCR-007 ラック列一覧]
    RackRow --> RackList[SCR-008 ラック一覧]
    RackList --> RackDetail[SCR-009 ラック詳細]
    RackList --> RackEdit[SCR-010 ラック登録・編集]
    RackDetail --> RackEdit
    RackDetail --> DeviceDetail[SCR-013 機器詳細]

    DeviceList --> DeviceDetail
    DeviceList --> DeviceEdit[SCR-014 機器登録・編集]
    DeviceDetail --> DeviceEdit
    DeviceDetail --> IPList[SCR-015 IPサブネット一覧]
    DeviceDetail --> MaintenanceDetail[SCR-018 保守契約詳細]

    IPList --> IPEdit[SCR-016 IPサブネット登録・編集]

    MaintenanceList[SCR-017 保守契約一覧] --> MaintenanceDetail
    MaintenanceList --> MaintenanceEdit[SCR-019 保守契約登録・編集]
    MaintenanceDetail --> MaintenanceEdit
    MaintenanceDetail --> DeviceDetail
    NoMaintenance[SCR-020 保守未設定機器一覧] --> DeviceDetail
    MaintenanceAlert --> MaintenanceDetail

    CloudAccount[SCR-022 クラウドアカウント一覧] --> CloudResourceList[SCR-023 クラウドリソース一覧]
    CloudResourceList --> CloudResourceDetail[SCR-024 クラウドリソース詳細]

    Tag[SCR-025 タグ管理]
    Notification[SCR-026 通知設定]
    UserList[SCR-027 ユーザー一覧] --> UserEdit[SCR-028 ユーザー登録・編集]
    Role[SCR-029 権限ロール一覧]
    Usage --> Option[SCR-031 オプション追加]
    Audit[SCR-032 監査ログ]
    SystemSetting[SCR-033 システム設定]
```

## 4. メニュー構成

| メニュー | 遷移先 | 備考 |
|---|---|---|
| ダッシュボード | SCR-002 | ログイン後の初期画面 |
| データセンター | SCR-003 | DC、建物、フロア、エリア管理の入口 |
| ラック | SCR-008 | ラック管理の入口 |
| 機器 | SCR-012 | サーバー/NW機器管理の入口 |
| IPサブネット | SCR-015 | IPサブネット・IP利用状況管理の入口 |
| 保守契約 | SCR-017 | 保守契約管理の入口 |
| 保守未設定 | SCR-020 | 保守契約未設定機器の確認 |
| 保守期限アラート | SCR-021 | 期限切れ/期限間近契約の確認 |
| クラウド | SCR-022 | クラウドアカウント・リソース管理 |
| タグ管理 | SCR-025 | 共通タグ管理 |
| 通知設定 | SCR-026 | 通知条件・通知先管理 |
| ユーザー管理 | SCR-027 | ユーザー・ロール管理 |
| 契約プラン | SCR-030 | 利用状況・オプション管理 |
| 監査ログ | SCR-032 | 操作履歴確認 |
| システム設定 | SCR-033 | テナント設定 |

## 5. 代表的な業務遷移

### 5.1 データセンター・ラック・機器登録

```mermaid
sequenceDiagram
    actor User as 管理者/運用者
    User->>Dashboard: ログイン後表示
    User->>DCList: データセンター一覧へ遷移
    User->>DCEdit: データセンター登録
    User->>RackList: ラック一覧へ遷移
    User->>RackEdit: ラック登録
    User->>DeviceList: 機器一覧へ遷移
    User->>DeviceEdit: 機器登録
    User->>DeviceDetail: 登録結果確認
```

### 5.2 保守期限確認

```mermaid
sequenceDiagram
    actor User as 管理者/運用者
    User->>Dashboard: 保守期限アラート確認
    User->>MaintenanceAlert: 保守期限アラート一覧へ遷移
    User->>MaintenanceDetail: 対象契約を確認
    User->>DeviceDetail: 対象機器を確認
```

### 5.3 プラン上限確認とオプション追加

```mermaid
sequenceDiagram
    actor Admin as 管理者
    Admin->>Usage: 契約プラン・利用状況確認
    Admin->>Option: オプション追加画面へ遷移
    Admin->>Usage: 追加後の上限を確認
```

## 6. 遷移制御

| 条件 | 制御内容 |
|---|---|
| 未ログイン | ログイン画面へリダイレクト |
| 権限不足 | 403相当の権限エラー画面またはメッセージ表示 |
| プラン上限到達 | 登録不可、またはオプション追加画面への導線を表示 |
| 対象データなし | 一覧へ戻す、またはデータ不存在メッセージ表示 |
| テナント不一致 | アクセス拒否 |
