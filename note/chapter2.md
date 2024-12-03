# 第2章 CSSスタイル
現在、ホームページにはスタイルがありません。Next.jsアプリケーションをスタイル設定するさまざまな方法を見てみましょう。

## この章では...

この章で扱うトピックは次のとおりです。
* アプリケーションにグローバル CSS ファイルを追加する方法。
* スタイリングの2つの異なる方法： Tailwind と CSS モジュール。
* clsx ユーティリティパッケージを使って条件付きでクラス名を追加する方法。

## グローバルスタイル
/app/uiフォルダの中を見ると、global.cssというファイルがあります。このファイルを使って、アプリケーションのすべてのルートにCSSルールを追加できます。たとえば、CSSのリセットルール、リンクのようなHTML要素のサイト全体のスタイルなどです。

アプリケーションのどのコンポーネントでもglobal.cssをインポートできますが、通常はトップレベルのコンポーネントに追加するのがよい習慣です。Next.jsでは、これがルートレイアウトです（これについては後で詳しく説明します）。

グローバルスタイルをアプリケーションに追加するには、/app/layout.tsxに移動して、global.cssファイルをインポートします：

```typescript
import '@/app/ui/global.css';

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  );
}
```
開発サーバーを起動したまま、変更を保存し、ブラウザでプレビューしてください。ホームページはこのようになるはずです。
![プレビュー](./images/image1.png)

でもちょっと待って、CSSルールを追加していないのに、スタイルはどこから来たのでしょうか？

global.cssの中を見てみると、@tailwindディレクティブがいくつかあることに気づくだろう

```typescript
@tailwind base;
@tailwind components;
@tailwind utilities;
```
