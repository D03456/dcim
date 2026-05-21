# 4. 詳細設計 - テーブル定義

## 1. 目的

本書は、Data Center Asset & Infrastructure Manager のMariaDB向けテーブル定義案を示す。

## 2. 共通方針

### 2.1 命名規則

| 対象 | 規則 | 例 |
|---|---|---|
| テーブル名 | スネークケース・単数または業務名 | data_center |
| カラム名 | スネークケース | tenant_id |
| 主キー | `{table}_id` | data_center_id |
| 外部キー | 参照先主キー名 | tenant_id |
| 論理削除 | deleted boolean | deleted |

### 2.2 共通カラム

原則として業務テーブルには以下を付与する。

| カラム | 型 | NULL | 説明 |
|---|---|---:|---|
| created_at | datetime(6) | NO | 作成日時 |
| updated_at | datetime(6) | NO | 更新日時 |
| deleted | boolean | NO | 論理削除フラグ |

### 2.3 マルチテナント方針

- 業務データには `tenant_id` を必ず保持する。
- Repository検索時は原則として `tenant_id` と `deleted = false` を条件に含める。
- 一意制約は必要に応じて `tenant_id` を含める。

## 3. テーブル一覧

| No | テーブル名 | 概要 |
|---:|---|---|
| 1 | tenant | テナント |
| 2 | subscription_plan | 契約プラン |
| 3 | tenant_add_on | 追加利用枠 |
| 4 | app_user | 利用者 |
| 5 | role | ロール |
| 6 | user_role | ユーザー・ロール関連 |
| 7 | region | 地域 |
| 8 | data_center | データセンター |
| 9 | building | 棟 |
| 10 | floor | フロア |
| 11 | area | 区画 |
| 12 | rack_row | ラック列 |
| 13 | rack | ラック |
| 14 | device | 機器 |
| 15 | ip_address | IPアドレス |
| 16 | maintenance_contract | 保守契約 |
| 17 | maintenance_contract_device | 保守契約・機器関連 |
| 18 | contact | 連絡先 |
| 19 | data_center_contact | DC・連絡先関連 |
| 20 | tag | タグ |
| 21 | tagged_resource | タグ関連 |
| 22 | notification_setting | 通知設定 |
| 23 | notification_log | 通知ログ |
| 24 | cloud_account | クラウドアカウント |
| 25 | cloud_resource | クラウドリソース |

## 4. 主要テーブル定義

## 4.1 tenant

| カラム | 型 | NULL | キー | 説明 |
|---|---|---:|---|---|
| tenant_id | bigint | NO | PK | テナントID |
| tenant_name | varchar(100) | NO |  | テナント名 |
| plan_code | varchar(30) | NO | FK | 契約プランコード |
| status | varchar(30) | NO |  | ACTIVE / SUSPENDED |
| created_at | datetime(6) | NO |  | 作成日時 |
| updated_at | datetime(6) | NO |  | 更新日時 |
| deleted | boolean | NO |  | 論理削除 |

### インデックス

| 名称 | カラム | 種別 |
|---|---|---|
| uk_tenant_name | tenant_name | UNIQUE |

## 4.2 subscription_plan

| カラム | 型 | NULL | キー | 説明 |
|---|---|---:|---|---|
| plan_id | bigint | NO | PK | プランID |
| plan_code | varchar(30) | NO | UK | FREE / STARTER / BUSINESS / ENTERPRISE |
| plan_name | varchar(100) | NO |  | プラン名 |
| max_data_centers | int | NO |  | DC上限 |
| max_rack_rows | int | NO |  | ラック列上限 |
| max_devices | int | NO |  | 機器上限 |
| max_ip_addresses | int | NO |  | IP上限 |
| max_users | int | NO |  | ユーザー上限 |

### 初期データ

| plan_code | max_data_centers | max_rack_rows | max_devices | max_ip_addresses | max_users |
|---|---:|---:|---:|---:|---:|
| FREE | 1 | 1 | 5 | 5 | 1 |
| STARTER | 2 | 5 | 50 | 50 | 3 |
| BUSINESS | 5 | 50 | 100 | 100 | 10 |
| ENTERPRISE | 10 | 100 | 1000 | 1000 | 30 |

## 4.3 tenant_add_on

| カラム | 型 | NULL | キー | 説明 |
|---|---|---:|---|---|
| tenant_add_on_id | bigint | NO | PK | 追加枠ID |
| tenant_id | bigint | NO | FK | テナントID |
| add_on_type | varchar(30) | NO |  | IP_ADDRESS / DEVICE |
| quantity_unit | int | NO |  | 追加単位数 |
| effective_from | date | NO |  | 有効開始日 |
| effective_to | date | YES |  | 有効終了日 |
| created_at | datetime(6) | NO |  | 作成日時 |
| updated_at | datetime(6) | NO |  | 更新日時 |
| deleted | boolean | NO |  | 論理削除 |

## 4.4 data_center

| カラム | 型 | NULL | キー | 説明 |
|---|---|---:|---|---|
| data_center_id | bigint | NO | PK | DC ID |
| tenant_id | bigint | NO | FK | テナントID |
| region_id | bigint | YES | FK | 地域ID |
| formal_name | varchar(150) | NO |  | 正式名称 |
| display_name | varchar(150) | YES |  | 通称・呼称名 |
| address | varchar(255) | YES |  | 所在地 |
| status | varchar(30) | NO |  | ACTIVE / INACTIVE |
| created_at | datetime(6) | NO |  | 作成日時 |
| updated_at | datetime(6) | NO |  | 更新日時 |
| deleted | boolean | NO |  | 論理削除 |

### インデックス

| 名称 | カラム | 種別 |
|---|---|---|
| idx_dc_tenant | tenant_id, deleted | INDEX |
| uk_dc_name | tenant_id, formal_name, deleted | UNIQUE |

## 4.5 rack

| カラム | 型 | NULL | キー | 説明 |
|---|---|---:|---|---|
| rack_id | bigint | NO | PK | ラックID |
| tenant_id | bigint | NO | FK | テナントID |
| rack_row_id | bigint | NO | FK | ラック列ID |
| formal_name | varchar(150) | NO |  | 正式名称 |
| display_name | varchar(150) | YES |  | 通称・呼称名 |
| rack_number | varchar(50) | NO |  | ラック番号 |
| height_unit | int | NO |  | ラック高さ |
| status | varchar(30) | NO |  | ACTIVE / INACTIVE |
| created_at | datetime(6) | NO |  | 作成日時 |
| updated_at | datetime(6) | NO |  | 更新日時 |
| deleted | boolean | NO |  | 論理削除 |

## 4.6 device

| カラム | 型 | NULL | キー | 説明 |
|---|---|---:|---|---|
| device_id | bigint | NO | PK | 機器ID |
| tenant_id | bigint | NO | FK | テナントID |
| rack_id | bigint | YES | FK | ラックID |
| device_type | varchar(30) | NO |  | SERVER / SWITCH 等 |
| formal_name | varchar(150) | NO |  | 正式名称 |
| display_name | varchar(150) | YES |  | 通称・呼称名 |
| hostname | varchar(150) | YES |  | ホスト名 |
| serial_number | varchar(100) | YES |  | シリアル番号 |
| manufacturer | varchar(100) | YES |  | メーカー |
| model_name | varchar(100) | YES |  | 型番 |
| rack_unit_start | int | YES |  | 開始U位置 |
| rack_unit_size | int | YES |  | 使用U数 |
| lifecycle_status | varchar(30) | NO |  | ACTIVE / SPARE / RETIRED 等 |
| created_at | datetime(6) | NO |  | 作成日時 |
| updated_at | datetime(6) | NO |  | 更新日時 |
| deleted | boolean | NO |  | 論理削除 |

### インデックス

| 名称 | カラム | 種別 |
|---|---|---|
| idx_device_tenant | tenant_id, deleted | INDEX |
| idx_device_rack | tenant_id, rack_id, deleted | INDEX |
| idx_device_hostname | tenant_id, hostname, deleted | INDEX |
| uk_device_formal_name | tenant_id, formal_name, deleted | UNIQUE |

## 4.7 ip_address

| カラム | 型 | NULL | キー | 説明 |
|---|---|---:|---|---|
| ip_address_id | bigint | NO | PK | IPアドレスID |
| tenant_id | bigint | NO | FK | テナントID |
| ip_address | varchar(45) | NO |  | IPv4 / IPv6 |
| ip_version | varchar(10) | NO |  | IPV4 / IPV6 |
| device_id | bigint | YES | FK | 割当機器ID |
| usage_status | varchar(30) | NO |  | UNUSED / IN_USE / RESERVED / RETIRED |
| description | varchar(255) | YES |  | 備考 |
| created_at | datetime(6) | NO |  | 作成日時 |
| updated_at | datetime(6) | NO |  | 更新日時 |
| deleted | boolean | NO |  | 論理削除 |

### インデックス

| 名称 | カラム | 種別 |
|---|---|---|
| uk_ip_address | tenant_id, ip_address, deleted | UNIQUE |
| idx_ip_device | tenant_id, device_id, deleted | INDEX |

## 4.8 maintenance_contract

| カラム | 型 | NULL | キー | 説明 |
|---|---|---:|---|---|
| maintenance_contract_id | bigint | NO | PK | 保守契約ID |
| tenant_id | bigint | NO | FK | テナントID |
| contract_name | varchar(150) | NO |  | 契約名 |
| vendor_name | varchar(150) | NO |  | ベンダー名 |
| contract_number | varchar(100) | YES |  | 契約番号 |
| start_date | date | NO |  | 開始日 |
| end_date | date | NO |  | 終了日 |
| notification_enabled | boolean | NO |  | 通知有効 |
| notification_days_before | int | NO |  | 期限前通知日数。標準60 |
| created_at | datetime(6) | NO |  | 作成日時 |
| updated_at | datetime(6) | NO |  | 更新日時 |
| deleted | boolean | NO |  | 論理削除 |

## 4.9 maintenance_contract_device

| カラム | 型 | NULL | キー | 説明 |
|---|---|---:|---|---|
| maintenance_contract_device_id | bigint | NO | PK | 関連ID |
| tenant_id | bigint | NO | FK | テナントID |
| maintenance_contract_id | bigint | NO | FK | 保守契約ID |
| device_id | bigint | NO | FK | 機器ID |
| created_at | datetime(6) | NO |  | 作成日時 |
| updated_at | datetime(6) | NO |  | 更新日時 |
| deleted | boolean | NO |  | 論理削除 |

### インデックス

| 名称 | カラム | 種別 |
|---|---|---|
| uk_contract_device | tenant_id, maintenance_contract_id, device_id, deleted | UNIQUE |
| idx_contract_device_device | tenant_id, device_id, deleted | INDEX |

## 4.10 contact

| カラム | 型 | NULL | キー | 説明 |
|---|---|---:|---|---|
| contact_id | bigint | NO | PK | 連絡先ID |
| tenant_id | bigint | NO | FK | テナントID |
| contact_type | varchar(30) | NO |  | DC / VENDOR / INTERNAL |
| name | varchar(150) | NO |  | 名称 |
| email | varchar(255) | YES |  | メールアドレス |
| phone_number | varchar(50) | YES |  | 電話番号 |
| created_at | datetime(6) | NO |  | 作成日時 |
| updated_at | datetime(6) | NO |  | 更新日時 |
| deleted | boolean | NO |  | 論理削除 |

## 4.11 tag

| カラム | 型 | NULL | キー | 説明 |
|---|---|---:|---|---|
| tag_id | bigint | NO | PK | タグID |
| tenant_id | bigint | NO | FK | テナントID |
| tag_name | varchar(50) | NO |  | タグ名 |
| color_code | varchar(20) | YES |  | 表示色 |
| created_at | datetime(6) | NO |  | 作成日時 |
| updated_at | datetime(6) | NO |  | 更新日時 |
| deleted | boolean | NO |  | 論理削除 |

## 4.12 tagged_resource

| カラム | 型 | NULL | キー | 説明 |
|---|---|---:|---|---|
| tagged_resource_id | bigint | NO | PK | 関連ID |
| tenant_id | bigint | NO | FK | テナントID |
| tag_id | bigint | NO | FK | タグID |
| resource_type | varchar(50) | NO |  | DATA_CENTER / RACK / DEVICE 等 |
| resource_id | bigint | NO |  | 対象ID |
| created_at | datetime(6) | NO |  | 作成日時 |
| updated_at | datetime(6) | NO |  | 更新日時 |
| deleted | boolean | NO |  | 論理削除 |

## 4.13 notification_setting

| カラム | 型 | NULL | キー | 説明 |
|---|---|---:|---|---|
| notification_setting_id | bigint | NO | PK | 通知設定ID |
| tenant_id | bigint | NO | FK | テナントID |
| notification_type | varchar(50) | NO |  | MAINTENANCE_EXPIRY 等 |
| enabled | boolean | NO |  | 有効フラグ |
| email_enabled | boolean | NO |  | メール通知有効 |
| days_before | int | YES |  | 期限前日数 |
| created_at | datetime(6) | NO |  | 作成日時 |
| updated_at | datetime(6) | NO |  | 更新日時 |
| deleted | boolean | NO |  | 論理削除 |

## 4.14 notification_log

| カラム | 型 | NULL | キー | 説明 |
|---|---|---:|---|---|
| notification_log_id | bigint | NO | PK | 通知ログID |
| tenant_id | bigint | NO | FK | テナントID |
| notification_type | varchar(50) | NO |  | 通知種別 |
| target_type | varchar(50) | NO |  | 対象種別 |
| target_id | bigint | NO |  | 対象ID |
| recipient | varchar(255) | NO |  | 送信先 |
| subject | varchar(255) | NO |  | 件名 |
| status | varchar(30) | NO |  | PENDING / SENT / FAILED / SKIPPED |
| sent_at | datetime(6) | YES |  | 送信日時 |
| error_message | text | YES |  | エラー内容 |
| created_at | datetime(6) | NO |  | 作成日時 |
| updated_at | datetime(6) | NO |  | 更新日時 |
| deleted | boolean | NO |  | 論理削除 |

## 4.15 cloud_account

| カラム | 型 | NULL | キー | 説明 |
|---|---|---:|---|---|
| cloud_account_id | bigint | NO | PK | クラウドアカウントID |
| tenant_id | bigint | NO | FK | テナントID |
| provider | varchar(30) | NO |  | AWS 等 |
| account_name | varchar(150) | NO |  | アカウント名 |
| account_identifier | varchar(150) | YES |  | AWSアカウントID等 |
| created_at | datetime(6) | NO |  | 作成日時 |
| updated_at | datetime(6) | NO |  | 更新日時 |
| deleted | boolean | NO |  | 論理削除 |

## 4.16 cloud_resource

| カラム | 型 | NULL | キー | 説明 |
|---|---|---:|---|---|
| cloud_resource_id | bigint | NO | PK | クラウドリソースID |
| tenant_id | bigint | NO | FK | テナントID |
| cloud_account_id | bigint | NO | FK | クラウドアカウントID |
| region_name | varchar(100) | YES |  | リージョン |
| resource_type | varchar(50) | NO |  | EC2 / CONTAINER / EKS_POD 等 |
| resource_name | varchar(150) | NO |  | リソース名 |
| resource_identifier | varchar(255) | YES |  | インスタンスID等 |
| status | varchar(50) | YES |  | 状態 |
| created_at | datetime(6) | NO |  | 作成日時 |
| updated_at | datetime(6) | NO |  | 更新日時 |
| deleted | boolean | NO |  | 論理削除 |

## 5. 外部キー方針

- 参照整合性はDB外部キーとアプリケーションチェックの併用とする。
- 論理削除を採用するため、親データ削除時はService層で子データの存在を確認する。
- 履歴保全が必要な保守契約・通知ログは物理削除しない。
