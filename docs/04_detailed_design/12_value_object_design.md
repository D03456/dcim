# 4. 詳細設計 - Value Object設計

## 1. 目的

本書は、本システムで利用するValue Object、Enum、ID型、期間・状態表現の設計方針を定義する。

## 2. 設計方針

- 業務上の意味を持つ値はプリミティブ型のまま扱わず、Value Object化を検討する。
- 不正な値を生成時点で拒否する。
- JPA Entityへの保存時は、必要に応じて文字列・数値へ変換する。
- 画面DTOでは扱いやすさを優先し、Domain変換時に検証する。

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

初期実装ではDB上は `bigint` を前提とする。Domain層でID Value Objectを導入するかは実装コストと相談して段階導入可能とする。

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
| `NotificationType` | MAINTENANCE_EXPIRY, PLAN_LIMIT, TRIAL_EXPIRY, SYSTEM |

## 9. 実装注意

- Value Object内でDBアクセスしない。
- テナント分離や重複確認はService / Repositoryで行う。
- 画面入力値はCommand DTOで受け取り、Application ServiceでValue Objectへ変換する。
- Enum値はDB保存値、画面表示名、日本語ラベルの変換方針を分ける。
