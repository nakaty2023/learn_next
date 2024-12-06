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

## パターン：アクティブリンクの表示
一般的なUIパターンは、現在どのページにいるのかをユーザーに示すために、アクティブなリンクを表示することです。これを行うには、URLからユーザーの現在のパスを取得する必要があります。Next.jsは[usePathname()](https://nextjs.org/docs/app/api-reference/functions/use-pathname)というフックを提供しており、これを使用してパスをチェックし、このパターンを実装することができます。

usePathname()はフックなので、nav-links.tsxをクライアントコンポーネントにする必要があります。Reactの 「use client 」ディレクティブをファイルの先頭に追加し、next/navigationからusePathname()をインポートします。

```tsx
// app/ui/dashboard/nav-links.tsx

'use client';

import {
  UserGroupIcon,
  HomeIcon,
  InboxIcon,
} from '@heroicons/react/24/outline';
import Link from 'next/link';
import { usePathname } from 'next/navigation';

// ...
```

次に、`<NavLinks />`コンポーネント内のpathnameという変数にパスを代入します
```tsx
// app/ui/dashboard/nav-links.tsx

export default function NavLinks() {
  const pathname = usePathname();
  // ...
}
```

CSSスタイリングの章で紹介したclsxライブラリを使えば、リンクがアクティブなときに条件付きでクラス名を適用することができる。link.hrefがパス名と一致するとき、リンクは青いテキストと水色の背景で表示されるはずです。

これがnav-links.tsxの最終コードです
```tsx
// app/ui/dashboard/nav-links.tsx

'use client';

import {
  UserGroupIcon,
  HomeIcon,
  DocumentDuplicateIcon,
} from '@heroicons/react/24/outline';
import Link from 'next/link';
import { usePathname } from 'next/navigation';
import clsx from 'clsx';

// ...

export default function NavLinks() {
  const pathname = usePathname();

  return (
    <>
      {links.map((link) => {
        const LinkIcon = link.icon;
        return (
          <Link
            key={link.name}
            href={link.href}
            className={clsx(
              'flex h-[48px] grow items-center justify-center gap-2 rounded-md bg-gray-50 p-3 text-sm font-medium hover:bg-sky-100 hover:text-blue-600 md:flex-none md:justify-start md:p-2 md:px-3',
              {
                'bg-sky-100 text-blue-600': pathname === link.href,
              },
            )}
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
保存してローカルホストをチェックする。アクティブなリンクが青くハイライトされているはずです。

第5章を修了しました

Next.jsでページ間をリンクし、クライアントサイドナビゲーションを活用する方法を学びました。
