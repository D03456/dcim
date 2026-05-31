# 4. 詳細設計 - Value Object設計

## 1. 目的

本書は、本システムで利用するValue Object、Enum、ID型、期間・状態表現の設計方針を定義する。

## 2. 設計方針

- 業務上の意味を持ち、不正値による障害リスクが高い値は初期リリースからValue Object化する。
- 不正な値を生成時点で拒否する。
- JPA Entityへの保存時は、必要に応じて文字列・数値へ変換する。
- 画面DTOでは扱いやすさを優先し、Domain変換時に検証する。

## 2.1 初期必須 / 段階導入の区分

| 区分 | 対象 | 方針 |
|---|---|---|
| 初期必須 | `IpSubnetCidr`, `IpAddressValue`, `RackMountRange`, `RackHeightUnit`, `NotificationDaysBefore`, `EmailAddress`, `DateRange` | 形式・範囲・整合性の不正が重大なため初期から導入する |
| 初期推奨 | `FormalName`, `DisplayName`, `RackNumber`, `HostName`, `SerialNumber`, `TagName`, `PhoneNumber` | 入力仕様の一元化に有効。実装コストに応じて優先導入する |
| 段階導入 | `TenantId`, `UserId`, `DataCenterId`, `RackId`, `DeviceId` などID系 | 初期DBはbigint前提。外部公開IDやDomain分離を強める段階で導入する |
| Enum必須 | `TenantStatus`, `PlanType`, `DeviceLifecycleStatus`, `IpSubnetStatus`, `IpUsageStatus`, `NotificationType`, `NotificationStatus` | 文字列直書きを避け、初期からEnumで扱う |

## 3. ID系

| Value Object | 内容 |
|---|---|
| `TenantId` | テナントID |
| `UserId` | ユーザーID |
| `DataCenterId` | データセンターID |
| `RackId` | ラックID |
| `DeviceId` | 機器ID |
| `IpSubnetId` | IPサブネットID |
| `IpAddressId` | IPアドレスID |
| `MaintenanceContractId` | 保守契約ID |

初期実装ではDB上は `bigint` を前提とする。ID系Value Objectは初期必須ではなく、Domain層の分離を強める段階で導入する。

## 4. 名称・文字列系

| Value Object | 主な制約 |
|---|---|
| `FormalName` | 必須、最大150文字、前後trim |
| `DisplayName` | 任意、最大150文字 |
| `RackNumber` | 必須、最大50文字 |
| `HostName` | 任意、最大150文字 |
| `SerialNumber` | 任意、最大100文字 |
| `TagName` | 必須、最大50文字 |
| `EmailAddress` | メール形式、最大255文字 |
| `PhoneNumber` | 最大50文字 |

## 5. ネットワーク系

| Value Object | 主な制約 |
|---|---|
| `IpSubnetCidr` | CIDR形式、IPv4/IPv6判定 |
| `IpAddressValue` | IPv4/IPv6形式、所属サブネット範囲内 |
| `IpVersion` | IPV4 / IPV6 |

## 6. ラック配置系

| Value Object | 主な制約 |
|---|---|
| `RackUnitPosition` | 1以上 |
| `RackUnitSize` | 1以上 |
| `RackMountRange` | `start + size - 1 <= rack.heightUnit` |
| `RackHeightUnit` | 1〜60 |

## 7. 日付・期間系

| Value Object | 主な制約 |
|---|---|
| `DateRange` | 開始日 <= 終了日 |
| `NotificationDaysBefore` | 0〜365 |
| `TrialPeriod` | Freeは14日間 |
| `TrialEndDate` | Freeトライアル終了日 |

## 8. 状態系Enum

| Enum | 値 |
|---|---|
| `TenantStatus` | ACTIVE, SUSPENDED, TRIAL_EXPIRED, CANCELLED |
| `PlanType` | FREE, STARTER, BUSINESS, ENTERPRISE |
| `DeviceLifecycleStatus` | ACTIVE, SPARE, PLANNED_RETIREMENT, RETIRED |
| `IpSubnetStatus` | ACTIVE, RESERVED, RETIRED |
| `IpUsageStatus` | UNUSED, IN_USE, RESERVED, RETIRED |
| `NotificationStatus` | PENDING, SENT, FAILED, SKIPPED |
| `NotificationType` | MAINTENANCE_EXPIRY, PLAN_LIMIT, TRIAL_EXPIRY, PASSWORD_RESET, USER_INVITATION, OPERATION_ERROR, SYSTEM |

## 9. 実装注意

- Value Object内でDBアクセスしない。
- テナント分離や重複確認はService / Repositoryで行う。
- 画面入力値はCommand DTOで受け取り、Application ServiceでValue Objectへ変換する。
- Enum値はDB保存値、画面表示名、日本語ラベルの変換方針を分ける。

<!-- issue-fixes-216-217-218 -->

## 付録A. Issue対応追補: ラック関連Value Object

| Value Object | 責務 |
|---|---|
| `RackUnitPosition` | 下から1U基準の開始Uを表す |
| `RackUnitRange` | `startUnit` と `unitSize` から搭載範囲を表し、重複判定を行う |
| `RackMountItemType` | DEVICE / RESERVED_U / BLANK_PANEL を表す |
| `RackTemplateAppliedSnapshot` | テンプレート適用時点の予約U・標準構成をラック実体側へコピーした結果を表す |

ラック未搭載機器は `RackUnitRange` を持たない。搭載機器は `rackId` と `RackUnitRange` を必ず同時に持つ。

<!-- issue-fixes-262 -->

## 付録B. Issue対応追補: IP/CIDR正規化

`IpSubnetCidr` と `IpAddressValue` は保存前にcanonical形式へ正規化する。

| 対象 | 正規化方針 |
|---|---|
| IPv4アドレス | 先頭ゼロ等の表記揺れを排除し、ドット10進表記へ統一 |
| IPv6アドレス | RFC 5952相当の小文字・省略表記へ統一 |
| CIDR | ホスト部をネットワークアドレスへ正規化し、prefix長を明示 |
| 比較/検索 | 入力値を同じcanonical形式へ変換してから比較 |

DBの一意制約・Repository検索・CSV入出力ではcanonical値を正本とする。元入力文字列を保存する場合は表示補助に限定し、一意判定には使わない。
