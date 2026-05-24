# 4. 詳細設計 - エンティティ定義

## 1. 目的

本書は、Data Center Asset & Infrastructure Manager（以下、本システム）の詳細設計における主要エンティティを定義する。

対象は、データセンター、ラック、機器、IPサブネット・IPアドレス、保守契約、通知、契約プラン、ユーザー・権限、タグ、CSV連携である。クラウド資産管理は将来拡張として扱う。

## 2. 前提

- アーキテクチャはDDDを意識したレイヤード構成とする。
- Java / Spring Boot / Spring Security / Vaadin / MariaDB / Lombok を前提とする。
- マルチテナントSaaSを前提とし、すべての業務データは原則として `tenant_id` を保持する。
- 論理削除を基本とし、主要な業務データには最低限の監査情報として `created_by`、`created_at`、`updated_by`、`updated_at`、`deleted` を持たせる。
- 料金プラン別の上限管理を行う。

## 3. 共通エンティティ

### 3.1 Tenant

| 項目 | 内容 |
|---|---|
| エンティティ名 | Tenant |
| 概要 | 契約組織・利用単位を表す |
| 集約 | Tenant集約 |
| 主キー | tenantId |

#### 主な属性

| 属性 | 型 | 必須 | 説明 |
|---|---:|:---:|---|
| tenantId | Long | ○ | テナントID |
| tenantName | String | ○ | テナント名 |
| planType | PlanType | ○ | 契約プラン |
| status | TenantStatus | ○ | 利用状態。ACTIVE / SUSPENDED / TRIAL_EXPIRED など |
| createdAt | LocalDateTime | ○ | 作成日時 |
| updatedAt | LocalDateTime | ○ | 更新日時 |

#### 主な振る舞い

- プラン変更
- 利用停止
- 利用再開
- プラン上限判定の起点

## 4. 契約・プラン系エンティティ

### 4.1 SubscriptionPlan

| 項目 | 内容 |
|---|---|
| エンティティ名 | SubscriptionPlan |
| 概要 | Free / Starter / Business / Enterprise の基本上限を表す |
| 集約 | Plan集約 |
| 主キー | planId |

#### 主な属性

| 属性 | 型 | 必須 | 説明 |
|---|---:|:---:|---|
| planId | Long | ○ | プランID |
| planCode | String | ○ | FREE / STARTER / BUSINESS / ENTERPRISE |
| planName | String | ○ | 表示名 |
| maxDataCenters | Integer | ○ | データセンター上限 |
| maxRacks | Integer | ○ | ラック上限。ラック列は物理階層として管理し、課金・上限はラック数を基本とする |
| maxDevices | Integer | ○ | 機器上限 |
| maxIpAddresses | Integer | ○ | 管理対象IP数上限 |
| trialDays | Integer | - | Freeプランのトライアル日数。標準14日 |
| maxUsers | Integer | ○ | ユーザー上限 |

### 4.2 TenantAddOn

| 項目 | 内容 |
|---|---|
| エンティティ名 | TenantAddOn |
| 概要 | テナントごとの追加枠を表す |
| 集約 | Tenant集約 |
| 主キー | addOnId |

#### 追加単位

| 種別 | 追加単位 |
|---|---:|
| 管理対象IP | 256IP単位 |
| 機器 | 100台単位 |

## 5. ロケーション系エンティティ

### 5.1 Region

| 項目 | 内容 |
|---|---|
| エンティティ名 | Region |
| 概要 | 地域・都道府県など、データセンター所在地の分類 |
| 集約 | Location集約 |
| 主キー | regionId |

#### 主な属性

| 属性 | 型 | 必須 | 説明 |
|---|---:|:---:|---|
| regionId | Long | ○ | 地域ID |
| tenantId | Long | ○ | テナントID |
| regionCode | String | - | 地域コード |
| regionName | String | ○ | 地域名 |
| prefecture | String | - | 都道府県 |
| displayOrder | Integer | - | 表示順 |
| status | RegionStatus | ○ | ACTIVE / INACTIVE |

### 5.2 DataCenter

| 項目 | 内容 |
|---|---|
| エンティティ名 | DataCenter |
| 概要 | データセンターを表す |
| 集約 | DataCenter集約 |
| 主キー | dataCenterId |

#### 主な属性

| 属性 | 型 | 必須 | 説明 |
|---|---:|:---:|---|
| dataCenterId | Long | ○ | データセンターID |
| tenantId | Long | ○ | テナントID |
| regionId | Long | - | 地域ID |
| formalName | String | ○ | 正式名称 |
| displayName | String | - | 代表表示名。複数の呼称名・別名はResourceAliasで保持 |
| address | String | - | 所在地 |
| status | DataCenterStatus | ○ | 利用状態 |

#### 主な振る舞い

- 名称変更
- 代表表示名設定
- 呼称名・別名追加
- 利用停止
- タグ付与
- 連絡先関連付け

### 5.3 Building

| 項目 | 内容 |
|---|---|
| エンティティ名 | Building |
| 概要 | データセンター内の棟を表す |
| 集約 | DataCenter集約 |
| 主キー | buildingId |

### 5.4 Floor

| 項目 | 内容 |
|---|---|
| エンティティ名 | Floor |
| 概要 | 建物内のフロアを表す |
| 集約 | DataCenter集約 |
| 主キー | floorId |

### 5.5 Area

| 項目 | 内容 |
|---|---|
| エンティティ名 | Area |
| 概要 | フロア内の区画を表す。東・西・南・北など |
| 集約 | DataCenter集約 |
| 主キー | areaId |

### 5.6 RackRow

| 項目 | 内容 |
|---|---|
| エンティティ名 | RackRow |
| 概要 | ラック列を表す |
| 集約 | Rack集約 |
| 主キー | rackRowId |

### 5.7 Rack

| 項目 | 内容 |
|---|---|
| エンティティ名 | Rack |
| 概要 | 物理ラックを表す |
| 集約 | Rack集約 |
| 主キー | rackId |

#### 主な属性

| 属性 | 型 | 必須 | 説明 |
|---|---:|:---:|---|
| rackId | Long | ○ | ラックID |
| tenantId | Long | ○ | テナントID |
| rackRowId | Long | ○ | ラック列ID |
| formalName | String | ○ | 正式名称 |
| displayName | String | - | 代表表示名。複数の呼称名・別名はResourceAliasで保持 |
| rackNumber | String | ○ | ラック番号 |
| heightUnit | Integer | ○ | ラック高さ。例: 42U |
| status | RackStatus | ○ | 利用状態 |

### 5.8 RackTemplate

| 項目 | 内容 |
|---|---|
| エンティティ名 | RackTemplate |
| 概要 | ラック作成時に利用する標準ラック構成テンプレート |
| 集約 | RackTemplate集約 |
| 主キー | rackTemplateId |

#### 主な属性

| 属性 | 型 | 必須 | 説明 |
|---|---:|:---:|---|
| rackTemplateId | Long | ○ | ラックテンプレートID |
| tenantId | Long | ○ | テナントID |
| templateName | String | ○ | テンプレート名 |
| heightUnit | Integer | ○ | ラック高さU数 |
| category | String | - | 用途・カテゴリ |
| active | Boolean | ○ | 有効フラグ |

### 5.9 RackTemplateItem

| 項目 | 内容 |
|---|---|
| エンティティ名 | RackTemplateItem |
| 概要 | テンプレート内の標準搭載構成・予約U範囲 |
| 集約 | RackTemplate集約 |
| 主キー | rackTemplateItemId |

| 属性 | 型 | 必須 | 説明 |
|---|---:|:---:|---|
| rackTemplateItemId | Long | ○ | 明細ID |
| rackTemplateId | Long | ○ | ラックテンプレートID |
| itemType | RackTemplateItemType | ○ | DEVICE_PLACEHOLDER / RESERVED_U 等 |
| startUnit | Integer | ○ | 開始U |
| unitSize | Integer | ○ | 使用U数 |
| label | String | - | 表示ラベル |

## 6. 機器系エンティティ

### 6.1 Device

| 項目 | 内容 |
|---|---|
| エンティティ名 | Device |
| 概要 | サーバー、NW機器などの物理・設備管理対象機器。クラウドリソースは将来拡張で別管理 |
| 集約 | Device集約 |
| 主キー | deviceId |

#### 主な属性

| 属性 | 型 | 必須 | 説明 |
|---|---:|:---:|---|
| deviceId | Long | ○ | 機器ID |
| tenantId | Long | ○ | テナントID |
| rackId | Long | - | 設置ラックID |
| deviceType | DeviceType | ○ | SERVER / SWITCH / ROUTER / FIREWALL / LOAD_BALANCER / CLOUD など |
| formalName | String | ○ | 正式名称 |
| displayName | String | - | 代表表示名。複数の呼称名・別名はResourceAliasで保持 |
| hostname | String | - | ホスト名 |
| serialNumber | String | - | シリアル番号 |
| manufacturer | String | - | メーカー |
| modelName | String | - | 型番 |
| rackUnitStart | Integer | - | 搭載開始U |
| rackUnitSize | Integer | - | 使用U数 |
| lifecycleStatus | DeviceLifecycleStatus | ○ | 利用中 / 予備 / 廃止予定 / 廃止済み |

#### 主な振る舞い

- ラックへの配置
- IPアドレス割当
- 保守契約紐付け
- タグ付与
- 代表表示名設定
- 呼称名・別名追加
- 廃止処理

### 6.2 IpSubnet

| 項目 | 内容 |
|---|---|
| エンティティ名 | IpSubnet |
| 概要 | CIDR単位のIP管理範囲。プラン上限・追加オプションのカウント対象 |
| 集約 | Network集約 |
| 主キー | ipSubnetId |

#### 主な属性

| 属性 | 型 | 必須 | 説明 |
|---|---:|:---:|---|
| ipSubnetId | Long | ○ | IPサブネットID |
| tenantId | Long | ○ | テナントID |
| subnetName | String | ○ | サブネット名 |
| cidr | String | ○ | CIDR表記。例: 192.0.2.0/24 |
| ipVersion | IpVersion | ○ | IPv4 / IPv6 |
| status | IpSubnetStatus | ○ | 利用中 / 予約 / 廃止 |
| description | String | - | 備考 |

### 6.3 IpAddress

| 項目 | 内容 |
|---|---|
| エンティティ名 | IpAddress |
| 概要 | IPサブネット配下の個別IP利用状況 |
| 集約 | Network集約 |
| 主キー | ipAddressId |

#### 主な属性

| 属性 | 型 | 必須 | 説明 |
|---|---:|:---:|---|
| ipAddressId | Long | ○ | IPアドレスID |
| tenantId | Long | ○ | テナントID |
| ipSubnetId | Long | ○ | IPサブネットID |
| ipAddress | String | ○ | IPアドレス |
| ipVersion | IpVersion | ○ | IPv4 / IPv6 |
| deviceId | Long | - | 割当機器ID |
| usageStatus | IpUsageStatus | ○ | 未使用 / 使用中 / 予約 / 廃止 |
| description | String | - | 備考 |

## 7. 保守契約系エンティティ

### 7.1 MaintenanceContract

| 項目 | 内容 |
|---|---|
| エンティティ名 | MaintenanceContract |
| 概要 | 保守契約情報を表す |
| 集約 | Maintenance集約 |
| 主キー | maintenanceContractId |

#### 主な属性

| 属性 | 型 | 必須 | 説明 |
|---|---:|:---:|---|
| maintenanceContractId | Long | ○ | 保守契約ID |
| tenantId | Long | ○ | テナントID |
| contractName | String | ○ | 契約名 |
| vendorName | String | ○ | 保守ベンダー名 |
| contractNumber | String | - | 契約番号 |
| startDate | LocalDate | ○ | 開始日 |
| endDate | LocalDate | ○ | 終了日 |
| notificationEnabled | Boolean | ○ | 通知有効フラグ |
| notificationDaysBefore | Integer | ○ | 期限何日前に通知するか。標準60日 |

### 7.2 MaintenanceContractDevice

| 項目 | 内容 |
|---|---|
| エンティティ名 | MaintenanceContractDevice |
| 概要 | 保守契約と機器の紐付け |
| 集約 | Maintenance集約 |
| 主キー | maintenanceContractDeviceId |

### 7.3 MaintenanceContractContact

| 項目 | 内容 |
|---|---|
| エンティティ名 | MaintenanceContractContact |
| 概要 | 保守契約と連絡先の紐付け |
| 集約 | Maintenance集約 |
| 主キー | maintenanceContractContactId |

#### 主な属性

| 属性 | 型 | 必須 | 説明 |
|---|---:|:---:|---|
| maintenanceContractContactId | Long | ○ | 保守契約・連絡先関連ID |
| tenantId | Long | ○ | テナントID |
| maintenanceContractId | Long | ○ | 保守契約ID |
| contactId | Long | ○ | 連絡先ID |
| contactRole | ContactRole | - | VENDOR / EMERGENCY / BILLING / OTHER 等 |

## 8. 連絡先・タグ系エンティティ

### 8.1 Contact

| 項目 | 内容 |
|---|---|
| エンティティ名 | Contact |
| 概要 | データセンター、保守ベンダー、社内担当などの連絡先 |
| 集約 | Contact集約 |
| 主キー | contactId |

#### 主な属性

| 属性 | 型 | 必須 | 説明 |
|---|---:|:---:|---|
| contactId | Long | ○ | 連絡先ID |
| tenantId | Long | ○ | テナントID |
| contactType | ContactType | ○ | DC / VENDOR / INTERNAL など |
| organizationName | String | - | 会社・組織名 |
| departmentName | String | - | 部署名 |
| positionName | String | - | 役職・肩書 |
| personName | String | ○ | 担当者名または窓口名 |
| email | String | - | メールアドレス |
| phoneNumber | String | - | 電話番号 |
| address | String | - | 住所・所在地 |
| preferredContactMethod | ContactMethod | - | EMAIL / PHONE / OTHER |
| note | String | - | 備考 |
| active | Boolean | ○ | 有効フラグ |

### 8.2 Tag

| 項目 | 内容 |
|---|---|
| エンティティ名 | Tag |
| 概要 | 任意の分類ラベル |
| 集約 | Tag集約 |
| 主キー | tagId |

### 8.3 TaggedResource

| 項目 | 内容 |
|---|---|
| エンティティ名 | TaggedResource |
| 概要 | タグと対象リソースの関連 |
| 集約 | Tag集約 |
| 主キー | taggedResourceId |

### 8.4 ResourceAlias

| 項目 | 内容 |
|---|---|
| エンティティ名 | ResourceAlias |
| 概要 | 正式名称とは別に検索対象とする呼称名・別名・略称・運用名 |
| 集約 | 対象リソースの集約に従属 |
| 主キー | resourceAliasId |

#### 主な属性

| 属性 | 型 | 必須 | 説明 |
|---|---:|:---:|---|
| resourceAliasId | Long | ○ | 別名ID |
| tenantId | Long | ○ | テナントID |
| resourceType | ResourceType | ○ | DATA_CENTER / RACK / DEVICE |
| resourceId | Long | ○ | 対象ID |
| aliasName | String | ○ | 呼称名・別名・略称・運用名 |
| aliasType | AliasType | - | DISPLAY / ABBREVIATION / OPERATION / OTHER |

## 9. 通知系エンティティ

### 9.1 NotificationSetting

| 項目 | 内容 |
|---|---|
| エンティティ名 | NotificationSetting |
| 概要 | テナント別通知設定 |
| 集約 | Notification集約 |
| 主キー | notificationSettingId |

### 9.2 NotificationLog

| 項目 | 内容 |
|---|---|
| エンティティ名 | NotificationLog |
| 概要 | 通知送信履歴 |
| 集約 | Notification集約 |
| 主キー | notificationLogId |

## 10. 将来拡張：クラウド資産系エンティティ

### 10.1 CloudAccount

| 項目 | 内容 |
|---|---|
| エンティティ名 | CloudAccount |
| 概要 | AWS等のクラウドアカウント情報。初期リリース対象外 |
| 集約 | CloudResource集約 |
| 主キー | cloudAccountId |

### 10.2 CloudResource

| 項目 | 内容 |
|---|---|
| エンティティ名 | CloudResource |
| 概要 | EC2、コンテナ、EKS Pod等のクラウドリソース。初期リリース対象外 |
| 集約 | CloudResource集約 |
| 主キー | cloudResourceId |

## 11. ユーザー・権限系エンティティ

### 11.1 AppUser

| 項目 | 内容 |
|---|---|
| エンティティ名 | AppUser |
| 概要 | システム利用者 |
| 集約 | User集約 |
| 主キー | userId |

#### 主な属性

| 属性 | 型 | 必須 | 説明 |
|---|---:|:---:|---|
| userId | Long | ○ | ユーザーID |
| tenantId | Long | ○ | テナントID |
| email | String | ○ | ログインIDとなるメールアドレス |
| displayName | String | ○ | 表示名 |
| passwordHash | String | ○ | ハッシュ化済みパスワード。平文パスワードは保持しない |
| passwordUpdatedAt | LocalDateTime | - | パスワード最終更新日時 |

### 11.1A PasswordResetToken

| 項目 | 内容 |
|---|---|
| エンティティ名 | PasswordResetToken |
| 概要 | パスワード再設定用の有効期限付きワンタイムトークン。平文トークンは保持しない |
| 主キー | passwordResetTokenId |

| 属性 | 型 | 必須 | 説明 |
|---|---:|:---:|---|
| passwordResetTokenId | Long | ○ | トークンID |
| tenantId | Long | ○ | テナントID |
| userId | Long | ○ | 対象ユーザーID |
| tokenHash | String | ○ | ハッシュ化済みトークン |
| expiresAt | LocalDateTime | ○ | 有効期限 |
| usedAt | LocalDateTime | - | 使用日時 |
| status | UserStatus | ○ | ACTIVE / INVITED / SUSPENDED |

### 11.2 Role

| 項目 | 内容 |
|---|---|
| エンティティ名 | Role |
| 概要 | 管理者、編集者、閲覧者などのロール |
| 集約 | User集約 |
| 主キー | roleId |

## 12. 列挙型

| 列挙型 | 値 |
|---|---|
| PlanType | FREE, STARTER, BUSINESS, ENTERPRISE |
| TenantStatus | ACTIVE, SUSPENDED, TRIAL_EXPIRED, CANCELLED |
| DeviceType | SERVER, SWITCH, ROUTER, FIREWALL, LOAD_BALANCER, STORAGE, OTHER |
| IpVersion | IPV4, IPV6 |
| IpSubnetStatus | ACTIVE, RESERVED, RETIRED |
| IpUsageStatus | UNUSED, IN_USE, RESERVED, RETIRED |
| DeviceLifecycleStatus | ACTIVE, SPARE, PLANNED_RETIREMENT, RETIRED |
| NotificationType | MAINTENANCE_EXPIRY, PLAN_LIMIT, TRIAL_EXPIRY, PASSWORD_RESET, USER_INVITATION, SYSTEM |
| NotificationStatus | PENDING, SENT, FAILED, SKIPPED |
| NotificationChannel | EMAIL, IN_APP |
| RegionStatus | ACTIVE, INACTIVE |
| ResourceType | DATA_CENTER, RACK, DEVICE, IP_SUBNET, IP_ADDRESS, MAINTENANCE_CONTRACT |
| AliasType | DISPLAY, ABBREVIATION, OPERATION, OTHER |
| ContactMethod | EMAIL, PHONE, OTHER |
| UserStatus | ACTIVE, INVITED, SUSPENDED |
| DataCenterStatus | ACTIVE, INACTIVE |
| RackStatus | ACTIVE, INACTIVE |
| RackTemplateItemType | DEVICE_PLACEHOLDER, RESERVED_U, BLANK_PANEL, NETWORK_AREA, OTHER |
| ContactType | DC, VENDOR, INTERNAL |
| ContactRole | PRIMARY, VENDOR, EMERGENCY, BILLING, OTHER |
| AddOnType | IP_ADDRESS, DEVICE |
| Direction | EAST, WEST, SOUTH, NORTH, OTHER |
