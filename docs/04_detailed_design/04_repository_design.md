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
  ├─ DataCenterRepository.java
  ├─ RackRepository.java
  ├─ DeviceRepository.java
  ├─ IpSubnetRepository.java
  ├─ IpAddressRepository.java
  ├─ MaintenanceContractRepository.java
  ├─ ContactRepository.java
  ├─ TagRepository.java
  ├─ ResourceAliasRepository.java
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
| AppUserRepository | app_user | ユーザー取得 |
| DataCenterRepository | data_center | DC検索 |
| RackRowRepository | rack_row | ラック列検索 |
| RackRepository | rack | ラック検索 |
| DeviceRepository | device | 機器検索 |
| IpSubnetRepository | ip_subnet | IPサブネット検索・上限件数確認 |
| IpAddressRepository | ip_address | IP利用状況検索 |
| MaintenanceContractRepository | maintenance_contract | 保守契約検索 |
| MaintenanceContractDeviceRepository | maintenance_contract_device | 保守契約・機器関連検索 |
| MaintenanceContractContactRepository | maintenance_contract_contact | 保守契約・連絡先関連検索 |
| ContactRepository | contact | 連絡先検索 |
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

## 6.1 DataCenterRepository

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

## 6.2 RackRepository

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

## 6.3 DeviceRepository

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

## 6.4 IpSubnetRepository / IpAddressRepository

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
```

プラン上限判定では `IpSubnetRepository.countByTenantIdAndDeletedFalse()` を利用し、個別IPアドレス数は上限カウントに含めない。

## 6.5 MaintenanceContractRepository

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
  and c.endDate between :fromDate and :toDate
""")
List<MaintenanceContract> findExpiringContracts(
    Long tenantId,
    LocalDate fromDate,
    LocalDate toDate
);
```

## 6.6 MaintenanceContractDeviceRepository

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

## 6.7 MaintenanceContractContactRepository

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

## 6.8 TagRepository

### 主なメソッド

```java
Optional<Tag> findByTagIdAndTenantIdAndDeletedFalse(Long tagId, Long tenantId);

Optional<Tag> findByTenantIdAndTagNameAndDeletedFalse(Long tenantId, String tagName);

boolean existsByTenantIdAndTagNameAndDeletedFalse(Long tenantId, String tagName);
```

## 6.9 TaggedResourceRepository

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

## 6.10 ResourceAliasRepository

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

## 6.11 NotificationLogRepository

### 主なメソッド

```java
boolean existsByTenantIdAndNotificationTypeAndTargetTypeAndTargetIdAndRecipientAndStatus(
    Long tenantId,
    NotificationType notificationType,
    TargetType targetType,
    Long targetId,
    String recipient,
    NotificationStatus status
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
