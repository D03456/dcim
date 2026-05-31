# 4. 詳細設計 - 例外処理

## 1. 目的

本書は、Data Center Asset & Infrastructure Manager の例外分類、例外クラス、ハンドリング方針、ログ出力方針を定義する。

## 2. 基本方針

- 想定可能な業務エラーは独自例外として扱う。
- システムエラーは利用者に詳細を表示しない。
- Vaadin画面では利用者向けメッセージに変換して表示する。
- REST APIを将来提供する場合は、HTTPステータスとエラーコードに変換する。
- 例外には可能な限りエラーコードを付与する。

## 3. 例外分類

| 分類 | 例 | 利用者表示 | ログレベル |
|---|---|---|---|
| 入力エラー | 必須未入力、形式不正 | 表示する | WARN |
| 業務エラー | プラン上限、重複、状態不正 | 表示する | WARN |
| 認可エラー | 権限不足、他テナント参照 | 表示する | WARN |
| データ不整合 | 参照先なし、不整合 | 表示する | ERROR |
| 外部連携エラー | メール送信失敗 | 表示または管理者通知 | ERROR |
| システムエラー | DB障害、予期しない例外 | 汎用表示 | ERROR |

## 4. 例外クラス構成

```text
DcimException
  ├─ ValidationException
  ├─ BusinessException
  │   ├─ PlanLimitExceededException
  │   ├─ DuplicateResourceException
  │   ├─ InvalidStateException
  │   └─ ResourceInUseException
  ├─ ResourceNotFoundException
  ├─ AccessDeniedBusinessException
  ├─ ExternalServiceException
  │   └─ NotificationSendException
  └─ SystemException
```

## 5. 共通例外基底クラス

### DcimException

| 項目 | 内容 |
|---|---|
| 目的 | 独自例外の基底クラス |
| 主な属性 | errorCode, message, detail, cause |

```java
public abstract class DcimException extends RuntimeException {
    private final String errorCode;
    private final String userMessage;

    protected DcimException(String errorCode, String userMessage) {
        super(userMessage);
        this.errorCode = errorCode;
        this.userMessage = userMessage;
    }
}
```

## 6. 主要例外定義

## 6.1 ValidationException

| 項目 | 内容 |
|---|---|
| 用途 | 入力値不正 |
| 例 | 必須未入力、形式不正、桁数超過 |
| 表示 | 項目別メッセージ |

### メッセージ例

- 正式名称を入力してください。
- CIDRまたはIPアドレスの形式が正しくありません。
- 終了日は開始日以降の日付を指定してください。

## 6.2 ResourceNotFoundException

| 項目 | 内容 |
|---|---|
| 用途 | 指定されたデータが存在しない |
| 例 | rackIdが存在しない、他テナントのデータを参照 |
| 表示 | 指定されたデータが見つかりません。 |

### 注意

他テナントのデータを指定された場合も、情報漏えいを避けるため「見つかりません」と表示する。

## 6.3 PlanLimitExceededException

| 項目 | 内容 |
|---|---|
| 用途 | 契約プラン上限超過 |
| 例 | Freeで6台目の機器を登録しようとした |
| 表示 | 現在のプランで登録可能な上限に達しています。 |

### 対象

- データセンター
- ラック
- 機器
- 管理対象IP数（`ip_address` 件数）
- タグ
- ユーザー

ラック列は物理階層であり上限対象外とする。IPサブネット数も上限なしとし、個別IPアドレス数のみをプラン上限判定に利用する。

## 6.4 DuplicateResourceException

| 項目 | 内容 |
|---|---|
| 用途 | 一意制約違反、重複登録 |
| 例 | 同一テナント内で同じ正式名称の機器を登録 |
| 表示 | 同じ内容のデータが既に登録されています。 |

## 6.5 InvalidStateException

| 項目 | 内容 |
|---|---|
| 用途 | 状態遷移不正 |
| 例 | RETIREDのIPを機器へ割当 |
| 表示 | 現在の状態ではこの操作を実行できません。 |

## 6.6 ResourceInUseException

| 項目 | 内容 |
|---|---|
| 用途 | 利用中データの削除不可 |
| 例 | 機器が存在するラックを削除 |
| 表示 | 関連データが存在するため削除できません。 |

## 6.7 AccessDeniedBusinessException

| 項目 | 内容 |
|---|---|
| 用途 | 業務上の権限不足 |
| 例 | 閲覧者が更新処理を実行 |
| 表示 | この操作を実行する権限がありません。 |

## 6.8 NotificationSendException

| 項目 | 内容 |
|---|---|
| 用途 | メール等の通知送信失敗 |
| 例 | SMTP接続失敗、送信先不正 |
| 表示 | 通知送信に失敗しました。 |

## 6.9 SystemException

| 項目 | 内容 |
|---|---|
| 用途 | 予期しないシステムエラー |
| 例 | DB接続失敗、NullPointerException |
| 表示 | システムエラーが発生しました。時間をおいて再度お試しください。 |

## 7. エラーコード体系

| 範囲 | 分類 | 例 |
|---|---|---|
| VAL-xxxx | 入力チェック | VAL-0001 |
| BIZ-xxxx | 業務エラー | BIZ-0001 |
| AUTH-xxxx | 認可エラー | AUTH-0001 |
| DATA-xxxx | データ不整合 | DATA-0001 |
| EXT-xxxx | 外部連携 | EXT-0001 |
| SYS-xxxx | システム | SYS-0001 |

## 8. エラーコード例

| エラーコード | 例外 | メッセージ |
|---|---|---|
| VAL-0001 | ValidationException | 必須項目が入力されていません。 |
| VAL-0002 | ValidationException | 入力形式が正しくありません。 |
| BIZ-0001 | PlanLimitExceededException | 現在のプランで登録可能な上限に達しています。 |
| BIZ-0002 | DuplicateResourceException | 同じデータが既に登録されています。 |
| BIZ-0003 | InvalidStateException | 現在の状態ではこの操作を実行できません。 |
| BIZ-0004 | ResourceInUseException | 関連データが存在するため削除できません。 |
| AUTH-0001 | AccessDeniedBusinessException | この操作を実行する権限がありません。 |
| DATA-0001 | ResourceNotFoundException | 指定されたデータが見つかりません。 |
| EXT-0001 | NotificationSendException | 通知送信に失敗しました。 |
| SYS-0001 | SystemException | システムエラーが発生しました。 |

## 9. Vaadin画面でのハンドリング

### 9.1 表示方針

| 例外 | 表示方法 |
|---|---|
| ValidationException | 対象項目付近に表示 |
| BusinessException | 画面上部またはNotification表示 |
| ResourceNotFoundException | ダイアログ表示後、一覧へ戻る |
| AccessDeniedBusinessException | エラー通知後、操作不可 |
| SystemException | 汎用エラー表示 |

### 9.2 表示例

```java
try {
    deviceService.create(command);
    Notification.show("機器を登録しました。");
} catch (BusinessException e) {
    Notification.show(e.getUserMessage(), 5000, Notification.Position.TOP_CENTER);
}
```

## 10. REST APIを提供する場合のHTTPステータス

| 例外 | HTTPステータス |
|---|---|
| ValidationException | 400 Bad Request |
| ResourceNotFoundException | 404 Not Found |
| AccessDeniedBusinessException | 403 Forbidden |
| PlanLimitExceededException | 409 Conflict |
| DuplicateResourceException | 409 Conflict |
| InvalidStateException | 409 Conflict |
| ExternalServiceException | 502 Bad Gateway |
| SystemException | 500 Internal Server Error |

## 11. ログ出力方針

### 11.1 出力項目

| 項目 | 内容 |
|---|---|
| timestamp | 発生日時 |
| errorCode | エラーコード |
| tenantId | テナントID |
| userId | ユーザーID |
| operation | 操作名 |
| resourceType | 対象種別 |
| resourceId | 対象ID |
| message | エラーメッセージ |

### 11.2 注意事項

- パスワード、APIキー、認証トークンはログ出力しない。
- 個人情報は必要最小限にする。
- システム例外はスタックトレースを出力する。
- 業務例外は原則スタックトレース不要。

## 12. 通知処理での例外扱い

- 1件の通知失敗でバッチ全体を停止しない。
- 失敗した通知は `notification_log.status = FAILED` とする。
- エラー内容は `error_message` に保存する。
- 同じ対象への重複送信を避けるため、成功ログを確認する。

## 13. テスト方針

| テスト | 内容 |
|---|---|
| 単体テスト | 各Serviceで想定例外が発生すること |
| Repositoryテスト | DB制約違反の確認 |
| 画面テスト | エラー表示確認 |
| バッチテスト | 通知失敗時に処理継続されること |

<!-- issue-fixes-237 -->

## 付録A. Issue対応追補: 例外クラス配置方針

| 種別 | 配置先 | 例 |
|---|---|---|
| 共通基底例外 | `common.error` | `DcimException`, `ErrorCode` |
| 入力・認可・状態不正 | `common.error` または `application.error` | `ValidationException`, `AuthorizationException`, `ConflictException` |
| 業務ルール例外 | `domain.exception` | `PlanLimitExceededException`, `RackUnitConflictException`, `InvalidCidrException` |
| インフラ例外 | `infrastructure.error` | `MailSendException`, `FileStorageException` |

Application ServiceはDomain例外を捕捉して画面/API向けのメッセージコードへ変換する。内部詳細、SQL、トークン、個人情報は例外メッセージに含めない。

<!-- issue-fixes-274 -->

## 付録B. Issue対応追補: ValidationException詳細

ValidationExceptionは複数エラーを保持できる構造とする。

```java
public class ValidationErrorDetail {
    private String field;
    private String messageCode;
    private String message;
    private Object rejectedValue; // 秘密情報は必ずマスク済みにする
}

public class ValidationException extends DcimException {
    private List<ValidationErrorDetail> fieldErrors;
    private List<ValidationErrorDetail> globalErrors;
}
```

Controller/APIを提供する場合も同じ構造をレスポンスへ変換する。Vaadinでは `field` があるものを項目横、`field` がないものを画面上部に表示する。

<!-- issue-fixes-295 -->

## 付録C. Issue対応追補: 認証系例外

| 例外 | エラーコード | 表示方針 |
|---|---|---|
| AuthenticationFailedException | AUTH-001 | メールまたはパスワードが正しくない旨を汎用表示し、存在有無を推測させない |
| TenantSuspendedException | AUTH-002 | 利用停止中である旨と問い合わせ先を表示 |
| RateLimitExceededException | AUTH-003 | 一定時間後の再試行を案内 |
| InvalidTokenException | AUTH-004 | トークンが無効である旨を表示 |
| TokenExpiredException | AUTH-005 | 有効期限切れ、再発行導線を表示 |
| InvalidInvitationException | AUTH-006 | 招待が無効/取消済み/使用済みである旨を表示 |

これらの例外では内部理由、トークン値、認証失敗詳細を画面・ログへ出さない。監査ログには結果コードとrequest_idのみを残す。
