# 4. 詳細設計 - パッケージ構成設計

## 1. 目的

本書は、本システムのJava / Spring Boot / Vaadin実装におけるパッケージ構成と責務分担を定義する。

## 2. 基本方針

- DDDを意識したレイヤード構成とする。
- Presentation層からRepositoryを直接呼び出さない。
- 業務ルールはApplication ServiceまたはDomain Serviceに集約する。
- Infrastructureは外部技術詳細を閉じ込める。
- テナント分離、認可、プラン上限、Free期限超過制御はService層で必ず実施する。

## 3. パッケージ構成

```text
com.example.dcim
  ├─ common
  │   ├─ error
  │   ├─ time
  │   └─ util
  ├─ domain
  │   ├─ model
  │   │   ├─ tenant
  │   │   ├─ location
  │   │   ├─ rack
  │   │   ├─ device
  │   │   ├─ network
  │   │   ├─ maintenance
  │   │   ├─ notification
  │   │   ├─ csv
  │   │   └─ user
  │   ├─ repository
  │   ├─ service
  │   └─ exception
  ├─ application
  │   ├─ service
  │   ├─ command
  │   ├─ query
  │   └─ dto
  ├─ infrastructure
  │   ├─ persistence
  │   │   ├─ entity
  │   │   ├─ repository
  │   │   └─ mapper
  │   ├─ mail
  │   ├─ csv
  │   ├─ scheduler
  │   └─ security
  ├─ presentation
  │   └─ vaadin
  │       ├─ layout
  │       ├─ view
  │       └─ component
  └─ config
```

## 4. 各パッケージの責務

| パッケージ | 責務 |
|---|---|
| `common` | 共通例外、日時、ユーティリティ |
| `domain.model` | 業務概念、集約、Value Object、Enum |
| `domain.repository` | ドメイン視点のRepositoryインターフェース |
| `domain.service` | 集約をまたぐ業務判断 |
| `application.service` | ユースケース実行、トランザクション境界 |
| `application.command` | 登録・更新リクエスト |
| `application.query` | 検索条件 |
| `application.dto` | 画面・API返却用DTO |
| `infrastructure.persistence.entity` | JPA Entity |
| `infrastructure.persistence.repository` | Spring Data JPA実装 |
| `infrastructure.persistence.mapper` | Domain / Entity変換 |
| `infrastructure.mail` | メール送信 |
| `infrastructure.csv` | CSV入出力 |
| `infrastructure.scheduler` | バッチ起動 |
| `infrastructure.security` | 認証・ログインユーザー取得 |
| `presentation.vaadin` | Vaadin画面、コンポーネント、レイアウト |
| `config` | Spring設定 |

## 5. 依存関係方針

```text
presentation -> application -> domain
infrastructure -> domain
application -> domain.repository
infrastructure.persistence -> domain.repository
```

禁止する依存:

- `domain` から `application` / `infrastructure` / `presentation` への依存
- `presentation` から JPA Repository への直接依存
- `domain.model` から JPA Entity への依存
- CSV、メール、セキュリティ実装をDomainに持ち込むこと

## 6. 命名方針

| 種別 | 命名例 |
|---|---|
| Application Service | `DeviceApplicationService` |
| Domain Service | `PlanLimitDomainService` |
| Command | `DeviceCreateCommand` |
| Query | `DeviceSearchQuery` |
| DTO | `DeviceDetailDto` |
| JPA Entity | `DeviceEntity` |
| Repository interface | `DeviceRepository` |
| Repository implementation | `JpaDeviceRepository` |
| Vaadin View | `DeviceListView` |

## 7. 実装注意

- JPA Entityを画面へ直接返却しない。
- Command / Query / DTOは用途ごとに分け、肥大化させない。
- MapperはInfrastructure側に置き、Domainを永続化技術から分離する。
- 小規模実装で開始する場合も、PresentationからRepositoryへの直接呼び出しは避ける。
