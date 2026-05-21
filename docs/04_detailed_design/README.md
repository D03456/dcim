# 詳細設計

## 1. 目的

本フォルダは、Data Center Asset & Infrastructure Manager の詳細設計を管理する。

基本設計で定義した業務・画面・権限・DB方針を、実装担当者が具体的なクラス、Service、Repository、バリデーション、トランザクション、ログ、テストへ落とし込める粒度まで補完する。

## 2. 詳細設計一覧

| No | ファイル | 内容 |
|---:|---|---|
| 01 | `01_entity_definition.md` | ドメイン・エンティティ定義 |
| 02 | `02_table_definition.md` | テーブル定義 |
| 03 | `03_service_design.md` | Service設計 |
| 04 | `04_repository_design.md` | Repository設計 |
| 05 | `05_validation_specification.md` | バリデーション仕様 |
| 06 | `06_exception_handling.md` | 例外処理設計 |
| 07 | `07_notification_specification.md` | 通知処理仕様 |
| 08 | `08_package_structure.md` | パッケージ構成設計 |
| 09 | `09_authorization_design.md` | 認可制御設計 |
| 10 | `10_transaction_design.md` | トランザクション設計 |
| 11 | `11_logging_design.md` | ログ設計 |
| 12 | `12_value_object_design.md` | Value Object設計 |
| 13 | `13_test_design.md` | テスト設計 |
| 14 | `14_review_checklist.md` | 詳細設計レビュー確認事項 |

## 3. 画面詳細・モックの扱い

画面詳細設計、画面項目定義、UIモックは別工程で作成する。

本段階では、画面モックに依存しない以下を優先する。

- ドメイン・DB・Service・Repositoryの整合性
- 権限とテナント分離
- Freeトライアル期限超過時の制御
- CSV、通知、バッチ、ログ、トランザクションの基本方針
- 実装前レビュー観点

## 4. レビュー方針

- 要件定義・基本設計と矛盾しないこと。
- 初期リリース対象と将来拡張を混在させないこと。
- 画面表示制御だけでなく、Service層で業務・権限・テナント分離を必ず担保すること。
- 秘密情報、認証情報、個人情報をログや通知に出力しないこと。
