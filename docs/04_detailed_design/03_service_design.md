# 4. 詳細設計 - Service設計

## 1. 目的

本書は、Data Center Asset & Infrastructure Manager のService層の責務、主要クラス、主要処理を定義する。

## 2. Service層の基本方針

- Service層は業務ルールを実装する。
- Controller / Vaadin View からRepositoryを直接呼び出さない。
- 登録・更新・削除時は、プラン上限、権限、テナント分離、整合性をService層で検証する。
- 参照系は検索条件DTOを受け取り、PageまたはListで返却する。
- 更新系はCommand DTOを受け取り、結果DTOまたはIDを返却する。
- トランザクション境界はService層に置く。

## 3. パッケージ構成案

```text
com.example.dcim
  ├─ application
  │   ├─ service
  │   ├─ command
  │   ├─ query
  │   └─ dto
  ├─ domain
  │   ├─ model
  │   ├─ repository
  │   └─ exception
  ├─ infrastructure
  │   ├─ repository
  │   ├─ notification
  │   └─ security
  └─ presentation
      └─ vaadin
```

## 4. Service一覧

| Service | 主な責務 |
|---|---|
| TenantService | テナント管理、プラン変更、トライアル状態管理 |
| PlanLimitService | プラン上限判定、Free期限超過時の更新可否判定 |
| UserService | ユーザー管理 |
| RoleService | 権限管理 |
| DataCenterService | DC登録・更新・削除・検索 |
| RegionService | 地域・都道府県分類管理 |
| LocationService | 棟、フロア、区画管理 |
| RackService | ラック列、ラック管理 |
| DeviceService | 機器管理 |
| IpSubnetService | IPサブネット管理、CIDR重複判定 |
| IpAddressService | サブネット配下のIPアドレス利用状況管理 |
| MaintenanceContractService | 保守契約管理 |
| ContactService | 連絡先管理 |
| ResourceAliasService | 呼称名・別名管理 |
| TagService | タグ管理 |
| NotificationService | 通知作成・送信 |
| MaintenanceNotificationService | 保守期限通知処理 |
| AuditLogService | 操作履歴保存・検索 |
| CloudResourceService | 将来拡張：クラウド資産管理 |
| CsvExportService | CSVエクスポート |
| CsvImportService | CSVインポート（初期追加対象） |
| SearchService | 横断検索 |

## 5. 主要Service詳細

## 5.1 PlanLimitService

### 責務

- 契約プランの基本上限取得
- 追加オプションの反映
- 登録前の上限チェック
- 画面表示用の使用量取得

### 主なメソッド

| メソッド | 概要 |
|---|---|
| getCurrentLimit(tenantId) | テナントの現在上限を取得 |
| getCurrentUsage(tenantId) | テナントの現在使用量を取得 |
| validateCanCreateDataCenter(tenantId) | DC追加可否を検証 |
| validateCanCreateRack(tenantId) | ラック追加可否を検証 |
| validateCanCreateDevice(tenantId) | 機器追加可否を検証 |
| validateCanGenerateIpAddresses(tenantId, plannedCount) | 管理対象IP生成可否を検証 |
| validateCanCreateTag(tenantId) | タグ追加可否を検証 |
| validateCanCreateUser(tenantId) | ユーザー追加可否を検証 |

### 判定方針

```text
利用可能上限 = プラン基本上限 + 有効な追加オプション数
登録可能 = 現在使用数 < 利用可能上限
```

Freeトライアル期限超過時は、テナント状態を `TRIAL_EXPIRED` として扱い、参照・CSVエクスポート・契約プラン確認・有料プラン変更以外の更新系操作を不可とする。通常ロールより前にテナント状態を判定する。

### 例外

上限超過時は `PlanLimitExceededException` を送出する。

## 5.2 UserService

### 責務

- ユーザー招待
- ユーザー情報更新
- ロール付与・変更
- ユーザー無効化
- 初期テナント管理者の削除・無効化保護
- パスワードハッシュ保存・更新日時管理
- パスワード再設定依頼、再設定トークン発行・検証・使用済み化
- 管理者によるパスワード再設定メール再送

### 主なメソッド

| メソッド | 概要 |
|---|---|
| invite(command) | ユーザー招待 |
| update(userId, command) | ユーザー情報更新 |
| changeRoles(userId, roleIds) | ロール変更 |
| suspend(userId) | ユーザー停止。ただし初期テナント管理者は通常フローでは不可 |
| updatePasswordHash(userId, passwordHash) | パスワードハッシュ更新 |
| requestPasswordReset(email) | パスワード再設定メール送信依頼。メール存在有無を画面に出さない |
| resetPassword(token, newPassword) | 再設定トークン検証後に新パスワードを設定 |
| resendPasswordReset(userId) | 管理者による再設定メール再送 |

### 業務ルール

- 同一テナント内でメールアドレスを重複させない。
- パスワード平文は保存しない。保存対象はハッシュ値のみとする。
- ユーザー作成前に `PlanLimitService.validateCanCreateUser()` を実行する。
- テナント初期作成時に登録される初期テナント管理者は、通常のユーザー無効化・削除対象にできない。
- 初期テナント管理者を利用停止扱いにする場合は、テナント解約フローでテナント状態変更と合わせて行う。
- パスワード再設定トークンはハッシュ化して保存し、有効期限付き・一度限り利用とする。
- パスワード再設定依頼時は、メールアドレスの存在有無を推測できない共通メッセージを返す。
- 再設定トークン、仮パスワード、平文パスワードはログ・DB・通知本文に残さない。

## 5.3 DataCenterService

### 責務

- データセンター登録
- データセンター更新
- データセンター削除
- DC連絡先紐付け
- タグ付与
- 呼称名・別名の登録・更新

### 主なメソッド

| メソッド | 概要 |
|---|---|
| create(command) | DC登録 |
| update(dataCenterId, command) | DC更新 |
| delete(dataCenterId) | DC論理削除 |
| findById(dataCenterId) | DC詳細取得 |
| search(query) | DC検索 |
| assignContact(dataCenterId, contactId) | 連絡先紐付け |
| addAlias(dataCenterId, aliasName, aliasType) | DC呼称名・別名追加 |
| removeAlias(dataCenterId, aliasId) | DC呼称名・別名削除 |

### 登録処理

1. ログインユーザーのテナントIDを取得する。
2. `PlanLimitService.validateCanCreateDataCenter()` を実行する。
3. 正式名称の重複を確認する。
4. 入力値を検証する。
5. `DataCenter` を保存する。
6. 必要に応じてタグ、連絡先、呼称名・別名を関連付ける。

## 5.4 RegionService

### 責務

- リージョン登録・更新・削除
- DC登録時の選択肢提供
- 表示順制御

### 主なメソッド

| メソッド | 概要 |
|---|---|
| create(command) | リージョン登録 |
| update(regionId, command) | リージョン更新 |
| delete(regionId) | リージョン論理削除 |
| findActive(tenantId) | 有効リージョン一覧取得 |

### 業務ルール

- 同一テナント内で `regionName` を重複させない。
- DCで利用中のリージョンは削除不可またはINACTIVE化とする。

## 5.5 RackService

### 責務

- ラック列管理
- ラック管理
- ラック高さ・搭載位置の管理
- ラック内機器の配置整合性チェック

### 主なメソッド

| メソッド | 概要 |
|---|---|
| createRackRow(command) | ラック列登録 |
| createRack(command) | ラック登録 |
| updateRack(rackId, command) | ラック更新 |
| deleteRack(rackId) | ラック削除 |
| validateRackUnitAvailable(rackId, startUnit, unitSize) | U位置重複チェック |

### ラック搭載位置チェック

- `rack_unit_start` は1以上とする。
- `rack_unit_start + rack_unit_size - 1` が `height_unit` を超えないこと。
- 同一ラック内で使用U範囲が既存機器と重複しないこと。

## 5.5A RackTemplateService

### 責務

- ラックテンプレート登録・更新・複製・無効化
- テンプレート明細（標準搭載構成・予約U範囲）の管理
- テンプレートからラックを作成する

### 主なメソッド

| メソッド | 概要 |
|---|---|
| createTemplate(command) | ラックテンプレート登録 |
| updateTemplate(templateId, command) | ラックテンプレート更新 |
| copyTemplate(templateId) | ラックテンプレート複製 |
| disableTemplate(templateId) | ラックテンプレート無効化 |
| createRackFromTemplate(templateId, command) | テンプレートからラック作成 |

### 業務ルール

- 同一テナント内でテンプレート名を重複させない。
- テンプレート明細の`startUnit + unitSize - 1`は`heightUnit`を超えない。
- テンプレート明細同士のU範囲は重複させない。
- テンプレートから作成したラックも通常のラック上限対象に含める。
- テンプレート適用は作成時コピーを基本とし、テンプレート変更を既存ラックへ自動反映しない。

## 5.6 DeviceService

### 責務

- 機器登録・更新・削除
- ラック配置
- 機器種別管理
- 保守契約との関連確認
- 機器別名の追加・更新・削除
- 機器タグの付与・解除
- 保守未契約機器の検索

### 主なメソッド

| メソッド | 概要 |
|---|---|
| create(command) | 機器登録 |
| update(deviceId, command) | 機器更新 |
| delete(deviceId) | 機器論理削除 |
| assignRack(deviceId, rackId, rackUnitStart, rackUnitSize) | ラック配置 |
| assignIpAddress(deviceId, ipAddressId) | IP割当 |
| addAlias(deviceId, aliasName, aliasType) | 機器別名追加 |
| removeAlias(deviceId, aliasId) | 機器別名削除 |
| assignTag(deviceId, tagId) | 機器タグ付与 |
| removeTag(deviceId, tagId) | 機器タグ解除 |
| findWithoutMaintenance(query) | 保守未紐付け機器検索 |
| search(query) | 機器検索 |

### 登録処理

1. テナントIDを取得する。
2. `PlanLimitService.validateCanCreateDevice()` を実行する。
3. 正式名称・ホスト名・シリアル番号の重複を確認する。
4. ラック指定がある場合、ラック存在確認とU位置重複チェックを行う。
5. 機器を保存する。
6. 別名指定がある場合は `ResourceAliasService` で関連付ける。
7. タグ指定がある場合は `TagService` で関連付ける。

## 5.7 IpSubnetService / IpAddressService

### IpSubnetServiceの責務

- IPサブネット登録・更新・削除
- 管理対象IP数上限チェック
- CIDR重複チェック
- サブネット配下IPの生成・再生成方針管理

### IpAddressServiceの責務

- IPサブネット配下の個別IP利用状況管理
- 機器への割当・解除
- IPアドレス検索

### 主なメソッド

| メソッド | 概要 |
|---|---|
| createSubnet(command) | IPサブネット登録 |
| updateSubnet(ipSubnetId, command) | IPサブネット更新 |
| deleteSubnet(ipSubnetId) | IPサブネット論理削除 |
| generateAddresses(ipSubnetId) | サブネット配下IPの利用状況を生成 |
| countActiveAddresses(tenantId) | 有効な管理対象IP数を取得 |
| assignToDevice(ipAddressId, deviceId) | 機器へ割当 |
| release(ipAddressId) | 割当解除 |
| searchSubnets(query) | IPサブネット検索 |
| searchAddresses(query) | IPアドレス検索 |

### IP管理ルール

- `ip_subnet` の有効件数はプラン上限判定対象外とし、生成・管理する `ip_address` の有効件数のみを上限判定する。
- CIDRプレフィックスはプランで固定しない。サブネット登録・個別IP生成時に、生成予定IP数を含めた管理対象IP数上限（プラン上限 + IP上限追加オプション数 × 256）を超えないことを確認する。
- 同一テナント内でCIDRは重複不可とする。
- 個別IPは必ず1つのIPサブネットに属する。
- `UNUSED` または `RESERVED` のIPのみ機器へ割当可能。
- 割当後は `IN_USE` とする。
- 解除後は原則 `UNUSED` とする。

## 5.8 MaintenanceContractService

### 責務

- 保守契約登録・更新・削除
- 保守契約と機器の紐付け
- 保守契約と連絡先の紐付け
- 保守期限検索
- 保守未契約機器検索

### 主なメソッド

| メソッド | 概要 |
|---|---|
| create(command) | 保守契約登録 |
| update(contractId, command) | 保守契約更新 |
| delete(contractId) | 保守契約論理削除 |
| assignDevice(contractId, deviceId) | 機器紐付け |
| removeDevice(contractId, deviceId) | 機器紐付け解除 |
| assignContact(contractId, contactId) | 保守契約連絡先紐付け |
| removeContact(contractId, contactId) | 保守契約連絡先紐付け解除 |
| findExpiringContracts(daysBefore) | 期限到来契約検索 |
| findDevicesWithoutMaintenance(query) | 保守未契約機器検索 |

### 保守期限判定

```text
通知対象日 = 契約終了日 - 通知設定.days_before（60日前/30日前/当日/期限切れをtiming_codeごとに判定）
現在日付 >= 通知対象日。期限切れ通知では契約終了日 < 現在日付も対象に含める
```

標準は60日前通知とする。

## 5.9 NotificationService

### 責務

- 通知メッセージ生成
- メール通知送信
- 画面内通知作成
- 通知設定取得・更新
- 通知ログ保存
- 未読/既読管理
- 重複通知抑止

### 主なメソッド

| メソッド | 概要 |
|---|---|
| createNotification(command) | 通知作成 |
| createInAppNotification(command) | 画面内通知作成 |
| send(notificationId) | 通知送信 |
| markAsRead(notificationId, userId) | 画面内通知を既読化 |
| updateSetting(command) | 通知設定更新 |
| sendMaintenanceExpiryNotification(contractId) | 保守期限通知送信 |
| hasAlreadySent(type, channel, targetId, recipient) | メール通知の重複通知確認 |
| hasAlreadySentInApp(type, targetId, recipientUserId) | 画面内通知の重複通知確認 |

## 5.10 MaintenanceNotificationService

### 責務

- 定期実行により保守期限が近い契約を抽出する。
- 通知対象の連絡先・ユーザーを決定する。
- メール通知と画面内通知の作成・送信を依頼する。

### スケジュール案

| 項目 | 内容 |
|---|---|
| 実行頻度 | 1日1回 |
| 実行時刻 | 09:00 |
| 対象 | notification_enabled = true の保守契約 |
| 標準通知 | 60日前 |


## 5.10A AuditLogService

### 責務

- 初期必須の操作履歴を保存する。
- ログイン成功/失敗、主要データの登録・更新・削除、ユーザー招待、権限変更、CSVエクスポート、プラン変更申請を記録する。
- SCR-032 操作履歴画面向けに、権限に応じた検索結果を返す。
- パスワード、トークン、APIキー、CSV内容そのものなどの秘密情報は保存しない。

### 主なメソッド

| メソッド | 概要 |
|---|---|
| `recordLoginSuccess(command)` | ログイン成功を記録 |
| `recordLoginFailure(command)` | ログイン失敗を記録。未認証のためuserIdはNULL可 |
| `recordDataChange(command)` | CRUD、招待、権限変更、契約変更などを記録 |
| `recordCsvExport(command)` | CSV出力操作を記録 |
| `searchAuditLogs(query)` | 操作履歴画面向け検索 |

## 5.11 ContactService

### 責務

- 連絡先登録・更新・削除
- DC・保守契約への連絡先関連付け
- 通知先候補の提供

### 主なメソッド

| メソッド | 概要 |
|---|---|
| create(command) | 連絡先登録 |
| update(contactId, command) | 連絡先更新 |
| delete(contactId) | 連絡先論理削除 |
| assignToDataCenter(dataCenterId, contactId, role) | DC連絡先紐付け |
| assignToMaintenanceContract(contractId, contactId, role) | 保守契約連絡先紐付け |
| findNotificationContacts(target) | 通知対象連絡先取得 |

### 業務ルール

- 同一テナント内の対象にのみ紐付ける。
- 通知先に利用する場合はメールアドレスを必須とする。

## 5.12 ResourceAliasService

### 責務

- 呼称名・別名登録・更新・削除
- 正式名称・代表表示名と合わせた検索条件提供

### 主なメソッド

| メソッド | 概要 |
|---|---|
| addAlias(resourceType, resourceId, aliasName, aliasType) | 別名追加 |
| updateAlias(aliasId, command) | 別名更新 |
| removeAlias(aliasId) | 別名削除 |
| findByResource(resourceType, resourceId) | 対象リソースの別名一覧 |
| searchByAlias(query) | 別名検索 |

### 業務ルール

- 同一対象内で同じ別名を重複登録しない。
- 正式名称と同一の別名は登録不可とする。
- 機器の別名は正式名称・代表表示名と合わせて機器一覧、詳細検索、横断検索の検索対象に含める。

## 5.13 TagService

### 責務

- タグ登録・更新・削除
- タグ登録前の `PlanLimitService.validateCanCreateTag()` 実行
- 各リソースへのタグ付与・解除
- タグによる検索条件提供

### 対象リソース

初期対象は `DATA_CENTER`、`RACK`、`DEVICE`、`IP_SUBNET`、`IP_ADDRESS`、`MAINTENANCE_CONTRACT` とする。機器タグは機器一覧、詳細検索、横断検索の検索対象に含める。

- DataCenter
- Rack
- Device
- IpSubnet
- IpAddress
- MaintenanceContract
- CloudResource（将来拡張）

### タグ上限

タグ数のプラン上限はタグマスタ件数で判定し、リソースへのタグ付け件数は上限対象外とする。

## 5.14 CloudResourceService（将来拡張）

### 責務

- クラウドアカウント管理
- EC2 / コンテナ / EKS Pod等の手動登録
- 将来の外部連携取込に備えたデータ管理

初期リリースでは画面・業務処理の対象外とし、設計上の拡張ポイントとして保持する。

## 5.15 CsvExportService / CsvImportService

### 責務

- CSVエクスポートは初期リリース必須機能として、主要マスタ・一覧データを対象に提供する。
- CSVインポートは初期追加対象として、履歴・エラー管理を含めて設計する。
- FreeプランのCSVインポートは1ファイル100行まで許可する。101行以上はファイル単位エラーとする。
- 権限、テナント分離、プラン上限、入力チェックをService層で検証する。

### 主なメソッド

| メソッド | 概要 |
|---|---|
| export(targetType, query) | 対象データをCSV出力し、履歴を保存する |
| importCsv(targetType, file) | CSVを検証・登録し、取込履歴を保存する |
| validateHeader(targetType, file) | ヘッダ妥当性を検証する |
| findImportErrors(historyId) | 取込エラー一覧を取得する |

## 6. トランザクション方針

| 処理 | トランザクション |
|---|---|
| 登録・更新・削除 | REQUIRED |
| 検索・参照 | readOnly |
| 通知送信 | ログ作成と送信結果更新を分離 |
| 一括登録 | ヘッダ不一致・ファイル形式不正はファイル単位でFAILED。行単位の入力/参照/上限エラーは正常行のみ登録し、失敗行を `csv_import_error` に記録してPARTIAL_FAILEDとする |

## 7. 認可チェック方針

- 画面表示制御はVaadin側で実施する。
- 業務処理の最終的な権限チェックはService層で実施する。
- 基本設計の7ロール定義に合わせ、テナント管理者、運用管理者、編集者、閲覧者、契約管理者、監査担当者を想定する。システム管理者はSaaS全体のテナント・契約確定処理のみを扱い、通常テナント内データの運用操作は行わない。

| 操作 | テナント管理者 | 運用管理者 | 編集者 | 閲覧者 | 契約管理者 | 監査担当者 |
|---|:---:|:---:|:---:|:---:|:---:|:---:|
| 参照 | ○ | ○ | ○ | ○ | △ | ○ |
| 登録 | ○ | ○ | ○ | × | × | × |
| 更新 | ○ | ○ | ○ | × | × | × |
| 削除 | ○ | ○ | × | × | × | × |
| CSVエクスポート | ○ | ○ | ○ | ○ | × | ○ |
| CSVインポート | ○ | ○ | ○ | × | × | × |
| Free期限超過中の参照 | ○ | ○ | ○ | ○ | ○ | ○ |
| Free期限超過中のCSVエクスポート | ○ | ○ | ○ | ○ | × | ○ |
| Free期限超過中の登録・更新・削除 | × | × | × | × | × | × |
| Free期限超過中の有料プラン変更 | ○ | × | × | × | ○ | × |
| ユーザー管理 | ○ | × | × | × | × | × |
| プラン管理 | ○ | × | × | × | ○ | × |
| 監査ログ閲覧 | ○ | × | × | × | × | ○ |

## 8. DTO方針

### Command DTO

登録・更新用。

例:

```java
public class DeviceCreateCommand {
    private String formalName;
    private String displayName;
    private DeviceType deviceType;
    private Long rackId;
    private Integer rackUnitStart;
    private Integer rackUnitSize;
}
```

### Query DTO

検索条件用。

```java
public class DeviceSearchQuery {
    private String keyword;
    private DeviceType deviceType;
    private Long dataCenterId;
    private Long rackId;
    private List<Long> tagIds;
}
```

### Result DTO

画面表示用。

```java
public class DeviceDetailDto {
    private Long deviceId;
    private String formalName;
    private String displayName;
    private String hostname;
    private String rackName;
    private String maintenanceStatus;
}
```

<!-- issue-fixes-220-231-232-233-234 -->

## 付録A. Issue対応追補: Service設計の補完

### A.1 CSVインポートの初期リリース無効化

CSVインポートは初期追加フェーズ対象であり、初期リリースでは以下の扱いに統一する。

| 層 | 初期リリース方針 |
|---|---|
| 画面 | メニュー非表示。直接URLアクセス時は利用不可画面または404相当 |
| API/Controller | 実装しない、またはFeature Flagで無効化し403/404を返す |
| 権限 | CSV_IMPORT権限は初期リリースでは付与しない |
| Service | 将来追加の設計メモに留め、業務導線から呼び出さない |
| DB | 履歴テーブルを先行作成する場合も、画面/APIからは利用不可 |

### A.2 LocationApplicationService

物理階層管理の責務を明確化するため、LocationApplicationServiceを追加する。

| メソッド例 | 責務 |
|---|---|
| `createBuilding`, `updateBuilding`, `disableBuilding` | DC配下の棟管理 |
| `createFloor`, `updateFloor`, `disableFloor` | 建物配下のフロア管理 |
| `createArea`, `updateArea`, `disableArea` | フロア配下の区画管理、グリッド位置管理 |
| `createRackRow`, `updateRackRow`, `disableRackRow` | 区画配下のラック列管理、方向・位置管理 |
| `searchLocations` | DC配下の物理階層検索 |

親子の `tenantId` 一致、親が有効であること、配下有効データが存在する場合の削除不可をService層で検証する。

### A.3 SearchApplicationService詳細

横断検索は以下を初期対象とする。

| 対象 | 検索キー | 結果項目 |
|---|---|---|
| DataCenter | 正式名称、表示名、別名、タグ、住所、リージョン | 種別、表示名、所在地、タグ、詳細URL |
| Rack | 正式名称、表示名、別名、タグ、ラック番号、設置場所 | 種別、表示名、DC/フロア/列、空きU、詳細URL |
| Device | 正式名称、表示名、別名、タグ、ホスト名、シリアル、IP | 種別、表示名、設置ラック、IP、保守状態、詳細URL |
| IpSubnet / IpAddress | CIDR、IPアドレス、用途、タグ | 種別、CIDR/IP、利用状態、関連機器、詳細URL |
| MaintenanceContract | 契約名、契約番号、ベンダー、タグ、期限状態 | 種別、契約名、期限、更新状態、詳細URL |

検索は必ず `tenantId`、論理削除、ロール別閲覧権限でフィルタリングし、ページングとソートを適用する。契約管理者には利用状況に必要な限定項目のみ返す。

### A.4 CSVインポートの行単位トランザクション

初期追加フェーズでCSVインポートを有効化する際は、以下の方式を採用する。

1. ファイル全体のヘッダ・文字コード・行数・必須列を事前検証する。
2. 行間依存があるデータは事前検証で参照関係を解決する。
3. 登録処理は行単位または小チャンク単位の `REQUIRES_NEW` 相当で実行し、正常行のみ確定する。
4. 失敗行は `csv_import_error` に行番号、項目名、エラーコード、メッセージ、元値を保存する。
5. ファイル全体の履歴には成功件数、失敗件数、ステータス、`correlation_id` を保存する。

### A.5 Service命名と責務分担

| 種別 | 命名 | 責務 |
|---|---|---|
| Application Service | `XxxApplicationService` | 画面/APIユースケース、トランザクション境界、認可、Repository呼び出し |
| Domain Service | `XxxDomainService` | Entity単体に閉じない業務ルール、プラン上限、重複判定、期限判定 |
| Infrastructure Service | `XxxClient` / `XxxGateway` | メール送信、外部連携、ファイル保存など |

例: `DeviceApplicationService` は機器登録ユースケースを担当し、ラックU重複判定の純粋な業務ルールは `RackPlacementDomainService` に分離できる。`PlanLimitDomainService` は上限判定に限定し、申請・画面表示は `SubscriptionApplicationService` が担当する。

<!-- issue-fixes-270-271-272-273-275 -->

## 付録B. Issue対応追補: Application Service詳細

### B.1 AuthApplicationService

| メソッド例 | 責務 | 主な例外 |
|---|---|---|
| `login(email, password)` | メールログイン、失敗回数更新、監査ログ依頼 | AuthenticationFailedException, TenantSuspendedException |
| `requestPasswordReset(email)` | 再設定トークン発行、通知依頼、ユーザー存在推測防止 | RateLimitExceededException |
| `resetPassword(token, newPassword)` | トークン検証、パスワード更新、使用済み化 | InvalidTokenException, TokenExpiredException |
| `acceptInvitation(token, password)` | 招待承諾、初回パスワード設定、ユーザー有効化 | InvalidInvitationException |

通知送信はNotificationApplicationServiceへ、操作履歴はAuditLogServiceへ委譲する。

### B.2 SubscriptionApplicationService / TenantApplicationService

| Service | メソッド例 | 責務 |
|---|---|---|
| SubscriptionApplicationService | `requestPlanChange`, `requestAddOnChange`, `approveRequest`, `rejectRequest`, `getUsageLimit` | 契約変更申請、利用量/上限取得、Free期限判定 |
| TenantApplicationService | `createTenant`, `updateTenant`, `suspendTenant`, `resumeTenant`, `cancelTenant`, `expireTrialTenants` | システム管理者向けテナント管理、状態変更、初期管理者保護 |
| AuthorizationApplicationService | `hasPermission`, `canAccessScreen`, `canOperateResource` | RolePermissionを正本にした認可判定 |

契約変更申請は `ContractChangeRequest` を正本とし、申請中の同種リクエスト重複を禁止する。承認/却下/取消は監査ログ対象とする。

### B.3 IPv4 / IPv6 IP管理

`IpSubnetService` と `IpAddressService` はIPバージョンごとに処理を分岐する。

| IPバージョン | 登録方式 | 上限カウント |
|---|---|---|
| IPv4 | CIDRから範囲生成可。必要に応じて明示登録も可 | 生成予定または明示登録した個別IP数 |
| IPv6 | 全範囲生成は禁止。利用・予約する個別IPのみ明示登録 | 明示登録したIPv6アドレス数のみ |

IPv6 CIDRに対して `generateAddresses` を実行してはならない。UI/APIでもIPv6の全生成導線は表示しない。

### B.4 期限切れ保守通知の7日ごと再通知

期限切れ保守契約は、期限切れ初回通知後、更新済み/終了/通知無効/削除になるまで7日ごとに再通知する。

| 項目 | 方針 |
|---|---|
| 初回期限切れ通知 | `endDate < currentDate` になった最初のバッチ日 |
| 再通知周期 | 前回期限切れ通知の `referenceDate` から7日以上経過 |
| 重複キー | tenantId, notificationType, maintenanceContractId, timing=EXPIRED, referenceDate, channel, recipient |
| 停止条件 | 更新済み、終了、通知無効、契約削除、対象者なし |
| 送信失敗 | FAILEDを記録し、次回バッチで再評価する |

通知ログ作成とメール送信は、契約単位または通知単位で失敗分離する。

### B.5 Region重複判定

Regionの一意条件は、同一テナント内の有効データで `regionName + prefecture` とする。`regionCode` は任意項目だが、入力される場合は同一テナント内で重複不可とする。

### B.6 AuditLogService対象イベント

AuditLogServiceは以下を初期必須イベントとして記録する。

| イベント | 例 |
|---|---|
| 認証 | ログイン成功/失敗、ログアウト、パスワード再設定依頼/完了 |
| ユーザー管理 | 招待、再招待、招待取消、権限変更、無効化 |
| CRUD | DC、ラック、機器、IP、保守契約、タグ、連絡先の登録/更新/削除 |
| CSV | エクスポート要求、ダウンロード、インポート実行/失敗 |
| 契約 | プラン変更申請、オプション変更申請、承認、却下、取消 |
| 通知 | 通知設定変更、通知送信失敗の運用イベント |
| システム管理 | テナント登録、編集、停止、再開、解約 |

共通Commandには `eventType`, `tenantId`, `actorUserId`, `targetType`, `targetId`, `result`, `requestId`, `detail` を含める。
