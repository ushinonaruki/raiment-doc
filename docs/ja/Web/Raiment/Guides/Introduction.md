# 概要 (Introduction)

このガイドでは、Raimentの基本的な概念、AI時代になぜデータ層の分離が重要なのか、そして既存のWeb標準やフレームワークとどのように共存・補完するのかについて学びます。

## Raimentとは

Raimentとは、データバインディングのための「第四のWebプレーン（Data Plane）」を確立することを目指す宣言的ポリフィルです。

- 第一のWebプレーン：HTML（構造）
- 第二のWebプレーン：CSS（見た目）
- 第三のWebプレーン：JavaScript（振る舞い）
- **第四のWebプレーン：Raiment（データ）**

## Raimentの目的

人間向けUI（React / Vue 等）の構築ではなく、機械（AI / クローラー）によるデータ解釈コストを削減することです。

CSSがHTMLの構造を壊さずに見た目を適用するのと同様に、RaimentはHTMLから内容（データ）を分離し、宣言的に流し込みます。

実装上は `<link rel="raiment">` で指定したYAMLをHTML内のプレースホルダー `(( ... ))` にバインドします。

### 実装例

#### HTML

```html
<div id="profile">
  <h1 id="profile-name">Taro Yamada</h1>
  <p id="profile-job">Frontend Developer</p>
</div>
```

#### HTML + Raiment

```html
<link rel="raiment" href="profile.yaml">
<div id="profile">
  <h1 id="profile-name">(( user.name ))</h1>
  <p id="profile-job">(( user.job ))</p>
</div>
```

```yaml
user:
  name: "Taro Yamada"
  job: "Frontend Developer"
```

## 次に読む (See also)

### ガイド (Guide)

### 手引 (Learn)

### リファレンス (Reference)
