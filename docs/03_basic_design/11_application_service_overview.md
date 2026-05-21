# Application Service設計概要

## 1. 目的

本書は、本システムのApplication Service層の責務と主要サービスを基本設計レベルで整理する。

詳細なメソッド、Repository、例外、バリデーションは詳細設計で定義する。

## 2. レイヤ構成

```text
presentation.vaadin
  ↓
application.service
  ↓
domain.model / domain.repository
  ↓
infrastructure.repository / notification / security
```

## 3. 共通設計方針

- Presentation層からRepositoryを直接呼び出さない。
- Application Serviceはユースケース単位の処理を担当する。
- テナント分離、権限、プラン上限、参照整合性はApplication Serviceで必ず確認する。
- 更新系処理はトランザクション境界をApplication Serviceに置く。
- 画面表示用DTOと更新用Command DTOを分離する。

## 4. Application Service一覧

| Service | 主な責務 |
|---|---|
| TenantApplicationService | テナント情報、状態管理 |
| SubscriptionApplicationService | 契約プラン、Freeトライアル、オプション管理 |
| UserApplicationService | ユーザー招待、ユーザー状態管理 |
| AuthorizationApplicationService | 権限・ロール判定 |
| DataCenterApplicationService | DC登録・更新・削除・検索 |
| LocationApplicationService | 棟、フロア、区画、ラック列管理 |
| RackApplicationService | ラック管理、U位置整合性 |
| DeviceApplicationService | 機器管理、ラック配置、保守・IP関連付け |
| IpManagementApplicationService | IPサブネット、IPアドレス割当管理 |
| MaintenanceApplicationService | 保守契約、対象機器、期限検索 |
| NotificationApplicationService | 通知設定、通知ログ、メール送信依頼 |
| CsvApplicationService | CSVエクスポート、インポート履歴、行エラー管理 |
| SearchApplicationService | 横断検索、タグ検索 |
| CloudResourceApplicationService | 将来拡張。クラウド資産管理 |

## 5. 主要Service概要

## 5.1 SubscriptionApplicationService

### 主な責務

- 契約プランの取得・変更
- Freeトライアル期限の管理
- オプション追加状態の管理
- 利用量と上限の取得

### 主な業務ルール

- Freeプランは14日間トライアルとする。
- プラン上限はDC、ラック、機器、IPサブネット、ユーザーを対象とする。
- サブネット追加は10サブネット単位、機器追加は100台単位とする。

## 5.2 DataCenterApplicationService

### 主な責務

- データセンター登録・更新・削除
- 連絡先、タグの関連付け
- 物理階層の起点管理

### 主な業務ルール

- 同一テナント内で正式名称を重複させない。
- DC登録時にプラン上限を確認する。
- 削除時は配下の棟、フロア、区画、ラック、機器の存在を確認する。

## 5.3 RackApplicationService

### 主な責務

- ラック登録・更新・削除
- ラック列との関連管理
- 搭載U位置の整合性チェック

### 主な業務ルール

- ラック数はプラン上限の対象とする。
- 同一ラック内でU位置の重複を許可しない。
- ラック高さ変更時は既存搭載機器の範囲を下回らない。

## 5.4 DeviceApplicationService

### 主な責務

- 機器登録・更新・削除
- ラック配置
- IP割当
- 保守契約との関連確認

### 主な業務ルール

- 機器登録時に機器数上限を確認する。
- 正式名称、ホスト名、シリアル番号の重複を確認する。
- 廃止済み機器には新規IP割当を行わない。

## 5.5 IpManagementApplicationService

### 主な責務

- IPサブネット登録・更新・削除
- サブネット配下のIP利用状況管理
- 機器へのIP割当・解除

### 主な業務ルール

- プラン上限は個別IP数ではなくIPサブネット数で判定する。
- 同一テナント内でCIDRを重複させない。
- 個別IPは所属サブネット範囲内であること。

## 5.6 MaintenanceApplicationService

### 主な責務

- 保守契約登録・更新・削除
- 保守契約と機器の紐付け
- 保守期限検索
- 保守未契約機器検索

### 主な業務ルール

- 保守契約の終了日は開始日以降とする。
- 同一保守契約に同一機器を重複紐付けしない。
- 標準通知日は終了日の60日前とする。

## 5.7 NotificationApplicationService

### 主な責務

- 通知設定の管理
- 通知ログ作成
- メール送信処理への依頼
- 重複通知抑止

### 主な業務ルール

- 初期リリースの通知チャネルはメールを基本とする。
- 同一対象・同一宛先への保守期限通知は初期リリースでは1回のみとする。
- メール送信失敗時は通知ログにFAILEDを記録する。

## 5.8 CsvApplicationService

### 主な責務

- CSVエクスポート
- CSVインポート
- 取込履歴・行単位エラー管理

### 主な業務ルール

- CSVエクスポートは初期リリース必須とする。
- CSVインポートは初期追加対象とする。
- インポート時もプラン上限、参照存在、権限を検証する。

## 6. 共通例外方針

| 種別 | 例 |
|---|---|
| 入力不正 | 必須未入力、形式不正 |
| 参照なし | 指定IDが存在しない |
| 重複 | 同一名称、同一CIDR |
| 上限超過 | プラン上限超過 |
| 権限不足 | 閲覧者による更新、他テナントデータ参照 |
| 状態不正 | 廃止済み機器へのIP割当 |

## 7. 後続確認事項

| 項目 | 確認内容 |
|---|---|
| API公開範囲 | Vaadin内部利用に限定するか、REST APIも外部公開するか |
| CSV非同期化 | 初期から非同期にするか、小規模同期で開始するか |
| Free期限超過時の制御 | 読み取り専用、ログイン制限、登録停止のどれにするか |
| クラウド資産Service | 将来拡張時の初期対象クラウド |
