# API・サービス設計方針

## 1. 目的

本書は、Data Center Asset & Infrastructure Manager のAPIおよびサービス設計方針を定義する。

画面はVaadinを主とするが、内部サービス層を明確化し、将来的なREST API公開や外部連携にも耐えられる構成とする。

## 2. 基本方針

- Spring Bootを利用する。
- Vaadin画面からApplication Serviceを呼び出す。
- ドメインロジックはDomain ServiceまたはEntityに寄せる。
- Repositoryは永続化責務に限定する。
- APIは当初は内部利用中心とし、将来的にREST API化できる命名・DTO設計にする。
- テナント分離、権限チェック、プラン上限チェックをサービス層で必ず実施する。

## 3. レイヤ構成

```mermaid
flowchart TD
    UI[Vaadin UI / View] --> AppService[Application Service]
    AppService --> DomainService[Domain Service]
    DomainService --> DomainModel[Domain Model / Entity]
    AppService --> Repository[Repository]
    Repository --> DB[(MariaDB)]
    AppService --> External[External Integration]
```

## 4. パッケージ構成案

```text
com.example.dcim
  ├── config
  ├── security
  ├── common
  │   ├── exception
  │   ├── validation
  │   └── audit
  ├── tenant
  │   ├── domain
  │   ├── application
  │   ├── infrastructure
  │   └── ui
  ├── location
  ├── rack
  ├── device
  ├── ipsubnet
  ├── maintenance
  ├── cloud
  ├── notification
  └── user
```

## 5. サービス一覧

| サービスID | サービス名 | 主な責務 |
|---|---|---|
| SVC-001 | AuthService | ログイン、認証情報取得 |
| SVC-002 | TenantService | テナント情報取得、テナント状態確認 |
| SVC-003 | SubscriptionService | プラン情報取得、利用上限算出 |
| SVC-004 | UsageLimitService | DC、ラック、機器、管理対象IP、タグ、ユーザー、Freeトライアル期間、期限超過時の更新可否チェック |
| SVC-005 | DataCenterService | データセンター登録、更新、検索、詳細取得 |
| SVC-006 | LocationService | 建物、フロア、エリア、ラック列管理 |
| SVC-006A | RegionService | 地域・都道府県分類の登録、更新、検索 |
| SVC-006B | ContactService | 連絡先登録、更新、検索、関連付け |
| SVC-006C | ResourceAliasService | 呼称名・別名の登録、更新、検索 |
| SVC-007 | RackService | ラック登録、更新、検索、詳細取得 |
| SVC-008 | RackTemplateService | ラックテンプレート管理 |
| SVC-009 | RackPlacementService | ラック搭載位置チェック、空きU算出 |
| SVC-010 | DeviceService | 機器登録、更新、検索、詳細取得 |
| SVC-011 | IpSubnetService | IPサブネット登録、配下IP割当、解放、検索 |
| SVC-012 | MaintenanceContractService | 保守契約登録、対象機器ひもづけ、検索 |
| SVC-013 | MaintenanceAlertService | 保守期限通知対象抽出、保守未設定機器抽出 |
| SVC-014 | CloudAccountService | 将来拡張。クラウドアカウント管理 |
| SVC-015 | CloudResourceService | 将来拡張。クラウドリソース管理、同期結果保存 |
| SVC-016 | TagService | タグ登録、更新、付与、解除 |
| SVC-017 | NotificationService | 通知設定、通知履歴管理、メール通知実行、画面内通知作成、未読/既読管理、送信失敗管理 |
| SVC-021 | CsvExportService | 初期必須。検索結果と主要一覧のCSV出力 |
| SVC-022 | CsvImportService | 初期追加対象。CSV取込、検証、エラー結果管理 |
| SVC-018 | UserAccountService | ユーザー招待、更新、無効化、ロール変更 |
| SVC-019 | AuthorizationService | 権限チェック、ロール判定 |
| SVC-020 | AuditLogService | 将来拡張。操作ログ記録、検索 |

## 6. API設計方針

### 6.1 URL設計方針

将来的にREST APIを公開する場合は、以下のようなURL体系とする。

| リソース | API例 |
|---|---|
| データセンター | `/api/v1/data-centers` |
| リージョン | `/api/v1/regions` |
| 連絡先 | `/api/v1/contacts` |
| 呼称名・別名 | `/api/v1/resource-aliases` または `/api/v1/{resource}/{id}/aliases` |
| ラック | `/api/v1/racks` |
| 機器 | `/api/v1/devices` |
| IPサブネット | `/api/v1/ip-subnets` |
| IP利用状況 | `/api/v1/ip-addresses` |
| 保守契約 | `/api/v1/maintenance-contracts` |
| 通知設定 | `/api/v1/notification-settings` |
| 通知 | `/api/v1/notifications`、`/api/v1/notifications/{id}/read` |
| CSV出力 | `/api/v1/csv/exports` |
| CSV取込 | `/api/v1/csv/imports` |
| 将来拡張：クラウドアカウント | `/api/v1/cloud-accounts` |
| 将来拡張：クラウドリソース | `/api/v1/cloud-resources` |
| タグ | `/api/v1/tags` |
| ユーザー | `/api/v1/users` |
| 利用状況 | `/api/v1/usage` |

### 6.2 HTTPメソッド方針

| メソッド | 用途 |
|---|---|
| GET | 一覧取得、詳細取得 |
| POST | 新規作成、検索条件が複雑な検索 |
| PUT | 全体更新 |
| PATCH | 一部更新、ステータス変更 |
| DELETE | 削除。内部的には論理削除を原則とする |

### 6.3 DTO方針

| DTO種別 | 用途 |
|---|---|
| SearchCondition | 検索条件 |
| SummaryDto | 一覧表示用 |
| DetailDto | 詳細表示用 |
| CreateCommand | 新規登録要求 |
| UpdateCommand | 更新要求 |
| ResultDto | 登録・更新結果 |

## 7. 代表API案

### 7.1 データセンター

| API | メソッド | 概要 |
|---|---|---|
| `/api/v1/data-centers` | GET | データセンター一覧取得 |
| `/api/v1/data-centers/{id}` | GET | データセンター詳細取得 |
| `/api/v1/data-centers` | POST | データセンター登録 |
| `/api/v1/data-centers/{id}` | PUT | データセンター更新 |
| `/api/v1/data-centers/{id}` | DELETE | データセンター削除 |

### 7.2 機器

| API | メソッド | 概要 |
|---|---|---|
| `/api/v1/devices` | GET | 機器一覧取得 |
| `/api/v1/devices/{id}` | GET | 機器詳細取得 |
| `/api/v1/devices` | POST | 機器登録 |
| `/api/v1/devices/{id}` | PUT | 機器更新 |
| `/api/v1/devices/{id}/maintenance-contracts` | GET | 機器に紐づく保守契約取得 |
| `/api/v1/devices/without-maintenance` | GET | 保守未設定機器一覧取得 |

### 7.3 保守契約

| API | メソッド | 概要 |
|---|---|---|
| `/api/v1/maintenance-contracts` | GET | 保守契約一覧取得 |
| `/api/v1/maintenance-contracts/{id}` | GET | 保守契約詳細取得 |
| `/api/v1/maintenance-contracts` | POST | 保守契約登録 |
| `/api/v1/maintenance-contracts/{id}` | PUT | 保守契約更新 |
| `/api/v1/maintenance-contracts/{id}/devices` | POST | 対象機器ひもづけ |
| `/api/v1/maintenance-contracts/alerts` | GET | 保守期限アラート取得 |

### 7.4 利用状況

| API | メソッド | 概要 |
|---|---|---|
| `/api/v1/usage` | GET | 現在の利用状況取得 |
| `/api/v1/usage/limits` | GET | プラン上限取得 |
| `/api/v1/add-ons` | POST | オプション追加 |

## 8. 共通処理方針

| 項目 | 方針 |
|---|---|
| 認証 | Spring Securityでログインセッション管理 |
| 認可 | メソッド実行前にロール・権限チェック |
| テナント分離 | すべての検索・更新でtenantId条件を必須化 |
| 入力チェック | Bean Validation + ドメイン制約チェック |
| 例外処理 | 業務例外、認可例外、システム例外を分類 |
| 最低限監査情報 | 登録・更新時に作成者、作成日時、更新者、更新日時を保存 |
| 監査ログ | 将来拡張。登録、更新、削除、権限変更、契約変更を記録 |
| トランザクション | Application Service単位で制御 |
| ページング | 一覧取得はページングを標準化 |
| CSV出力 | 初期必須。一覧検索条件に基づく出力を標準化 |
| CSV取込 | 初期追加対象。事前検証、行単位エラー、取込履歴を標準化 |
| メール通知 | 初期必須。通知履歴、送信失敗、再送要否を管理できる構造にする |
| 画面内通知 | 初期必須。通知ログ、未読/既読、ダッシュボード表示を管理できる構造にする |

## 9. プラン上限チェック方針

```mermaid
sequenceDiagram
    actor User as 利用者
    User->>UI: 登録操作
    UI->>ApplicationService: create(command)
    ApplicationService->>AuthorizationService: 権限確認
    ApplicationService->>UsageLimitService: 上限確認
    UsageLimitService->>SubscriptionService: プラン・オプション取得
    UsageLimitService-->>ApplicationService: 登録可否
    ApplicationService->>Repository: 保存
    ApplicationService->>Repository: 最低限監査情報を含めて保存
    ApplicationService-->>UI: 結果返却
```

### 9.1 管理対象IP数上限

- IPサブネット数には上限を設けず、サブネット配下に生成・管理する個別IP数の上限をプラン別に持つ。
- Free 256、Starter 512、Business 1,024、Enterprise 2,048 IPを初期上限とする。
- IP上限追加オプションは256IP単位で、管理対象IP数上限に256件を追加する。
- CIDRプレフィックスは固定せず、個別IP生成予定数を含めて上限判定する。

## 10. 保守期限通知方針

- 保守契約の終了日から60日前を標準通知対象とする。
- 通知日数は契約単位またはテナント設定で変更可能とする。
- 通知対象はバッチまたはスケジューラで抽出する。
- 通知履歴を保存し、同一契約への重複通知を制御する。

## 11. 外部連携方針

| 外部連携 | 方針 |
|---|---|
| メール通知 | 初期必須。SMTPまたはクラウドメールサービスを利用可能な設計にする |
| AWS連携 | 将来拡張。手動登録およびAPI同期を検討する |
| CSVインポート | 初期追加対象。データセンター、ラック、機器、IPサブネット/IP利用状況、保守契約の一括登録対象とする |
| CSVエクスポート | 初期必須。一覧画面の検索結果出力として実装する |

## 12. 命名方針

| 対象 | 方針 |
|---|---|
| Service | `XxxService` |
| Repository | `XxxRepository` |
| Entity | ドメイン名を単数形で命名 |
| DTO | `XxxSummaryDto`, `XxxDetailDto` |
| Command | `CreateXxxCommand`, `UpdateXxxCommand` |
| Enum | 業務用語 + `Type` / `Status` |

## 13. 実装上の注意

- Controller/APIを後から追加してもUI層を大きく変更しないよう、Application Serviceを中心に設計する。
- Vaadin Viewに業務ロジックを持たせない。
- RepositoryをUIから直接呼ばない。
- 権限チェックとテナントチェックを画面側だけに依存しない。
- 将来的なクラウドリソース自動同期に備え、外部IDを保持する。


## 14. Freeトライアル期限超過時のAPI制御

Freeトライアル期限超過テナントは `TRIAL_EXPIRED` として扱い、API・Service層で通常ロールより前に更新可否を判定する。

| 処理種別 | 可否 | 備考 |
|---|:---:|---|
| 参照系API | ○ | 既存データ確認のため許可 |
| CSVエクスポート | ○ | データ退避・棚卸のため許可 |
| 契約プラン変更 | ○ | 有料プラン移行のため許可 |
| 登録・更新・削除API | × | 期限超過中は不可 |
| CSVインポート | × | 新規登録扱いのため不可 |
| ユーザー招待 | × | テナント拡張扱いのため不可 |
