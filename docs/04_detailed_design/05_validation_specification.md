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
| IPサブネット | 上限なし |
| 管理対象IP | プラン別 + IP上限追加オプション（256IP単位） |
| タグ | プラン別 |
| ユーザー | プラン別 |

### 4.2 プラン初期上限

| プラン | 利用期間 | DC | ラック | 機器 | IP数 | タグ | ユーザー |
|---|---:|---:|---:|---:|---:|---:|---:|
| Free | 14日間 | 1 | 3 | 40 | 256 | 10 | 3 |
| Starter | 契約期間中 | 2 | 5 | 50 | 512 | 50 | 3 |
| Business | 契約期間中 | 5 | 50 | 100 | 1,024 | 200 | 10 |
| Enterprise | 契約期間中 | 10 | 100 | 1000 | 2,048 | 1000 | 30 |

### 4.3 エラー条件

```text
現在登録数 >= プラン上限 + 追加枠

管理対象IPについては、現在の有効な `ip_address` 件数 + 生成予定IP数 > プラン上限 + IP上限追加オプション数 × 256 の場合に上限超過とする。
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

## 5.6A RackTemplate

| 項目 | 必須 | ルール |
|---|:---:|---|
| templateName | ○ | 1〜150文字。同一テナント内で重複不可 |
| heightUnit | ○ | 1以上 |
| item.startUnit | ○ | 1以上 |
| item.unitSize | ○ | 1以上 |

### ラックテンプレート明細チェック

| チェック | 内容 |
|---|---|
| 上限 | `startUnit + unitSize - 1 <= heightUnit` |
| 重複 | 同一テンプレート内の明細U範囲が重複しない |
| 作成時コピー | テンプレート変更は既存ラックへ自動反映しない |

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
- IPサブネット数には上限を設けないこと。
- サブネット登録または個別IP生成時に、生成・管理する個別IP数が管理対象IP数上限を超えないこと。
- CIDRプレフィックスはプランで固定しないが、生成・管理する個別IP数はプラン上限内に収めること。
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
| organizationName | - | 150文字以内 |
| departmentName | - | 150文字以内 |
| positionName | - | 100文字以内 |
| personName | ○ | 1〜150文字 |
| email | - | メール形式、255文字以内 |
| phoneNumber | - | 50文字以内 |
| address | - | 255文字以内 |
| preferredContactMethod | - | EMAIL / PHONE / OTHER |
| contactType | ○ | DC / VENDOR / INTERNAL |
| active | ○ | true / false |

### 業務チェック

- emailまたはphoneNumberのどちらか一方は入力推奨。
- 通知先に利用する場合はemail必須。

## 5.11 Tag

| 項目 | 必須 | ルール |
|---|:---:|---|
| tagName | ○ | 1〜50文字、同一テナント内で重複不可 |
| colorCode | - | `#RRGGBB` 形式 |

### 業務チェック

- タグ登録時にタグ数上限を超えないこと。
- タグ数上限は有効なタグマスタ件数で判定し、タグ付け件数は対象外とする。

## 5.12 ResourceAlias

| 項目 | 必須 | ルール |
|---|:---:|---|
| resourceType | ○ | DATA_CENTER / RACK / DEVICE |
| resourceId | ○ | 同一テナント内に存在 |
| aliasName | ○ | 1〜150文字、同一対象内で重複不可 |
| aliasType | - | DISPLAY / ABBREVIATION / OPERATION / OTHER |

### 業務チェック

- 正式名称と同一の別名は登録不可とする。
- 機器別名は `resourceType=DEVICE` で登録し、対象機器が同一テナント内に存在すること。
- 横断検索では正式名称、代表表示名、呼称名・別名、タグ名を検索対象に含める。
- 機器タグ付与では対象機器とタグが同一テナント内に存在し、同一機器への同一タグ重複付与を拒否する。

## 5.13 Region

| 項目 | 必須 | ルール |
|---|:---:|---|
| regionCode | - | 50文字以内、同一テナント内で重複不可とする場合は個別制約で定義 |
| regionName | ○ | 1〜100文字、同一テナント内で重複不可 |
| prefecture | - | 100文字以内 |
| displayOrder | - | 0以上の整数 |
| status | ○ | ACTIVE / INACTIVE |

### 業務チェック

- DCで利用中のリージョンは物理削除しない。削除操作は論理削除またはINACTIVE化とする。

## 5.14 AppUser / 招待・パスワード

| 項目 | 必須 | ルール |
|---|:---:|---|
| email | ○ | メール形式、255文字以内、同一テナント内で重複不可 |
| displayName | ○ | 1〜150文字 |
| passwordHash | ○ | ハッシュ化済み文字列のみ保存。平文パスワードは保存不可 |
| status | ○ | ACTIVE / INVITED / SUSPENDED |
| roleIds | ○ | 同一テナントで利用可能なロールID |

### 業務チェック

- ユーザー招待時にユーザー数上限を超えないこと。
- パスワード登録・変更時は、平文パスワードをログ・DB・通知に残さないこと。
- パスワード再設定トークンは有効期限内・未使用・対象ユーザーに紐づく場合のみ利用可能とすること。
- 再設定依頼時は、対象メールアドレスの存在有無にかかわらず同一メッセージを返すこと。
- 再設定トークンはハッシュ化して保存し、平文トークンをログ・DB・通知に残さないこと。
- 招待中ユーザーの再招待は既存状態を確認し、重複ユーザーを作成しないこと。
- 初期テナント管理者を通常のユーザー無効化・削除フローで指定した場合は業務エラーとすること。
- 初期テナント管理者の利用停止はテナント解約フロー内でのみ許可すること。

## 5.15 NotificationSetting

| 項目 | 必須 | ルール |
|---|:---:|---|
| notificationType | ○ | MAINTENANCE_EXPIRY / PLAN_LIMIT / TRIAL_EXPIRY / PASSWORD_RESET / USER_INVITATION / OPERATION_ERROR / SYSTEM |
| enabled | ○ | true / false |
| emailEnabled | ○ | true / false |
| inAppEnabled | ○ | true / false |
| timingCode | - | 60_DAYS_BEFORE / 30_DAYS_BEFORE / DUE_DATE / EXPIRED / WARNING / REACHED / ERROR 等 |
| daysBefore | - | 0以上の整数。期限系通知で使用 |
| targetRoles | - | 通知種別ごとに許可されたロールのみ指定可能 |

### 業務チェック

- `enabled=true` の場合、少なくとも1つの通知チャネル（メールまたは画面内通知）を有効にする。
- MAINTENANCE_EXPIRY は60日前・30日前・当日・期限切れを別タイミングとして扱い、いずれかの送信済みが他タイミングを抑止しない。
- TRIAL_EXPIRY の標準値は3日前および期限日とする。

## 5.16 CloudResource（将来拡張）

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
| Free期限超過 | Freeトライアル期間が終了しています。有料プランへ変更すると登録・更新できます。 |
| 参照なし | 指定されたラックが見つかりません。 |
| ラックU重複 | 指定されたU位置には既に別の機器が登録されています。 |

## 8. Vaadin画面での扱い

- 入力フォームではBean Validationを利用する。
- 保存ボタン押下時にService層の業務チェックを実行する。
- ValidationExceptionは画面上部または項目横に表示する。
- 業務例外はNotificationまたはDialogで表示する。


## 9. CSVバリデーション

- CSVエクスポートは閲覧権限がある対象のみ出力可能とする。
- CSVインポートはテナント管理者、運用管理者、編集者のみ実行可能とする。
- Freeプランは1ファイル100行まで実行可能とし、101行以上はファイル単位エラーにする。
- ヘッダ不一致、文字コード不正、対象種別不正、ファイル全体の必須列欠落はファイル単位エラーとして `FAILED` にする。
- 行単位の必須欠落、形式不正、参照先なし、プラン上限超過は該当行のみエラーにする。正常行は登録し、1件以上の行エラーがある場合は `PARTIAL_FAILED` にする。
- 全行成功時は `SUCCEEDED`、全行失敗時は `FAILED` とする。
- 取込結果は `csv_import_history` と `csv_import_error` に記録する。


## 10. Freeトライアル期限超過時のバリデーション

Freeプランの14日間トライアル期限を超過したテナントは `TRIAL_EXPIRED` として扱い、通常ロールの権限判定より前に更新可否を判定する。

| 処理 | 可否 | 備考 |
|---|:---:|---|
| ログイン | ○ | 期限超過案内を表示する |
| 参照 | ○ | 既存データ確認を許可する |
| CSVエクスポート | ○ | データ退避・棚卸のため許可する |
| 契約プラン確認 | ○ | 有料プラン移行導線を表示する |
| 有料プラン変更 | ○ | 継続利用のため許可する |
| 登録・更新・削除 | × | テナント管理者でも不可 |
| CSVインポート | × | 新規データ追加扱いのため不可 |
| ユーザー招待 | × | テナント拡張扱いのため不可 |

更新不可時は業務例外として扱い、利用者には有料プラン変更を促す日本語メッセージを表示する。

<!-- issue-fixes-217-218-222-235-236 -->

## 付録A. Issue対応追補: バリデーション仕様

### A.1 機器のラック搭載項目

| 状態 | `rackId` | `rackUnitStart` | `rackUnitSize` | 判定 |
|---|---:|---:|---:|---|
| 未搭載 | NULL | NULL | NULL | 許可 |
| 搭載 | NOT NULL | NOT NULL | NOT NULL | 許可 |
| 一部のみ入力 | NULL/NOT NULL混在 | NULL/NOT NULL混在 | NULL/NOT NULL混在 | 不可 |

搭載時は `rackUnitStart >= 1`、`rackUnitSize >= 1`、`rackUnitStart + rackUnitSize - 1 <= rack.heightUnit`、同一ラック内の有効機器・予約Uとの重複なしを検証する。

### A.2 ラックU番号の基準

ラックU番号は **下から1U** を正本とする。画面表示、CSV、Repository検索、重複判定はすべてこの基準で扱う。画面で上から描画する場合も、内部値は下からのU番号を保持し、表示時に変換する。

### A.3 TaggedResource

`resourceType/resourceId/tagId` 登録時は以下を検証する。

- `tagId` が同一テナントの有効タグであること
- `resourceType` が初期対象種別であること
- 対象リソースが同一テナントに存在し、削除済みでないこと
- 同一対象に同一タグが未付与であること
- タグ数上限・対象リソース権限を満たすこと

### A.4 RackTemplate.heightUnit

`RackTemplate.heightUnit` は初期リリースでは `Rack.heightUnit` と同じく `1〜60` とする。将来、特殊ラックを扱う場合はテンプレート専用上限を別Issueで再定義する。

### A.5 Rack/Device項目名対応

| 概念 | Javaプロパティ | DBカラム | 画面表示名 |
|---|---|---|---|
| ラック高さ | `heightUnit` | `height_unit` | U数 / ラック高さ |
| ラック搭載開始U | `rackUnitStart` | `rack_unit_start` | 搭載開始U |
| ラック搭載U数 | `rackUnitSize` | `rack_unit_size` | 使用U数 |
| 機器メーカー | `manufacturer` | `manufacturer` | メーカー |
| 機器型番 | `modelNumber` | `model_number` | 型番 |
| ホスト名 | `hostName` | `host_name` | ホスト名 |

`heightU`、`vendor` などの別名表記は説明文に限定し、Command/DTO/Entityでは上記の正規名を使う。
