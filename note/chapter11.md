# 第11章 検索とページネーションの追加
前の章では、ストリーミングによってダッシュボードの初期読み込みパフォーマンスを改善しました。次に /invoices ページに移動して、検索とページ分割を追加する方法を学びましょう！

## この章では...

この章で取り上げるトピックは次のとおりです。

* Next.jsのAPI、useSearchParams、usePathname、useRouterの使い方を学びます。
* URL検索パラメータを使用して、検索とページ分割を実装します。

## 開始コード
dashboard/invoices/page.tsxファイルの中に、以下のコードを貼り付けます

```tsx
// app/dashboard/invoices/page.tsx

import Pagination from '@/app/ui/invoices/pagination';
import Search from '@/app/ui/search';
import Table from '@/app/ui/invoices/table';
import { CreateInvoice } from '@/app/ui/invoices/buttons';
import { lusitana } from '@/app/ui/fonts';
import { InvoicesTableSkeleton } from '@/app/ui/skeletons';
import { Suspense } from 'react';

export default async function Page() {
  return (
    <div className="w-full">
      <div className="flex w-full items-center justify-between">
        <h1 className={`${lusitana.className} text-2xl`}>Invoices</h1>
      </div>
      <div className="mt-4 flex items-center justify-between gap-2 md:mt-8">
        <Search placeholder="Search invoices..." />
        <CreateInvoice />
      </div>
      {/*  <Suspense key={query + currentPage} fallback={<InvoicesTableSkeleton />}>
        <Table query={query} currentPage={currentPage} />
      </Suspense> */}
      <div className="mt-5 flex w-full justify-center">
        {/* <Pagination totalPages={totalPages} /> */}
      </div>
    </div>
  );
}
```

このページと、これから扱うコンポーネントに慣れるのに時間をかけましょう

1. `<Search/>`はユーザーが特定の請求書を検索できるようにします。
2. `<Pagination/>`は、ユーザーが請求書のページ間を移動できるようにします。
3. `<Table/>`は請求書を表示します。

検索機能はクライアントとサーバーにまたがります。ユーザーがクライアント上で請求書を検索すると、URLパラメータが更新され、サーバー上でデータが取得され、新しいデータでテーブルがサーバー上で再レンダリングされます。

## なぜURL検索パラメータを使うのか？
上述したように、検索状態を管理するためにURL検索パラメータを使用します。クライアントサイドのステートで検索を行うことに慣れていると、このパターンは新鮮かもしれません。

URLパラメータを使って検索を実装することには、いくつかの利点があります

* ブックマーク可能なURLと共有可能なURL： 検索パラメータはURL内にあるので、ユーザーは検索クエリとフィルタを含むアプリケーションの現在の状態をブックマークして、将来の参照や共有に利用することができます。
* サーバーサイドレンダリングと初期ロード： 初期状態をレンダリングするために、URLパラメータをサーバー上で直接消費することができるため、サーバーレンダリングの処理が容易になります。
* アナリティクスとトラッキング: 検索クエリとフィルタを直接URL内に持つことで、クライアントサイドのロジックを追加することなく、ユーザーの行動をトラッキングしやすくなります。

## 検索機能の追加
次に示すのは、検索機能を実装するために使用する Next.js のクライアントフックです。
* useSearchParams: 現在のURLのパラメータにアクセスできます。たとえば、/dashboard/invoices?page=1&query=pending というURLの場合、{page: '1', query: 'pending'} のような検索パラメータになります。
* usePathname: 現在のURLのパス名を取得できます。たとえば、ルートが /dashboard/invoices の場合、usePathname は '/dashboard/invoices' を返します。
* useRouter: クライアントコンポーネント内でプログラム的にルート間のナビゲーションを可能にします。利用できるメソッドはいくつか存在します。

以下は、実装手順の概要です。
1.	ユーザーの入力を取得する
2.	URLを検索パラメータ付きで更新する
3.	入力フィールドとURLを同期させる
4.	テーブルを検索クエリに応じて更新する

### 1. ユーザーの入力を取得する

`<Search>`コンポーネント（/app/ui/search.tsx）に移動します。以下の点に注目してください。
* "use client": このコンポーネントはクライアントコンポーネントであるため、イベントリスナーやフックを使用できます。
* `<input>`: これは検索入力フィールドです。

handleSearch という新しい関数を作成し、`<input>`要素にonChangeリスナーを追加します。onChangeは、入力値が変わるたびにhandleSearchを呼び出します。

```tsx
// app/ui/search.tsx

'use client';

import { MagnifyingGlassIcon } from '@heroicons/react/24/outline';

export default function Search({ placeholder }: { placeholder: string }) {
  function handleSearch(term: string) {
    console.log(term);
  }

  return (
    <div className="relative flex flex-1 flex-shrink-0">
      <label htmlFor="search" className="sr-only">
        Search
      </label>
      <input
        className="peer block w-full rounded-md border border-gray-200 py-[9px] pl-10 text-sm outline-2 placeholder:text-gray-500"
        placeholder={placeholder}
        onChange={(e) => {
          handleSearch(e.target.value);
        }}
      />
      <MagnifyingGlassIcon className="absolute left-3 top-1/2 h-[18px] w-[18px] -translate-y-1/2 text-gray-500 peer-focus:text-gray-900" />
    </div>
  );
}
```

デベロッパーツールのコンソールを開き、検索フィールドに文字を入力して、正しく動作していることをテストしてください。コンソールに検索語が記録されるはずです。

素晴らしい！ユーザーの検索入力をキャプチャしています。次に、検索語でURLを更新する必要があります。

### 2. 検索パラメータでURLを更新する
'next/navigation'からuseSearchParamsフックをインポートし、変数に代入する

```tsx
// app/ui/search.tsx

'use client';

import { MagnifyingGlassIcon } from '@heroicons/react/24/outline';
import { useSearchParams } from 'next/navigation';

export default function Search() {
  const searchParams = useSearchParams();

  function handleSearch(term: string) {
    console.log(term);
  }
  // ...
}
```

handleSearchの内部で、新しいsearchParams変数を使用して新しいURLSearchParamsインスタンスを作成します。

```tsx
// app/ui/search.tsx

'use client';

import { MagnifyingGlassIcon } from '@heroicons/react/24/outline';
import { useSearchParams } from 'next/navigation';

export default function Search() {
  const searchParams = useSearchParams();

  function handleSearch(term: string) {
    const params = new URLSearchParams(searchParams);
  }
  // ...
}
```

URLSearchParamsは、URLクエリ・パラメータを操作するためのユーティリティ・メソッドを提供するWeb APIです。複雑な文字列リテラルを作成する代わりに、?page=1&query=aのようなparams文字列を取得するために使用することができます。

次に、ユーザーの入力に基づいてparams文字列を設定します。入力が空の場合は、それを削除します

```tsx
// app/ui/search.tsx

'use client';

import { MagnifyingGlassIcon } from '@heroicons/react/24/outline';
import { useSearchParams } from 'next/navigation';

export default function Search() {
  const searchParams = useSearchParams();

  function handleSearch(term: string) {
    const params = new URLSearchParams(searchParams);
    if (term) {
      params.set('query', term);
    } else {
      params.delete('query');
    }
  }
  // ...
}
```

これでクエリ文字列ができました。Next.jsのuseRouterとusePathnameフックを使ってURLを更新できます。

useRouterとusePathnameを'next/navigation'からインポートし、handleSearch内でuseRouter()のreplaceメソッドを使用します

```tsx
// app/ui/search.tsx

'use client';

import { MagnifyingGlassIcon } from '@heroicons/react/24/outline';
import { useSearchParams, usePathname, useRouter } from 'next/navigation';

export default function Search() {
  const searchParams = useSearchParams();
  const pathname = usePathname();
  const { replace } = useRouter();

  function handleSearch(term: string) {
    const params = new URLSearchParams(searchParams);
    if (term) {
      params.set('query', term);
    } else {
      params.delete('query');
    }
    replace(`${pathname}?${params.toString()}`);
  }
}
```

何が起きているかの内訳は以下の通り

* `$[pathname}`は現在のパスで、あなたの場合は`/dashboard/invoices`です。
* ユーザーが検索バーに入力すると、`params.toString()`がこの入力をURLフレンドリーなフォーマットに変換します。
* `replace(${pathname}?${params.toString()})`は、ユーザーの検索データでURLを更新します。例えば、ユーザが 「Lee 」と検索した場合、`/dashboard/invoices?query=lee`となります。
* Next.jsのクライアントサイドナビゲーション（ページ間のナビゲーションの章で説明しました）のおかげで、ページをリロードすることなくURLが更新されます。

### 3. URLと入力を同期させる
入力フィールドがURLと同期し、共有時に入力されるようにするには、searchParamsから読み込んでinputにdefaultValueを渡します

```tsx
// app/ui/search.tsx

<input
  className="peer block w-full rounded-md border border-gray-200 py-[9px] pl-10 text-sm outline-2 placeholder:text-gray-500"
  placeholder={placeholder}
  onChange={(e) => {
    handleSearch(e.target.value);
  }}
  defaultValue={searchParams.get('query')?.toString()}
/>
```

**defaultValue vs. value / 制御される vs. 制御されない**

入力の値を管理するために状態を使用する場合、value属性を使用して制御されたコンポーネントにします。これは、Reactが入力の状態を管理することを意味します。

しかし、状態を使用しないので、defaultValueを使用することができます。これは、ネイティブ入力が自身の状態を管理することを意味する。ステートの代わりに検索クエリをURLに保存するので、これは問題ない。

### 4. テーブルの更新
最後に、検索クエリを反映させるためにテーブルコンポーネントを更新する必要があります。

請求書ページに戻ってください。

ページコンポーネントはsearchParamsと呼ばれるpropを受け入れるので、現在のURLパラメータを`<Table>`コンポーネントに渡すことができます。

```tsx
// app/dashboard/invoices/page.tsx

import Pagination from '@/app/ui/invoices/pagination';
import Search from '@/app/ui/search';
import Table from '@/app/ui/invoices/table';
import { CreateInvoice } from '@/app/ui/invoices/buttons';
import { lusitana } from '@/app/ui/fonts';
import { Suspense } from 'react';
import { InvoicesTableSkeleton } from '@/app/ui/skeletons';

export default async function Page(props: {
  searchParams?: Promise<{
    query?: string;
    page?: string;
  }>;
}) {
  const searchParams = await props.searchParams;
  const query = searchParams?.query || '';
  const currentPage = Number(searchParams?.page) || 1;

  return (
    <div className="w-full">
      <div className="flex w-full items-center justify-between">
        <h1 className={`${lusitana.className} text-2xl`}>Invoices</h1>
      </div>
      <div className="mt-4 flex items-center justify-between gap-2 md:mt-8">
        <Search placeholder="Search invoices..." />
        <CreateInvoice />
      </div>
      <Suspense key={query + currentPage} fallback={<InvoicesTableSkeleton />}>
        <Table query={query} currentPage={currentPage} />
      </Suspense>
      <div className="mt-5 flex w-full justify-center">
        {/* <Pagination totalPages={totalPages} /> */}
      </div>
    </div>
  );
}
```

`<Table>`コンポーネントに移動すると、queryとcurrentPageの2つのpropsがfetchFilteredInvoices()関数に渡され、クエリに一致する請求書を返すことがわかります。

これらの変更を行って、テストしてみてください。用語を検索すると、URLが更新され、サーバに新しいリクエストが送信され、サーバでデータが取得され、クエリにマッチする請求書だけが返されます。

**useSearchParams()フックとsearchParamsプロップの使い分けは？**

検索パラメータを抽出するために2つの異なる方法を使用していることにお気づきかもしれません。どちらを使うかは、クライアントで作業しているかサーバで作業しているかによります。

* `<Search>`はクライアント・コンポーネントなので、クライアントからパラメータにアクセスするためにuseSearchParams()フックを使用しました。
* `<Table>`はそれ自身のデータを取得するServer Componentなので、ページからコンポーネントにsearchParams propを渡すことができます。

一般的なルールとして、クライアントからパラメータを読み込みたい場合、useSearchParams()フックを使用します。

### ベストプラクティス：デバウンス
おめでとうございます！Next.jsで検索を実装しましたね！しかし、最適化するためにできることがあります。

handleSearch 関数の中に、次の console.log を追加してください

```tsx
// app/ui/search.tsx

function handleSearch(term: string) {
  console.log(`Searching... ${term}`);

  const params = new URLSearchParams(searchParams);
  if (term) {
    params.set('query', term);
  } else {
    params.delete('query');
  }
  replace(`${pathname}?${params.toString()}`);
}
```
それから検索バーに「Delba」と入力し、開発ツールのコンソールを確認してください。何が起こっているのか？

```
Searching... D
Searching... De
Searching... Del
Searching... Delb
Searching... Delba
```

キーストロークのたびにURLを更新しているので、キーストロークのたびにデータベースに問い合わせをしていることになります！私たちのアプリケーションは小さいので、これは問題ではありませんが、もしあなたのアプリケーションに何千人ものユーザーがいて、それぞれがキーストロークのたびにデータベースに新しいリクエストを送信していたらと想像してみてください。

デバウンスは、関数が起動するレートを制限するプログラミング手法です。この例では、ユーザーが入力を止めたときだけデータベースに問い合わせるようにします。

**デバウンスの仕組み**

1. トリガーイベント： デバウンスすべきイベント（検索ボックスのキー入力など）が発生すると、タイマーがスタートします。
2. 待機：タイマーが切れる前に新しいイベントが発生すると、タイマーがリセットされます。
3. 実行： タイマーのカウントダウンが終了すると、デバウンスされた関数が実行されます。

デバウンスを実装するには、独自のデバウンス関数を手動で作成するなど、いくつかの方法があります。シンプルにするために、use-debounceというライブラリを使います。

use-debounceをインストールする：

```bash
pnpm i use-debounce
```

`<Search>` ComponentにuseDebouncedCallbackという関数をインポートします

```tsx
// app/ui/search.tsx

// ...
import { useDebouncedCallback } from 'use-debounce';

// Inside the Search Component...
const handleSearch = useDebouncedCallback((term) => {
  console.log(`Searching... ${term}`);

  const params = new URLSearchParams(searchParams);
  if (term) {
    params.set('query', term);
  } else {
    params.delete('query');
  }
  replace(`${pathname}?${params.toString()}`);
}, 300);
```

この関数はhandleSearchの内容をラップし、ユーザーが入力をやめてから特定の時間（300ミリ秒）後にのみコードを実行する。

もう一度検索バーを入力し、開発ツールのコンソールを開いてください。以下のように表示されるはずです

```
Searching... Delba
```

デバウンスすることで、データベースに送られるリクエストの数を減らし、リソースを節約することができます。

## ページネーションの追加
検索機能を導入した後、テーブルが一度に6つの請求書しか表示しないことに気づくでしょう。これは、data.tsのfetchFilteredInvoices()関数が1ページに最大6件の請求書を返すためです。

ページ分割を追加することで、ユーザーは異なるページをナビゲートしてすべての請求書を見ることができます。検索で行ったように、URLパラメータを使用してページ分割を実装する方法を見てみましょう。

`<Pagination/>`コンポーネントに移動すると、それがクライアントコンポーネントであることがわかります。クライアントでデータを取得することは、データベースの秘密を暴露することになるので避けたいでしょう（APIレイヤーを使用していないことを忘れないでください）。代わりに、サーバでデータを取得し、それをpropとしてコンポーネントに渡すことができます。

/dashboard/invoices/page.tsxで、fetchInvoicesPagesという新しい関数をインポートし、searchParamsからのクエリを引数として渡します

```tsx
// app/dashboard/invoices/page.tsx

// ...
import { fetchInvoicesPages } from '@/app/lib/data';

export default async function Page(
  props: {
    searchParams?: Promise<{
      query?: string;
      page?: string;
    }>;
  }
) {
  const searchParams = await props.searchParams;
  const query = searchParams?.query || '';
  const currentPage = Number(searchParams?.page) || 1;
  const totalPages = await fetchInvoicesPages(query);

  return (
    // ...
  );
}
```

fetchInvoicesPages は、検索クエリに基づくページの総数を返します。例えば、検索クエリにマッチする請求書が12件あり、各ページが6件の請求書を表示する場合、総ページ数は2ページとなります。

次に、totalPagesプロップを`<Pagination/>`コンポーネントに渡します

```tsx
// app/dashboard/invoices/page.tsx

// ...

export default async function Page(props: {
  searchParams?: Promise<{
    query?: string;
    page?: string;
  }>;
}) {
  const searchParams = await props.searchParams;
  const query = searchParams?.query || '';
  const currentPage = Number(searchParams?.page) || 1;
  const totalPages = await fetchInvoicesPages(query);

  return (
    <div className="w-full">
      <div className="flex w-full items-center justify-between">
        <h1 className={`${lusitana.className} text-2xl`}>Invoices</h1>
      </div>
      <div className="mt-4 flex items-center justify-between gap-2 md:mt-8">
        <Search placeholder="Search invoices..." />
        <CreateInvoice />
      </div>
      <Suspense key={query + currentPage} fallback={<InvoicesTableSkeleton />}>
        <Table query={query} currentPage={currentPage} />
      </Suspense>
      <div className="mt-5 flex w-full justify-center">
        <Pagination totalPages={totalPages} />
      </div>
    </div>
  );
}
```

`<Pagination/>`コンポーネントに移動し、usePathnameとuseSearchParamsフックをインポートする。これを使用して、現在のページを取得し、新しいページを設定します。このコンポーネントのコードもアンコメントしてください。まだ`<Pagination/>`ロジックを実装していないので、アプリケーションが一時的に壊れてしまいます。今すぐ実装しましょう！

```tsx
// app/ui/invoices/pagination.tsx

'use client';

import { ArrowLeftIcon, ArrowRightIcon } from '@heroicons/react/24/outline';
import clsx from 'clsx';
import Link from 'next/link';
import { generatePagination } from '@/app/lib/utils';
import { usePathname, useSearchParams } from 'next/navigation';

export default function Pagination({ totalPages }: { totalPages: number }) {
  const pathname = usePathname();
  const searchParams = useSearchParams();
  const currentPage = Number(searchParams.get('page')) || 1;

  // ...
}
```

次に、`<Pagination>`コンポーネントの中にcreatePageURLという新しい関数を作成します。検索と同様に、URLSearchParamsを使って新しいページ番号を設定し、pathNameを使ってURL文字列を作成します。

```tsx
// app/ui/invoices/pagination.tsx

'use client';

import { ArrowLeftIcon, ArrowRightIcon } from '@heroicons/react/24/outline';
import clsx from 'clsx';
import Link from 'next/link';
import { generatePagination } from '@/app/lib/utils';
import { usePathname, useSearchParams } from 'next/navigation';

export default function Pagination({ totalPages }: { totalPages: number }) {
  const pathname = usePathname();
  const searchParams = useSearchParams();
  const currentPage = Number(searchParams.get('page')) || 1;

  const createPageURL = (pageNumber: number | string) => {
    const params = new URLSearchParams(searchParams);
    params.set('page', pageNumber.toString());
    return `${pathname}?${params.toString()}`;
  };

  // ...
}
```

何が起こっているかの内訳は以下の通り

* createPageURLは現在の検索パラメータのインスタンスを作成します。
* 次に、「page 」パラメータを指定されたページ番号に更新します。
* 最後に、パス名と更新された検索パラメータを使って完全なURLを作成します。

`<Pagination>`コンポーネントの残りの部分は、スタイリングとさまざまな状態（最初、最後、アクティブ、無効など）を扱います。このコースでは詳細には触れませんが、createPageURLがどこで呼び出されているか、自由にコードを見てください。

最後に、ユーザが新しい検索クエリを入力したときに、ページ番号を1にリセットしたい場合、`<Search>`コンポーネントのhandleSearch関数を更新することでこれを行うことができます

```tsx
// app/ui/search.tsx

'use client';

import { MagnifyingGlassIcon } from '@heroicons/react/24/outline';
import { usePathname, useRouter, useSearchParams } from 'next/navigation';
import { useDebouncedCallback } from 'use-debounce';

export default function Search({ placeholder }: { placeholder: string }) {
  const searchParams = useSearchParams();
  const { replace } = useRouter();
  const pathname = usePathname();

  const handleSearch = useDebouncedCallback((term) => {
    const params = new URLSearchParams(searchParams);
    params.set('page', '1');
    if (term) {
      params.set('query', term);
    } else {
      params.delete('query');
    }
    replace(`${pathname}?${params.toString()}`);
  }, 300);

```

## まとめ
おめでとうございます！URL ParamsとNext.js APIを使って、検索とページ遷移を実装しました。

要約すると、この章では

* クライアントステートの代わりにURL検索パラメータを使用して検索とページ分割を処理しました。
* サーバーでデータを取得しました。
* useRouterルーターフックを使って、クライアントサイドの遷移をスムーズにした。

これらのパターンは、クライアントサイドのReactで作業するときに慣れ親しんできたものとは異なりますが、URL検索パラメータを使用し、この状態をサーバーに持ち上げることの利点をよりよく理解していただけたと思います。

第11章を修了しました

ダッシュボードに検索機能とページネーション機能が追加されました！
