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
    Login --> PasswordResetRequest[SCR-001A パスワード再設定依頼]
    PasswordResetRequest --> PasswordResetMail[再設定メール送信完了]
    PasswordResetMail -. メールリンク .-> PasswordResetForm[SCR-001B 新パスワード設定]
    PasswordResetForm --> Login
    PasswordResetForm --> PasswordResetError[期限切れ・使用済み・不正トークン]
    PasswordResetError --> PasswordResetRequest
    InvitationMail[招待メール] -. メールリンク .-> InvitationAccept[SCR-001C 招待承諾・初回パスワード設定]
    InvitationAccept --> Login

    Dashboard --> DCList[SCR-003 データセンター一覧]
    Dashboard --> DeviceList[SCR-012 機器一覧]
    Dashboard --> MaintenanceAlert[SCR-021 保守期限アラート一覧]
    Dashboard --> NotificationList[SCR-036 通知一覧]
    Dashboard --> Usage[SCR-030 契約プラン・利用状況]
    Dashboard --> CrossSearch[SCR-041 横断検索]
    CrossSearch --> DCDetail
    CrossSearch --> DeviceDetail
    CrossSearch --> MaintenanceDetail
    SystemTenantList[SCR-039 テナント一覧] --> SystemTenantEdit[SCR-040 テナント登録・編集]

    DCList --> DCDetail[SCR-004 データセンター詳細]
    DCList --> DCEdit[SCR-005 データセンター登録・編集]
    DCDetail --> DCEdit
    DCDetail --> BuildingFloor[SCR-006 建物・フロア一覧]
    DCDetail --> ContactList[SCR-037 連絡先一覧]
    ContactList --> ContactEdit[SCR-038 連絡先登録・編集]
    BuildingFloor --> RackRow[SCR-007 ラック列一覧]
    RackRow --> RackList[SCR-008 ラック一覧]
    RackList --> RackDetail[SCR-009 ラック詳細]
    RackList --> RackEdit[SCR-010 ラック登録・編集]
    RackDetail --> RackEdit
    RackTemplate[SCR-011 ラックテンプレート一覧]
    RackTemplate --> RackEdit
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
    MaintenanceDetail --> ContactList
    NoMaintenance[SCR-020 保守未設定機器一覧] --> DeviceDetail
    MaintenanceAlert --> MaintenanceDetail

    FutureCloud[将来拡張: クラウド管理] -.-> CloudAccount[SCR-022 クラウドアカウント一覧]
    CloudAccount -.-> CloudResourceList[SCR-023 クラウドリソース一覧]
    CloudResourceList -.-> CloudResourceDetail[SCR-024 クラウドリソース詳細]

    Tag[SCR-025 タグ管理]
    Notification[SCR-026 通知設定]
    NotificationList --> Notification
    NotificationList --> MaintenanceDetail
    NotificationList --> DeviceDetail
    UserList[SCR-027 ユーザー一覧] --> UserEdit[SCR-028 ユーザー登録・編集]
    Role[SCR-029 権限ロール一覧]
    Usage --> Option[SCR-031 オプション追加]
    AuditLog[SCR-032 操作履歴]
    SystemSetting[SCR-033 システム設定]
    CsvImport[SCR-034 CSVインポート画面]
    Region[SCR-035 リージョン管理画面]
    ContactMenu[SCR-037 連絡先一覧] --> ContactEdit
```

## 4. メニュー構成

| メニュー | 遷移先 | 備考 |
|---|---|---|
| パスワード再設定 | SCR-001A〜SCR-001B | ログイン画面から遷移。メールリンク経由で新パスワード設定へ進む |
| 招待承諾 | SCR-001C | 招待メールリンクから遷移。初回パスワード設定後にログインへ誘導する |
| テナント管理 | SCR-039〜SCR-040 | システム管理者専用。テナント登録・編集・解約を行う |
| 横断検索 | SCR-041 | キーワード、別名、タグから主要リソースを横断検索する |
| ダッシュボード | SCR-002 | ログイン後の初期画面 |
| データセンター | SCR-003 | DC、建物、フロア、エリア管理の入口。フロア図はグリッド表示を基本とする |
| ラック | SCR-008 | ラック管理の入口。ラック詳細ではU単位グリッドで搭載状態を表示する |
| ラックテンプレート | SCR-011 | ラックテンプレート管理、テンプレートからラック作成 |
| 機器 | SCR-012 | サーバー/NW機器管理の入口 |
| IPサブネット | SCR-015 | IPサブネット・IP利用状況管理の入口 |
| 保守契約 | SCR-017 | 保守契約管理の入口 |
| 保守未設定 | SCR-020 | 保守契約未設定機器の確認 |
| 保守期限アラート | SCR-021 | 期限切れ/期限間近契約の確認 |
| 将来拡張：クラウド | SCR-022 | クラウドアカウント・リソース管理 |
| タグ管理 | SCR-025 | 共通タグ管理 |
| 通知設定 | SCR-026 | 通知条件・通知先管理 |
| 通知一覧 | SCR-036 | 未読通知確認、通知詳細、既読化 |
| ユーザー管理 | SCR-027 | ユーザー・ロール管理 |
| 契約プラン | SCR-030 | 利用状況・オプション管理 |
| 操作履歴 | SCR-032 | 初期必須の操作履歴検索・閲覧。変更差分履歴は将来拡張 |
| システム設定 | SCR-033 | テナント設定 |
| CSVインポート | SCR-034 | 初期追加対象。初期リリースでは非表示または無効化 |
| リージョン管理 | SCR-035 | 地域・都道府県分類の管理 |
| 連絡先管理 | SCR-037〜SCR-038 | DC・保守契約に紐づく連絡先の管理 |

## 5. 代表的な業務遷移

### 5.1 データセンター・ラック・機器登録

```mermaid
sequenceDiagram
    actor User as テナント管理者/運用管理者/編集者
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
    actor User as テナント管理者/運用管理者/編集者
    User->>Dashboard: 保守期限アラート確認
    User->>MaintenanceAlert: 保守期限アラート一覧へ遷移
    User->>MaintenanceDetail: 対象契約を確認
    User->>DeviceDetail: 対象機器を確認
```

### 5.3 画面内通知確認

```mermaid
sequenceDiagram
    actor User as 利用者
    User->>Dashboard: 未読通知を確認
    User->>NotificationList: 通知一覧へ遷移
    User->>NotificationList: 通知を既読化
    User->>MaintenanceDetail: 対象契約・機器を確認
```

### 5.4 プラン上限確認とオプション追加

```mermaid
sequenceDiagram
    actor Billing as 管理者/契約管理者
    Billing->>Usage: 契約プラン・利用状況確認
    Billing->>Option: オプション追加画面へ遷移
    Billing->>Option: オプション追加を申請
    Billing->>Usage: 申請状態を確認
```

### 5.5 パスワード再設定

```mermaid
sequenceDiagram
    actor User as 未認証ユーザー
    User->>Login: パスワード再設定を選択
    User->>PasswordResetRequest: メールアドレスを入力
    PasswordResetRequest-->>User: 存在有無を推測させない完了表示
    User->>PasswordResetForm: メールリンクから新パスワード設定
    alt トークン有効
        PasswordResetForm-->>User: 設定完了、ログイン画面へ誘導
    else 期限切れ・使用済み・不正
        PasswordResetForm-->>User: エラー表示、再設定依頼へ誘導
    end
```
### 5.6 横断検索

```mermaid
sequenceDiagram
    actor User as 利用者
    User->>CrossSearch: キーワード・種別・タグで検索
    CrossSearch-->>User: 権限のある結果のみ表示
    User->>CrossSearch: 結果を選択
    CrossSearch-->>User: 対象詳細画面へ遷移
```

### 5.7 テナント管理

```mermaid
sequenceDiagram
    actor SysAdmin as システム管理者
    SysAdmin->>SystemTenantList: テナント一覧を確認
    SysAdmin->>SystemTenantEdit: テナント登録・編集
    SysAdmin->>SystemTenantEdit: 解約/状態変更
    SystemTenantEdit-->>SysAdmin: 操作履歴を記録
```


### 5.8 ユーザー招待承諾

```mermaid
sequenceDiagram
    actor Invitee as 招待ユーザー
    Invitee->>InvitationAccept: 招待メールリンクからアクセス
    InvitationAccept-->>Invitee: トークン検証結果を表示
    alt トークン有効
        Invitee->>InvitationAccept: 初回パスワード設定
        InvitationAccept-->>Invitee: 利用開始完了、ログイン画面へ誘導
    else 期限切れ・取消・使用済み
        InvitationAccept-->>Invitee: エラー表示、管理者への再招待依頼を案内
    end
```

## 6. 遷移制御

| 条件 | 制御内容 |
|---|---|
| 未ログイン | ログイン画面へリダイレクト |
| 権限不足 | 403相当の権限エラー画面またはメッセージ表示 |
| プラン上限到達 | 登録不可、またはオプション追加画面への導線を表示 |
| 対象データなし | 一覧へ戻す、またはデータ不存在メッセージ表示 |
| テナント不一致 | アクセス拒否 |
