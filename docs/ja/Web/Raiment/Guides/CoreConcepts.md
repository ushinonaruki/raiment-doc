# 基本概念 / 用語 (Core Concepts)

Raimentを理解し活用するために必要な、主要概念と用語を定義します。

## 1. Data Plane（データプレーン）

Raimentは、HTML/CSS/JavaScriptに続く、データバインディングのための **第四のWebプレーン（Data Plane）** を確立することを目指します。

| プレーン | 技術 | 役割 |
|---|---|---|
| 構造層 | HTML | 骨格 |
| 見た目層 | CSS | 肌 |
| 振る舞い層 | JavaScript | 運動 |
| **データ層** | **Raiment** | データ |

HTMLが構造を定義し、CSSが見た目を適用し、JavaScriptが振る舞いを制御するのと同様に、Raimentはデータを宣言的に流し込みます。

CSSがHTMLの構造を壊さずに見た目を適用するように、RaimentはHTMLの骨格に触れることはありません。

機械（AI/クローラー）によるデータ解釈コストの削減を主目的とし、人間が可読な「構造（HTML）」から、機械が処理しやすい「データ（YAML）」を完全に分離します。

この分離により、AIはHTML全体をパースすることなく、データファイルのみを読み取ることで即座に情報を取得できます。

## 2. 6つの契約（責務）

RaimentがCSS/JSと並ぶ「第4のプレーン」として自立するために定義された、根本仕様です。

| 契約 | 対立する層 | 概要 |
|---|---|---|
| 非破壊性 | HTML | HTMLの要素ノード（骨格）を破壊せず、textNode（内容）のみを適用 |
| 環境追従性 | CSS/Context | 言語設定やテーマの変化に対し、宣言的にデータソースを切り替える責務 |
| JS API | JavaScript | JS層がRaiment層を命令的に操作するための公式な接点（RaimentOM）を提供 |
| Security契約 | Data | 安全なデータの定義と、サニタイズのルールを明確化 |
| A11y契約 | User | 動的な内容変更を支援技術へ適切に伝達する責務 |
| Tooling契約 | Developer | 開発者ツールがRaimentの状態を可視化・デバッグできるAPIを提供 |

これら6つの契約は、それぞれ対応するガイドページで詳細に説明されます。

- 非破壊性の設計原則 → Design Guidelines
- 環境追従性の実装 → Performance Considerations
- JS APIの詳細仕様 → Reference（RaimentOM APIリファレンス）
- Security契約の詳細 → Security Considerations
- A11y契約の詳細 → Accessibility Considerations
- Tooling契約の実装 → Best Practices

## 3. Data Source（データソース）

外部ファイル（YAML）でデータを定義し、`<link rel="raiment">` でHTMLに関連付けます。

```html
<link rel="raiment" href="data.yaml">
```

データソースは `<link>` タグを使用し、`rel` 属性に `raiment` を指定します。

データ形式の解決は、HTTPレスポンスの `Content-Type` ヘッダ、`type` 属性、ファイル拡張子の順で行われます。

複数の `<link>` が存在する場合、すべてのリソース取得が完了するまでレンダリングは開始されません。

### 3.1 環境追従属性

環境の変化に応じて、宣言的にデータソースを切り替える仕組みです。

#### `raiment-lang` 属性

`<html>` タグの `lang` 属性値と一致する `<link>` タグのみを有効化します。

`lang` 属性が変更された場合、Raimentは自動でデータソースを再評価し、再描画します。

```html
<link rel="raiment" href="data.ja.yaml" raiment-lang="ja">
<link rel="raiment" href="data.en.yaml" raiment-lang="en">
```

## 4. Substitution Engine（置換エンジン）

データソースから取得したデータを、HTML内のプレースホルダーに適用する仕組みです。

Substitution Engineは「何を置換し、どのように処理するか」を定義します。

### 4.1 プレースホルダー記法

HTML内でデータを埋め込む位置を示すマーカーです。

テキストノードおよび属性値に使用できます。

| 記法 | 動作 | セキュリティ処理 |
|---|---|---|
| `(( ... ))` | HTMLエスケープあり | Raiment Coreがエスケープによる出力整形を実施 |
| `((( ... )))` | HTMLエスケープなし | 外部のSecurity Engineに完全委譲 |

```html
<h1>(( post.title ))</h1>
<p>(( post.author ))</p>
<div>((( post.htmlContent )))</div>
```

`(( ... ))` は安全な文字列への整形（`<` を `&lt;` に変換など）を自動で行います。

`((( ... )))` は非エスケープ指定としてマークし、実際のXSS防御は外部のSecurity Engineに委譲します。

### 4.2 非破壊レンダリング

RaimentはDOMの要素ノード（骨格）を一切変更せず、textNode（内容）のみを管理対象とします。

CSSがRender Treeにスタイルを適用する思想を模倣した設計です。

#### レンダリング時機

- **初回描画**:
  `DOMContentLoaded` 後に一括適用
- **動的更新**:
  `MutationObserver` による自動監視は行わず、JS API（`window.raiment.refresh()`）による明示的なトリガーのみ

この設計により、Raimentは既存のHTML構造やCSSセレクタ、JavaScript操作に一切影響を与えません。

### 4.3 フォールバック

データパスの解決に失敗した値（`undefined`、`null` を含む）は、一律で空文字 `""` としてレンダリングされます。

```yaml
user:
  name: "Taro Yamada"
  # age は未定義
```

```html
<h1>(( user.name ))</h1>  <!-- "Taro Yamada" -->
<p>(( user.age ))</p>      <!-- "" (空文字) -->
```

配列やオブジェクトが参照された場合も、一律で空文字 `""` としてレンダリングされます。

再試行や代替ソースの読み込みなどの挙動は禁止です。

複雑な分岐処理や再試行による肥大化を防ぐため、フォールバック動作は極めてシンプルに設計されています。

## 5. `__metadata`（メタデータ）

データオブジェクト内に `__metadata` キーが存在する場合、その配下の値群は「外部仕様で定義されたメタ情報」として扱われます。

`__metadata` は、Substitution Engineの処理対象外となる領域を定義する概念です。

### Raimentの責務 (構文の定義)

Raimentは `__metadata` という「場所（構文）」を定義するだけであり、その配下の内容の意味・構文・解釈については一切関知しません。

レンダリング（`(( ... ))`）の対象からも完全に除外されます。

### 外部の責務 (語彙の定義)

__metadata の意味・構文・解釈をSEOやAI（RAG）のヒントとして利用するのは、クローラーやAIなど外部エンジンの責務です。

これにより、外部の団体（GoogleやW3C等）が独自の語彙（`reviewed-by`、`license` 等）を自由に定義できます。

```yaml
# post.yaml

# --- Raimentが処理するデータ ---
title: "Raiment 仕様書"
author: "Taro Yamada"

# --- Raimentが処理しないメタ情報 ---
__metadata:
  # 1. JSON-LD (AI/クローラーの標準語彙)
  "@context": "https://schema.org"
  "@type": "Article"
  "headline": "Raiment 仕様書"
  # 2. 外部団体が定義する独自の語彙
  "reviewed-by": "google.com"
  "license": "CC-BY-4.0"
```

`__metadata` は、Raimentと外部エンジンの責務分離を明確化し、Raimentの主目的である「機械可読性の向上」を実現する中核概念です。

AIは `__metadata` を読み取ることで、HTMLを解析することなく構造化されたメタ情報を直接取得できます。

## 6. RaimentOM（JS API）

JavaScript層がRaiment層を命令的に操作するための公式な接点です。

6つの契約の一つ（JS API契約）として定義されています。

### 6.1 役割

- 単一パス原子操作の提供（UIデータ編集の基底語彙）
- 観測と再同期のフック提供（observe/refresh）
- 上位（React/Vue 等）やSDKの糖衣APIからの呼び出し先（常にプリミティブへ正規化）

### 6.2 コアプリミティブ（概念）

- `window.raiment.get(path)`
  指定パスの現在値を取得（読み取り専用）。

- `window.raiment.set(path, value)`
  丸ごと置換。
  オブジェクト/配列/スカラーをそのまま入れ替える。

- `window.raiment.merge(path, partial)`
  同一階層の部分更新。
  キーの追加/上書きのみ（shallow 限定／削除非対応）。

- `window.raiment.unset(path)`
  オブジェクトのキー削除。
  `path` が指すプロパティを取り除く。

- `window.raiment.splice(path, index, deleteCount, ...values)`
  配列の位置編集（挿入/削除/置換/並べ替えの基底）。
  `index` 位置で `deleteCount` 個削除し、`values` を挿入。
  配列操作はこのコアプリミティブのみでの対応。

- `window.raiment.add(path, delta)`
  数値の加減算（原子的 read–modify–write）。
  減算は `delta` を負値に。

- `window.raiment.observe(path, callback)`
  指定パスの変更を購読。
  購読解除関数を返す。

- `window.raiment.refresh(scope?)`
  指定スコープ（省略時は全体）のDOMバインディングを再同期。

- `window.raiment.snapshot()`
  全データの読み取り専用の複製（スナップショット）を返す（DevTools/外部エンジン向け）。

複合操作（deep-merge、unique push、rename、CAS、トランザクション等）は上位のSDKで提供され、内部で上記プリミティブに変換されます。

### 6.3 例（最小の書き味）

```javascript
// 読み取り
const title = window.raiment.get('post.title');

// 置換（再描画は明示）
window.raiment.set('post.title', '新しいタイトル');
window.raiment.refresh('post');

// オブジェクトの部分更新（shallow）
window.raiment.merge('user', { nickname: 'ta' });

// オブジェクトのキー削除
window.raiment.unset('user.nickname');

// 配列: 2番目に要素を挿入
window.raiment.splice('post.tags', 1, 0, 'raiment');

// 数値: 在庫を5減らす（原子的）
window.raiment.add('inventory.stock', -5);

// 変更の購読
const off = window.raiment.observe('post.title', (next) => {
  console.debug('title changed:', next);
});
// 変更の購読を解除
off();

// スナップショット（読み取り専用）
const dump = window.raiment.snapshot();
```

#### 実装ルール（概念）

各プリミティブは型に厳格（対象が期待型でなければ `TypeError`）かつ原子的に実行。

`merge` は shallow 限定、削除は `unset` に分離、配列編集は常に `splice` で表現、数値演算は `add` に統一。

RaimentOM は、Core Concepts では最小語彙と責務の境界のみ定義する。

詳細な引数検証、エラー仕様、バッチ/トランザクション、SDK糖衣API（例: `push`, `reorder`, `deepMerge`, `compareAndSet` 等）の挙動は Reference を参照。

## 7. 次に読む (See also)

### ガイド (Guide)

- **[Introduction](./Introduction.md)** - 概要

### 手引 (Learn)

### リファレンス (Reference)
