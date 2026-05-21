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
| LocationService | 棟、フロア、区画管理 |
| RackService | ラック列、ラック管理 |
| DeviceService | 機器管理 |
| IpSubnetService | IPサブネット管理、サブネット上限判定 |
| IpAddressService | サブネット配下のIPアドレス利用状況管理 |
| MaintenanceContractService | 保守契約管理 |
| ContactService | 連絡先管理 |
| TagService | タグ管理 |
| NotificationService | 通知作成・送信 |
| MaintenanceNotificationService | 保守期限通知処理 |
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
| validateCanCreateIpSubnet(tenantId) | IPサブネット追加可否を検証 |
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

## 5.2 DataCenterService

### 責務

- データセンター登録
- データセンター更新
- データセンター削除
- DC連絡先紐付け
- タグ付与

### 主なメソッド

| メソッド | 概要 |
|---|---|
| create(command) | DC登録 |
| update(dataCenterId, command) | DC更新 |
| delete(dataCenterId) | DC論理削除 |
| findById(dataCenterId) | DC詳細取得 |
| search(query) | DC検索 |
| assignContact(dataCenterId, contactId) | 連絡先紐付け |

### 登録処理

1. ログインユーザーのテナントIDを取得する。
2. `PlanLimitService.validateCanCreateDataCenter()` を実行する。
3. 正式名称の重複を確認する。
4. 入力値を検証する。
5. `DataCenter` を保存する。
6. 必要に応じてタグ、連絡先を関連付ける。

## 5.3 RackService

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

## 5.4 DeviceService

### 責務

- 機器登録・更新・削除
- ラック配置
- 機器種別管理
- 保守契約との関連確認
- 保守未契約機器の検索

### 主なメソッド

| メソッド | 概要 |
|---|---|
| create(command) | 機器登録 |
| update(deviceId, command) | 機器更新 |
| delete(deviceId) | 機器論理削除 |
| assignRack(deviceId, rackId, rackUnitStart, rackUnitSize) | ラック配置 |
| assignIpAddress(deviceId, ipAddressId) | IP割当 |
| findWithoutMaintenance(query) | 保守未紐付け機器検索 |
| search(query) | 機器検索 |

### 登録処理

1. テナントIDを取得する。
2. `PlanLimitService.validateCanCreateDevice()` を実行する。
3. 正式名称・ホスト名・シリアル番号の重複を確認する。
4. ラック指定がある場合、ラック存在確認とU位置重複チェックを行う。
5. 機器を保存する。
6. タグ指定がある場合は `TagService` で関連付ける。

## 5.5 IpSubnetService / IpAddressService

### IpSubnetServiceの責務

- IPサブネット登録・更新・削除
- サブネット上限チェック
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
| assignToDevice(ipAddressId, deviceId) | 機器へ割当 |
| release(ipAddressId) | 割当解除 |
| searchSubnets(query) | IPサブネット検索 |
| searchAddresses(query) | IPアドレス検索 |

### IP管理ルール

- プラン上限は個別IP数ではなく、`ip_subnet` の有効件数で判定する。
- 同一テナント内でCIDRは重複不可とする。
- 個別IPは必ず1つのIPサブネットに属する。
- `UNUSED` または `RESERVED` のIPのみ機器へ割当可能。
- 割当後は `IN_USE` とする。
- 解除後は原則 `UNUSED` とする。

## 5.6 MaintenanceContractService

### 責務

- 保守契約登録・更新・削除
- 保守契約と機器の紐付け
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
| findExpiringContracts(daysBefore) | 期限到来契約検索 |
| findDevicesWithoutMaintenance(query) | 保守未契約機器検索 |

### 保守期限判定

```text
通知対象日 = 契約終了日 - notification_days_before
現在日付 >= 通知対象日 かつ 契約終了日 >= 現在日付
```

標準は60日前通知とする。

## 5.7 NotificationService

### 責務

- 通知メッセージ生成
- 通知送信
- 通知ログ保存
- 重複通知抑止

### 主なメソッド

| メソッド | 概要 |
|---|---|
| createNotification(command) | 通知作成 |
| send(notificationId) | 通知送信 |
| sendMaintenanceExpiryNotification(contractId) | 保守期限通知送信 |
| hasAlreadySent(type, targetId, recipient) | 重複通知確認 |

## 5.8 MaintenanceNotificationService

### 責務

- 定期実行により保守期限が近い契約を抽出する。
- 通知対象の連絡先を決定する。
- 通知送信を依頼する。

### スケジュール案

| 項目 | 内容 |
|---|---|
| 実行頻度 | 1日1回 |
| 実行時刻 | 09:00 |
| 対象 | notification_enabled = true の保守契約 |
| 標準通知 | 60日前 |

## 5.9 TagService

### 責務

- タグ登録・更新・削除
- タグ登録前の `PlanLimitService.validateCanCreateTag()` 実行
- 各リソースへのタグ付与・解除
- タグによる検索条件提供

### 対象リソース

- DataCenter
- Rack
- Device
- IpSubnet
- IpAddress
- MaintenanceContract
- CloudResource（将来拡張）

### タグ上限

タグ数のプラン上限はタグマスタ件数で判定し、リソースへのタグ付け件数は上限対象外とする。

## 5.10 CloudResourceService（将来拡張）

### 責務

- クラウドアカウント管理
- EC2 / コンテナ / EKS Pod等の手動登録
- 将来の外部連携取込に備えたデータ管理

初期リリースでは画面・業務処理の対象外とし、設計上の拡張ポイントとして保持する。

## 5.11 CsvExportService / CsvImportService

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
- 基本設計のロール定義に合わせ、テナント管理者、運用者、閲覧者、契約管理者、監査閲覧者を想定する。

| 操作 | テナント管理者 | 運用者 | 閲覧者 | 契約管理者 | 監査閲覧者 |
|---|:---:|:---:|:---:|:---:|:---:|
| 参照 | ○ | ○ | ○ | △ | ○ |
| 登録 | ○ | ○ | × | × | × |
| 更新 | ○ | ○ | × | × | × |
| 削除 | ○ | ○ | × | × | × |
| CSVエクスポート | ○ | ○ | ○ | △ | ○ |
| CSVインポート | ○ | ○ | × | × | × |
| Free期限超過中の参照 | ○ | ○ | ○ | ○ | ○ |
| Free期限超過中のCSVエクスポート | ○ | ○ | ○ | △ | ○ |
| Free期限超過中の登録・更新・削除 | × | × | × | × | × |
| Free期限超過中の有料プラン変更 | ○ | × | × | ○ | × |
| ユーザー管理 | ○ | × | × | × | × |
| プラン管理 | ○ | × | × | ○ | × |
| 監査ログ閲覧 | ○ | × | × | × | ○ |

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
