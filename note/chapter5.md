# 第５章 ページ間の移動

前の章では、ダッシュボードのレイアウトとページを作成しました。では、ユーザがダッシュボードのルート間を移動できるようにリンクを追加しましょう。

## この章では...

この章で扱うトピックは次のとおりです。

* next/linkコンポーネントの使い方
* usePathname()フックを使ってアクティブなリンクを表示する方法。
* Next.jsでのナビゲーションの仕組み


## なぜナビゲーションを最適化するのか？

ページ間をリンクするためには、伝統的に `<a>` HTML要素を使用します。

現在、サイドバーのリンクは `<a>` 要素を使用していますが、ブラウザでホーム、請求書、顧客ページ間を移動するときに何が起こるかに注目してください。

見えましたか？

各ページへのナビゲーション時に、ページ全体がリフレッシュされます！

もちろん、以下にご依頼の英文を日本語に翻訳しました。

## `<Link>`コンポーネント

Next.js では、アプリケーション内のページ間をリンクするために `<Link />` コンポーネントを使用できます。`<Link>` を使用すると、JavaScript を使ったクライアントサイドのナビゲーションが可能になります。

`<Link />` コンポーネントを使用するには、/app/ui/dashboard/nav-links.tsx を開き、next/link から Link コンポーネントをインポートします。その後、`<a>` タグを `<Link>` に置き換えます

```tsx
// app/ui/dashboard/nav-links.tsx

import {
  UserGroupIcon,
  HomeIcon,
  DocumentDuplicateIcon,
} from '@heroicons/react/24/outline';
import Link from 'next/link';

// ...

export default function NavLinks() {
  return (
    <>
      {links.map((link) => {
        const LinkIcon = link.icon;
        return (
          <Link
            key={link.name}
            href={link.href}
            className="flex h-[48px] grow items-center justify-center gap-2 rounded-md bg-gray-50 p-3 text-sm font-medium hover:bg-sky-100 hover:text-blue-600 md:flex-none md:justify-start md:p-2 md:px-3"
          >
            <LinkIcon className="w-6" />
            <p className="hidden md:block">{link.name}</p>
          </Link>
        );
      })}
    </>
  );
}
```

ご覧のように、Linkコンポーネントは`<a>`タグを使うのと似ていますが、`<a href=「...」>`の代わりに`<Link href=「...」>`を使います。

変更を保存して、ローカルホストで動作するか確認してください。これで、完全な更新を見ることなくページ間を移動できるようになるはずです。アプリケーションの一部はサーバー上でレンダリングされますが、完全なページ更新がないため、Webアプリケーションのように感じられます。なぜでしょうか？

## 自動コード分割とプリフェッチ
ナビゲーション体験を向上させるために、Next.jsはルートセグメントごとにアプリケーションを自動的にコード分割します。これは、ブラウザが初期ロード時にすべてのアプリケーションコードをロードする従来のReact SPAとは異なります。

ルートによってコードが分割されるということは、ページが分離されるということです。特定のページがエラーをスローしても、アプリケーションの残りの部分は動作します。

さらに本番環境では、`<Link>`コンポーネントがブラウザのビューポートに表示されるたびに、Next.jsはバックグラウンドでリンクされたルートのコードを自動的にプリフェッチします。ユーザーがリンクをクリックするころには、リンク先ページのコードはすでにバックグラウンドで読み込まれています！

[ナビゲーションの仕組みについてもっと知る。](https://nextjs.org/docs/app/building-your-application/routing/linking-and-navigating#how-routing-and-navigation-works)
