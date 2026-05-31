# ドメインモデル

## 1. 目的

本書は、Data Center Asset & Infrastructure Manager の基本設計におけるドメインモデルを定義する。

開発方針は、Java / Spring Boot / Vaadin / MariaDB を前提とし、DDDを意識したドメイン分割を行う。

## 2. 境界づけられたコンテキスト

| コンテキスト | 概要 | 主な集約 |
|---|---|---|
| テナント・契約管理 | SaaSテナント、契約プラン、利用上限、オプションを管理する | Tenant、SubscriptionPlan、UsageLimit、TenantAddOn |
| ロケーション管理 | リージョン、データセンター、建物、フロア、エリア、ラック列を管理する | Region、DataCenter、Building、Floor、Area、RackRow |
| ラック管理 | ラックとラック内搭載位置を管理する | Rack、RackMountPosition、RackTemplate |
| 機器管理 | サーバー、ネットワーク機器、呼称名・別名、タグを管理する | Device、ResourceAlias、Tag |
| IPサブネット管理 | IPサブネットと配下IPの割当状況を管理する | IpSubnet、IpAddress |
| 保守契約管理 | 保守契約、対象機器、連絡先、期限通知を管理する | MaintenanceContract、MaintenanceContractDevice、MaintenanceContractContact、MaintenanceAlert |
| 将来拡張：クラウドリソース管理 | AWSアカウント、リージョン、EC2/EKS/コンテナ等を管理する | CloudAccount、CloudResource |
| ユーザー・権限管理 | ユーザー、ロール、権限、招待を管理する | UserAccount、Role、Permission、RolePermission、UserInvitationToken |
| 通知管理 | メール/画面内通知条件、通知先、通知履歴、未読/既読を管理する | NotificationSetting、NotificationLog |
| 操作履歴管理 | 初期必須の操作履歴保存・検索・閲覧を管理する | AuditLog |

## 3. 主要エンティティ

ID型は初期リリースでは詳細設計・DB設計に合わせて `Long`（DB上は `bigint`）を採用する。UUIDは将来、外部公開IDや分散構成が必要になった場合に再検討する。

### 3.1 Tenant

| 属性 | 型 | 説明 |
|---|---|---|
| tenantId | Long | テナントID |
| name | String | テナント名 |
| planCode | String | 契約プランコード。SubscriptionPlanマスタのplanCodeを参照 |
| planType | Enum | Free / Starter / Business / Enterprise。画面表示・判定用にSubscriptionPlanから取得 |
| status | Enum | Active / Suspended / TrialExpired / Cancelled |
| trialStartDate | Date | Freeトライアル開始日。Free以外はnull可 |
| trialEndDate | Date | Freeトライアル終了日。Free以外はnull可 |
| createdAt | DateTime | 作成日時 |
| updatedAt | DateTime | 更新日時 |

### 3.2 SubscriptionPlan

| 属性 | 型 | 説明 |
|---|---|---|
| planId | Long | プランID |
| planCode | String | プランコード。FREE / STARTER / BUSINESS / ENTERPRISE |
| planType | Enum | プラン種別 |
| maxDataCenters | Integer | DC上限 |
| maxRacks | Integer | ラック上限 |
| maxDevices | Integer | 機器上限 |
| maxIpAddresses | Integer | 管理対象IP数上限 |
| maxTags | Integer | タグマスタ件数上限 |
| trialDays | Integer | Freeトライアル日数 |
| maxUsers | Integer | ユーザー上限 |

### 3.3 Region

| 属性 | 型 | 説明 |
|---|---|---|
| regionId | Long | リージョンID |
| tenantId | Long | テナントID |
| regionName | String | 地域名 |
| prefecture | String | 都道府県 |
| displayOrder | Integer | 表示順 |
| status | Enum | Active / Inactive |

### 3.4 DataCenter

| 属性 | 型 | 説明 |
|---|---|---|
| dataCenterId | Long | データセンターID |
| tenantId | Long | テナントID |
| formalName | String | 正式名称 |
| displayName | String | 代表表示名。複数の呼称名・別名はResourceAliasで保持 |
| regionId | Long | 地域・都道府県分類ID。`Region` を参照 |
| address | String | 所在地 |
| status | Enum | Active / Inactive |

### 3.5 Location階層

| エンティティ | 説明 |
|---|---|
| Building | データセンター内の建物・棟 |
| Floor | 建物内のフロア |
| Area | フロア内のエリア。東/西/南/北など |
| RackRow | エリア内のラック列 |
| Rack | ラック列内のラック |

### 3.6 Rack

| 属性 | 型 | 説明 |
|---|---|---|
| rackId | Long | ラックID |
| rackRowId | Long | ラック列ID |
| formalName | String | 正式名称 |
| displayName | String | 代表表示名。複数の呼称名・別名はResourceAliasで保持 |
| heightU | Integer | ラック高さU数 |
| positionNo | String | ラック列内位置 |
| status | Enum | ACTIVE / INACTIVE |

### 3.6A RackTemplate

| 属性 | 型 | 説明 |
|---|---|---|
| rackTemplateId | Long | ラックテンプレートID |
| tenantId | Long | テナントID |
| templateName | String | テンプレート名 |
| heightU | Integer | ラック高さU数 |
| category | String | 用途・カテゴリ |
| active | Boolean | 有効フラグ |
| note | String | 備考 |
| templateItems | List | 標準搭載構成・予約U範囲。作成時コピーとしてラックへ反映 |

### 3.7 Device

| 属性 | 型 | 説明 |
|---|---|---|
| deviceId | Long | 機器ID |
| tenantId | Long | テナントID |
| rackId | Long | 設置ラックID |
| formalName | String | 正式名称 |
| displayName | String | 代表表示名。複数の呼称名・別名はResourceAliasで保持 |
| deviceType | Enum | Server / Switch / Router / Firewall / LoadBalancer / Other |
| vendor | String | ベンダー |
| modelName | String | 型番 |
| serialNumber | String | シリアル番号 |
| rackStartU | Integer | 搭載開始U |
| rackSizeU | Integer | 搭載U数 |
| lifecycleStatus | Enum | ACTIVE / SPARE / PLANNED_RETIREMENT / RETIRED |

### 3.8 IpSubnet / IpAddress

| 属性 | 型 | 説明 |
|---|---|---|
| ipSubnetId | Long | IPサブネットID |
| tenantId | Long | テナントID |
| cidr | String | CIDR表記 |
| name | String | サブネット名 |
| ipVersion | Enum | IPv4 / IPv6 |
| status | Enum | ACTIVE / RESERVED / RETIRED |
| ipAddressId | Long | 個別IP利用状況ID |
| address | String | IPアドレス |
| assignedDeviceId | Long | 割当機器ID |
| purpose | String | 用途 |
| usageStatus | Enum | UNUSED / IN_USE / RESERVED / RETIRED |
| registrationMode | Enum | RANGE_GENERATED / MANUAL。IPv4は範囲生成、IPv6は明示登録を基本とする |

### 3.9 MaintenanceContract

| 属性 | 型 | 説明 |
|---|---|---|
| maintenanceContractId | Long | 保守契約ID |
| tenantId | Long | テナントID |
| contractName | String | 契約名 |
| vendorName | String | ベンダー名 |
| contractNo | String | 契約番号 |
| contractDescription | Text | 契約内容。サポート範囲、SLA、更新条件などの要約 |
| renewalStatus | Enum | 更新状態。ACTIVE / RENEWAL_REQUIRED / RENEWED / TERMINATED |
| startDate | Date | 開始日 |
| endDate | Date | 終了日 |
| expiryStatus | Enum | 期限状態。OK / DUE_SOON / EXPIRES_TODAY / EXPIRED。原則としてendDateと基準日から算出し、検索用DTOに保持する |
| notificationEnabled | Boolean | 通知有効フラグ |
| notificationDaysBefore | Integer | 主通知日数。標準60日。30日前・当日・期限切れは通知設定/バッチ条件で扱う |
| note | Text | 備考 |

### 3.10 CloudResource（将来拡張）

| 属性 | 型 | 説明 |
|---|---|---|
| cloudResourceId | Long | クラウドリソースID |
| cloudAccountId | Long | クラウドアカウントID |
| resourceType | Enum | EC2 / Container / EksCluster / EksPod / Other |
| resourceName | String | リソース名 |
| region | String | リージョン |
| externalResourceId | String | クラウド側ID |
| status | String | クラウド側状態 |

### 3.11 共通監査情報

主要エンティティは、最低限の監査情報として以下を保持する。

| 属性 | 型 | 説明 |
|---|---|---|
| createdBy | Long | 作成者 |
| createdAt | DateTime | 作成日時 |
| updatedBy | Long | 更新者 |
| updatedAt | DateTime | 更新日時 |

初期リリースでは操作履歴をAuditLogに保存する。変更差分を含む完全な変更履歴は将来拡張で扱う。

### 3.12 連絡先・タグ・通知・CSV関連

| エンティティ | 説明 |
|---|---|
| Contact | データセンターや保守契約に紐づく連絡先 |
| ResourceAlias | データセンター、ラック、機器など対象リソースの呼称名・別名 |
| Tag | 検索・分類用タグ |
| TaggedResource | タグと対象リソースの関連 |
| NotificationSetting | 通知条件、通知先、通知チャネル設定 |
| NotificationLog | 画面内通知・メール通知の履歴 |
| UserInvitationToken | ユーザー招待トークン。有効期限、一度限り利用、取消、再招待を管理 |
| AuditLog | 操作履歴。登録、更新、削除、招待、権限変更、契約変更、通知設定変更などを保存 |
| CsvImportHistory | CSV取込履歴 |
| CsvImportError | CSV取込時の行単位エラー |
| MaintenanceContractDevice | 保守契約と対象機器の関連 |
| MaintenanceContractContact | 保守契約と連絡先の関連 |
### 3.13 権限・操作履歴・招待

| エンティティ | 説明 |
|---|---|
| Role | 要件定義の7ロールを保持する |
| Permission | 固定権限IDをDBへ初期投入し、画面/API認可で参照する |
| RolePermission | ロールと権限の標準対応を保持する。初期リリースでは標準ロールの権限変更は行わない |
| UserInvitationToken | 招待メールに含める一度限りのトークンハッシュ、有効期限、使用日時、取消日時を保持する |
| AuditLog | 操作履歴を保持する。変更差分を含む完全な変更履歴は将来拡張とする |

### 3.14 IPv4/IPv6管理方式

| 区分 | サブネット登録 | 個別IP登録 | 上限カウント |
|---|---|---|---|
| IPv4 | CIDR登録後、必要範囲の個別IPを範囲生成できる | 範囲生成または明示登録 | 生成・明示登録した個別IP数を管理対象IP数としてカウント |
| IPv6 | CIDR情報は登録するが、全範囲生成は行わない | 利用・予約するアドレスのみ明示登録 | 明示登録した個別IPv6数を管理対象IP数としてカウント |

IPv6はアドレス空間が大きいため、初期リリースではサブネット情報と明示登録IPのみを扱う。IPv4の範囲生成でも、生成予定数がプラン上限を超える場合は登録不可とする。


## 4. 値オブジェクト候補

| 値オブジェクト | 説明 |
|---|---|
| TenantId | テナント識別子 |
| DataCenterId | データセンター識別子 |
| RackPosition | ラック内U位置 |
| IpSubnetCidr | IPサブネットCIDR |
| IpAddressValue | IPアドレス値 |
| ContactInfo | メール・電話番号 |
| DateRange | 開始日・終了日の期間 |
| UsageLimit | 契約プラン上限 |
| AliasName | 別名・呼称名 |

## 5. 集約と関連

```mermaid
classDiagram
    class Tenant
    class SubscriptionPlan
    class TenantAddOn
    class Region
    class DataCenter
    class Building
    class Floor
    class Area
    class RackRow
    class Rack
    class RackTemplate
    class Device
    class IpSubnet
    class IpAddress
    class MaintenanceContract
    class MaintenanceContractDevice
    class MaintenanceContractContact
    class NotificationSetting
    class NotificationLog
    class TaggedResource
    class CloudAccount
    class CloudResource
    class UserAccount
    class Role
    class Contact
    class Tag
    class ResourceAlias

    SubscriptionPlan "1" --> "0..*" Tenant
    Tenant "1" --> "0..*" TenantAddOn
    Tenant "1" --> "0..*" Region
    Tenant "1" --> "0..*" DataCenter
    Tenant "1" --> "0..*" Device
    Tenant "1" --> "0..*" UserAccount
    Tenant "1" --> "0..*" CloudAccount

    Region "1" --> "0..*" DataCenter
    DataCenter "1" --> "0..*" Building
    Building "1" --> "0..*" Floor
    Floor "1" --> "0..*" Area
    Area "1" --> "0..*" RackRow
    RackRow "1" --> "0..*" Rack
    RackTemplate "1" --> "0..*" Rack
    Rack "1" --> "0..*" Device

    IpSubnet "1" --> "0..*" IpAddress
    Device "1" --> "0..*" IpAddress

    MaintenanceContract "1" --> "0..*" MaintenanceContractDevice
    MaintenanceContractDevice "*" --> "1" Device
    MaintenanceContract "1" --> "0..*" MaintenanceContractContact
    MaintenanceContractContact "*" --> "1" Contact
    DataCenter "*" --> "*" Contact
    DataCenter "1" --> "0..*" ResourceAlias
    Rack "1" --> "0..*" ResourceAlias
    Device "1" --> "0..*" ResourceAlias

    CloudAccount "1" --> "0..*" CloudResource

    UserAccount "*" --> "*" Role
    Device "*" --> "0..*" TaggedResource
    Rack "*" --> "0..*" TaggedResource
    DataCenter "*" --> "0..*" TaggedResource
    IpSubnet "*" --> "0..*" TaggedResource
    IpAddress "*" --> "0..*" TaggedResource
    MaintenanceContract "*" --> "0..*" TaggedResource
    TaggedResource "*" --> "1" Tag
    NotificationSetting "1" --> "0..*" NotificationLog
    NotificationLog "*" --> "0..1" MaintenanceContract
```

## 6. ドメインサービス候補

| サービス | 役割 |
|---|---|
| PlanLimitService | プラン別上限チェック、オプション加算後の上限算出 |
| RackPlacementService | ラック搭載位置の重複チェック、空きU計算 |
| IpAllocationService | IPアドレス割当、解放、重複チェック |
| MaintenanceNotificationService | 保守期限60日前・30日前・当日・期限切れ再通知対象の抽出 |
| DeviceSearchService | 保守未設定機器、タグ、別名、設置場所による検索 |
| ResourceAliasService | 対象リソースの呼称名・別名の登録、重複確認、検索 |
| CloudResourceSyncService | 将来拡張。クラウドリソースの同期方針管理 |
| AuthorizationService | ロール・権限による操作可否判定 |

## 7. ドメイン制約

| 制約ID | 制約内容 |
|---|---|
| DRC-001 | すべての主要データはtenantIdで分離する |
| DRC-002 | 契約プラン上限を超えてDC、ラック、機器、管理対象IP、タグ、ユーザーを登録できない |
| DRC-003 | Freeプランは14日間トライアルとし、DC 1件、ラック3本、機器40台、管理対象IP 256件、タグ10件、ユーザー3名まで。IPサブネット数には上限を設けない |
| DRC-003-1 | Freeトライアル期限超過後は `TRIAL_EXPIRED` として扱い、ログイン・参照・CSVエクスポート・契約プラン確認・有料プラン変更申請のみ許可する |
| DRC-004 | 管理対象IP数上限と機器台数はオプションで追加できる |
| DRC-005 | IP上限追加は256IP単位とする |
| DRC-006 | 機器追加は100台単位とする |
| DRC-007 | ラック内のU位置は同一ラック内で重複不可とする |
| DRC-007-1 | ラックテンプレートの標準搭載構成・予約U範囲もU重複不可とし、ラック作成時コピーを基本とする |
| DRC-008 | 保守契約には複数の機器をひもづけ可能とする |
| DRC-009 | 保守契約未設定の機器を検索可能とする |
| DRC-010 | 保守期限通知は60日前、30日前、当日、期限切れを初期対象とし、通知ログでチャネル・受信者単位に重複抑止する |
| DRC-010-1 | 保守契約の期限状態は、原則として永続化せず、検索・一覧表示時に終了日と基準日から算出する。大量検索で性能上必要な場合のみ詳細設計でキャッシュ列を検討する |
| DRC-010-2 | 保守契約の更新状態は契約業務上の状態として永続化し、期限状態とは別に検索条件・表示項目として扱う |
| DRC-010-3 | 期限切れ通知は初回期限切れ日に送信し、その後は7日ごとに再通知する。保守契約の更新状態がRENEWEDまたはTERMINATEDになった場合、または通知無効化時に停止する |
| DRC-011 | 正式名称とは別に表示名・別名を持てる |
| DRC-012 | 主要エンティティにタグを付与できる。初期対象はDataCenter、Rack、Device、IpSubnet、IpAddress、MaintenanceContractとし、CloudResourceは将来拡張対象とする |
| DRC-013 | タグ数上限は有効なタグマスタ件数で判定し、リソースへのタグ付け件数は対象外とする |

<!-- issue-fixes-239-240-243 -->

## 付録A. Issue対応追補: ドメインモデル補足

### A.1 IPサブネットCIDRの重複判定

同一テナント内ではCIDRの完全一致だけでなく、範囲重複・包含関係も原則禁止する。例: `192.168.0.0/24` が有効な場合、`192.168.0.0/25`、`192.168.0.128/25`、`192.168.0.10/32` は同一範囲を二重管理するため登録不可とする。

将来、意図的な階層管理を許容する場合は別途設計し、IP利用状況の正本サブネットを一意に決める。

### A.2 ResourceAliasの参照整合性

`ResourceAlias` は `resourceType + resourceId` によるポリモーフィック参照であり、DB外部キーではなくService層で整合性を保証する。対象リソースは同一テナント・有効データに限り、削除済み対象への別名付与は禁止する。同一対象内の同一別名は重複不可とする。

### A.3 日付・時刻基準

- DB保存時刻はシステム基準タイムゾーンで統一する。
- 初期リリースのシステム基準タイムゾーンは `Asia/Tokyo` とする。
- `createdAt` / `updatedAt` は `LocalDateTime` + DB `datetime(6)` を基本とする。
- Free期限・保守期限・バッチ判定の「現在日付」は、システム基準タイムゾーンの日付で判定する。
- テナント別タイムゾーンは将来拡張とする。

<!-- issue-fixes-286-290 -->

## 付録B. Issue対応追補: 資産項目の用語・備考

### B.1 Deviceのメーカー・型番

Deviceのメーカー項目は `manufacturer` を正本とし、旧表記の `vendor` は説明上の同義語として扱う。型番は `modelName` または詳細設計の正規名に合わせた `modelNumber` のどちらかに統一する。

| 概念 | 基本設計上の正本 |
|---|---|
| メーカー | `manufacturer` |
| 型番 | `modelName`（詳細設計で `modelNumber` を採用する場合はそちらへ統一） |

### B.2 DC・ラック・機器の備考

要件上の備考項目は、以下のEntityで保持する。

| Entity | 属性 | 用途 |
|---|---|---|
| DataCenter | `note` | DC運用メモ、補足事項 |
| Rack | `note` | ラック設置・利用上の補足 |
| Device | `note` | 機器運用メモ、棚卸補足 |

備考は検索補助には使えるが、識別子や一意制約対象にはしない。機微情報・パスワード・トークンは入力禁止とする。
