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

## Tailwind
Tailwindは、TSXマークアップに直接ユーティリティ・クラスを素早く記述できるようにすることで、開発プロセスをスピードアップするCSSフレームワークです。

Tailwindでは、クラス名を追加することで要素をスタイリングします。たとえば、"text-blue-500 "というクラスを追加すると、`<h1>`のテキストが青くなります
```html
<h1 className="text-blue-500">青いです！</h1>
```
CSSスタイルはグローバルに共有されますが、各クラスは各要素に個別に適用されます。つまり、要素を追加または削除しても、別々のスタイルシートを維持したり、スタイルが衝突したり、アプリケーションの拡張に伴ってCSSバンドルのサイズが大きくなったりする心配はありません。

create-next-appを使用して新しいプロジェクトを開始すると、Next.jsはTailwindを使用するかどうかを尋ねます。yesを選択すると、Next.jsは自動的に必要なパッケージをインストールし、アプリケーションにTailwindを設定します。

`/app/page.tsx`を見ると、サンプルでTailwindクラスを使っていることがわかります。

Tailwindを使うのが初めてでもご心配なく。Tailwindを使うのが初めての方でもご安心ください。

Tailwindで遊んでみましょう！以下のコードをコピーして、/app/page.tsxの`<p>`要素の上に貼り付けてください

```tsx
<div
  className="relative w-0 h-0 border-l-[15px] border-r-[15px] border-b-[26px] border-l-transparent border-r-transparent border-b-black"
/>
```
※ 黒い三角形が表示される。

伝統的なCSSルールを書きたい場合や、スタイルをJSXから分離したい場合は、CSSモジュールが最適な選択肢となります。
