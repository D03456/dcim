# 4. 詳細設計 - Repository設計

## 1. 目的

本書は、Data Center Asset & Infrastructure Manager のRepository層の設計方針、主要Repository、検索メソッドを定義する。

## 2. 基本方針

- Spring Data JPAを基本とする。
- Repositoryは永続化と検索に責務を限定する。
- 業務判断、プラン上限判定、権限判定はService層で行う。
- すべての業務検索は原則として `tenantId` と `deleted = false` を条件に含める。
- 複雑な検索は `Specification`、`QueryDSL`、またはカスタムRepositoryで実装する。
- 一覧検索は `Pageable` に対応する。

## 3. パッケージ構成案

```text
com.example.dcim.domain.repository
  ├─ TenantRepository.java
  ├─ SubscriptionPlanRepository.java
  ├─ TenantAddOnRepository.java
  ├─ AppUserRepository.java
  ├─ PasswordResetTokenRepository.java
  ├─ UserInvitationTokenRepository.java
  ├─ RoleRepository.java
  ├─ PermissionRepository.java
  ├─ RolePermissionRepository.java
  ├─ AuditLogRepository.java
  ├─ RegionRepository.java
  ├─ DataCenterRepository.java
  ├─ RackRepository.java
  ├─ DeviceRepository.java
  ├─ IpSubnetRepository.java
  ├─ IpAddressRepository.java
  ├─ MaintenanceContractRepository.java
  ├─ ContactRepository.java
  ├─ TagRepository.java
  ├─ TaggedResourceRepository.java
  ├─ ResourceAliasRepository.java
  ├─ NotificationSettingRepository.java
  ├─ NotificationLogRepository.java
  ├─ CsvExportHistoryRepository.java
  ├─ CsvImportHistoryRepository.java
  ├─ CsvImportErrorRepository.java
  └─ CloudResourceRepository.java（将来拡張）
```

## 4. 共通Repository方針

### 4.1 命名方針

| 種別 | 命名例 |
|---|---|
| ID検索 | findByIdAndTenantIdAndDeletedFalse |
| 一覧検索 | findByTenantIdAndDeletedFalse |
| 重複確認 | existsByTenantIdAndFormalNameAndDeletedFalse |
| 件数確認 | countByTenantIdAndDeletedFalse |

### 4.2 共通検索条件

```java
tenant_id = :tenantId
and deleted = false
```

### 4.3 Optional方針

- 単一検索は `Optional<Entity>` を返却する。
- Service層で存在しない場合に `ResourceNotFoundException` を送出する。

## 5. Repository一覧

| Repository | 対象テーブル | 主な用途 |
|---|---|---|
| TenantRepository | tenant | テナント取得 |
| SubscriptionPlanRepository | subscription_plan | プラン上限取得 |
| TenantAddOnRepository | tenant_add_on | 追加枠取得 |
| AppUserRepository | app_user | ユーザー取得・メール重複確認 |
| PasswordResetTokenRepository | password_reset_token | パスワード再設定トークン検索・使用済み更新 |
| UserInvitationTokenRepository | user_invitation_token | 招待トークン検索・承諾/取消更新 |
| RoleRepository | role | ロール取得 |
| PermissionRepository | permission | 権限取得 |
| RolePermissionRepository | role_permission | ロール権限取得 |
| AuditLogRepository | audit_log | 操作履歴保存・検索 |
| RegionRepository | region | リージョン検索 |
| DataCenterRepository | data_center | DC検索 |
| BuildingRepository | building | 棟検索 |
| FloorRepository | floor | フロア検索 |
| AreaRepository | area | 区画検索 |
| RackRowRepository | rack_row | ラック列検索 |
| RackRepository | rack | ラック検索 |
| RackTemplateRepository | rack_template | ラックテンプレート検索 |
| RackTemplateItemRepository | rack_template_item | ラックテンプレート明細検索 |
| DeviceRepository | device | 機器検索 |
| IpSubnetRepository | ip_subnet | IPサブネット検索・CIDR重複確認 |
| IpAddressRepository | ip_address | IP利用状況検索 |
| MaintenanceContractRepository | maintenance_contract | 保守契約検索 |
| MaintenanceContractDeviceRepository | maintenance_contract_device | 保守契約・機器関連検索 |
| MaintenanceContractContactRepository | maintenance_contract_contact | 保守契約・連絡先関連検索 |
| ContactRepository | contact | 連絡先検索 |
| DataCenterContactRepository | data_center_contact | DC・連絡先関連検索 |
| TagRepository | tag | タグ検索 |
| TaggedResourceRepository | tagged_resource | タグ関連検索 |
| ResourceAliasRepository | resource_alias | 呼称名・別名検索 |
| NotificationSettingRepository | notification_setting | 通知設定検索 |
| NotificationLogRepository | notification_log | 通知履歴検索 |
| CsvExportHistoryRepository | csv_export_history | CSV出力履歴検索 |
| CsvImportHistoryRepository | csv_import_history | CSV取込履歴検索 |
| CsvImportErrorRepository | csv_import_error | CSV取込エラー検索 |
| CloudAccountRepository | cloud_account | 将来拡張：クラウドアカウント検索 |
| CloudResourceRepository | cloud_resource | 将来拡張：クラウド資産検索 |

## 6. 主要Repository詳細

## 6.1 AppUserRepository

### 主なメソッド

```java
Optional<AppUser> findByUserIdAndTenantIdAndDeletedFalse(Long userId, Long tenantId);

Optional<AppUser> findByTenantIdAndEmailAndDeletedFalse(Long tenantId, String email);

boolean existsByTenantIdAndEmailAndDeletedFalse(Long tenantId, String email);

long countByTenantIdAndDeletedFalse(Long tenantId);
```

### 認証・権限・操作履歴Repository

```java
Optional<PasswordResetToken> findByTokenHashAndDeletedFalse(String tokenHash);

Optional<UserInvitationToken> findByTokenHashAndDeletedFalse(String tokenHash);

List<Role> findByRoleCodeIn(Collection<String> roleCodes);

List<Permission> findByPermissionCodeIn(Collection<String> permissionCodes);

List<RolePermission> findByRoleIdIn(Collection<Long> roleIds);

AuditLog save(AuditLog auditLog);

Page<AuditLog> search(AuditLogSearchCriteria criteria, Pageable pageable);
```

## 6.2 RegionRepository

### 主なメソッド

```java
Optional<Region> findByRegionIdAndTenantIdAndDeletedFalse(Long regionId, Long tenantId);

List<Region> findByTenantIdAndStatusAndDeletedFalseOrderByDisplayOrderAsc(
    Long tenantId,
    RegionStatus status
);

boolean existsByTenantIdAndRegionNameAndDeletedFalse(Long tenantId, String regionName);
```

## 6.3 DataCenterRepository

### 主なメソッド

```java
Optional<DataCenter> findByDataCenterIdAndTenantIdAndDeletedFalse(Long dataCenterId, Long tenantId);

Page<DataCenter> findByTenantIdAndDeletedFalse(Long tenantId, Pageable pageable);

boolean existsByTenantIdAndFormalNameAndDeletedFalse(Long tenantId, String formalName);

long countByTenantIdAndDeletedFalse(Long tenantId);
```

### カスタム検索条件

| 条件 | 内容 |
|---|---|
| keyword | 正式名称、通称、住所を部分一致検索 |
| regionId | 地域指定 |
| status | 状態指定 |
| tagIds | タグ指定 |

## 6.4 RackRepository

### 主なメソッド

```java
Optional<Rack> findByRackIdAndTenantIdAndDeletedFalse(Long rackId, Long tenantId);

Page<Rack> findByTenantIdAndDeletedFalse(Long tenantId, Pageable pageable);

List<Rack> findByTenantIdAndRackRowIdAndDeletedFalse(Long tenantId, Long rackRowId);

boolean existsByTenantIdAndRackRowIdAndRackNumberAndDeletedFalse(
    Long tenantId,
    Long rackRowId,
    String rackNumber
);
```

### ラック配置チェック用検索

- 対象ラック内の機器一覧を取得する。
- `DeviceRepository.findByTenantIdAndRackIdAndDeletedFalse()` を利用する。

## 6.5 DeviceRepository

### 主なメソッド

```java
Optional<Device> findByDeviceIdAndTenantIdAndDeletedFalse(Long deviceId, Long tenantId);

Page<Device> findByTenantIdAndDeletedFalse(Long tenantId, Pageable pageable);

List<Device> findByTenantIdAndRackIdAndDeletedFalse(Long tenantId, Long rackId);

boolean existsByTenantIdAndFormalNameAndDeletedFalse(Long tenantId, String formalName);

boolean existsByTenantIdAndHostnameAndDeletedFalse(Long tenantId, String hostname);

long countByTenantIdAndDeletedFalse(Long tenantId);
```

### 保守未契約機器検索

```sql
select d.*
from device d
left join maintenance_contract_device mcd
  on d.device_id = mcd.device_id
 and mcd.deleted = false
where d.tenant_id = :tenantId
  and d.deleted = false
  and mcd.device_id is null
```

### 検索条件

| 条件 | 内容 |
|---|---|
| keyword | 正式名称、通称、ホスト名、シリアル番号 |
| deviceType | 機器種別 |
| lifecycleStatus | ライフサイクル状態 |
| dataCenterId | DC配下検索 |
| rackId | ラック指定 |
| hasMaintenance | 保守有無 |
| tagIds | タグ指定 |

## 6.6 IpSubnetRepository / IpAddressRepository

### IpSubnetRepository 主なメソッド

```java
Optional<IpSubnet> findByIpSubnetIdAndTenantIdAndDeletedFalse(Long ipSubnetId, Long tenantId);

Optional<IpSubnet> findByTenantIdAndCidrAndDeletedFalse(Long tenantId, String cidr);

Page<IpSubnet> findByTenantIdAndDeletedFalse(Long tenantId, Pageable pageable);

boolean existsByTenantIdAndCidrAndDeletedFalse(Long tenantId, String cidr);

long countByTenantIdAndDeletedFalse(Long tenantId);
```

### IpAddressRepository 主なメソッド

```java
Optional<IpAddress> findByIpAddressIdAndTenantIdAndDeletedFalse(Long ipAddressId, Long tenantId);

Optional<IpAddress> findByTenantIdAndIpAddressAndDeletedFalse(Long tenantId, String ipAddress);

Page<IpAddress> findByTenantIdAndIpSubnetIdAndDeletedFalse(Long tenantId, Long ipSubnetId, Pageable pageable);

List<IpAddress> findByTenantIdAndDeviceIdAndDeletedFalse(Long tenantId, Long deviceId);

boolean existsByTenantIdAndIpSubnetIdAndIpAddressAndDeletedFalse(Long tenantId, Long ipSubnetId, String ipAddress);

long countByTenantIdAndDeletedFalse(Long tenantId);
```

プラン上限判定では `IpAddressRepository.countByTenantIdAndDeletedFalse()` など管理対象の個別IPアドレス数を利用する。IPサブネット数は上限なしのため、`IpSubnetRepository.countByTenantIdAndDeletedFalse()` はプラン上限判定に利用しない。

## 6.7 MaintenanceContractRepository

### 主なメソッド

```java
Optional<MaintenanceContract> findByMaintenanceContractIdAndTenantIdAndDeletedFalse(
    Long maintenanceContractId,
    Long tenantId
);

Page<MaintenanceContract> findByTenantIdAndDeletedFalse(Long tenantId, Pageable pageable);

List<MaintenanceContract> findByTenantIdAndNotificationEnabledTrueAndDeletedFalse(Long tenantId);
```

### 期限到来検索

```java
@Query("""
select c from MaintenanceContract c
where c.tenantId = :tenantId
  and c.deleted = false
  and c.notificationEnabled = true
  and (
      (c.endDate = :targetDate)
      or (:includeExpired = true and c.endDate < :today)
  )
""")
List<MaintenanceContract> findExpiringContracts(
    Long tenantId,
    LocalDate targetDate,
    LocalDate today,
    boolean includeExpired
);
```

## 6.8 MaintenanceContractDeviceRepository

### 主なメソッド

```java
boolean existsByTenantIdAndMaintenanceContractIdAndDeviceIdAndDeletedFalse(
    Long tenantId,
    Long maintenanceContractId,
    Long deviceId
);

List<MaintenanceContractDevice> findByTenantIdAndDeviceIdAndDeletedFalse(
    Long tenantId,
    Long deviceId
);

List<MaintenanceContractDevice> findByTenantIdAndMaintenanceContractIdAndDeletedFalse(
    Long tenantId,
    Long maintenanceContractId
);
```

## 6.9 MaintenanceContractContactRepository

### 主なメソッド

```java
boolean existsByTenantIdAndMaintenanceContractIdAndContactIdAndDeletedFalse(
    Long tenantId,
    Long maintenanceContractId,
    Long contactId
);

List<MaintenanceContractContact> findByTenantIdAndMaintenanceContractIdAndDeletedFalse(
    Long tenantId,
    Long maintenanceContractId
);

List<MaintenanceContractContact> findByTenantIdAndContactIdAndDeletedFalse(
    Long tenantId,
    Long contactId
);
```

## 6.10 ContactRepository

### 主なメソッド

```java
Optional<Contact> findByContactIdAndTenantIdAndDeletedFalse(Long contactId, Long tenantId);

Page<Contact> findByTenantIdAndDeletedFalse(Long tenantId, Pageable pageable);

Page<Contact> findByTenantIdAndContactTypeAndDeletedFalse(
    Long tenantId,
    ContactType contactType,
    Pageable pageable
);

List<Contact> findByTenantIdAndContactIdInAndDeletedFalse(Long tenantId, List<Long> contactIds);
```

## 6.11 TagRepository

### 主なメソッド

```java
Optional<Tag> findByTagIdAndTenantIdAndDeletedFalse(Long tagId, Long tenantId);

Optional<Tag> findByTenantIdAndTagNameAndDeletedFalse(Long tenantId, String tagName);

boolean existsByTenantIdAndTagNameAndDeletedFalse(Long tenantId, String tagName);
```

## 6.12 TaggedResourceRepository

### 主なメソッド

```java
List<TaggedResource> findByTenantIdAndResourceTypeAndResourceIdAndDeletedFalse(
    Long tenantId,
    ResourceType resourceType,
    Long resourceId
);

boolean existsByTenantIdAndTagIdAndResourceTypeAndResourceIdAndDeletedFalse(
    Long tenantId,
    Long tagId,
    ResourceType resourceType,
    Long resourceId
);
```

## 6.13 ResourceAliasRepository

### 主なメソッド

```java
List<ResourceAlias> findByTenantIdAndResourceTypeAndResourceIdAndDeletedFalse(
    Long tenantId,
    ResourceType resourceType,
    Long resourceId
);

Page<ResourceAlias> findByTenantIdAndAliasNameContainingAndDeletedFalse(
    Long tenantId,
    String aliasName,
    Pageable pageable
);

boolean existsByTenantIdAndResourceTypeAndResourceIdAndAliasNameAndDeletedFalse(
    Long tenantId,
    ResourceType resourceType,
    Long resourceId,
    String aliasName
);
```

## 6.14 NotificationSettingRepository

### 主なメソッド

```java
Optional<NotificationSetting> findByTenantIdAndNotificationTypeAndTimingCodeAndDeletedFalse(
    Long tenantId,
    NotificationType notificationType,
    String timingCode
);

List<NotificationSetting> findByTenantIdAndDeletedFalse(Long tenantId);
```

## 6.15 NotificationLogRepository

### 主なメソッド

```java
boolean existsByTenantIdAndNotificationTypeAndTimingCodeAndNotificationLevelAndChannelAndTargetTypeAndTargetIdAndRecipientAndStatus(
    Long tenantId,
    NotificationType notificationType,
    String timingCode,
    String notificationLevel,
    NotificationChannel channel,
    TargetType targetType,
    Long targetId,
    String recipient,
    NotificationStatus status
);

boolean existsByTenantIdAndNotificationTypeAndTimingCodeAndNotificationLevelAndChannelAndTargetTypeAndTargetIdAndRecipientUserIdAndStatus(
    Long tenantId,
    NotificationType notificationType,
    String timingCode,
    String notificationLevel,
    NotificationChannel channel,
    TargetType targetType,
    Long targetId,
    Long recipientUserId,
    NotificationStatus status
);

List<NotificationLog> findByTenantIdAndRecipientUserIdAndChannelAndReadAtIsNullAndDeletedFalse(
    Long tenantId,
    Long recipientUserId,
    NotificationChannel channel
);

List<NotificationLog> findByTenantIdAndStatusAndDeletedFalse(
    Long tenantId,
    NotificationStatus status
);
```

## 7. 複雑検索の実装方針

### 7.1 Specification利用案

以下のような検索は `Specification` を利用する。

- 機器一覧検索
- IPサブネット一覧検索
- IPアドレス利用状況一覧検索
- 保守契約一覧検索
- タグを含む横断検索

### 7.2 カスタムRepository利用案

以下のような検索はカスタムRepositoryを検討する。

- DC → 棟 → フロア → 区画 → ラック → 機器 の階層検索
- 保守未契約機器検索
- 保守期限間近機器検索
- 複数タグAND/OR検索

## 8. ページング・ソート方針

| 項目 | 方針 |
|---|---|
| デフォルトページサイズ | 20件 |
| 最大ページサイズ | 100件 |
| デフォルトソート | updated_at desc |
| 名称ソート | formal_name asc |

## 9. Repositoryで実施しないこと

- 権限判定
- プラン上限判定
- 通知送信
- 入力DTOのバリデーション
- 画面表示用メッセージ生成

## 10. テスト方針

- Repository単体は `@DataJpaTest` を利用する。
- テナント分離条件が必ず効いていることを確認する。
- 論理削除データが検索されないことを確認する。
- 一意制約に関するテストを実施する。


## 11. CSV関連Repository

```java
Page<CsvExportHistory> findByTenantIdAndDeletedFalse(Long tenantId, Pageable pageable);

Page<CsvImportHistory> findByTenantIdAndDeletedFalse(Long tenantId, Pageable pageable);

List<CsvImportError> findByCsvImportHistoryId(Long csvImportHistoryId);
```

CSV履歴は監査・問い合わせ対応に利用するため、原則として物理削除しない。

## 12. 将来拡張Repository

`CloudAccountRepository`、`CloudResourceRepository` はクラウド資産管理の将来拡張用であり、初期リリースでは実装対象外とする。


### CSVインポート同時実行制御

`CsvImportHistoryRepository` は `tenant_id + target_type + process_key + status(RUNNING/PENDING)` で処理中データを確認し、同一テナント・同一対象種別の同時取込を拒否する。実装では悲観ロックまたは一意制約を併用し、画面二重送信や複数利用者の同時実行でも一方だけが開始できるようにする。


### 6.x 競合更新用Repository

プラン上限、ラックU配置、IP割当の同時更新を防ぐため、以下のRepositoryメソッドを追加する。

```java
@Lock(PESSIMISTIC_WRITE)
Optional<Tenant> findByTenantIdForUpdate(Long tenantId);

@Lock(PESSIMISTIC_WRITE)
Optional<Rack> findByRackIdAndTenantIdForUpdate(Long rackId, Long tenantId);

boolean existsOverlappingRackMount(Long tenantId, Long rackId, Integer startUnit, Integer endUnit);

@Lock(PESSIMISTIC_WRITE)
Optional<IpSubnet> findByIpSubnetIdAndTenantIdForUpdate(Long ipSubnetId, Long tenantId);
```

DB制約とRepositoryロックを併用し、Service層の事前チェックだけに依存しない。

<!-- issue-fixes-219-221-222 -->

## 付録A. Issue対応追補: Repository境界の補完

### A.1 CSV履歴Repositoryとdeleted条件

CSV履歴テーブルに `deleted` を持たせる場合のみ `findByTenantIdAndDeletedFalse` を使用する。履歴系として `deleted` を持たせない場合は、Repositoryメソッドを `findByTenantId` / `findByTenantIdAndStatus` に統一する。Table定義とRepositoryメソッド名の不一致を禁止する。

### A.2 CsvImportErrorRepository

`csv_import_error` は親履歴のテナント境界を確認してから取得する。

| メソッド | 方針 |
|---|---|
| `findByHistoryForTenant(tenantId, historyId)` | 親履歴を `tenantId` 付きで検証してから明細を返す |
| `findByCsvImportHistoryId(historyId)` | Service外へ公開しない内部用途に限定 |

### A.3 TaggedResourceRepository

タグ付与はDB FKで対象リソースを表現できないため、Service層で `resourceType` ごとのRepositoryを使って存在確認・同一テナント確認・削除済みでないことを検証する。Repositoryでは同一対象への重複付与を防ぐため、`tenantId, resourceType, resourceId, tagId` の有効データ一意制約を前提にする。

<!-- issue-fixes-258-261-263 -->

## 付録B. Issue対応追補: 契約・保守・追加枠Repository

### B.1 ContractChangeRequestRepository

| メソッド例 | 用途 |
|---|---|
| `findByTenantIdAndStatusIn(tenantId, statuses)` | 申請中/承認待ち一覧 |
| `existsPendingRequest(tenantId, requestType, addOnType)` | 同種の未完了申請重複チェック |
| `findByRequestedBy(userId)` | 申請者別検索 |
| `findApprovedRequests(tenantId, from, to)` | 承認済み申請の履歴検索 |

### B.2 TenantAddOnRepository

| メソッド例 | 用途 |
|---|---|
| `findActiveByTenantIdAndDate(tenantId, baseDate)` | 基準日時点で有効な追加枠取得 |
| `sumQuantityUnitByTenantIdAndType(tenantId, addOnType, baseDate)` | 上限計算用の追加単位合算 |
| `findByTenantIdAndAddOnType(tenantId, addOnType)` | 追加枠履歴検索 |

### B.3 MaintenanceContractRepository契約番号チェック

| メソッド例 | 用途 |
|---|---|
| `existsByTenantIdAndContractNumberIncludingDeleted(tenantId, contractNumber)` | 削除済みを含む契約番号重複チェック |
| `findByTenantIdAndContractNumber(tenantId, contractNumber)` | 契約番号検索 |

`contractNumber` がNULLまたは空の場合は重複チェック対象外とする。入力値はtrim後の値で比較する。

<!-- issue-fixes-303-304-305 -->

## 付録C. Issue対応追補: 関連Repository・認証トークン

### C.1 DataCenterContactRepository

| メソッド例 | 用途 |
|---|---|
| `existsActiveByTenantIdAndDataCenterIdAndContactIdAndRole(...)` | 同一役割の重複関連チェック |
| `findByTenantIdAndDataCenterIdAndDeletedFalse(...)` | DC配下連絡先一覧 |
| `findByTenantIdAndContactIdAndDeletedFalse(...)` | 連絡先利用先確認 |

### C.2 MaintenanceContractDeviceRepository

同一契約・同一機器の有効関連重複を禁止するため、`existsActiveByTenantIdAndMaintenanceContractIdAndDeviceId` を用意する。解除時は関連行を論理削除/無効化し、履歴確認できるようにする。

### C.3 PasswordResetTokenRepository

システム管理者の `tenant_id = NULL` に対応するため、パスワード再設定トークンは `tokenHash`、`userId`、期限、ステータスで検証する。tenantId条件は通常ユーザーの補助条件として扱い、必須検索条件にしない。
