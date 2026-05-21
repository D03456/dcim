# 4. 詳細設計 - バリデーション仕様

## 1. 目的

本書は、Data Center Asset & Infrastructure Manager における入力チェック、業務チェック、整合性チェックの仕様を定義する。

## 2. バリデーション分類

| 分類 | 実施場所 | 内容 |
|---|---|---|
| 入力チェック | DTO / Vaadin Form | 必須、桁数、形式、数値範囲 |
| 業務チェック | Service | プラン上限、重複、状態遷移、権限 |
| 整合性チェック | Service / Repository | 親子関係、参照存在、ラックU重複 |
| DB制約 | DB | 一意制約、NOT NULL、外部キー |

## 3. 共通バリデーション

### 3.1 文字列

| 項目 | ルール |
|---|---|
| 前後空白 | 登録前にtrimする |
| 空文字 | null扱い、または必須エラー |
| 最大長 | DB定義に合わせる |
| 禁止文字 | 制御文字は不可 |

### 3.2 ID

| 項目 | ルール |
|---|---|
| Long ID | null不可の場合、正の値のみ許可 |
| 参照ID | 同一テナント内に存在すること |
| 他テナントID | 指定されても参照不可 |

### 3.3 日付

| 項目 | ルール |
|---|---|
| 開始日 | 必須項目では現在日以前・以後の制限を業務ごとに定義 |
| 終了日 | 開始日以降であること |
| 通知日数 | 0以上365以下 |

## 4. プラン上限チェック

### 4.1 対象

| 対象 | 上限 |
|---|---:|
| データセンター | プラン別 |
| ラック | プラン別 |
| 機器 | プラン別 + 100台単位追加 |
| IPサブネット | プラン別 + 10サブネット単位追加 |
| ユーザー | プラン別 |

### 4.2 プラン初期上限

| プラン | 利用期間 | DC | ラック | 機器 | サブネット | ユーザー |
|---|---:|---:|---:|---:|---:|---:|
| Free | 14日間 | 1 | 3 | 20 | 3 | 1 |
| Starter | 契約期間中 | 2 | 5 | 50 | 10 | 3 |
| Business | 契約期間中 | 5 | 50 | 100 | 50 | 10 |
| Enterprise | 契約期間中 | 10 | 100 | 1000 | 200 | 30 |

### 4.3 エラー条件

```text
現在登録数 >= プラン上限 + 追加枠
```

上限超過時は `PlanLimitExceededException` を送出する。

## 5. エンティティ別バリデーション

## 5.1 DataCenter

| 項目 | 必須 | ルール |
|---|:---:|---|
| formalName | ○ | 1〜150文字、同一テナント内で重複不可 |
| displayName | - | 150文字以内 |
| regionId | - | 指定時は同一テナント内に存在 |
| address | - | 255文字以内 |
| status | ○ | ACTIVE / INACTIVE |

### 業務チェック

- DC登録時にプラン上限を超えないこと。
- 削除時、配下に有効な棟・ラック・機器がある場合は削除不可、または確認付き論理削除とする。

## 5.2 Building

| 項目 | 必須 | ルール |
|---|:---:|---|
| dataCenterId | ○ | 同一テナント内に存在 |
| formalName | ○ | 1〜150文字 |
| displayName | - | 150文字以内 |

### 業務チェック

- 同一DC内で正式名称が重複しないこと。

## 5.3 Floor

| 項目 | 必須 | ルール |
|---|:---:|---|
| buildingId | ○ | 同一テナント内に存在 |
| floorName | ○ | 1〜100文字 |
| floorNumber | - | 整数または文字列表現 |

## 5.4 Area

| 項目 | 必須 | ルール |
|---|:---:|---|
| floorId | ○ | 同一テナント内に存在 |
| areaName | ○ | 1〜100文字 |
| direction | - | EAST / WEST / SOUTH / NORTH / OTHER |

## 5.5 RackRow

| 項目 | 必須 | ルール |
|---|:---:|---|
| areaId | ○ | 同一テナント内に存在 |
| rowName | ○ | 1〜100文字 |

### 業務チェック

- ラック登録時にラック上限を超えないこと。ラック列は物理階層として管理する。

## 5.6 Rack

| 項目 | 必須 | ルール |
|---|:---:|---|
| rackRowId | ○ | 同一テナント内に存在 |
| formalName | ○ | 1〜150文字 |
| displayName | - | 150文字以内 |
| rackNumber | ○ | 1〜50文字 |
| heightUnit | ○ | 1〜60 |

### 業務チェック

- 同一ラック列内でラック番号が重複しないこと。
- ラック高さ変更時、既存機器の搭載Uを下回らないこと。

## 5.7 Device

| 項目 | 必須 | ルール |
|---|:---:|---|
| formalName | ○ | 1〜150文字、同一テナント内で重複不可 |
| displayName | - | 150文字以内 |
| deviceType | ○ | 定義済み種別のみ |
| hostname | - | 150文字以内、指定時は同一テナント内で重複不可 |
| serialNumber | - | 100文字以内 |
| manufacturer | - | 100文字以内 |
| modelName | - | 100文字以内 |
| rackId | - | 指定時は同一テナント内に存在 |
| rackUnitStart | - | 1以上 |
| rackUnitSize | - | 1以上 |

### ラック搭載チェック

| チェック | 内容 |
|---|---|
| 開始U | 1以上 |
| 使用U数 | 1以上 |
| 上限 | `rackUnitStart + rackUnitSize - 1 <= rack.heightUnit` |
| 重複 | 同一ラック内の既存機器とU範囲が重複しない |

### 業務チェック

- 登録時に機器上限を超えないこと。
- 廃止済み機器には新規IP割当不可。
- 削除時、保守契約やIP割当がある場合は確認または解除を求める。

## 5.8 IpSubnet / IpAddress

### IpSubnet

| 項目 | 必須 | ルール |
|---|:---:|---|
| subnetName | ○ | 1〜150文字 |
| cidr | ○ | CIDR形式。IPv4/IPv6形式と一致 |
| ipVersion | ○ | CIDR形式と一致 |
| status | ○ | ACTIVE / RESERVED / RETIRED |
| description | - | 255文字以内 |

### IpAddress

| 項目 | 必須 | ルール |
|---|:---:|---|
| ipSubnetId | ○ | 同一テナント内に存在 |
| ipAddress | ○ | サブネット範囲内のIPv4またはIPv6形式 |
| ipVersion | ○ | IPアドレス形式と一致 |
| deviceId | - | 指定時は同一テナント内に存在 |
| usageStatus | ○ | UNUSED / IN_USE / RESERVED / RETIRED |
| description | - | 255文字以内 |

### 業務チェック

- 同一テナント内でCIDR重複不可。
- 登録時にIPサブネット上限を超えないこと。
- 個別IPは所属サブネット範囲内であること。
- deviceId指定時は `usageStatus = IN_USE` と整合すること。
- `RETIRED` のIPは機器へ割当不可。

## 5.9 MaintenanceContract

| 項目 | 必須 | ルール |
|---|:---:|---|
| contractName | ○ | 1〜150文字 |
| vendorName | ○ | 1〜150文字 |
| contractNumber | - | 100文字以内 |
| startDate | ○ | 日付形式 |
| endDate | ○ | startDate以降 |
| notificationEnabled | ○ | true / false |
| notificationDaysBefore | ○ | 0〜365、標準60 |

### 業務チェック

- 保守契約に紐付ける機器は同一テナント内に存在すること。
- 同じ保守契約に同一機器を重複紐付けしないこと。
- 終了日が過去でも登録は可能。ただし状態表示は期限切れとする。

## 5.10 Contact

| 項目 | 必須 | ルール |
|---|:---:|---|
| name | ○ | 1〜150文字 |
| email | - | メール形式、255文字以内 |
| phoneNumber | - | 50文字以内 |
| contactType | ○ | DC / VENDOR / INTERNAL |

### 業務チェック

- emailまたはphoneNumberのどちらか一方は入力推奨。
- 通知先に利用する場合はemail必須。

## 5.11 Tag

| 項目 | 必須 | ルール |
|---|:---:|---|
| tagName | ○ | 1〜50文字、同一テナント内で重複不可 |
| colorCode | - | `#RRGGBB` 形式 |

## 5.12 CloudResource（将来拡張）

| 項目 | 必須 | ルール |
|---|:---:|---|
| cloudAccountId | ○ | 同一テナント内に存在 |
| provider | ○ | AWS等、定義済み値。初期リリースでは入力画面・登録処理の対象外 |
| regionName | - | 100文字以内 |
| resourceType | ○ | EC2 / CONTAINER / EKS_POD 等 |
| resourceName | ○ | 1〜150文字 |
| resourceIdentifier | - | 255文字以内 |

## 6. 状態遷移バリデーション

### 6.1 DeviceLifecycleStatus

| 現在 | 遷移可能 |
|---|---|
| ACTIVE | SPARE, PLANNED_RETIREMENT, RETIRED |
| SPARE | ACTIVE, RETIRED |
| PLANNED_RETIREMENT | ACTIVE, RETIRED |
| RETIRED | 原則変更不可。ただしテナント管理者のみ復旧可 |

### 6.2 IpUsageStatus

| 現在 | 遷移可能 |
|---|---|
| UNUSED | RESERVED, IN_USE, RETIRED |
| RESERVED | UNUSED, IN_USE, RETIRED |
| IN_USE | UNUSED, RETIRED |
| RETIRED | 原則変更不可 |

## 7. エラーメッセージ方針

- 利用者向けには分かりやすい日本語メッセージを返す。
- 内部エラー詳細、SQL、スタックトレースは画面に表示しない。
- どの項目が原因か分かるメッセージにする。

### 例

| ケース | メッセージ |
|---|---|
| 必須未入力 | 正式名称を入力してください。 |
| 重複 | 同じ正式名称が既に登録されています。 |
| プラン上限 | 現在のプランで登録可能な機器数の上限に達しています。 |
| 参照なし | 指定されたラックが見つかりません。 |
| ラックU重複 | 指定されたU位置には既に別の機器が登録されています。 |

## 8. Vaadin画面での扱い

- 入力フォームではBean Validationを利用する。
- 保存ボタン押下時にService層の業務チェックを実行する。
- ValidationExceptionは画面上部または項目横に表示する。
- 業務例外はNotificationまたはDialogで表示する。


## 9. CSVバリデーション

- CSVエクスポートは閲覧権限がある対象のみ出力可能とする。
- CSVインポートはテナント管理者または運用者のみ実行可能とする。
- ヘッダ不一致、必須欠落、形式不正、参照先なし、プラン上限超過は行単位またはファイル単位でエラーにする。
- 取込結果は `csv_import_history` と `csv_import_error` に記録する。
