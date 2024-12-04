# 第3章 フォントと画像の最適化
前の章では、Next.jsアプリケーションのスタイル設定方法を学びました。引き続き、カスタムフォントとヒーロー画像を追加して、ホームページを作成しましょう。

## この章では...
この章で取り上げるトピックは次のとおりです。

* next/fontを使用してカスタムフォントを追加する方法。
* next/imageで画像を追加する方法
* Next.jsでフォントと画像を最適化する方法。

## フォントを最適化する理由
フォントはウェブサイトのデザインにおいて重要な役割を果たしますが、プロジェクトでカスタムフォントを使用すると、フォントファイルの取得と読み込みが必要になり、パフォーマンスに影響を与える可能性があります。

CLS（Cumulative Layout Shift）は、Googleがウェブサイトのパフォーマンスとユーザー体験を評価するために使用する指標です。フォントの場合、レイアウトシフトは、ブラウザが最初にフォールバックフォントまたはシステムフォントでテキストをレンダリングし、それが読み込まれるとカスタムフォントに入れ替わるときに発生します。この入れ替えにより、テキストのサイズ、間隔、レイアウトが変更され、周囲の要素が移動することがあります。

next.jsは、next/fontモジュールを使用すると、アプリケーション内のフォントを自動的に最適化します。ビルド時にフォントファイルをダウンロードし、他の静的アセットと一緒にホストします。つまり、ユーザーがアプリケーションにアクセスしたときに、パフォーマンスに影響するようなフォントの追加ネットワークリクエストが発生しません。

## 主要フォントの追加
アプリケーションにカスタムGoogleフォントを追加して、その動作を確認してみましょう！

app/uiフォルダに、fonts.tsという新しいファイルを作成します。このファイルは、アプリケーション全体で使用されるフォントを保持するために使用します。

next/font/googleモジュールからInterフォントをインポートしてください。次に、どのサブセットを読み込むかを指定します。この場合は'latin'です

```typescript
import { Inter } from 'next/font/google';

export const inter = Inter({ subsets: ['latin'] });
```

最後に、/app/layout.tsxの<body>要素にフォントを追加する。

```tsx
import '@/app/ui/global.css';
import { inter } from '@/app/ui/fonts';

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body className={`${inter.className} antialiased`}>{children}</body>
    </html>
  );
}
```

`<body>`要素にInterを追加することで、アプリケーション全体にフォントが適用されます。ここでは、フォントを滑らかにする Tailwind antialiased クラスも追加しています。このクラスを使う必要はありませんが、いいアクセントになります。

ブラウザに移動してdev toolsを開き、body要素を選択します。スタイルの下にInterとInter_Fallbackが適用されているのが見えるはずです。

## 練習：セカンダリフォントの追加
アプリケーションの特定の要素にフォントを追加することもできます。

次はあなたの番です！fonts.tsファイルで、Lusitanaというセカンダリーフォントをインポートし、/app/page.tsxファイルの`<p>`要素に渡します。前と同じようにサブセットを指定するだけでなく、フォントの太さも指定する必要があります。

準備ができたら、下のコード・スニペットを展開して解決策を見てください。

ヒント
* フォントに渡すウェイトオプションがわからない場合は、コードエディタでTypeScriptのエラーを確認してください。
* Google Fontsのウェブサイトにアクセスし、Lusitanaを検索して、どのようなオプションが利用可能かを確認する。
* 複数のフォントを追加するためのドキュメントと、オプションの完全なリストを参照してください。

/app/ui/fonts.ts
```typescript
import { Inter, Lusitana } from 'next/font/google';

export const inter = Inter({ subsets: ['latin'] });

export const lusitana = Lusitana({
  weight: ['400', '700'],
  subsets: ['latin'],
});
```

/app/page.tsx

```tsx
import AcmeLogo from '@/app/ui/acme-logo';
import { ArrowRightIcon } from '@heroicons/react/24/outline';
import Link from 'next/link';
import { lusitana } from '@/app/ui/fonts';

export default function Page() {
  return (
    // ...
    <p
      className={`${lusitana.className} text-xl text-gray-800 md:text-3xl md:leading-normal`}
    >
      <strong>Welcome to Acme.</strong> This is the example for the{' '}
      <a href="https://nextjs.org/learn/" className="text-blue-500">
        Next.js Learn Course
      </a>
      , brought to you by Vercel.
    </p>
    // ...
  );
}
```

最後に、`<AcmeLogo />`コンポーネントもLusitanaを使用しています。エラーを防ぐためにコメントアウトされていますが、コメントアウトを解除してください

/app/page.tsx
```tsx
// ...
export default function Page() {
  return (
    <main className="flex min-h-screen flex-col p-6">
      <div className="flex h-20 shrink-0 items-end rounded-lg bg-blue-500 p-4 md:h-52">
        <AcmeLogo />
        {/* ... */}
      </div>
    </main>
  );
}
```
これで、アプリケーションに2つのカスタムフォントが追加されました！次に、ホームページにヒーロー画像を追加してみましょう。
