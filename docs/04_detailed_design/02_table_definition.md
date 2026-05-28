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
| created_by | bigint | NO | 作成者ユーザーID |
| created_at | datetime(6) | NO | 作成日時 |
| updated_by | bigint | NO | 更新者ユーザーID |
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
| 5 | password_reset_token | パスワード再設定トークン |
| 6 | user_invitation_token | ユーザー招待トークン |
| 7 | role | ロール |
| 8 | permission | 権限マスタ |
| 9 | role_permission | ロール・権限関連 |
| 10 | user_role | ユーザー・ロール関連 |
| 11 | audit_log | 操作履歴 |
| 12 | region | 地域 |
| 8 | data_center | データセンター |
| 9 | building | 棟 |
| 10 | floor | フロア |
| 11 | area | 区画 |
| 12 | rack_row | ラック列 |
| 13 | rack | ラック |
| 13A | rack_template | ラックテンプレート |
| 13B | rack_template_item | ラックテンプレート明細 |
| 14 | device | 機器 |
| 15 | ip_subnet | IPサブネット |
| 16 | ip_address | IPアドレス利用状況 |
| 17 | maintenance_contract | 保守契約 |
| 18 | maintenance_contract_device | 保守契約・機器関連 |
| 19 | maintenance_contract_contact | 保守契約・連絡先関連 |
| 20 | contact | 連絡先 |
| 21 | data_center_contact | DC・連絡先関連 |
| 22 | tag | タグ |
| 23 | tagged_resource | タグ関連 |
| 24 | notification_setting | 通知設定 |
| 25 | notification_log | 通知ログ |
| 25A | resource_alias | 呼称名・別名 |
| 26 | csv_export_history | CSVエクスポート履歴 |
| 27 | csv_import_history | CSVインポート履歴 |
| 28 | csv_import_error | CSVインポートエラー |
| 29 | cloud_account | 将来拡張：クラウドアカウント |
| 30 | cloud_resource | 将来拡張：クラウドリソース |

> Noは分類用の整理番号であり、追加テーブルにより一部枝番・既存番号が残る場合がある。実装時はテーブル名を正とする。

## 4. 主要テーブル定義

## 4.1 tenant

| カラム | 型 | NULL | キー | 説明 |
|---|---|---:|---|---|
| tenant_id | bigint | NO | PK | テナントID |
| tenant_name | varchar(100) | NO |  | テナント名 |
| plan_code | varchar(30) | NO | FK | 契約プランコード |
| trial_start_date | date | YES |  | Freeトライアル開始日 |
| trial_end_date | date | YES |  | Freeトライアル終了日 |
| status | varchar(30) | NO |  | ACTIVE / SUSPENDED / TRIAL_EXPIRED / CANCELLED |
| created_by | bigint | NO | FK | 作成者 |
| created_at | datetime(6) | NO |  | 作成日時 |
| updated_by | bigint | NO | FK | 更新者 |
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
| max_racks | int | NO |  | ラック上限 |
| max_devices | int | NO |  | 機器上限 |
| max_ip_addresses | int | NO |  | 管理対象IP数上限 |
| max_tags | int | NO |  | タグマスタ件数上限 |
| trial_days | int | YES |  | Freeトライアル日数。標準14 |
| max_users | int | NO |  | ユーザー上限 |

### 初期データ

| plan_code | trial_days | max_data_centers | max_racks | max_devices | max_ip_addresses | max_tags | max_users |
|---|---:|---:|---:|---:|---:|---:|---:|
| FREE | 14 | 1 | 3 | 40 | 256 | 10 | 3 |
| STARTER | null | 2 | 5 | 50 | 512 | 50 | 3 |
| BUSINESS | null | 5 | 50 | 100 | 1024 | 200 | 10 |
| ENTERPRISE | null | 10 | 100 | 1000 | 2048 | 1000 | 30 |

## 4.3 tenant_add_on

| カラム | 型 | NULL | キー | 説明 |
|---|---|---:|---|---|
| tenant_add_on_id | bigint | NO | PK | 追加枠ID |
| tenant_id | bigint | NO | FK | テナントID |
| add_on_type | varchar(30) | NO |  | IP_ADDRESS / DEVICE |
| quantity_unit | int | NO |  | 追加単位数 |
| effective_from | date | NO |  | 有効開始日 |
| effective_to | date | YES |  | 有効終了日 |
| created_by | bigint | NO | FK | 作成者 |
| created_at | datetime(6) | NO |  | 作成日時 |
| updated_by | bigint | NO | FK | 更新者 |
| updated_at | datetime(6) | NO |  | 更新日時 |
| deleted | boolean | NO |  | 論理削除 |

## 4.3A region

データセンター所在地を分類する地域マスタ。初期リリースではテナントごとの任意マスタとして管理し、都道府県・地域コード・表示順を保持する。

| カラム | 型 | NULL | キー | 説明 |
|---|---|---:|---|---|
| region_id | bigint | NO | PK | 地域ID |
| tenant_id | bigint | NO | FK | テナントID |
| region_code | varchar(50) | YES |  | 地域コード |
| region_name | varchar(100) | NO |  | 地域名 |
| prefecture | varchar(100) | YES |  | 都道府県 |
| display_order | int | YES |  | 表示順 |
| status | varchar(30) | NO |  | ACTIVE / INACTIVE |
| created_by | bigint | NO | FK | 作成者 |
| created_at | datetime(6) | NO |  | 作成日時 |
| updated_by | bigint | NO | FK | 更新者 |
| updated_at | datetime(6) | NO |  | 更新日時 |
| deleted | boolean | NO |  | 論理削除 |

### インデックス

| 名称 | カラム | 種別 |
|---|---|---|
| idx_region_tenant | tenant_id, deleted | INDEX |
| uk_region_name | tenant_id, region_name, deleted | UNIQUE |

## 4.4 data_center

| カラム | 型 | NULL | キー | 説明 |
|---|---|---:|---|---|
| data_center_id | bigint | NO | PK | DC ID |
| tenant_id | bigint | NO | FK | テナントID |
| region_id | bigint | YES | FK | 地域ID |
| formal_name | varchar(150) | NO |  | 正式名称 |
| display_name | varchar(150) | YES |  | 代表表示名。複数の呼称名・別名は `resource_alias` で保持する |
| address | varchar(255) | YES |  | 所在地 |
| status | varchar(30) | NO |  | ACTIVE / INACTIVE |
| created_by | bigint | NO | FK | 作成者 |
| created_at | datetime(6) | NO |  | 作成日時 |
| updated_by | bigint | NO | FK | 更新者 |
| updated_at | datetime(6) | NO |  | 更新日時 |
| deleted | boolean | NO |  | 論理削除 |

### インデックス

| 名称 | カラム | 種別 |
|---|---|---|
| idx_dc_tenant | tenant_id, deleted | INDEX |
| uk_dc_name | tenant_id, formal_name, deleted | UNIQUE |

## 4.5 building / floor / area / rack_row

初期リリースから物理階層を必須で扱うため、以下の階層テーブルを実装対象とする。

### building

| カラム | 型 | NULL | キー | 説明 |
|---|---|---:|---|---|
| building_id | bigint | NO | PK | 棟ID |
| tenant_id | bigint | NO | FK | テナントID |
| data_center_id | bigint | NO | FK | DC ID |
| formal_name | varchar(150) | NO |  | 正式名称 |
| display_name | varchar(150) | YES |  | 代表表示名。複数の呼称名・別名は `resource_alias` で保持する |
| created_by | bigint | NO | FK | 作成者 |
| created_at | datetime(6) | NO |  | 作成日時 |
| updated_by | bigint | NO | FK | 更新者 |
| updated_at | datetime(6) | NO |  | 更新日時 |
| deleted | boolean | NO |  | 論理削除 |

### floor

| カラム | 型 | NULL | キー | 説明 |
|---|---|---:|---|---|
| floor_id | bigint | NO | PK | フロアID |
| tenant_id | bigint | NO | FK | テナントID |
| building_id | bigint | NO | FK | 棟ID |
| floor_name | varchar(100) | NO |  | フロア名 |
| floor_number | varchar(30) | YES |  | 階数・表記 |
| created_by | bigint | NO | FK | 作成者 |
| created_at | datetime(6) | NO |  | 作成日時 |
| updated_by | bigint | NO | FK | 更新者 |
| updated_at | datetime(6) | NO |  | 更新日時 |
| deleted | boolean | NO |  | 論理削除 |

### area

| カラム | 型 | NULL | キー | 説明 |
|---|---|---:|---|---|
| area_id | bigint | NO | PK | 区画ID |
| tenant_id | bigint | NO | FK | テナントID |
| floor_id | bigint | NO | FK | フロアID |
| area_name | varchar(100) | NO |  | 区画名 |
| direction | varchar(30) | YES |  | EAST / WEST / SOUTH / NORTH / OTHER |
| created_by | bigint | NO | FK | 作成者 |
| created_at | datetime(6) | NO |  | 作成日時 |
| updated_by | bigint | NO | FK | 更新者 |
| updated_at | datetime(6) | NO |  | 更新日時 |
| deleted | boolean | NO |  | 論理削除 |

### rack_row

| カラム | 型 | NULL | キー | 説明 |
|---|---|---:|---|---|
| rack_row_id | bigint | NO | PK | ラック列ID |
| tenant_id | bigint | NO | FK | テナントID |
| area_id | bigint | NO | FK | 区画ID |
| row_name | varchar(100) | NO |  | ラック列名 |
| created_by | bigint | NO | FK | 作成者 |
| created_at | datetime(6) | NO |  | 作成日時 |
| updated_by | bigint | NO | FK | 更新者 |
| updated_at | datetime(6) | NO |  | 更新日時 |
| deleted | boolean | NO |  | 論理削除 |

## 4.6 rack

| カラム | 型 | NULL | キー | 説明 |
|---|---|---:|---|---|
| rack_id | bigint | NO | PK | ラックID |
| tenant_id | bigint | NO | FK | テナントID |
| rack_row_id | bigint | NO | FK | ラック列ID |
| formal_name | varchar(150) | NO |  | 正式名称 |
| display_name | varchar(150) | YES |  | 代表表示名。複数の呼称名・別名は `resource_alias` で保持する |
| rack_number | varchar(50) | NO |  | ラック番号 |
| height_unit | int | NO |  | ラック高さ |
| status | varchar(30) | NO |  | ACTIVE / INACTIVE |
| created_by | bigint | NO | FK | 作成者 |
| created_at | datetime(6) | NO |  | 作成日時 |
| updated_by | bigint | NO | FK | 更新者 |
| updated_at | datetime(6) | NO |  | 更新日時 |
| deleted | boolean | NO |  | 論理削除 |

## 4.6A rack_template / rack_template_item

### rack_template

| カラム | 型 | NULL | キー | 説明 |
|---|---|---:|---|---|
| rack_template_id | bigint | NO | PK | ラックテンプレートID |
| tenant_id | bigint | NO | FK | テナントID |
| template_name | varchar(150) | NO |  | テンプレート名 |
| height_unit | int | NO |  | ラック高さU数 |
| category | varchar(100) | YES |  | 用途・カテゴリ |
| active | boolean | NO |  | 有効フラグ |
| created_by | bigint | NO | FK | 作成者 |
| created_at | datetime(6) | NO |  | 作成日時 |
| updated_by | bigint | NO | FK | 更新者 |
| updated_at | datetime(6) | NO |  | 更新日時 |
| deleted | boolean | NO |  | 論理削除 |

### rack_template_item

| カラム | 型 | NULL | キー | 説明 |
|---|---|---:|---|---|
| rack_template_item_id | bigint | NO | PK | 明細ID |
| tenant_id | bigint | NO | FK | テナントID |
| rack_template_id | bigint | NO | FK | ラックテンプレートID |
| item_type | varchar(30) | NO |  | DEVICE_PLACEHOLDER / RESERVED_U / BLANK_PANEL 等 |
| start_unit | int | NO |  | 開始U |
| unit_size | int | NO |  | 使用U数 |
| label | varchar(150) | YES |  | 表示ラベル |
| created_by | bigint | NO | FK | 作成者 |
| created_at | datetime(6) | NO |  | 作成日時 |
| updated_by | bigint | NO | FK | 更新者 |
| updated_at | datetime(6) | NO |  | 更新日時 |
| deleted | boolean | NO |  | 論理削除 |

## 4.7 device

| カラム | 型 | NULL | キー | 説明 |
|---|---|---:|---|---|
| device_id | bigint | NO | PK | 機器ID |
| tenant_id | bigint | NO | FK | テナントID |
| rack_id | bigint | YES | FK | ラックID |
| device_type | varchar(30) | NO |  | SERVER / SWITCH 等 |
| formal_name | varchar(150) | NO |  | 正式名称 |
| display_name | varchar(150) | YES |  | 代表表示名。複数の呼称名・別名は `resource_alias` で保持する |
| hostname | varchar(150) | YES |  | ホスト名 |
| serial_number | varchar(100) | YES |  | シリアル番号 |
| manufacturer | varchar(100) | YES |  | メーカー |
| model_name | varchar(100) | YES |  | 型番 |
| rack_unit_start | int | YES |  | 開始U位置 |
| rack_unit_size | int | YES |  | 使用U数 |
| lifecycle_status | varchar(30) | NO |  | ACTIVE / SPARE / PLANNED_RETIREMENT / RETIRED |
| created_by | bigint | NO | FK | 作成者 |
| created_at | datetime(6) | NO |  | 作成日時 |
| updated_by | bigint | NO | FK | 更新者 |
| updated_at | datetime(6) | NO |  | 更新日時 |
| deleted | boolean | NO |  | 論理削除 |

### インデックス

| 名称 | カラム | 種別 |
|---|---|---|
| idx_device_tenant | tenant_id, deleted | INDEX |
| idx_device_rack | tenant_id, rack_id, deleted | INDEX |
| idx_device_hostname | tenant_id, hostname, deleted | INDEX |
| uk_device_formal_name | tenant_id, formal_name, deleted | UNIQUE |

## 4.8 ip_subnet

| カラム | 型 | NULL | キー | 説明 |
|---|---|---:|---|---|
| ip_subnet_id | bigint | NO | PK | IPサブネットID |
| tenant_id | bigint | NO | FK | テナントID |
| subnet_name | varchar(150) | NO |  | サブネット名 |
| cidr | varchar(64) | NO |  | CIDR表記 |
| ip_version | varchar(10) | NO |  | IPV4 / IPV6 |
| status | varchar(30) | NO |  | ACTIVE / RESERVED / RETIRED |
| description | varchar(255) | YES |  | 備考 |
| created_by | bigint | NO | FK | 作成者 |
| created_at | datetime(6) | NO |  | 作成日時 |
| updated_by | bigint | NO | FK | 更新者 |
| updated_at | datetime(6) | NO |  | 更新日時 |
| deleted | boolean | NO |  | 論理削除 |

### インデックス

| 名称 | カラム | 種別 |
|---|---|---|
| uk_ip_subnet_cidr | tenant_id, cidr | UNIQUE |
| idx_ip_subnet_tenant | tenant_id, deleted | INDEX |

## 4.9 ip_address

| カラム | 型 | NULL | キー | 説明 |
|---|---|---:|---|---|
| ip_address_id | bigint | NO | PK | IPアドレスID |
| tenant_id | bigint | NO | FK | テナントID |
| ip_subnet_id | bigint | NO | FK | IPサブネットID |
| ip_address | varchar(45) | NO |  | IPv4 / IPv6 |
| ip_version | varchar(10) | NO |  | IPV4 / IPV6 |
| device_id | bigint | YES | FK | 割当機器ID |
| usage_status | varchar(30) | NO |  | UNUSED / IN_USE / RESERVED / RETIRED |
| description | varchar(255) | YES |  | 備考 |
| created_by | bigint | NO | FK | 作成者 |
| created_at | datetime(6) | NO |  | 作成日時 |
| updated_by | bigint | NO | FK | 更新者 |
| updated_at | datetime(6) | NO |  | 更新日時 |
| deleted | boolean | NO |  | 論理削除 |

### インデックス

| 名称 | カラム | 種別 |
|---|---|---|
| uk_ip_address | tenant_id, ip_address, deleted | UNIQUE |
| idx_ip_device | tenant_id, device_id, deleted | INDEX |

## 4.10 maintenance_contract

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
| created_by | bigint | NO | FK | 作成者 |
| created_at | datetime(6) | NO |  | 作成日時 |
| updated_by | bigint | NO | FK | 更新者 |
| updated_at | datetime(6) | NO |  | 更新日時 |
| deleted | boolean | NO |  | 論理削除 |

## 4.11 maintenance_contract_device

| カラム | 型 | NULL | キー | 説明 |
|---|---|---:|---|---|
| maintenance_contract_device_id | bigint | NO | PK | 関連ID |
| tenant_id | bigint | NO | FK | テナントID |
| maintenance_contract_id | bigint | NO | FK | 保守契約ID |
| device_id | bigint | NO | FK | 機器ID |
| created_by | bigint | NO | FK | 作成者 |
| created_at | datetime(6) | NO |  | 作成日時 |
| updated_by | bigint | NO | FK | 更新者 |
| updated_at | datetime(6) | NO |  | 更新日時 |
| deleted | boolean | NO |  | 論理削除 |

### インデックス

| 名称 | カラム | 種別 |
|---|---|---|
| uk_contract_device | tenant_id, maintenance_contract_id, device_id, deleted | UNIQUE |
| idx_contract_device_device | tenant_id, device_id, deleted | INDEX |

## 4.12 maintenance_contract_contact

| カラム | 型 | NULL | キー | 説明 |
|---|---|---:|---|---|
| maintenance_contract_contact_id | bigint | NO | PK | 保守契約・連絡先関連ID |
| tenant_id | bigint | NO | FK | テナントID |
| maintenance_contract_id | bigint | NO | FK | 保守契約ID |
| contact_id | bigint | NO | FK | 連絡先ID |
| contact_role | varchar(30) | YES |  | VENDOR / EMERGENCY / BILLING / OTHER 等の用途 |
| created_by | bigint | NO | FK | 作成者 |
| created_at | datetime(6) | NO |  | 作成日時 |
| updated_by | bigint | NO | FK | 更新者 |
| updated_at | datetime(6) | NO |  | 更新日時 |
| deleted | boolean | NO |  | 論理削除 |

### インデックス

| 名称 | カラム | 種別 |
|---|---|---|
| uk_contract_contact | tenant_id, maintenance_contract_id, contact_id, deleted | UNIQUE |
| idx_contract_contact_contact | tenant_id, contact_id, deleted | INDEX |

`MaintenanceContractService.assignContact(contractId, contactId)` は、同一テナント内の保守契約と連絡先のみ関連付ける。既に有効な関連が存在する場合は重複登録せず、業務例外として扱う。

## 4.13 contact

| カラム | 型 | NULL | キー | 説明 |
|---|---|---:|---|---|
| contact_id | bigint | NO | PK | 連絡先ID |
| tenant_id | bigint | NO | FK | テナントID |
| contact_type | varchar(30) | NO |  | DC / VENDOR / INTERNAL |
| organization_name | varchar(150) | YES |  | 会社・組織名 |
| department_name | varchar(150) | YES |  | 部署名 |
| position_name | varchar(100) | YES |  | 役職・肩書 |
| person_name | varchar(150) | NO |  | 担当者名または窓口名 |
| email | varchar(255) | YES |  | メールアドレス |
| phone_number | varchar(50) | YES |  | 電話番号 |
| address | varchar(255) | YES |  | 住所・所在地 |
| preferred_contact_method | varchar(30) | YES |  | EMAIL / PHONE / OTHER |
| note | text | YES |  | 備考 |
| active | boolean | NO |  | 有効フラグ |
| created_by | bigint | NO | FK | 作成者 |
| created_at | datetime(6) | NO |  | 作成日時 |
| updated_by | bigint | NO | FK | 更新者 |
| updated_at | datetime(6) | NO |  | 更新日時 |
| deleted | boolean | NO |  | 論理削除 |

## 4.14 data_center_contact

| カラム | 型 | NULL | キー | 説明 |
|---|---|---:|---|---|
| data_center_contact_id | bigint | NO | PK | DC・連絡先関連ID |
| tenant_id | bigint | NO | FK | テナントID |
| data_center_id | bigint | NO | FK | DC ID |
| contact_id | bigint | NO | FK | 連絡先ID |
| contact_role | varchar(30) | YES |  | PRIMARY / EMERGENCY / BILLING / OTHER 等の用途 |
| created_by | bigint | NO | FK | 作成者 |
| created_at | datetime(6) | NO |  | 作成日時 |
| updated_by | bigint | NO | FK | 更新者 |
| updated_at | datetime(6) | NO |  | 更新日時 |
| deleted | boolean | NO |  | 論理削除 |

### インデックス

| 名称 | カラム | 種別 |
|---|---|---|
| uk_dc_contact | tenant_id, data_center_id, contact_id, deleted | UNIQUE |
| idx_dc_contact_contact | tenant_id, contact_id, deleted | INDEX |

`DataCenterService.assignContact(dataCenterId, contactId)` は、同一テナント内のDCと連絡先のみ関連付ける。既に有効な関連が存在する場合は重複登録せず、業務例外として扱う。

## 4.15 tag

| カラム | 型 | NULL | キー | 説明 |
|---|---|---:|---|---|
| tag_id | bigint | NO | PK | タグID |
| tenant_id | bigint | NO | FK | テナントID |
| tag_name | varchar(50) | NO |  | タグ名 |
| color_code | varchar(20) | YES |  | 表示色 |
| created_by | bigint | NO | FK | 作成者 |
| created_at | datetime(6) | NO |  | 作成日時 |
| updated_by | bigint | NO | FK | 更新者 |
| updated_at | datetime(6) | NO |  | 更新日時 |
| deleted | boolean | NO |  | 論理削除 |

## 4.16 tagged_resource

| カラム | 型 | NULL | キー | 説明 |
|---|---|---:|---|---|
| tagged_resource_id | bigint | NO | PK | 関連ID |
| tenant_id | bigint | NO | FK | テナントID |
| tag_id | bigint | NO | FK | タグID |
| resource_type | varchar(50) | NO |  | DATA_CENTER / RACK / DEVICE / IP_SUBNET / IP_ADDRESS / MAINTENANCE_CONTRACT |
| resource_id | bigint | NO |  | 対象ID |
| created_by | bigint | NO | FK | 作成者 |
| created_at | datetime(6) | NO |  | 作成日時 |
| updated_by | bigint | NO | FK | 更新者 |
| updated_at | datetime(6) | NO |  | 更新日時 |
| deleted | boolean | NO |  | 論理削除 |

## 4.17 notification_setting

| カラム | 型 | NULL | キー | 説明 |
|---|---|---:|---|---|
| notification_setting_id | bigint | NO | PK | 通知設定ID |
| tenant_id | bigint | NO | FK | テナントID |
| notification_type | varchar(50) | NO |  | MAINTENANCE_EXPIRY 等 |
| enabled | boolean | NO |  | 有効フラグ |
| email_enabled | boolean | NO |  | メール通知有効 |
| in_app_enabled | boolean | NO |  | 画面内通知有効 |
| days_before | int | YES |  | 期限前日数 |
| created_by | bigint | NO | FK | 作成者 |
| created_at | datetime(6) | NO |  | 作成日時 |
| updated_by | bigint | NO | FK | 更新者 |
| updated_at | datetime(6) | NO |  | 更新日時 |
| deleted | boolean | NO |  | 論理削除 |

## 4.18 notification_log

| カラム | 型 | NULL | キー | 説明 |
|---|---|---:|---|---|
| notification_log_id | bigint | NO | PK | 通知ログID |
| tenant_id | bigint | NO | FK | テナントID |
| notification_type | varchar(50) | NO |  | 通知種別 |
| channel | varchar(30) | NO |  | EMAIL / IN_APP |
| target_type | varchar(50) | NO |  | 対象種別 |
| target_id | bigint | NO |  | 対象ID |
| recipient | varchar(255) | YES |  | 送信先メールアドレス。画面内通知ではNULL可 |
| recipient_user_id | bigint | YES | FK | 画面内通知の受信ユーザーID |
| subject | varchar(255) | NO |  | 件名 |
| status | varchar(30) | NO |  | PENDING / SENT / FAILED / SKIPPED |
| sent_at | datetime(6) | YES |  | 送信日時 |
| read_at | datetime(6) | YES |  | 画面内通知の既読日時 |
| error_message | text | YES |  | エラー内容 |
| created_by | bigint | NO | FK | 作成者 |
| created_at | datetime(6) | NO |  | 作成日時 |
| updated_by | bigint | NO | FK | 更新者 |
| updated_at | datetime(6) | NO |  | 更新日時 |
| deleted | boolean | NO |  | 論理削除 |

## 4.19 csv_export_history

| カラム | 型 | NULL | キー | 説明 |
|---|---|---:|---|---|
| csv_export_history_id | bigint | NO | PK | CSV出力履歴ID |
| tenant_id | bigint | NO | FK | テナントID |
| target_type | varchar(50) | NO |  | DEVICE / RACK / IP_SUBNET 等 |
| condition_summary | text | YES |  | 出力条件概要 |
| file_name | varchar(255) | NO |  | 出力ファイル名 |
| record_count | int | NO |  | 出力件数 |
| requested_by | bigint | NO | FK | 実行者 |
| created_by | bigint | NO | FK | 作成者 |
| created_at | datetime(6) | NO |  | 作成日時 |

## 4.20 csv_import_history

| カラム | 型 | NULL | キー | 説明 |
|---|---|---:|---|---|
| csv_import_history_id | bigint | NO | PK | CSV取込履歴ID |
| tenant_id | bigint | NO | FK | テナントID |
| target_type | varchar(50) | NO |  | DEVICE / RACK / IP_SUBNET 等 |
| file_name | varchar(255) | NO |  | 取込ファイル名 |
| status | varchar(30) | NO |  | PENDING / SUCCEEDED / FAILED / PARTIAL_FAILED |
| total_count | int | NO |  | 総行数 |
| success_count | int | NO |  | 成功行数 |
| failure_count | int | NO |  | 失敗行数 |
| requested_by | bigint | NO | FK | 実行者 |
| created_by | bigint | NO | FK | 作成者 |
| created_at | datetime(6) | NO |  | 作成日時 |

## 4.21 csv_import_error

| カラム | 型 | NULL | キー | 説明 |
|---|---|---:|---|---|
| csv_import_error_id | bigint | NO | PK | CSV取込エラーID |
| csv_import_history_id | bigint | NO | FK | 取込履歴ID |
| row_number | int | NO |  | 行番号 |
| column_name | varchar(100) | YES |  | カラム名 |
| error_message | varchar(500) | NO |  | エラー内容 |

## 4.22 app_user / password_reset_token / user_invitation_token / role / permission / role_permission / user_role / audit_log

### app_user

| カラム | 型 | NULL | キー | 説明 |
|---|---|---:|---|---|
| user_id | bigint | NO | PK | ユーザーID |
| tenant_id | bigint | NO | FK | テナントID |
| email | varchar(255) | NO |  | メールアドレス |
| display_name | varchar(150) | NO |  | 表示名 |
| password_hash | varchar(255) | NO |  | ハッシュ化済みパスワード。平文は保存しない |
| password_updated_at | datetime(6) | YES |  | パスワード最終更新日時 |
| status | varchar(30) | NO |  | ACTIVE / INVITED / SUSPENDED |
| created_by | bigint | NO | FK | 作成者 |
| created_at | datetime(6) | NO |  | 作成日時 |
| updated_by | bigint | NO | FK | 更新者 |
| updated_at | datetime(6) | NO |  | 更新日時 |
| deleted | boolean | NO |  | 論理削除 |

### password_reset_token

| カラム | 型 | NULL | キー | 説明 |
|---|---|---:|---|---|
| password_reset_token_id | bigint | NO | PK | トークンID |
| tenant_id | bigint | NO | FK | テナントID |
| user_id | bigint | NO | FK | 対象ユーザーID |
| token_hash | varchar(255) | NO |  | ハッシュ化済み再設定トークン |
| expires_at | datetime(6) | NO |  | 有効期限 |
| used_at | datetime(6) | YES |  | 使用日時 |
| created_at | datetime(6) | NO |  | 作成日時 |
| deleted | boolean | NO |  | 論理削除 |

### user_invitation_token

| カラム | 型 | NULL | キー | 説明 |
|---|---|---:|---|---|
| user_invitation_token_id | bigint | NO | PK | 招待トークンID |
| tenant_id | bigint | NO | FK | テナントID |
| user_id | bigint | NO | FK | 招待対象ユーザーID |
| token_hash | varchar(255) | NO |  | ハッシュ化済み招待トークン |
| invited_email | varchar(255) | NO |  | 招待先メールアドレス |
| status | varchar(30) | NO |  | ACTIVE / ACCEPTED / CANCELLED / EXPIRED |
| expires_at | datetime(6) | NO |  | 有効期限 |
| accepted_at | datetime(6) | YES |  | 承諾日時 |
| cancelled_at | datetime(6) | YES |  | 取消日時 |
| created_by | bigint | NO | FK | 招待実行者 |
| created_at | datetime(6) | NO |  | 作成日時 |
| updated_at | datetime(6) | NO |  | 更新日時 |
| deleted | boolean | NO |  | 論理削除 |

### role / permission / role_permission / user_role

| テーブル | 主なカラム | 説明 |
|---|---|---|
| role | role_id, role_code, role_name | `ROLE_SYSTEM_ADMIN` / `ROLE_TENANT_ADMIN` / `ROLE_BILLING_ADMIN` / `ROLE_OPERATION_ADMIN` / `ROLE_EDITOR` / `ROLE_VIEWER` / `ROLE_AUDITOR` のロール定義 |
| permission | permission_id, permission_code, permission_name | 画面・Service操作単位の権限マスタ |
| role_permission | role_permission_id, role_id, permission_id | 標準ロールと権限の関連。初期ロール権限の正本 |
| user_role | user_role_id, tenant_id, user_id, role_id | ユーザー・ロール関連。システム管理者はtenant_idをNULL可とする |

### audit_log

| カラム | 型 | NULL | キー | 説明 |
|---|---|---:|---|---|
| audit_log_id | bigint | NO | PK | 操作履歴ID |
| tenant_id | bigint | YES | FK | 対象テナントID。システム管理操作ではNULL可 |
| user_id | bigint | YES | FK | 操作ユーザーID。未認証ログイン失敗ではNULL可 |
| action_type | varchar(50) | NO |  | LOGIN_SUCCESS / LOGIN_FAILURE / CREATE / UPDATE / DELETE / INVITE / ROLE_CHANGE / CSV_EXPORT / PLAN_CHANGE_REQUEST 等 |
| resource_type | varchar(50) | YES |  | 操作対象種別 |
| resource_id | bigint | YES |  | 操作対象ID |
| result | varchar(20) | NO |  | SUCCESS / FAILURE |
| reason | varchar(500) | YES |  | 失敗理由・操作理由。機密情報は含めない |
| ip_address | varchar(45) | YES |  | 操作元IP |
| user_agent | varchar(255) | YES |  | User-Agent。長文は切り詰める |
| created_at | datetime(6) | NO |  | 操作日時 |

## 4.22A resource_alias

正式名称とは別に、検索対象となる複数の呼称名・別名・略称・運用名を保持する。初期対象は `DATA_CENTER`、`RACK`、`DEVICE` とし、将来的に対象を拡張できる設計とする。

| カラム | 型 | NULL | キー | 説明 |
|---|---|---:|---|---|
| resource_alias_id | bigint | NO | PK | 別名ID |
| tenant_id | bigint | NO | FK | テナントID |
| resource_type | varchar(50) | NO |  | DATA_CENTER / RACK / DEVICE |
| resource_id | bigint | NO |  | 対象ID |
| alias_name | varchar(150) | NO |  | 呼称名・別名・略称・運用名 |
| alias_type | varchar(30) | YES |  | DISPLAY / ABBREVIATION / OPERATION / OTHER |
| created_by | bigint | NO | FK | 作成者 |
| created_at | datetime(6) | NO |  | 作成日時 |
| updated_by | bigint | NO | FK | 更新者 |
| updated_at | datetime(6) | NO |  | 更新日時 |
| deleted | boolean | NO |  | 論理削除 |

### インデックス

| 名称 | カラム | 種別 |
|---|---|---|
| idx_resource_alias_resource | tenant_id, resource_type, resource_id, deleted | INDEX |
| idx_resource_alias_name | tenant_id, alias_name, deleted | INDEX |

## 4.23 cloud_account（将来拡張）

| カラム | 型 | NULL | キー | 説明 |
|---|---|---:|---|---|
| cloud_account_id | bigint | NO | PK | クラウドアカウントID |
| tenant_id | bigint | NO | FK | テナントID |
| provider | varchar(30) | NO |  | AWS 等 |
| account_name | varchar(150) | NO |  | アカウント名 |
| account_identifier | varchar(150) | YES |  | AWSアカウントID等 |
| created_by | bigint | NO | FK | 作成者 |
| created_at | datetime(6) | NO |  | 作成日時 |
| updated_by | bigint | NO | FK | 更新者 |
| updated_at | datetime(6) | NO |  | 更新日時 |
| deleted | boolean | NO |  | 論理削除 |

## 4.24 cloud_resource（将来拡張）

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
| created_by | bigint | NO | FK | 作成者 |
| created_at | datetime(6) | NO |  | 作成日時 |
| updated_by | bigint | NO | FK | 更新者 |
| updated_at | datetime(6) | NO |  | 更新日時 |
| deleted | boolean | NO |  | 論理削除 |

## 5. 外部キー方針

- 参照整合性はDB外部キーとアプリケーションチェックの併用とする。
- 論理削除を採用するため、親データ削除時はService層で子データの存在を確認する。
- 履歴保全が必要な保守契約・通知ログは物理削除しない。


## 6. 初期リリース対象外テーブル

`cloud_account`、`cloud_resource` は将来拡張の設計候補として定義するが、初期リリースの実装・マイグレーション対象からは除外する。
