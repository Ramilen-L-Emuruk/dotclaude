---
paths:
  - "**/*.java"
---
# Java コーディングスタイル

> 本ファイルは [common/coding-style.md](../common/coding-style.md) を Java 固有の内容で拡張する。

## フォーマット

- **google-java-format** または **Checkstyle**（Google / Sun スタイル）で強制
- 1ファイルに1つの public トップレベル型
- 一貫したインデント: 2 または 4 スペース（プロジェクト標準に合わせる）
- メンバー順序: 定数、フィールド、コンストラクタ、public メソッド、protected、private

## イミュータビリティ

- 値型には `record` を優先（Java 16+）
- フィールドはデフォルトで `final` — 可変状態は必要な場合のみ
- public API からは防御的コピーを返す: `List.copyOf()`、`Map.copyOf()`、`Set.copyOf()`
- 既存オブジェクトを変更するのではなく、新しいインスタンスを返す

```java
// 良い例 — 不変の値型
public record OrderSummary(Long id, String customerName, BigDecimal total) {}

// 良い例 — final フィールド、setter なし
public class Order {
    private final Long id;
    private final List<LineItem> items;

    public List<LineItem> getItems() {
        return List.copyOf(items);
    }
}
```

## 命名

標準 Java 規約に従う:
- `PascalCase`: クラス、インターフェース、record、enum
- `camelCase`: メソッド、フィールド、パラメータ、ローカル変数
- `SCREAMING_SNAKE_CASE`: `static final` 定数
- パッケージ: すべて小文字、逆ドメイン（`com.example.app.service`）

## モダン Java 機能

明快さが向上する場面でモダンな言語機能を使用:
- **record**: DTO・値型（Java 16+）
- **sealed クラス**: 閉じた型階層（Java 17+）
- **パターンマッチング**: `instanceof` での明示的キャスト不要（Java 16+）
- **テキストブロック**: 複数行文字列 — SQL、JSON テンプレート（Java 15+）
- **switch 式**: アロー構文（Java 14+）
- **switch のパターンマッチング**: sealed 型の網羅的ハンドリング（Java 21+）

```java
// パターンマッチング instanceof
if (shape instanceof Circle c) {
    return Math.PI * c.radius() * c.radius();
}

// sealed 型階層
public sealed interface PaymentMethod permits CreditCard, BankTransfer, Wallet {}

// switch 式
String label = switch (status) {
    case ACTIVE -> "Active";
    case SUSPENDED -> "Suspended";
    case CLOSED -> "Closed";
};
```

## Optional の使用

- 結果がない可能性のある検索メソッドからは `Optional<T>` を返す
- `map()`、`flatMap()`、`orElseThrow()` を使用 — `isPresent()` なしの `get()` は禁止
- `Optional` をフィールド型やメソッドパラメータとして使用しない

```java
// 良い例
return repository.findById(id)
    .map(ResponseDto::from)
    .orElseThrow(() -> new OrderNotFoundException(id));

// 悪い例 — Optional をパラメータに使用
public void process(Optional<String> name) {}
```

## エラーハンドリング

- ドメインエラーには非チェック例外を優先
- `RuntimeException` を継承したドメイン固有例外を作成
- トップレベルハンドラ以外では広範な `catch (Exception e)` を避ける
- 例外メッセージにコンテキストを含める

```java
public class OrderNotFoundException extends RuntimeException {
    public OrderNotFoundException(Long id) {
        super("Order not found: id=" + id);
    }
}
```

## ストリーム

- 変換にストリームを使用。パイプラインは短く（3〜4操作が上限）
- 可読性が高ければメソッド参照を優先: `.map(Order::getTotal)`
- ストリーム操作内での副作用を避ける
- 複雑なロジックには、入り組んだストリームパイプラインよりループを優先
