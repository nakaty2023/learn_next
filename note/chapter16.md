# 第16章 メタデータの追加
メタデータはSEOと共有のために重要です。この章では、Next.jsアプリケーションにメタデータを追加する方法について説明します。

## この章では...

この章で取り上げるトピックは次のとおりです。

* メタデータとは何か
* メタデータの種類
* メタデータを使ってOpen Graph画像を追加する方法。
* メタデータを使ってファビコンを追加する方法

## メタデータとは？
ウェブ開発において、メタデータはウェブページに関する追加情報を提供します。メタデータは、ページを訪れたユーザーには見えません。その代わり、ページのHTML内、通常は<head>要素内に埋め込まれ、舞台裏で機能します。この隠された情報は、検索エンジンや、ウェブページのコンテンツをよりよく理解する必要のある他のシステムにとって非常に重要です。

## なぜメタデータが重要なのか？
メタデータはウェブページのSEOを強化する上で重要な役割を果たし、検索エンジンやソーシャルメディアプラットフォームにとってよりアクセスしやすく、理解しやすいものにします。適切なメタデータは、検索エンジンがウェブページを効果的にインデックスし、検索結果のランキングを向上させるのに役立ちます。さらに、Open Graphのようなメタデータは、ソーシャルメディア上で共有されたリンクの見栄えを改善し、コンテンツをユーザーにとってより魅力的で有益なものにします。

## メタデータの種類
メタデータにはさまざまな種類があり、それぞれが独自の目的をもっている。一般的なタイプには以下のようなものがある

タイトル・メタデータ： ブラウザのタブに表示されるウェブページのタイトルを担当します。検索エンジンがウェブページの内容を理解するのに役立つため、SEOには欠かせない。

```html
<title>Page Title</title>
```

説明メタデータ： このメタデータはウェブページコンテンツの簡単な概要を提供し、検索エンジンの検索結果によく表示されます。

```html
<meta name="description" content="A brief description of the page content." />
```

オープングラフメタデータ： このメタデータは、ソーシャルメディアプラットフォームでウェブページが共有される際の表現方法を強化し、タイトル、説明文、プレビュー画像などの情報を提供します。

```html
<meta property="og:title" content="Title Here" />
<meta property="og:description" content="Description Here" />
<meta property="og:image" content="image_url_here" />
```

ファビコンのメタデータ： このメタデータは、ブラウザのアドレスバーまたはタブに表示されるファビコン（小さなアイコン）をウェブページにリンクします。

```html
<link rel="icon" href="path/to/favicon.ico" />
```

## メタデータの追加
Next.jsには、アプリケーションのメタデータを定義するためのMetadata APIがあります。アプリケーションにメタデータを追加するには、2つの方法があります

* 設定ベース： 静的なメタデータオブジェクトまたは動的なgenerateMetadata関数をlayout.jsまたはpage.jsファイルにエクスポートします。
* ファイルベース： Next.jsには、メタデータ用に特別に使用されるさまざまなファイルがあります：
  * favicon.ico、apple-icon.jpg、icon.jpg：ファビコンやアイコンに使用します。
  * opengraph-image.jpg、twitter-image.jpg：ソーシャルメディアの画像に使用。
  * robots.txt： 検索エンジンのクロールを指示する
  * sitemap.xml： ウェブサイトの構造に関する情報を提供

これらのファイルは、静的なメタデータとして使用することも、プロジェクト内でプログラム的に生成することもできます。

どちらの方法でも、Next.jsは関連する`<head>`要素を自動的に生成します。

### ファビコンとオープングラフ画像
publicフォルダに、favicon.icoとopengraph-image.jpgの2つの画像があることに気づくだろう。

これらの画像を/appフォルダのルートに移動します。

こうすると、Next.jsは自動的にこれらのファイルをファビコンとOG画像として認識し、使用します。このことは、開発ツールでアプリケーションの`<head>`要素をチェックすることで確認できます。

知っておいて損はありません： ImageResponseコンストラクタを使用して、動的なOG画像を作成することもできます。

### ページのタイトルと説明
layout.jsまたはpage.jsファイルからmetadataオブジェクトをインクルードして、タイトルや説明文などのページ情報を追加することもできます。layout.js内のメタデータは、それを使用するすべてのページに継承されます。

ルート・レイアウトで、以下のフィールドを持つ新しいメタデータ・オブジェクトを作成してください

```tsx
// app/layout.tsx

import { Metadata } from 'next';

export const metadata: Metadata = {
  title: 'Acme Dashboard',
  description: 'The official Next.js Course Dashboard, built with App Router.',
  metadataBase: new URL('https://next-learn-dashboard.vercel.sh'),
};

export default function RootLayout() {
  // ...
}
```

Next.jsは自動的にタイトルとメタデータをアプリケーションに追加します。

しかし、特定のページにカスタムタイトルを追加したい場合はどうすればよいでしょうか？これは、ページ自体にメタデータオブジェクトを追加することで実現できます。入れ子になったページのメタデータは、親のメタデータを上書きします。

例えば、/dashboard/invoicesページでは、ページタイトルを更新することができます

```tsx
// app/dashboard/invoices/page.tsx

import { Metadata } from 'next';

export const metadata: Metadata = {
  title: 'Invoices | Acme Dashboard',
};
```

これは機能しますが、すべてのページでアプリケーションのタイトルを繰り返しています。会社名のように何かが変われば、すべてのページでそれを更新しなければなりません。

代わりに、メタデータオブジェクトのtitle.templateフィールドを使って、ページタイトルのテンプレートを定義することができます。このテンプレートには、ページタイトルのほか、任意の情報を含めることができます。

ルート・レイアウトで、テンプレートを含むようにメタデータ・オブジェクトを更新してください

```tsx
// app/layout.tsx

import { Metadata } from 'next';

export const metadata: Metadata = {
  title: {
    template: '%s | Acme Dashboard',
    default: 'Acme Dashboard',
  },
  description: 'The official Next.js Learn Dashboard built with App Router.',
  metadataBase: new URL('https://next-learn-dashboard.vercel.sh'),
};
```

テンプレート内の%sは、特定のページタイトルに置き換えられます。

これで、/dashboard/invoicesページにページタイトルを追加できます

```tsx
// app/dashboard/invoices/page.tsx

export const metadata: Metadata = {
  title: 'Invoices',
};
```

dashboard/invoicesページに移動し、`<head>`要素を確認してください。ページタイトルが「Invoices｜Acme Dashboard」になっていることが確認できるはずです。
